# INTERACTION-DESIGN-POLICY.md

## Purpose

Define standards for interaction design decisions embedded in code generation. This policy governs agent behavior when generating UI components, user flows, form interactions, feedback mechanisms, and error states. It bridges design principles and practical implementation.

---

## Scope / When to Use

Apply this policy when generating or modifying:
- UI components with interactive states (forms, buttons, navigation, modals, toasts).
- User flows and multi-step processes.
- Error messages, empty states, and loading states.
- Micro-interactions and affordances.
- Any UI code where the user must understand what to do, what happened, or what went wrong.

---

## Core Principles

- **Clarity over cleverness.** The user must always know where they are, what they can do, and what just happened.
- **Feedback is mandatory.** Every action must produce a visible, timely response.
- **Affordances telegraph function.** Visual design must communicate how an element behaves.
- **Errors prevent, not punish.** Design to prevent errors; when they occur, make recovery obvious.
- **Consistency reduces cognitive load.** Use the same patterns for the same types of actions everywhere in the product.
- **Accessibility is interaction design.** An interaction that works only with a mouse is not a complete interaction.

---

## Required Behavior

### Clarity and Hierarchy
- Every screen must have a clear primary action. Secondary and tertiary actions must be visually subordinate.
- Use plain language for labels, instructions, and error messages — no jargon, no technical identifiers.
- Group related controls together and separate unrelated ones with whitespace or dividers.
- Use progressive disclosure: show only what the user needs at this step; reveal advanced options on demand.

### Feedback
- Provide immediate visual feedback (< 100ms) for every interactive element (button press, checkbox toggle, link click).
- Show loading indicators for operations that take more than 500ms.
- Confirm successful actions with a non-blocking notification (toast or inline confirmation).
- Do not leave users on a blank screen or in an ambiguous loading state without feedback.

```
User action → Immediate visual response → Async operation → Result notification
```

### Affordances
- Buttons must look like buttons: distinct background, appropriate padding, clear label.
- Links must be distinguishable from non-interactive text (underline, color, or both).
- Disabled controls must be visually distinguished from enabled ones and must not be interactive.
- Form fields must have visible borders or backgrounds that clearly identify them as editable.
- Destructive actions (delete, remove, overwrite) must be visually distinct from constructive actions.

### Forms
- Label every input field — never rely on placeholder text as a label.
- Show inline validation errors next to the field they relate to, not only at form submission.
- Indicate required fields clearly (using `*` with a legend or explicit "Required" text).
- Preserve user input on validation failure — do not clear fields that were correctly filled.
- Provide helpful hint text below fields when the expected format is not obvious.
- Use the appropriate input type (`email`, `tel`, `number`, `date`) to trigger correct mobile keyboards.

### Error Prevention
- Confirm irreversible destructive actions with a modal dialog. The default button must be Cancel, not Confirm.
- Disable the submit button when a form is known to be invalid (only after the user has attempted to submit).
- Warn users before navigating away from unsaved changes.
- For file uploads, validate file type and size before submission and provide a clear error if they fail.

### Error Messages
- Error messages must: (1) describe what went wrong, (2) explain why, (3) tell the user what to do next.
- Never display raw error codes, stack traces, or technical IDs in user-facing messages.
- Position error messages adjacent to the element they describe, not only in a summary at the top.
- Use `role="alert"` or `aria-live="polite"` to announce errors to screen readers.

Example structure:
```
Bad:  "Error 422"
Good: "Email address is already in use. Sign in instead, or use a different email address."
```

### Loading and Empty States
- Every list or data view must have an empty state with an explanation and a suggested action.
- Loading skeletons are preferred over spinners for content areas that have a known structure.
- Do not show empty states and loading states simultaneously.

### Navigation and Wayfinding
- Always indicate the current location in navigation (active state, breadcrumb, or page title).
- Breadcrumbs are required for content more than two levels deep.
- Do not open links in a new tab/window without warning the user.
- Back button behavior must be predictable — do not override browser history unexpectedly.

### Consistency
- Use the same component for the same purpose everywhere. Do not create one-off variations without justification.
- Reuse the same terminology for the same actions and objects throughout the product.
- Maintain consistent spacing, iconography style, and motion duration.

### UX Writing
- Call-to-action labels must be verb-noun pairs that describe the outcome: "Save Changes", "Delete Account", "Add Item".
- Avoid vague labels: "OK", "Yes", "Submit" without context.
- Confirmation dialog titles should state what is being confirmed: "Delete this file?" not "Are you sure?".

---

## Things to Avoid

- Placeholder text used as the only label for form inputs.
- Operations with no loading state or success confirmation.
- Destructive actions without a confirmation step.
- Error messages that describe the technical cause but not the user action required.
- Multiple primary buttons on a single view.
- Interactions that work only with hover (no touch or keyboard equivalent).
- Animations or transitions that cannot be disabled (respect `prefers-reduced-motion`).
- Empty states with no explanation or suggested action.

---

## Output Expectations

Generated UI code must:
- Include all interactive states: default, hover, focus, active, disabled, loading, error, success.
- Pair every destructive action with a confirmation dialog.
- Include inline validation for form fields.
- Use `role="alert"` or `aria-live` for dynamic error messages.
- Respect `prefers-reduced-motion` for animations.
- Have visible focus indicators on all interactive elements.
- Include an empty state for any list or data view.

---

## Optional Checklist

- [ ] Every interactive element has feedback for hover, focus, active, and disabled states.
- [ ] Loading state provided for operations > 500ms.
- [ ] Success confirmation provided for completed actions.
- [ ] Form fields have associated labels (not placeholder-only).
- [ ] Inline validation errors shown adjacent to the field.
- [ ] Destructive actions require confirmation; default button is Cancel.
- [ ] Error messages explain cause and next action.
- [ ] `role="alert"` or `aria-live` used for dynamic content.
- [ ] `prefers-reduced-motion` respected.
- [ ] Empty states include explanation and suggested action.
- [ ] CTA labels are verb-noun pairs.
