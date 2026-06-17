# Mobile Accessibility Reference

## Table Of Contents

- Baseline
- Screen Reader Support
- Touch And Gesture Accessibility
- Text And Visual Accessibility
- React Native And Expo Implementation
- Native iOS Considerations
- Native Android Considerations
- Mobile Component Patterns
- Testing
- Output Templates

## Baseline

Use WCAG 2.2 Level A and AA with WCAG2Mobile and WCAG2ICT guidance as the practical baseline. Also use Apple accessibility guidance, Android accessibility guidance, and platform-specific assistive technology behavior.

Mobile accessibility must be verified manually with VoiceOver and TalkBack. Automated checks and lint rules are useful but incomplete.

## Screen Reader Support

Check the screen reader experience, not just properties in code.

- Accessible names: every actionable control needs a clear label matching visible intent.
- Hints: use hints for non-obvious outcomes, not to repeat the label.
- Roles/traits: buttons, links, images, headers, tabs, adjustable controls, and search fields should expose correct roles.
- States: selected, disabled, expanded, checked, busy, invalid, and current states must be announced.
- Values: sliders, progress, steppers, ratings, and counters need meaningful values.
- Grouping: group related text and actions when it improves navigation; do not group so much that users lose access to child actions.
- Reading order: screen reader order should match visual and task order.
- Focus order: focus should move predictably through controls, screens, modals, sheets, and error states.

Do not hide meaningful content from accessibility. Hide decorative content and duplicate icons.

## Touch And Gesture Accessibility

- Touch targets should generally be at least 44 by 44 points on iOS and 48 by 48 dp on Android.
- Avoid placing targets so close that users with motor impairments activate the wrong control.
- Provide alternatives for complex gestures: drag, pinch, rotate, long press, multi-finger, swipe-only, or precise path gestures.
- Support one-handed use for common flows where practical.
- Drag/drop needs buttons or menus for move/reorder actions.
- Swipe-only cards need visible actions or accessible custom actions.
- Camera, selfie, and document capture flows need spoken instructions, non-visual feedback, retake controls, and non-camera alternatives when possible.
- Maps and location flows need list alternatives, search, explicit location permission explanation, and accessible zoom/recenter controls.

## Text And Visual Accessibility

- Support iOS Dynamic Type and Android font scale.
- Do not disable font scaling unless there is a documented exception and an accessible alternative.
- Layouts must remain usable with large text, long translations, and bold text settings.
- Check contrast for text, icons, borders, focus indicators, disabled states, selected states, charts, and map overlays.
- Do not rely on color alone for errors, selection, status, or validation.
- Respect Reduce Motion and platform animation settings.
- Respect reduced transparency/increased contrast where applicable.
- Avoid tiny text inside images; provide real text when information matters.

## React Native And Expo Implementation

Use React Native accessibility APIs deliberately.

Common props:

- `accessibilityRole`: expose the role, such as `button`, `link`, `header`, `image`, `tab`, `search`, or `adjustable`.
- `accessibilityLabel`: provide an accessible name when visible text is absent or insufficient.
- `accessibilityHint`: explain the result of activation when not obvious.
- `accessibilityState`: expose `disabled`, `selected`, `checked`, `expanded`, or `busy`.
- `accessibilityValue`: expose min, max, current, and text values for adjustable or progress controls.
- `accessibilityActions`: expose custom screen reader actions for swipe/reorder/delete alternatives.
- `onAccessibilityAction`: handle custom accessibility actions.
- `importantForAccessibility`: hide background or duplicate content on Android when appropriate.
- `accessible`: group children into one accessible element when useful.
- `accessibilityLiveRegion`: announce dynamic Android updates.
- `AccessibilityInfo`: detect/reduce motion, announce changes, or manage screen reader behavior.
- `setAccessibilityFocus`: move focus after navigation, modal open, or blocking errors when appropriate.

### Pressable And Touchable Semantics

Use `Pressable` or platform components for actions. Provide role, label, state, and disabled behavior.

```tsx
const SaveButton = ({ disabled }: { disabled: boolean }) => {
  return (
    <Pressable
      accessibilityRole="button"
      accessibilityState={{ disabled }}
      disabled={disabled}
      onPress={saveProfile}
    >
      <Text>Save profile</Text>
    </Pressable>
  );
};
```

Icon-only buttons need labels:

```tsx
const CloseButton = () => {
  return (
    <Pressable accessibilityRole="button" accessibilityLabel="Close" onPress={close}>
      <CloseIcon importantForAccessibility="no" />
    </Pressable>
  );
};
```

### TextInput Labels And Errors

React Native does not provide the same native label association or invalid state semantics as web. Ensure visible labels, accessible labels, hints, and error text are clear, then announce blocking errors explicitly.

```tsx
const EmailInput = ({ error }: { error?: string }) => {
  return (
    <View>
      <Text nativeID="email-label">Email</Text>
      <TextInput
        accessibilityLabel="Email"
        accessibilityHint={error ? `Error: ${error}` : "Enter the email address for your account"}
        autoComplete="email"
        inputMode="email"
      />
      {error ? <Text accessibilityLiveRegion="polite">{error}</Text> : null}
    </View>
  );
};
```

### Modal Accessibility

- Move screen reader focus to the modal title or first useful control.
- Prevent screen readers from reaching background content.
- Restore focus to the opener after close when possible.
- Make close and cancel actions accessible.
- Avoid relying on swipe-down or backdrop tap as the only close mechanism.

### Lists

For `FlatList` and `SectionList`:

- Ensure item order matches visual order.
- Provide labels for item actions.
- Avoid nested touchables that create confusing focus.
- Expose item position when useful, such as "Item 3 of 10".
- Keep loading, empty, and error states accessible.

### Custom Components

Custom controls must expose the same role, name, state, value, and actions as platform controls. If this is too costly, use a native or simpler component.

## Native iOS Considerations

- Use clear `accessibilityLabel`, `accessibilityHint`, `accessibilityValue`, and traits.
- Use button traits for tappable controls.
- Use header traits for screen and section headings.
- Support Dynamic Type with scalable fonts and flexible layout.
- Respect Reduce Motion.
- Move VoiceOver focus on modal/screen transitions when needed.
- Use Accessibility Inspector to inspect hierarchy, labels, traits, contrast, hit targets, and Dynamic Type behavior.

## Native Android Considerations

- Use `contentDescription` for icon-only controls and meaningful images.
- Do not set `contentDescription` on purely decorative images.
- Associate labels with inputs.
- Ensure TalkBack order matches visual/task order.
- Use state descriptions when default state announcement is unclear.
- Use live regions for important dynamic updates.
- Meet touch target guidance.
- Test with Accessibility Scanner, but manually verify TalkBack behavior.

## Mobile Component Patterns

### Buttons

Expose role, label, disabled state, loading state, and hit target. Do not make plain text tappable without role and target padding.

### Icon Buttons

Provide an accessible label. Hide decorative icon paths from the accessibility tree when possible.

### Inputs

Provide visible labels, accessible labels, input type, autocomplete, helper text, errors, and clear focus order.

### OTP Fields

Support paste and autofill. Label each field clearly, for example "Digit 1 of 6". Announce errors and allow correction without restarting the flow.

### Pickers

Prefer native pickers when possible. Custom pickers need role, selected state, current value, focus management, and dismiss behavior.

### Bottom Sheets

Treat bottom sheets like modal dialogs when they block background interaction. Focus the sheet, hide background content, provide close controls, and avoid drag-only dismissal.

### Modals

Use a title, focus management, accessible close/cancel actions, and background exclusion.

### Tabs

Expose tab role, selected state, labels, and logical order. Screen reader users should know which tab is selected and what content changed.

### Lists And Cards

Cards with multiple actions should not be one ambiguous accessible element. Either group summary content with a primary action or expose child actions clearly.

### Toasts And Snackbars

Announce important messages. Do not rely on toasts for critical errors. Provide persistent recovery actions when needed.

### Loading States

Expose busy/loading state for blocking operations. Avoid repeatedly announcing progress unless meaningful.

### Error States

Move focus or announce errors when they block task completion. Put recovery actions after the error message in reading order.

### Camera, Biometric, And Selfie Flows

Provide spoken guidance, stable controls, retake and skip alternatives where possible, permission rationale, and non-visual progress or alignment feedback.

### Maps

Provide search, list results, accessible pins, current location controls, and non-gesture zoom controls.

### Notifications

Notification settings need clear labels and state. Push notification permission prompts should explain value before system prompt where appropriate.

## Testing

VoiceOver manual flow:

- Enable VoiceOver.
- Navigate by swipe and touch exploration.
- Verify labels, roles, hints, states, values, headings, and reading order.
- Complete happy path and error path.
- Test modals, sheets, tabs, lists, and forms.

TalkBack manual flow:

- Enable TalkBack.
- Navigate by swipe and touch exploration.
- Verify labels, roles, states, custom actions, live regions, and reading order.
- Complete happy path and error path.

Additional mobile tests:

- iOS Accessibility Inspector.
- Android Accessibility Scanner.
- React Native lint and component checks where available.
- Font scaling at large accessibility sizes.
- Landscape/orientation where supported.
- Reduced motion enabled.
- Large touch target pass.
- External keyboard, switch control, or keyboard navigation where relevant.
- Color contrast and color-independent state review.

## Output Templates

### Mobile Accessibility Audit Report

```txt
Accessibility risk summary
- Overall risk:
- Platforms tested:
- Assistive tech tested:
- Main blockers:

Issues found
- [Critical/High/Medium/Low] Screen/component:
  Problem:
  Impact:
  Evidence:
  Fix:
  Test:

Manual test checklist
- VoiceOver:
- TalkBack:
- Focus/reading order:
- Labels/roles/states:
- Font scaling:
- Touch targets:
- Gestures:
- Motion:
- Contrast:

Automated test suggestions
- Tool:
- What it can catch:
- What still requires manual review:

Acceptance criteria
- User can:
- Screen reader announces:
- Touch behavior:
- Visual/text scaling behavior:
```

### React Native Code Review Checklist

```txt
- Pressables expose role, label, state, and disabled behavior.
- Icon-only controls have accessibilityLabel.
- Inputs have visible labels, accessible labels, useful hints, and error announcements.
- Custom controls expose role, state, value, and custom actions where needed.
- Modals/sheets manage focus and hide background content.
- Lists have logical reading order and accessible item actions.
- Dynamic updates are announced without excessive noise.
- Font scaling does not truncate or overlap key content.
- Touch targets meet platform guidance.
- Gesture-only flows have alternatives.
```

### QA Test Script

```txt
1. Enable VoiceOver or TalkBack.
2. Start from the app entry screen.
3. Navigate to the target flow by swipe only.
4. Confirm every control announces name, role, state, and hint when needed.
5. Complete the happy path.
6. Trigger validation errors and recover.
7. Open and close every modal/sheet/menu.
8. Increase font size to an accessibility size and repeat core steps.
9. Enable Reduce Motion and repeat animated transitions.
10. Record blockers with screen, control, expected behavior, actual behavior, and fix.
```

### Acceptance Criteria

```txt
- Core task can be completed with VoiceOver and TalkBack.
- Reading order matches visual/task order.
- All interactive controls have clear names, roles, and states.
- Error messages are announced and recovery is possible.
- Touch targets meet platform size expectations.
- Gesture-only actions have accessible alternatives.
- Text remains usable at large font sizes.
- Motion respects platform reduced-motion settings.
- Automated scanner findings are fixed or documented with manual rationale.
```
