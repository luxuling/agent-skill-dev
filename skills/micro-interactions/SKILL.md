---
name: micro-interactions
description: The owner's rules for UI micro-interactions, covering password show/hide, confirm dialogs on hard-to-undo actions, pending buttons, visible success and failure, loading empty and error states, and keyboard focus for overlays. Use whenever you build or change an app screen, form, button, modal, password field, or other user-facing control, including a small edit, and even when the user never mentions interaction or polish. Also use when the user asks for micro-interactions or runs /micro-interactions.
license: MIT
---

# Micro-interactions

When you build or change UI, the interaction ships with the control. A control that works and gives no feedback is unfinished.

Use the dialog, toast, and button the project already has. Add a component library for this only when the user asks. Explicit project instructions win over this skill.

## Password reveal

Every password field has a show/hide control with an eye icon. It toggles the input between `type="password"` and `type="text"`. The control is `type="button"` so it does not submit the form. Its accessible name and pressed state follow the mode: "Show password" / `aria-pressed="false"` when hidden, "Hide password" / `aria-pressed="true"` when shown. The password label stays on the input. Keep a single input and change its type. Do not copy the value into a second field.

```html
<label for="password">Password</label>
<input id="password" name="password" type="password" />
<button type="button" aria-label="Show password" aria-pressed="false"></button>
```

Build that behavior with the project's components. The button contains the eye icon. The markup is the contract.

## Confirm before a hard-to-undo action

Ask first when the action is hard to undo: delete, discard, revoke access, charge money, publish, send to someone else, or leave a form that still has unsaved changes.

The dialog names the thing and the outcome. The confirming button uses the verb ("Delete invoice"), not "OK". Cancel is present. On a destructive confirm, focus starts on the safe action.

```html
<dialog aria-labelledby="delete-title">
  <h2 id="delete-title">Delete invoice INV-12?</h2>
  <p>This cannot be undone.</p>
  <button type="button">Cancel</button>
  <button type="button">Delete invoice</button>
</dialog>
```

Do not ask for a read, a filter, navigation that loses nothing, a toggle the person can flip back, or a routine save. Submitting a form the person just filled is already the confirmation. That save still needs a visible success or failure.

## Pending and outcome

A button that starts async work shows a pending label or spinner and ignores further clicks until the work settles. On failure it accepts clicks again.

Success and failure are visible, as a toast or inline message. An error about one field is shown on that field. A disabled control whose reason is not already on screen says why.

## Views that load data

A view that fetches data has a loading state, an empty state, and an error state. The empty state says what is missing and what to do next.

## Immediate feedback

A copy control switches to "Copied" and then back. A switch shows the new value at once. A clickable control has hover, pressed, and a visible focus style. A search field that has text has a clear control.

## Overlays

Opening a dialog moves focus into it. Tab stays inside. Escape, an explicit cancel, or a backdrop click closes it and does not run the action. Focus returns to the control that opened it.

## Motion

Transitions stay around 150–200ms. When the person prefers reduced motion, skip animation that is not required to understand the change.

## A case this file does not name

Show that the action happened, and ask first when it is hard to undo. Do not add a named rule to this file unless the owner confirms it.

## Before you finish

Reread the UI you touched.

- A password field with no eye toggle, or an eye button that submits the form.
- A hard-to-undo action with no confirm, or a routine save that asks for one.
- An async button that can fire twice, or that stays disabled after a failure.
- A success or failure the person cannot see, or a field error shown only in a banner.
- A fetched view missing loading, empty, or error.
- A dialog that does not move focus in, keep Tab inside, close on Escape or backdrop without running the action, and return focus.
