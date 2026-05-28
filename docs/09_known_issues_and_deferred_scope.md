# OneDone Known Issues, Limitations, and Deferred Scope

## 1. Purpose

This document separates:
- current implemented MVP runtime,
- production hardening still required,
- planned next capabilities not yet implemented,
- deferred v1.1+ scope.

Use this as the practical boundary for planning and release decisions.

## 2. Current implementation baseline (for context)

Implemented baseline includes:
- iOS remote runtime by default (mock only for preview/dev fallback).
- Email/password auth with session restore/logout.
- Access-state-driven routing and gating.
- New Task -> Analyze -> Clarification or Task Result flow.
- My Tasks and Task Detail remote reads.
- Draft reply and sent-status actions.
- Reminder sync endpoints with local-notification-first iOS behavior.
- StoreKit purchase/restore mirrored to backend via `ios_verified_mirror` scaffold.

## 3. Implemented MVP but still needs production hardening

Subscription maturity:
- StoreKit local/TestFlight scaffold works for MVP validation.
- `ios_verified_mirror` is a scaffold, not full production-grade subscription trust.
- Real TestFlight/App Store behavior still requires App Store Connect setup and operational validation.

Auth and account hardening:
- Email/password auth is implemented.
- Production email confirmation/deep-link behavior still needs final rollout validation.
- Sign in with Apple is not currently implemented and may be required before broader/public release.

Operational hardening:
- Monitoring/alerting and observability need broader production coverage.
- Rate limits are implemented but may require tuning on real traffic.
- Support/admin tooling remains minimal.

## 4. Planned next capabilities (not implemented yet)

Task Intake System is planned next major capability and is not currently implemented.

Not implemented today:
- Intake sessions (`intake_sessions`-style flow).
- Dedicated multi-task detection + split-review user flow.
- Pending-question persistence beyond current clarification loop.
- "Answer missing details later" intake resume flow.
- Structured per-item split confirmation before final task creation.

Reference spec (planned source of truth):
- `docs/15_task_intake_system.md`

## 5. Deferred product features (outside current MVP)

Deferred/not available:
- Attachments.
- OCR/PDF upload processing.
- Autonomous external actions (automatic cancellations/payments/sending).
- External account integrations (email/calendar/messaging providers).
- Advanced template systems beyond current app-side template set.

## 6. Deferred iOS/platform items

Not complete for broader public rollout:
- Sign in with Apple (if required by release policy).
- Final App Store Connect subscription setup and broader TestFlight matrix.
- Production deep-link/universal-link polish.
- Accessibility polish beyond current baseline.
- Offline/cache behavior polish beyond current flow.

## 7. Deferred backend/security items

Not complete yet:
- Full Apple Server API validation.
- App Store Server Notifications ingestion/reconciliation.
- Full production subscription reconciliation operations.
- Expanded admin/support operational tooling.
- Broader production analytics/observability and incident runbooks.

## 8. Performance status and caveats

Optimizations already present in current iOS implementation:
- Performance-safe repeated list rows (`listRow`-style surfaces/badges).
- Reduced heavy material/glass usage in repeated rows.
- Reduced duplicate full-screen background overdraw.
- Lazy list rendering and shared formatter use in key list/detail screens.

Known remaining risks:
- Some screens still use layered shadows/overlays that may cost on older hardware.
- Large text editor + rich card stacks can still be memory-sensitive.
- Concurrent detail refresh calls can feel bursty on weak networks.

Still required:
- Manual real-device memory/performance QA across long sessions and slower devices.

## 9. Known technical caveats

Current caveats:
- Shared Xcode scheme must remain value-free.
- Concrete env values belong in local unshared schemes only.
- Mock mode is not production runtime.
- `usage_events` must not store raw user content.
- Checklist toggles in current iOS Task Result/Task Detail behavior are local-only.
- If docs are updated in `onedone-docs`, synced copies in `onedone-ios/docs` and `onedone-backend/docs` must be refreshed.

## 10. Release blockers vs non-blockers

| Item | Status | Blocks internal MVP? | Blocks TestFlight? | Blocks public App Store? | Notes |
|---|---|---|---|---|---|
| Core remote runtime (auth/access/task/reminder/reply) | Implemented | No | No | No | Working baseline is present. |
| StoreKit local testing scaffold | Implemented scaffold | No | No | Yes | Dev/testing scaffold only. |
| `ios_verified_mirror` subscription mode | Implemented scaffold | No | No | Yes | Not full production validation model. |
| Full Apple Server API validation | Deferred | No | No | Yes | Required for strong production subscription validation. |
| App Store Server Notifications | Deferred | No | No | Yes | Needed for robust subscription lifecycle reconciliation. |
| App Store Connect subscription setup | Pending setup | No | Yes | Yes | Needed for real purchase lifecycle testing. |
| Task Intake System (sessions/split/pending answers) | Planned next capability | No | No | No | Not required for current MVP but not implemented yet. |
| Attachments/OCR/PDF | Deferred | No | No | No | Explicitly outside current MVP. |
| Real-device performance verification | In progress | No | Yes | Yes | Must complete before broader release confidence. |
| Sensitive log safety validation | Required | No | Yes | Yes | Must verify no tokens/password/raw content leaks. |

## 11. Recommended next roadmap

Before TestFlight confidence signoff:
1. Complete hosted env parity checks (migrations/functions/RLS/secrets).
2. Run full real-device QA for auth/session, StoreKit, reminders, and performance.
3. Validate log safety and scheme hygiene across team setups.

Before public App Store release:
1. Implement full Apple Server API validation.
2. Implement App Store Server Notifications pipeline.
3. Finalize auth-entry production details (including Sign in with Apple decision).
4. Expand observability and support/admin tooling.

Post-MVP v1.1:
1. Implement Task Intake System planned capabilities.
2. Add attachments/OCR/PDF flows.
3. Expand split/task-composition experiences after intake foundation is shipped.
