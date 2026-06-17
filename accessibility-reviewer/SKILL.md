---
name: accessibility-reviewer
description: Use this skill when reviewing, designing, implementing, or testing accessibility for web, React, Next.js, mobile, React Native, Expo, iOS, or Android applications. The skill helps find WCAG issues, fix semantic markup, improve screen reader support, validate keyboard and focus behavior, audit forms, modals, navigation, touch targets, labels, gestures, color contrast, motion, and create actionable accessibility reports.
---

# Accessibility Reviewer

## Core Baseline

Use WCAG 2.2 Level A and AA as the default baseline.

Treat automated accessibility tools as useful but incomplete. Never claim a UI is accessible only because it passes automated tests. Require manual checks for keyboard, screen reader behavior, focus order, semantics, contrast, zoom/text scaling, dynamic content, motion, forms, errors, and gestures.

Output direct, implementation-focused advice. Avoid vague guidance like "make it accessible"; name the exact behavior, role, label, focus rule, gesture alternative, code change, or testing step.

## Load References

Read `references/web.md` when the request is about websites, React, Next.js, HTML, CSS, DOM, ARIA, forms, modals, tables, SPA routing, keyboard support, browser testing, or web design systems.

Read `references/mobile.md` when the request is about mobile apps, React Native, Expo, iOS, Android, native screens, touch gestures, VoiceOver, TalkBack, font scaling, mobile assistive technology, or mobile design systems.

Read both references when the request involves shared design systems, cross-platform components, responsive web plus native apps, or a component that ships on both web and mobile.

## Workflow

1. Classify the target:
   - Web
   - Mobile
   - Cross-platform
   - Design/spec review
   - Code review
   - Test plan
   - Bug fix
2. Identify context:
   - Framework: React, Next.js, Vue, Angular, React Native, Expo, native iOS, native Android, or other
   - Component type: button, form, modal, dropdown, tabs, table, navigation, list, media, map, camera, authentication flow, or other
   - User need: audit, implementation, code fix, checklist, QA plan, or bug diagnosis
3. Inspect provided code, screenshots, specs, logs, or behavior reports. If exact compliance cannot be verified, state what is missing and list the checks needed.
4. Produce concrete findings and fixes. Prefer minimal, correct fixes over decorative advice.
5. Explain tradeoffs when accessibility requirements conflict with custom UI behavior.

## Default Output Format

Use this format unless the user asks for another shape:

```txt
Accessibility risk summary
Issues found or likely issues
Fixes with priority: Critical, High, Medium, Low
Code-level recommendations
Manual test checklist
Automated test suggestions
Acceptance criteria
```

For code review, lead with findings ordered by severity. Include file and line references when available.

## Implementation Rules

- Prefer native semantic elements on web.
- Avoid unnecessary ARIA.
- Use ARIA only when it improves the accessibility tree and interaction model.
- Preserve keyboard and screen reader behavior.
- Do not remove visible focus indicators.
- Do not rely on color alone.
- Respect reduced motion and font scaling.
- For React Native, use `accessibilityRole`, `accessibilityLabel`, `accessibilityHint`, `accessibilityState`, `accessibilityValue`, `accessibilityActions`, `onAccessibilityAction`, `importantForAccessibility`, `accessible`, `accessibilityLiveRegion`, `AccessibilityInfo`, and focus management where appropriate.

## Strict Review Checks

When reviewing AI-generated UI, check:

- keyboard access
- focus order
- visible focus
- accessible names
- semantic structure
- form labels and errors
- modals and dialogs
- dynamic updates
- touch target size
- screen reader order
- contrast and text scaling

Final reports must use concise developer language and include exact remediation steps.
