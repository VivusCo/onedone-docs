# OneDone Project Overview

This document is the current source-of-truth overview of the implemented OneDone MVP as of 2026-05-28.

Source basis:
- `../onedone-ios/docs/current_ios_state_audit.md`
- `../onedone-backend/docs/current_backend_state_audit.md`
- existing docs in this repository

## 1. Product summary

OneDone is a guided self-service AI assistant for everyday admin tasks.

It is designed for users who need help turning messy real-world requests into clear next actions, checklist steps, draft replies, and reminders.

Current MVP summary:
- iOS app is production-like in flow and uses remote runtime by default.
- Backend access-state controls routing and capability gating.
- Core loop is implemented: New Task -> Analyze -> Clarification or Result -> follow-through actions.
- Existing tasks remain viewable in limited states.
- Mock mode exists for previews/development fallback, not primary runtime.

## 2. Core user value (implemented)

- Convert vague task input into actionable next steps.
- Ask clarifying questions when needed in the current clarification loop.
- Provide checklist guidance and task timeline context.
- Generate draft replies and track sent/follow-up state.
- Support reminder scheduling/snooze/cancel with iOS local-notification-first behavior and backend sync.

## 3. Current iOS app flow and screens

Implemented app phases and routing:
- `auth`
- `welcome`
- `onboarding`
- `starterIntro`
- `access`
- `accessStateLoading`
- `accessStateError`
- `main`

Implemented main screens:
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
- Subscription Gate
- Access
- Settings

Navigation/runtime behavior:
- Custom bottom navigation with elevated center `Task` button opens New Task as a sheet.
- Home is an overview/shortcut hub (no large direct input block).
- Remote runtime is default for real usage.
- Mock runtime is development/preview fallback only.

## 4. Current task flow (implemented)

Primary chain:
1. User opens New Task from center `Task` button (or shortcut/template entry).
2. App submits to `POST /analyze-task` (with idempotency support).
3. Backend returns either clarification or task analysis.
4. Clarification answers are submitted via `POST /answer-clarification`.
5. User lands in Task Result, then continues via Task Detail, Draft Reply, and reminders.

Implemented follow-through actions:
- Draft reply generation via `POST /generate-reply`.
- Sent-state update via `POST /message-marked-sent`.
- Task status update via `POST /update-task-status`.
- Reminder create/update/cancel/snooze via reminder endpoints.

Current checklist behavior in iOS:
- Checklist toggles in Task Result and Task Detail are local-only in current iOS behavior.
- Do not treat checklist toggle state as persisted from current iOS interactions.

## 5. Current backend capability summary

Implemented backend function groups:
- Access/onboarding: `complete-onboarding`, `get-access-state`
- AI task loop: `analyze-task`, `answer-clarification`, `generate-reply`
- Task reads: `list-tasks`, `get-task-detail`, `get-task-outputs`, `get-task-events`, `get-checklist-items`, `get-reminders`
- Task actions: `update-task-status`, `message-marked-sent`
- Reminder actions: `reminder-create`, `reminder-update`, `reminder-cancel`, `reminder-snooze`, `notification-triggered`
- Subscription mirror scaffold: `validate-subscription`, `restore-purchases`
- Privacy/support: `feedback`, `delete-task`, `delete-all-data`, `delete-account`

Data/security posture (implemented):
- RLS and user ownership checks are active.
- `analyze-task` includes idempotency table support.
- Rate-limited responses are structured.
- OpenAI is called only from backend using Supabase secrets.
- `usage_events` explicitly avoids raw user content payloads.

## 6. Access and subscription model (implemented)

Access model:
- Onboarding required is checked first.
- Starter access is backend-controlled.
- Trial/subscription states are driven by mirrored StoreKit entitlement data.
- Limited mode allows viewing existing data while create/generate actions are gated.

Subscription scope today:
- StoreKit 2 purchase/restore flow is connected on iOS.
- Backend supports `ios_verified_mirror` scaffolding for `xcode`, `sandbox`, `testflight` environments.
- Full Apple server-side validation and App Store Server Notifications are deferred.

## 7. Current visual direction in implementation

Current shipped direction aligns to:
- Warm off-white base with soft radial gradient accents.
- Non-heavy-glass surface system (performance-safe, non-material repeated rows).
- Compact list rows and non-wrapping status badges for My Tasks.
- Custom bottom nav with elevated center `Task` button.
- No purple.
- No chart-heavy default UI motifs.
- No active attachments/OCR UI.

## 8. Current implementation limitations (explicit)

Not implemented yet:
- Task intake sessions model/surface.
- Multi-task detection split-review flow as a dedicated user flow.
- Pending question persistence model beyond current clarification loop.
- Answer-missing-details-later intake flow.
- Attachments/OCR/PDF processing flow.
- Sign in with Apple in current auth flow.

Important boundary note:
- The Task Intake System is planned next capability and is not currently implemented in app/backend runtime.

## 9. Performance status and remaining QA reality

Current improvements already present:
- Performance-safe repeated list rows (lightweight surfaces/badges).
- Reduced heavy material/glass usage in repeated rows.
- Reduced duplicate full-screen background overdraw.
- Lazy task list rendering and shared formatters in key screens.

Remaining manual QA required:
- Real-device memory/performance observation on long sessions and mixed flows.
- Real-device validation on older/slower hardware profiles.
- Real-network reminder-sync behavior under weak connectivity.

## 10. Release and QA reality checks

Still required before broader rollout:
- Real hosted-environment auth/session restore verification.
- Real StoreKit transaction lifecycle verification (purchase/restore conflicts and edge cases).
- Real-device reminder sync and permission-path verification.
- Log safety verification (no sensitive tokens/passwords/raw content in client-visible paths).

## 11. Security and privacy principles

- iOS never calls OpenAI directly.
- OpenAI key is backend-only (Supabase secrets).
- `service_role` must never be present in iOS config.
- Auth and task operations are user-scoped.
- `usage_events` must not store raw user content.

## 12. Planned next major capability

Task Intake System (planned next capability):
- Better intake classification for single-ready, single-needs-details, and multi-task candidates.
- Persisted intake context and questions for resume-later completion.
- Controlled split review and confirmation flow before creating final tasks.

Reference:
- `docs/15_task_intake_system.md` is the planning specification and is not a statement of current implementation.

## 13. Glossary

Starter Access:
- Backend-controlled initial access window after onboarding.

Limited Mode:
- User can view existing tasks/details, but creation/generation actions are gated.

access_state:
- Backend payload that drives app routing and capability gating.

task_analysis:
- Structured output returned from task analysis path.

clarification:
- Blocking question path used when additional information is required to proceed.

ios_verified_mirror:
- Current MVP/TestFlight subscription sync mode where iOS-verified entitlements are mirrored to backend.
