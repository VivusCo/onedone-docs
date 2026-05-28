# OneDone Approved Visual Direction

## 1. Purpose and Status

This document records the approved visual direction for OneDone MVP and aligns it to the current implemented iOS state.

Status:
- Approved baseline for SwiftUI visual implementation and QA.
- `design-prototype/` remains a visual reference only.
- React/Tailwind prototype code is not production code and must not be ported directly.

## 2. Approved Visual Direction (current implementation-aligned)

Foundation:
- Warm off-white base canvas.
- Soft radial gradient accents (green/orange family).
- Calm, practical, trustworthy tone.
- Simple and human visual hierarchy.

Surface system:
- Non-heavy-glass card/surface treatment.
- Subtle borders and restrained shadows.
- Avoid heavy per-row blur/material effects for repeated list content.
- Keep repeated list rows performance-safe.

Accent rules:
- Deep green primary accent.
- Warm orange accent used sparingly.
- No purple.
- No neon.

Visual constraints:
- No heavy dashboard density.
- No chart-first UI framing.
- No robot/chatbot-heavy AI visuals.
- No active attachments/OCR affordances.

## 3. Approved Navigation Structure

- Use custom bottom navigation.
- Elevated center button is the primary creation entry.
- Center button label is `Task`.
- Home is overview/shortcut hub only.
- New Task opens from the center `Task` button.

MVP alignment:
- Guided self-service flow, not chatbot conversation framing.
- Limited mode still allows viewing existing tasks/details.
- Create/generate actions remain gated in locked states.

## 4. Approved Screen Behaviors

### Home

Required:
- Access pill at top.
- Warm greeting.
- Illustration/support card (abstract, not chart-heavy).
- Quick shortcut cards.
- Small `Next up` card when applicable.

Rules:
- Home does not include a large direct input block.
- Creation entry is center `Task` button.

### New Task

Required:
- Text-first task entry context.
- Large task description area.
- Centered `Analyze Task` CTA.

Rules:
- No active attachments/OCR UI.

### Clarification

Required:
- One clear question surface.
- Option rows with lightweight card treatment.
- Centered primary/secondary actions.

### Task Result

Required:
- Clear next-step emphasis.
- Checklist with checked/unchecked interaction.
- Draft Reply and Reminder actions.

Note:
- Current iOS checklist toggles are local-only behavior.

### My Tasks

Required:
- Compact filters.
- Status badges must not wrap.
- Long titles/next-step previews must truncate safely.
- Repeated rows must stay lightweight/performance-safe.

### Task Detail

Required:
- Current next-step card.
- Timeline section.
- Checklist/progress section.
- Compact readable section rhythm.

### Draft Reply

Required:
- Reply text as primary content.
- Compact `Copy` action near reply text.
- No oversized copy CTA.

### Reminder

Required:
- Calm reminder controls.
- Centered key actions.

### Limited Mode

Required:
- Calm locked-state messaging.
- Explain what remains available.
- Centered trial/restore CTAs.

### Subscription Gate

Required:
- Calm modern gate surface.
- Centered `Start Trial` and `Restore` actions.
- No `Not now` bypass after Starter expiry.

## 5. Reusable UI Patterns

Approved patterns:
- Warm radial background wrapper.
- Lightweight surface cards for content.
- Compact status pills/badges.
- Compact list-row card variants for repeated content.
- Center-emphasis CTAs on critical decision screens.
- Error/info banners with plain non-technical copy.

Key CTA centering required on:
- New Task
- Clarification
- My Tasks Empty
- Reminder
- Limited Mode
- Subscription Gate

## 6. Interaction and Copy Guidance

Interaction:
- One primary action per screen should be obvious.
- Keep flows action-first and compact.
- Avoid visual clutter from decorative layers in repeated surfaces.

Copy:
- Keep user-facing copy clear and non-technical.
- Avoid backend/internal terminology in user UI.
- Avoid overstating AI language.

## 7. Implementation Guardrails

- Do not change backend logic.
- Do not change auth/session logic.
- Do not change StoreKit logic.
- Do not change remote service behavior.
- Do not introduce unsupported features.
- Do not present attachments/OCR as available.
- Do not present autonomous external actions as available.
- Keep mock/previews working as fallback.
- Keep readability and accessibility first-class.

Runtime guardrails:
- Remote runtime is default for real usage.
- Mock runtime is preview/dev fallback only.

## 8. SwiftUI Handoff Notes

- Treat prototype assets as visual reference only.
- Rebuild in SwiftUI-native components/tokens.
- Favor restrained, performant surfaces over heavy material stacks.
- Preserve current bottom-nav + center `Task` behavior.
- Preserve compact list-row rendering patterns.

Out of scope for this document:
- API/backend contract changes.
- Subscription business logic changes.
- New product capability design beyond approved MVP scope.

## 9. Acceptance Checklist

Visual system:
- [ ] Warm off-white + radial accent background is applied consistently.
- [ ] Surface cards remain lightweight and readable.
- [ ] Repeated list rows avoid heavy material effects.
- [ ] Deep green is primary accent; warm orange is sparse.
- [ ] No purple and no neon.
- [ ] No chart-heavy default visual language.
- [ ] No chatbot/robot-heavy visual identity.

Navigation/flow:
- [ ] Elevated center `Task` button is primary creation entry.
- [ ] Home remains overview/shortcut only.
- [ ] New Task opens from center `Task` button.
- [ ] Limited mode and subscription gate behavior are represented correctly.

Scope boundaries:
- [ ] Attachments/OCR are not shown as available.
- [ ] Multi-task split review is not presented as currently available.
- [ ] Document does not imply unsupported capability as implemented.
