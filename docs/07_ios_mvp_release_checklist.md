# OneDone iOS MVP / TestFlight Release Checklist

Purpose: practical release checklist for the current implemented MVP runtime.

## 1. Scope Snapshot

Current implementation assumptions:
- Remote backend runtime is default for real app usage.
- Mock mode is only for SwiftUI previews/development fallback.
- Supabase Auth email/password is implemented.
- Backend access-state drives routing/gating.
- StoreKit 2 purchase/restore flow is implemented.
- Subscription backend mirror scaffold is implemented (`ios_verified_mirror`).
- iOS never calls OpenAI directly.

## 2. App Configuration Checklist

Configure runtime values only in a local unshared scheme (example: `OneDone Local`):
- `ONEDONE_SUPABASE_URL`
- `ONEDONE_SUPABASE_ANON_KEY`
- `ONEDONE_FUNCTIONS_BASE_URL`
- `ONEDONE_SUBSCRIPTION_PRODUCT_ID`

Rules:
- Shared scheme must stay value-free (no concrete env values committed).
- Local unshared scheme holds concrete runtime values.
- Never place Supabase `service_role` in iOS config.
- Never place OpenAI key in iOS config.

## 3. Xcode Local Scheme Setup

- Confirm shared scheme is safe to commit (placeholders/empty values only).
- Confirm local unshared scheme has required runtime values.
- Confirm StoreKit local config file exists and is selected for local development testing:
  - `OneDone.storekit`
- Confirm local StoreKit product ID matches `ONEDONE_SUBSCRIPTION_PRODUCT_ID`.

## 4. Backend Deployment Checklist

Confirm these Edge Functions are deployed and reachable:
- [ ] `complete-onboarding`
- [ ] `get-access-state`
- [ ] `analyze-task`
- [ ] `answer-clarification`
- [ ] `generate-reply`
- [ ] `list-tasks`
- [ ] `get-task-detail`
- [ ] `get-task-outputs`
- [ ] `get-task-events`
- [ ] `get-checklist-items`
- [ ] `get-reminders`
- [ ] `update-task-status`
- [ ] `message-marked-sent`
- [ ] `reminder-create`
- [ ] `reminder-update`
- [ ] `reminder-cancel`
- [ ] `reminder-snooze`
- [ ] `notification-triggered`
- [ ] `validate-subscription`
- [ ] `restore-purchases`
- [ ] `feedback`
- [ ] `delete-task`
- [ ] `delete-all-data`
- [ ] `delete-account`

## 5. Supabase Secrets and Database Checklist

Auth/database:
- [ ] Hosted migrations applied.
- [ ] Supabase Auth email/password enabled and tested.
- [ ] Email confirmation policy explicitly chosen for this release phase.
- [ ] RLS enabled for user-scoped MVP tables.

Required backend secrets (Supabase only):
- [ ] `OPENAI_API_KEY`
- [ ] `OPENAI_MODEL`

Core table readiness:
- [ ] `profiles`
- [ ] `subscriptions`
- [ ] `subscription_events`
- [ ] `tasks`
- [ ] `task_outputs`
- [ ] `task_events`
- [ ] `clarifications`
- [ ] `checklist_items`
- [ ] `reminders`
- [ ] `task_feedback`
- [ ] `usage_events`

Security:
- [ ] No backend secrets in iOS project.
- [ ] No `service_role` in iOS project.
- [ ] OpenAI key is server-side only.

## 6. Manual QA Script

### 6.1 Auth and session (real environment)
- [ ] Sign up with email/password against hosted Supabase.
- [ ] Log in with valid credentials.
- [ ] Invalid credentials show safe user copy.
- [ ] Relaunch restores session correctly.
- [ ] Logout clears session and returns to auth.
- [ ] Session-expiry/token-refresh path is validated in real runtime.

### 6.2 Onboarding and access routing
- [ ] New account receives `onboarding_required`.
- [ ] Completing onboarding calls `complete-onboarding`.
- [ ] Starter access becomes active.
- [ ] Locked states gate create/generate actions and still allow existing-task viewing.

### 6.3 AI loop
- [ ] `analyze-task` success path works.
- [ ] Clarification path works through `answer-clarification`.
- [ ] Task result flow shows expected next-step/checklist/reply/reminder actions.
- [ ] `generate-reply` works for saved tasks.
- [ ] Rate-limit response is handled gracefully.

### 6.4 Task reads and actions
- [ ] My Tasks loads remote data (`list-tasks`) and pull-to-refresh works.
- [ ] Task Detail loads detail/output/events/checklist/reminder reads.
- [ ] `update-task-status` sync works.
- [ ] `message-marked-sent` sync works.

### 6.5 Reminders (real device)
- [ ] Reminder create/update/cancel/snooze flows work on real device.
- [ ] Local notification scheduling succeeds with system permission granted.
- [ ] Permission-denied path is clear and non-technical.
- [ ] Reminder sync payloads remain aligned with backend state.

### 6.6 Subscription and StoreKit (real transaction checks)
- [ ] Purchase flow triggers `validate-subscription` with expected mirror payload.
- [ ] Restore flow triggers `restore-purchases` and reconciles access-state.
- [ ] Access-state refresh reflects entitlement changes.
- [ ] Mirror environments are valid (`xcode`, `sandbox`, `testflight`).
- [ ] Real TestFlight transaction scenarios are validated where applicable.

### 6.7 Runtime mode and gating
- [ ] Normal launch uses remote runtime by default.
- [ ] Mock mode is only entered intentionally for preview/dev fallback.
- [ ] Center `Task` button path and gated behavior work in locked states.

### 6.8 Real-device memory/performance observation
- [ ] Observe long-list navigation (My Tasks/Task Detail) on real device.
- [ ] Observe New Task -> Clarification/Result loop for memory spikes/jank.
- [ ] Observe reminder-heavy task detail interactions on real device.
- [ ] Record any regressions for follow-up before wider release.

### 6.9 Log and safety checks
- [ ] No sensitive values appear in user-visible errors.
- [ ] No auth tokens/passwords are logged in client-visible channels.
- [ ] No raw private user content is logged in diagnostic paths.
- [ ] Shared scheme still has no concrete runtime values.

## 7. StoreKit Notes

- Local StoreKit file is for development testing only.
- TestFlight/App Store behavior still depends on App Store Connect product setup.
- Subscription mirror scaffold accepts only `xcode`, `sandbox`, `testflight` environments.
- If local StoreKit transactions conflict across test users, reset local StoreKit transactions and clean related test data before retry.

## 8. Known Limitations and Deferred Items

- Attachments/OCR are deferred (not available in current runtime).
- Multi-task split review flow is not available in current runtime.
- Pending-question persistence / answer-later intake flow is not available yet.
- App Store Server Notifications are deferred.
- Full Apple Server API validation is deferred.
- Sign in with Apple is not implemented in current MVP.
- No autonomous external actions.

## 9. Final Go / No-Go Checklist

Release/share only when all are true:
- [ ] iOS builds and runs on real device.
- [ ] Local scheme/runtime configuration is correct.
- [ ] Backend functions are deployed and reachable.
- [ ] Hosted migrations and RLS are verified.
- [ ] Required backend secrets are configured in Supabase.
- [ ] Auth/session/access flows pass real-environment QA.
- [ ] AI loop + task reads/actions + reminders pass QA.
- [ ] StoreKit purchase/restore mirror flow passes transaction QA.
- [ ] Real-device memory/performance review completed.
- [ ] No sensitive logs or leaked config values.
