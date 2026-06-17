# Web Accessibility Reference

## Table Of Contents

- Baseline
- Practical WCAG 2.2 A/AA Interpretation
- Semantic HTML First
- Keyboard Accessibility
- Focus Management
- ARIA Rules
- Component Patterns
- Visual Accessibility
- React And Next.js Notes
- Testing
- Output Templates

## Baseline

Use WCAG 2.2 Level A and AA as the default baseline for production web work. Also use WAI-ARIA Authoring Practices for custom widgets, semantic HTML, browser accessibility APIs, and observed assistive technology behavior.

Automated tools help find some failures, but they do not verify full accessibility. Always pair automated checks with manual keyboard, focus, screen reader, zoom, contrast, reduced motion, forms, error, and dynamic-content checks.

## Practical WCAG 2.2 A/AA Interpretation

Prioritize user-blocking failures:

- Critical: cannot complete a core task with keyboard or screen reader, focus trap, unlabeled required control, inaccessible authentication, missing dialog focus containment, hidden content announced incorrectly, or content lost at zoom.
- High: confusing semantics, poor focus order, missing error association, no visible focus, insufficient contrast on critical UI, gesture-only controls, motion that cannot be reduced.
- Medium: unclear labels, weak helper text, inconsistent headings, incomplete live updates, missing current-page state, non-critical contrast failures.
- Low: polish issues that do not block task completion but reduce usability.

Check WCAG 2.2 additions that commonly affect apps:

- Focus Appearance: focus indicators must be visible and not rely on subtle color changes.
- Focus Not Obscured: sticky headers, cookie banners, and toolbars must not cover focused controls.
- Target Size: interactive targets should be large enough and not crowded.
- Dragging Movements: provide non-drag alternatives when drag is required.
- Consistent Help: keep help/contact mechanisms consistently placed when present.

## Semantic HTML First

Prefer native elements before ARIA:

- Use `<button>` for actions.
- Use `<a href>` for navigation.
- Use headings in order to describe page structure.
- Use `<main>`, `<nav>`, `<header>`, `<footer>`, `<aside>`, and `<section>` with names when helpful.
- Use native `<input>`, `<select>`, `<textarea>`, `<label>`, `<fieldset>`, and `<legend>` for forms.
- Use real lists for grouped items and real tables for tabular data.

Do not create clickable `<div>` or `<span>` controls. If a custom element is unavoidable, it must provide role, accessible name, keyboard support, focusability, state, and disabled behavior equivalent to the native control.

### Headings And Landmarks

- One page should have a clear `h1` matching the current view.
- Do not skip heading levels for visual styling.
- Use landmarks to let screen reader users jump by region.
- Name duplicate landmarks, such as multiple navigation regions.

### Forms

- Every input needs a programmatic label using `<label for>` or `aria-labelledby`.
- Use `aria-describedby` for helper text and error text.
- Use `aria-invalid="true"` only when the value is currently invalid.
- Put grouped choices in `<fieldset>` with `<legend>`.
- Mark required fields in text and, when helpful, with `required` or `aria-required`.
- Error summaries should link or move focus to invalid fields after submit failure.

Example:

```tsx
const EmailField = ({ error }: { error?: string }) => {
  return (
    <div>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        name="email"
        type="email"
        autoComplete="email"
        aria-invalid={error ? "true" : undefined}
        aria-describedby={error ? "email-error" : "email-help"}
      />
      <p id="email-help">Use the email for your account.</p>
      {error ? <p id="email-error">{error}</p> : null}
    </div>
  );
};
```

## Keyboard Accessibility

All interactive controls must be reachable and operable without a mouse.

- Tab moves through interactive elements in logical order.
- Shift+Tab reverses order.
- Enter activates links and buttons.
- Space activates buttons, checkboxes, and other button-like controls.
- Escape closes dismissible dialogs, menus, popovers, and drawers.
- Arrow keys are used inside composite widgets according to ARIA APG patterns.
- No keyboard traps. Users must be able to leave every component.
- Do not use positive `tabIndex`; fix DOM order instead.
- Use `tabIndex={0}` sparingly for custom focus targets.
- Avoid `tabIndex={-1}` except for programmatic focus targets.

Visible focus must be obvious in light mode, dark mode, high contrast modes, and while zoomed. Do not remove outlines unless replacing them with an equally visible indicator.

## Focus Management

### SPA Route Changes

On client-side navigation:

- Move focus to the new page's `h1`, `main`, or a route-announcement target.
- Update `document.title`.
- Announce major content changes when the page does not reload.
- Do not leave focus on a removed nav item or hidden trigger.

### Modals

- Move focus into the dialog on open.
- Keep Tab and Shift+Tab inside the dialog.
- Give the dialog an accessible name.
- Hide or inert background content.
- Restore focus to the opener on close when it still exists.
- Close with Escape unless the flow requires explicit completion.

### Drawers And Menus

- Focus the first useful item on open.
- Close on Escape.
- Restore focus to the trigger.
- Expose expanded/collapsed state on the trigger.

### Toasts, Alerts, And Dynamic Content

- Use `role="status"` or `aria-live="polite"` for non-urgent updates.
- Use `role="alert"` only for urgent messages.
- Do not move focus to passive toasts.
- For destructive or blocking errors, move focus to the error summary or affected control.

## ARIA Rules

- No ARIA is better than bad ARIA.
- Prefer native HTML over ARIA roles.
- Use ARIA APG patterns for custom widgets.
- Do not override native semantics unless required.
- Do not put interactive controls inside elements with roles that flatten children, such as some button-like containers.
- Keep ARIA state synchronized with visual state.

Accessible names are calculated from visible text, associated labels, `aria-labelledby`, `aria-label`, alt text, and other host-language mechanisms. Prefer visible text or `aria-labelledby` over `aria-label` when possible.

Use common ARIA deliberately:

- `aria-label`: name icon-only controls when no visible text exists.
- `aria-labelledby`: name a region or control from visible text.
- `aria-describedby`: connect helper, hint, and error text.
- `aria-expanded`: expose expandable trigger state.
- `aria-controls`: connect a trigger to controlled content when useful.
- `aria-current`: mark current page, step, or location.
- `aria-selected`: selected item in tabs, listbox, grid, or similar composite widget.
- `aria-invalid`: invalid form field state.
- `aria-live`: dynamic updates that need announcement.

## Component Patterns

### Buttons Vs Links

- Button: changes state, submits, opens modal, deletes, toggles.
- Link: navigates to another URL or route.
- Icon buttons need accessible names, for example `aria-label="Close"`.

### Modals And Dialogs

Use native `<dialog>` carefully or an accessible dialog implementation. Required behavior: label, focus on open, focus trap, inert background, Escape close, focus restore, scroll lock that does not break zoom.

### Menus

Use menu semantics only for application-style command menus. For site navigation, prefer normal links inside `<nav>`. Custom menu buttons need `aria-haspopup`, `aria-expanded`, keyboard support, Escape, and focus management.

### Tabs

Use APG tabs pattern:

- `tablist`, `tab`, `tabpanel`
- one active tab in tab order
- arrow keys move between tabs
- selected state is exposed
- tab panel is labelled by its tab

### Accordions

Use a real button as each header trigger. Expose `aria-expanded`; use `aria-controls` when useful. Keep content order logical.

### Comboboxes

Native `<select>` is preferred when it satisfies the design. Custom comboboxes require text input or button semantics, popup ownership, active descendant or roving focus, keyboard support, clear selected value, and escape/collapse behavior.

### Tooltips

Tooltips must not contain required information. They should appear on hover and focus, dismiss with Escape, and not trap focus. Use visible helper text for important guidance.

### Tables And Data Grids

Use `<table>` for tabular data with `<th scope>`, captions when useful, and clear sort state. Data grids require much stronger keyboard and screen reader behavior; do not use grid roles unless implementing the full interaction model.

### Pagination And Breadcrumbs

Pagination should be a named nav region. Mark current page with `aria-current="page"`. Breadcrumbs should be a nav region with ordered links and current item.

### Search

Use `<search>` or a named search form when supported by the project. Label the input. Announce result count changes with polite live regions when results update without navigation.

### File Upload

Use native file input when possible. Provide accepted types, size limits, error messages, and keyboard accessible remove/retry actions.

### Date Pickers

Allow direct text entry or a native date input when possible. Custom calendars need keyboard navigation, visible focus, selected/current date state, and clear format guidance.

### Carousels

Avoid autoplay. If autoplay exists, provide pause/stop controls. Ensure controls are buttons, slides are announced sensibly, and focus does not move unexpectedly.

### Maps

Provide text alternatives for core map information, keyboard-accessible pins, list alternatives for locations, and non-gesture controls for zoom and pan.

### Video And Audio

Provide captions, transcripts when appropriate, keyboard accessible controls, visible focus, no autoplay with sound, and reduced-motion friendly animations.

### Authentication And OTP

Use labels, autocomplete tokens, paste support, visible errors, and non-time-limited alternatives where possible. OTP fields should support paste, clear focus order, and screen reader labels like "Digit 1 of 6".

## Visual Accessibility

- Contrast: text and meaningful icons need sufficient contrast. Check default, hover, focus, disabled, error, and selected states.
- Focus indicators: visible, high contrast, not clipped, not obscured.
- Text resize: content remains usable at 200% browser zoom.
- Reflow: no two-dimensional scrolling for ordinary content at narrow widths.
- Zoom: sticky UI must not cover focused controls.
- Dark mode: recheck contrast and focus indicators.
- Reduced motion: honor `prefers-reduced-motion`; avoid parallax and essential motion.
- Color-independent states: pair color with text, icon, shape, or pattern.

## React And Next.js Notes

- Hydration: avoid changing focus or accessible names unexpectedly during hydration.
- Client-side navigation: update title and move focus to the new view.
- Form libraries: ensure generated IDs, labels, errors, and descriptions are connected.
- Portals and modals: manage focus across portal boundaries and inert background content.
- Suspense/loading: expose meaningful loading states; do not repeatedly announce spinners.
- Error boundaries: focus or announce recovery UI when an error replaces content.
- Icon buttons: require visible text or `aria-label`; decorative icons need `aria-hidden="true"`.
- Design systems: bake semantics, keyboard behavior, focus style, and naming rules into primitives.

## Testing

Manual browser pass:

- Navigate the full flow using keyboard only.
- Verify focus order and visible focus.
- Confirm no keyboard traps.
- Test Escape behavior for overlays.
- Test forms, validation, required fields, and error recovery.
- Test at 200% zoom and narrow viewport.
- Test reduced motion.
- Check contrast for text, icons, borders, focus, and states.
- Inspect accessibility tree in browser DevTools.

Screen reader pass:

- NVDA with Firefox or Chrome on Windows.
- JAWS with Chrome or Edge when enterprise support matters.
- VoiceOver with Safari on macOS and iOS for responsive web.
- Verify names, roles, states, heading navigation, landmarks, form navigation, dynamic updates, and dialog behavior.

Automated suggestions:

- `axe-core` or framework integration
- Lighthouse accessibility checks
- Accessibility Insights
- ESLint JSX accessibility rules
- Playwright accessibility smoke checks for focus and keyboard behavior

Automated results must be reported as partial evidence, not proof of accessibility.

## Output Templates

### Web Accessibility Audit Report

```txt
Accessibility risk summary
- Overall risk:
- Main blockers:
- Standards baseline: WCAG 2.2 A/AA

Issues found
- [Critical/High/Medium/Low] Component or page:
  Problem:
  Impact:
  Evidence:
  Fix:
  Test:

Manual test checklist
- Keyboard:
- Screen reader:
- Focus:
- Forms/errors:
- Zoom/reflow:
- Motion:
- Contrast:

Automated test suggestions
- Tool:
- What it can catch:
- What still requires manual review:

Acceptance criteria
- User can:
- Screen reader announces:
- Keyboard behavior:
- Visual behavior:
```

### Pull Request Review Comment

```txt
[High] The modal opens without moving focus into the dialog.

Impact: Keyboard and screen reader users remain behind the overlay and may not discover the dialog.

Fix: Move focus to the dialog heading or first actionable control on open, trap focus while open, close on Escape, and restore focus to the trigger on close.

Test: Open the modal with keyboard only, verify focus starts inside, Tab cycles within it, Escape closes it, and focus returns to the opener.
```

### Acceptance Criteria

```txt
- All controls have accessible names that match visible intent.
- The flow can be completed with keyboard only.
- Focus order follows the visual and logical order.
- Focus indicators are visible and not obscured.
- Form errors are associated with fields and announced.
- Dynamic updates are announced only when useful.
- Text remains usable at 200% zoom.
- Automated checks run clean, with manual exceptions documented.
```

### QA Checklist

```txt
- Keyboard-only happy path
- Keyboard-only error path
- Screen reader happy path
- Screen reader error path
- Focus restore after modal/menu/drawer close
- 200% zoom and narrow viewport
- Contrast for all states
- Reduced motion
- Automated scan with documented manual follow-up
```
