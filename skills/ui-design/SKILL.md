---
name: ui-design
description: Use when the users asks for a ui/frontend change or generation.
---

# UI design

Take the user's brief through implementation and verification. Preserve the requested scope, existing product conventions, stack, and explicit design direction. Inspect the relevant code, dependencies, and project instructions first; identify the audience, primary task, content, and constraints. Make reasonable decisions where the brief leaves room, and ask only when a missing decision blocks progress.

## Route to specialist skills

Read the relevant `SKILL.md` from the paths below before applying it; resolve paths relative to the repository root. Pass the actual task and relevant files as context, rather than invoking a skill without a question.

- **Visual direction and build:** use `skills/frontend-design/SKILL.md` for new or substantially redesigned UI. Adapt its design process to the brief and existing design system.
- **Mobile web:** use `skills/mobile-native/SKILL.md` when mobile, touch, full-screen, or installed-web-app behavior matters. Apply relevant fixes and distinguish emulation from real-device verification.
- **Library selection:** consult `skills/pick-ui-library/SKILL.md` only when the user explicitly invokes it; otherwise inspect and reuse the project's existing dependencies and choose additions as needed. Respect its explicit-only setting.
- **Sonner:** use `skills/ask-sonner/SKILL.md` when implementing or troubleshooting Sonner notifications; do not introduce a toast library without a need.
- **Alternatives:** use `skills/prototype/SKILL.md` only when the user explicitly invokes it or explicitly requests multiple interactive variants. Follow its isolated exploration and selection workflow; do not hold up an ordinary implementation for a variant picker.
- **Stress testing:** when the user requests adversarial UI testing, use `skills/break-ui/SKILL.md` with the actual component. Follow its report-before-fix workflow. For ordinary builds, independently check realistic long text, missing data, empty states, and other relevant edge cases without adding a permanent toggle.
- **UI review:** use `skills/web-design-guidelines/SKILL.md` to review changed web UI; provide the changed files or patterns and fetch the current guidelines as directed. Fix relevant findings within the authorized implementation scope and report any unresolved issues.

## Deliver and verify

Implement the requested behavior, responsive layouts, meaningful content and interaction states. Use semantic controls, keyboard access, visible focus, accessible names, appropriate contrast, and reduced-motion support. Keep changes consistent with the product and avoid adding unrelated dependencies or flows.

Run the project's relevant checks and exercise the affected interactions and responsive states using the available verification tools and applicable tool policies. Where visual or device verification is unavailable, say what was and was not verified. Report the result, checks performed, and remaining issues succinctly.
