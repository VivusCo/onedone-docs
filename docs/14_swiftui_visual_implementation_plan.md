# OneDone SwiftUI Visual Implementation Plan

Status: implementation sync and remaining-polish plan.
Date basis: 2026-05-28 audits.
Source basis:
- `../onedone-ios/docs/current_ios_state_audit.md`
- existing docs in this repository (especially `docs/13_approved_visual_direction.md`)

## 1. Purpose

This plan tracks what is already implemented in the SwiftUI visual system and what remains for polish/verification.

It is not a request to change product logic.

## 2. Already implemented visual/runtime foundations

Implemented and in active use:
- Warm off-white background with radial accent treatment.
- Non-heavy surface/card system for core UI.
- Shared design tokens and components (colors/style/card/button/badge/banner primitives).
- Custom bottom navigation with elevated center `Task` button.
- Home as overview/shortcut screen (no large direct input block).
- New Task opened from center `Task` button.
- Compact My Tasks row rendering with non-wrapping status badges.
- Lightweight repeated-list row styles to reduce heavy material cost.

Runtime behavior preserved:
- Remote runtime is default for real usage.
- Mock runtime is fallback for previews/development.

## 3. Current screen-level status

Implemented screens in current flow:
- Auth
- Welcome
- Onboarding
- Starter Access Intro
- Home
- Templates
- New Task
- Clarification
- Task Result
- My Tasks
- Task Detail
- Draft Reply
- Access
- Subscription Gate
- Settings

Current task flow surfaces are present:
- New Task
- loading/analyzing state
- clarification
- task result
- checklist local toggle behavior
- draft reply
- reminder entry/actions

## 4. Already completed performance-oriented visual changes

Completed improvements:
- Performance-safe repeated list rows.
- Reduced heavy per-row material/glass effects.
- Reduced duplicate full-screen background layering.
- Lazy list rendering in task list surfaces.
- Shared date formatter usage in key list/detail surfaces.

Why this matters:
- Protects scrolling and memory behavior on weaker devices.
- Keeps the visual direction warm/modern without expensive row-level effects.

## 5. Remaining visual polish (without logic changes)

Remaining work is polish/consistency, not feature expansion:
- Ensure full consistency of lightweight row surfaces across all repeated lists.
- Continue tightening contrast/readability on layered cards.
- Review shadow intensity and overlay layering on older devices.
- Confirm all key CTA alignment rules remain consistent with approved direction.
- Ensure no screen regresses toward heavy dashboard density.

Out of scope for this plan:
- Backend/API changes.
- Auth/session flow changes.
- StoreKit logic changes.
- New product capability implementation.

## 6. Guardrails and boundaries

Must keep:
- No purple.
- No chart-heavy default visual framing.
- No attachments/OCR presented as active.
- No autonomous external action UX.
- No split-review intake flow presented as current capability.

Must preserve:
- Current access gating behavior.
- Current task creation entry via center `Task` button.
- Current mock/runtime separation.

## 7. QA and validation checklist (remaining)

Visual QA:
- [ ] Warm gradient background and restrained surfaces are consistent across flows.
- [ ] Repeated rows remain lightweight and readable.
- [ ] Status badges remain single-line and stable.
- [ ] No unsupported features are implied by UI affordances.

Real-device performance QA:
- [ ] Observe scrolling/performance on My Tasks with larger datasets.
- [ ] Observe New Task/Task Result/Task Detail transitions on real device.
- [ ] Observe memory behavior during long usage sessions.
- [ ] Verify no visual regressions under dynamic type and accessibility settings.

Safety QA:
- [ ] User-facing errors remain non-technical.
- [ ] No sensitive data appears in UI diagnostics.

## 8. Relationship to planned next capabilities

Task Intake System is planned next capability and not implemented yet.

This visual plan should not imply currently available intake features such as:
- intake sessions
- multi-task split review
- pending-question persistence
- answer-missing-details-later intake flow

Reference:
- `docs/15_task_intake_system.md` (planned spec only)
