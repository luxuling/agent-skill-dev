---
name: ci-cd
description: The owner's standard for CI/CD and deploying to a VPS with GitHub Actions, a self-hosted GitHub runner on the VPS, Docker Compose, images on GHCR, host nginx as reverse proxy, and Certbot for HTTPS. Covers the CI workflow, the tag-triggered deploy workflow, Dockerfiles for Next.js, Node APIs, Go, and static SPAs, compose.yml, secrets, database migrations, health checks with automatic rollback, the nginx server block, and first-time VPS setup. Use whenever you add or change a GitHub Actions workflow, Dockerfile, compose file, or nginx config, set up deployment, CI, or a server, or the user asks to deploy, ship, release, or host an app, even when they never mention the standard. Also use when the user runs /ci-cd.
license: MIT
---

# CI/CD and VPS deploy

Every app ships the same way, whatever the stack:

1. PRs and pushes to `main` run CI on GitHub-hosted runners.
2. Pushing a tag `v*` builds a Docker image on a GitHub-hosted runner and pushes it to GHCR.
3. The self-hosted runner on the VPS pulls that image, runs migrations, starts it with Docker Compose, checks health, and rolls back if the check fails.
4. Host nginx proxies the domain to the container port bound on `127.0.0.1`. Certbot handles HTTPS.

Explicit project instructions win over this skill. Do not swap in another tool (Kubernetes, PM2, systemd units, SSH deploys, nginx in a container, building images on the VPS) unless the user asks.

## Files in the repo

```text
.github/workflows/ci.yml       # PR + main checks
.github/workflows/deploy.yml   # tag → build → deploy
Dockerfile
.dockerignore
compose.yml                    # the production stack, copied to the VPS on deploy
deploy/nginx.conf              # host nginx server block (reference, installed by hand)
deploy/spa-nginx.conf          # static SPA only: config for the nginx inside the image
```

## On the VPS

- Each app lives in `/apps/<app>/`, holding `compose.yml` and `.env`, both written by the deploy job. Do not edit them by hand. They are overwritten on the next deploy.
- The app container publishes its port only on localhost: `127.0.0.1:${HOST_PORT}:<container port>`. Each app on the VPS gets its own `HOST_PORT`.
- The database runs as a service in the same `compose.yml`, with a named volume and no published port.

## CI workflow

Runs on `ubuntu-latest`. Uses the project's own lint, typecheck, test, and build commands. Add steps for what the project has; do not invent scripts.

```yaml
name: ci

on:
  pull_request:
  push:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # stack setup (setup-node / setup-go) with dependency caching
      # install, lint, typecheck, test, build
      - run: docker build .
```

The last step proves the Dockerfile still builds. It does not push.

## Deploy workflow

Triggers on a `v*` tag push, and on `workflow_dispatch` with an existing tag to redeploy or roll back by hand. A manual run skips the build and deploys the image already in GHCR.

```yaml
name: deploy

on:
  push:
    tags: ["v*"]
  workflow_dispatch:
    inputs:
      tag:
        description: Existing image tag to deploy, e.g. v1.4.2
        required: true

concurrency:
  group: deploy
  cancel-in-progress: false

env:
  APP: my-app
  HOST_PORT: 3001
  IMAGE: ghcr.io/${{ github.repository }}
  TAG: ${{ inputs.tag || github.ref_name }}

jobs:
  build:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: ${{ env.IMAGE }}:${{ env.TAG }}

  deploy:
    needs: build
    if: ${{ !failure() && !cancelled() }}
    runs-on: [self-hosted, vps]
    permissions:
      contents: read
      packages: read
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ env.TAG }}
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Deploy
        env:
          ENV_FILE: ${{ secrets.ENV_FILE }}
        run: |
          set -euo pipefail
          cd "/apps/$APP"

          [ -f compose.yml ] && cp compose.yml compose.prev.yml
          [ -f .env ] && cp .env .env.prev

          cp "$GITHUB_WORKSPACE/compose.yml" compose.yml
          {
            printf '%s\n' "$ENV_FILE"
            echo "IMAGE=$IMAGE"
            echo "IMAGE_TAG=$TAG"
            echo "HOST_PORT=$HOST_PORT"
          } > .env
          chmod 600 .env

          rollback() {
            echo "Deploy of $TAG failed, rolling back"
            [ -f .env.prev ] || exit 1
            mv compose.prev.yml compose.yml
            mv .env.prev .env
            docker compose up -d --remove-orphans
            exit 1
          }

          docker compose pull || rollback

          if docker compose --profile migrate config --services | grep -qx migrate; then
            docker compose run --rm migrate || rollback
          fi

          docker compose up -d --remove-orphans || rollback

          for _ in $(seq 30); do
            curl -fsS "http://127.0.0.1:$HOST_PORT/health" > /dev/null && exit 0
            sleep 2
          done
          rollback
```

Rules the workflow encodes:

- The image tag is the git tag. Never deploy `latest`.
- Secrets reach the app only through the `ENV_FILE` repository secret: the whole `.env` body, one `KEY=value` per line. Add a new variable to that secret, not to the workflow or the image.
- Migrations run before the new version starts. A failed migration aborts the deploy and the old version keeps running.
- The deploy passes only when `GET /health` on the container returns 2xx within 60 seconds. Otherwise the previous `compose.yml` and `.env` come back and the old image starts again, and the job fails.
- The first deploy has nothing to roll back to. It just fails.
- Set `APP` and `HOST_PORT` per repo. Everything else stays as written.

## Health endpoint

Every app serves `GET /health` returning 200 without auth. It checks the process is up, not every dependency. Add it if the app does not have one.

## compose.yml

```yaml
services:
  app:
    image: ${IMAGE}:${IMAGE_TAG}
    env_file: .env
    ports:
      - "127.0.0.1:${HOST_PORT}:3000"
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  migrate:
    image: ${IMAGE}:${IMAGE_TAG}
    env_file: .env
    command: ["npm", "run", "migrate"]
    depends_on:
      db:
        condition: service_healthy
    profiles: [migrate]

  db:
    image: postgres:17-alpine
    env_file: .env
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER}"]
      interval: 5s
      retries: 10
    restart: unless-stopped

volumes:
  db-data:
```

- `migrate` uses the app image with the project's migrate command. Leave out `migrate` and `db` when the app has no database.
- The container port (`3000` above) is whatever the app listens on inside the image.

## Dockerfiles

Multi-stage. The final stage holds only what runs. Add a `.dockerignore` with at least `.git`, `node_modules`, `.env*`, and build output.

### Next.js

Set `output: "standalone"` in `next.config`.

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production PORT=3000 HOSTNAME=0.0.0.0
COPY --from=build /app/.next/standalone ./
COPY --from=build /app/.next/static ./.next/static
COPY --from=build /app/public ./public
USER node
EXPOSE 3000
CMD ["node", "server.js"]
```

### Node API

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --omit=dev

FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

Use the package manager the repo's lockfile names.

### Go

```dockerfile
FROM golang:1.23-alpine AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app ./cmd/server

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=build /app /app
EXPOSE 8080
ENTRYPOINT ["/app"]
```

### Static SPA

Served from its own container like every other app, never as files on the host.

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY deploy/spa-nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

`deploy/spa-nginx.conf`:

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;

    location = /health {
        return 200 "ok";
    }

    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    location / {
        try_files $uri /index.html;
    }
}
```

Build-time env vars (`VITE_*`) are baked into the bundle, so they go to the build as `build-args`, not into `ENV_FILE`.

## Host nginx server block

`deploy/nginx.conf`, installed as `/etc/nginx/sites-available/<domain>` and linked into `sites-enabled`. Certbot adds the TLS part.

```nginx
server {
    listen 80;
    server_name example.com;

    client_max_body_size 20m;

    location / {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

The `proxy_pass` port equals the app's `HOST_PORT`.

## First-time VPS setup

Do once per server, then once per app. Give the user these steps; do not run them on a server yourself unless asked.

Per server:

1. Install Docker Engine with the compose plugin, nginx, and certbot with its nginx plugin (`python3-certbot-nginx`).
2. Create a `runner` user in the `docker` group.
3. Install the GitHub Actions runner as that user, register it with the label `vps`, and install it as a service (`sudo ./svc.sh install runner && sudo ./svc.sh start`). Register at the org level when several repos deploy to this VPS.

Per app:

1. `sudo mkdir -p /apps/<app> && sudo chown runner:runner /apps/<app>`.
2. Pick a free `HOST_PORT` and set it in `deploy.yml` and `deploy/nginx.conf`.
3. Add the `ENV_FILE` repository secret.
4. Copy `deploy/nginx.conf` to `/etc/nginx/sites-available/<domain>`, link it into `sites-enabled`, then `sudo nginx -t && sudo systemctl reload nginx`.
5. Point the domain's DNS at the VPS, then `sudo certbot --nginx -d <domain>`.
6. Push the first tag: `git tag v0.1.0 && git push origin v0.1.0`.
