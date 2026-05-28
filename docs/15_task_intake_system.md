# OneDone Task Intake System Specification

Status:
- Planned next major capability.
- Not implemented in current iOS/backend runtime as of 2026-05-28.
- This document is a future implementation source of truth, not a statement of shipped behavior.

Current implementation boundary:
- Current app/backend supports New Task -> analyze -> clarification/result in the existing loop.
- Dedicated intake sessions, split-review flow, and pending-question resume-later intake model are not implemented yet.

## 1. Purpose

The Task Intake System is the product capability that turns messy user input into a deterministic next step.

What this system does:
- Accepts free-form user input from New Task.
- Decides whether the input is one ready task, one task that needs details, or multiple tasks.
- Preserves unfinished intake context so the user can continue later without restarting.

Why it matters:
- Real-life admin requests are often incomplete or bundled.
- Strong AI output requires enough context.
- Users do not always have all details available immediately.

How this supports messy input:
- Keeps the primary path focused on collecting needed details.
- Uses a resilience fallback: keep pending questions attached to the task when the user leaves.
- Allows the user to return later, answer missing details, and generate or update results.

Product principle:
- This is not a "skip questions" feature.
- The system should ask for required details when they materially change the next step.
- If answers are delayed, context must be preserved and resumable.

## 2. Core scenarios

### Scenario A — Single ready task

Input example:
- "I was charged twice by Acme Fitness yesterday. Please help me ask for a refund. The charge was $79 and I canceled last month."

AI classification:
- `intake_type: single`
- `readiness_state: ready`
- One candidate task, enough actionable details.

Backend decision:
- Accept structured output.
- Create task record immediately.
- Trigger analysis/result generation flow.

App UI flow:
- New Task -> Analyzing -> Task Result.
- Show generated next step, checklist, and reply action.

Result state:
- Task status transitions to active workflow state.
- No pending intake questions.

### Scenario B — Single task with missing details

Input example:
- "Help me cancel my internet service."

Needed vs helpful details:
- Needed details: facts that change next action (for example, provider name, cancellation channel available, deadline risk).
- Helpful details: facts that improve quality but are not strictly blocking (for example, plan tier).

AI classification:
- `intake_type: single`
- `readiness_state: needs_details` is the default when required details are missing.
- `readiness_state: can_start_with_limited_details` is allowed only when backend rules confirm missing details do not block a useful first result.
- One candidate task plus missing questions.

Clarification screen behavior:
- Show concise question set ordered by importance.
- Focus on needed questions first.
- Avoid over-questioning.

Pending questions behavior:
- Save questions to intake/task context.
- Save partially answered responses.
- Mark unanswered items as pending.

Save and continue later:
- User can stop after seeing or answering some questions.
- Task remains visible with `Needs details` state.
- User can return from My Tasks/Task Detail without re-entering original request.

Answer later and update/generate result:
- User taps `Answer missing details`.
- Submits answers.
- Backend re-evaluates readiness and updates or generates result.

### Scenario C — Multiple tasks in one input

Input example:
- "I need to cancel my gym, ask my landlord about the deposit, and set a reminder to pay my electricity bill on Friday."

AI classification:
- `intake_type: multiple`
- `readiness_state: split_review`
- Multiple task candidates extracted.

Split Review behavior:
- Present each candidate as an item.
- User confirms which items to keep.
- Each item displays readiness (`ready` or `needs_details`).

Per-item routing:
- Ready items: create task records and continue to result generation only after split confirmation.
- Needs-details items: keep intake-backed candidate entries with pending questions preserved until confirmation.

Preserve questions for incomplete items:
- Questions stay attached per item.
- User can answer later item-by-item.

Split persistence rule:
- During `split_review`, store only `intake_items` as candidates.
- Do not create final task records before the user confirms the split.

Future support (out of v1 implementation scope):
- Edit candidate text before create.
- Remove candidate.
- Merge overlapping candidates.

## 3. Readiness states

| State | Meaning | User-facing label | Backend behavior | UI behavior |
|---|---|---|---|---|
| `ready` | Enough detail to produce strong next step now | `Ready` | Create/update task and trigger result generation | Route to result flow |
| `needs_details` | Missing required details that block strong output | `Needs details` | Store pending questions, defer full generation | Route to clarification; show resume-later actions |
| `can_start_with_limited_details` | Can produce a provisional first result only when missing details are non-blocking | `Can start now` + `More details improve this` | Generate limited result, keep pending questions, and require follow-up for quality improvement | Show provisional result with `Answer missing details` action |
| `split_review` | Input contains multiple candidate tasks requiring user confirmation | `Review tasks` | Store candidates, wait for split confirmation | Route to split review screen |

Notes:
- State evaluation comes from AI output but is enforced by backend product rules.
- Backend must normalize invalid or conflicting state output.
- `can_start_with_limited_details` is not a default skip route.
- If needed questions are still blocking, clarification remains the primary route.

## 4. Missing details model

Question importance tiers:
- `needed`: required for reliable next action.
- `helpful`: improves specificity or quality.
- `nice_to_have`: optional context with low impact.

Rules:
- Needed details are asked first.
- Helpful details should be limited and concise.
- Nice-to-have questions should be rare in v1.
- Do not ask questions that do not change next-step logic.
- Ask only when the answer materially changes what OneDone recommends.

Practical limits:
- Prefer one question at a time in UI.
- Keep wording plain and scenario-specific.

Question quality guardrails:
- Questions must be concrete, short, and directly answerable.
- Avoid abstract or generic prompts (for example, "Can you share more context?").
- Ask only if the answer changes next step quality or output quality.

UI display rules by importance:
- `needed` -> show as the primary clarification card/input.
- `helpful` -> show as compact chip or expandable row.
- `nice_to_have` -> hidden in v1 unless user explicitly requests deeper refinement.

## 5. AI responsibilities

AI should:
- Detect single-task vs multi-task input.
- Extract task candidates from messy text.
- Generate concise candidate title and summary.
- Determine readiness state per candidate.
- Generate missing-detail questions.
- Classify question importance (`needed`, `helpful`, `nice_to_have`).
- Suggest answer type (`text`, `single_select`, `multi_select`, `date`, `number`).
- Suggest options when constrained choices exist.
- Return structured output only.

AI should not:
- Decide final UI route directly.
- Create or persist task records by itself.
- Invent unsupported capabilities.
- Ask excessive questions.
- Expose internal model reasoning to users.

## 6. Backend responsibilities

Backend should:
- Call AI intake analysis endpoint/model.
- Validate structured response shape and values.
- Apply deterministic product rules before persistence.
- Enforce limits for number of candidates and questions.
- Store intake session and intake items.
- Store pending questions and submitted answers.
- Create task records only when product rules allow.
- Support idempotency on intake submission and answer submission.
- Re-run or update result generation after detail answers are provided.
- Preserve intake context if the user leaves and returns later.
- For `split_review`, persist candidates first and delay final task creation until user confirmation.

Deterministic enforcement examples:
- Clamp question counts beyond limits.
- Downgrade unsupported answer types to safe defaults.
- Fallback to `needs_details` if AI output is low-confidence/invalid.
- Reject or hold split candidates when confirmation payload is missing or inconsistent.

## 7. iOS app responsibilities

iOS should:
- Show analyzing state after intake submission.
- Route to result, clarification, or split review based on backend response.
- Render missing questions with appropriate controls.
- Render `needed` questions as primary clarification inputs.
- Render `helpful` questions as compact secondary controls.
- Keep `nice_to_have` hidden in v1 unless user explicitly requests more refinement.
- Allow answering now or later.
- Persist and display `Needs details` status in My Tasks.
- Show `Answer missing details` from Task Detail and Task Result when applicable.
- Refresh task/result UI after answers are submitted.
- Never require user to restart the full intake flow to continue.

iOS should not:
- Re-implement business rules that belong to backend.
- Infer unsupported routes when backend returns fallback states.

## 8. Suggested data model

Suggested entities for v1+:

### `intake_sessions`

Purpose:
- Container for one top-level user intake submission.

Suggested fields:
- `id` (uuid)
- `user_id` (uuid)
- `source` (`ios_new_task`, `task_detail_resume`, etc.)
- `raw_input_text` (text)
- `intake_type` (`single`, `multiple`)
- `status` (`analyzing`, `awaiting_split_confirmation`, `awaiting_details`, `completed`, `cancelled`, `error`)
- `idempotency_key` (text, unique per user/action)
- `created_at`, `updated_at`

### `intake_items`

Purpose:
- Candidate task units extracted from an intake session.

Suggested fields:
- `id` (uuid)
- `intake_session_id` (uuid)
- `user_id` (uuid)
- `candidate_index` (int)
- `title` (text)
- `summary` (text)
- `readiness_state` (`ready`, `needs_details`, `can_start_with_limited_details`, `split_review`)
- `confidence_score` (numeric)
- `status` (`pending_confirmation`, `confirmed`, `deferred`, `created_task`, `discarded`)
- `linked_task_id` (uuid, nullable)
- `created_at`, `updated_at`

### `intake_questions`

Purpose:
- Structured clarification questions attached to an intake item.

Suggested fields:
- `id` (uuid)
- `intake_item_id` (uuid)
- `user_id` (uuid)
- `importance` (`needed`, `helpful`, `nice_to_have`)
- `question_text` (text)
- `answer_type` (`text`, `single_select`, `multi_select`, `date`, `number`)
- `options_json` (jsonb, nullable)
- `is_answered` (boolean)
- `answer_value_json` (jsonb, nullable)
- `answered_at` (timestamp, nullable)
- `status` (`pending`, `answered`, `dismissed_system`)
- `created_at`, `updated_at`

### Task/result mapping additions

Suggested task fields:
- `readiness_state` (mirror for UI)
- `missing_details_count_needed`
- `missing_details_count_helpful`
- `has_pending_questions` (boolean)
- `intake_item_id` (nullable link)

Suggested result/output fields:
- `result_version`
- `result_quality_state` (`provisional`, `complete`)
- `generated_from_answers_at` (timestamp, nullable)
- `missing_details_snapshot_json` (jsonb, optional for explainability)

Migration note:
- This proposed model requires a separate migration plan before implementation.
- Migration planning should map these entities onto existing task/result/reminder/checklist structures.
- Rollout should include compatibility handling for existing tasks without intake metadata.

## 9. Suggested API endpoints

These are draft endpoint contracts for design and planning.

### `POST /analyze-task-intake`

Purpose:
- Analyze new input and return routing decision payload.

Input (draft):
- `input_text`
- `source`
- `idempotency_key`

Output (draft):
- `intake_session`
- `routing_state` (`ready`, `needs_details`, `can_start_with_limited_details`, `split_review`)
- `items[]` with per-item readiness
- `questions[]` per item as needed
- optional `created_tasks[]` for ready items

Behavior:
- Validates AI output and applies product rules.
- Persists session/items/questions before response.
- If routing is `split_review`, stores candidates only and does not create final task records yet.

Error handling notes:
- `invalid_request`
- `rate_limited`
- `analysis_unavailable` (AI/system issue)
- `invalid_ai_output_fallback_applied`

### `POST /submit-intake-answers`

Purpose:
- Submit answers for pending intake questions.

Input (draft):
- `intake_item_id`
- `answers[]` keyed by question id
- `idempotency_key`

Output (draft):
- Updated `readiness_state`
- Updated `pending_questions`
- `result_update_status` (`generated`, `updated`, `still_needs_details`)
- optional updated `task_result`

Behavior:
- Validates question ownership/state.
- Stores answers and triggers readiness re-evaluation.

Error handling notes:
- `not_found`
- `forbidden`
- `conflict_question_already_answered`
- `validation_failed`

### `POST /confirm-task-split`

Purpose:
- Confirm which candidates from split review become real tasks.

Input (draft):
- `intake_session_id`
- `confirmed_item_ids[]`
- optional `discarded_item_ids[]`
- `idempotency_key`

Output (draft):
- `created_tasks[]`
- `pending_detail_items[]`
- `discarded_items[]`

Behavior:
- Creates ready tasks only for user-confirmed candidates.
- Preserves needs-details items and questions for later completion.

Error handling notes:
- `invalid_split_state`
- `item_mismatch`
- `already_confirmed`

### `POST /update-task-with-details`

Purpose:
- Re-run or update task result after new details are added.

Input (draft):
- `task_id`
- `answer_payload` or reference to answered intake questions
- `idempotency_key`

Output (draft):
- `task_id`
- `result_version`
- `result_quality_state`
- updated `task_result`

Behavior:
- Generates updated output while preserving prior history/version metadata.

Error handling notes:
- `task_not_eligible`
- `no_new_details`
- `generation_failed_retryable`

## 10. Suggested AI structured output schema

AI output must be structured JSON only.

### Example A — single ready

```json
{
  "intake_type": "single",
  "routing_state": "ready",
  "items": [
    {
      "candidate_index": 0,
      "title": "Request duplicate charge refund",
      "summary": "User was charged twice by a gym and wants a refund request plan.",
      "readiness_state": "ready",
      "confidence": 0.93,
      "missing_questions": []
    }
  ]
}
```

### Example B — single needs details

```json
{
  "intake_type": "single",
  "routing_state": "needs_details",
  "items": [
    {
      "candidate_index": 0,
      "title": "Cancel internet service",
      "summary": "User wants to cancel internet service but key details are missing.",
      "readiness_state": "needs_details",
      "confidence": 0.89,
      "missing_questions": [
        {
          "question_id": "q1",
          "importance": "needed",
          "question_text": "Which internet provider is this for?",
          "answer_type": "text",
          "suggested_options": []
        },
        {
          "question_id": "q2",
          "importance": "needed",
          "question_text": "Do you need cancellation before a specific date?",
          "answer_type": "date",
          "suggested_options": []
        },
        {
          "question_id": "q3",
          "importance": "helpful",
          "question_text": "Are you under a contract term right now?",
          "answer_type": "single_select",
          "suggested_options": ["Yes", "No", "Not sure"]
        }
      ]
    }
  ]
}
```

### Example C — multiple tasks mixed readiness

```json
{
  "intake_type": "multiple",
  "routing_state": "split_review",
  "items": [
    {
      "candidate_index": 0,
      "title": "Cancel gym membership",
      "summary": "User wants to cancel gym membership.",
      "readiness_state": "ready",
      "confidence": 0.9,
      "missing_questions": []
    },
    {
      "candidate_index": 1,
      "title": "Ask landlord about deposit",
      "summary": "User needs to follow up on deposit return.",
      "readiness_state": "needs_details",
      "confidence": 0.86,
      "missing_questions": [
        {
          "question_id": "q1",
          "importance": "needed",
          "question_text": "When did your lease end?",
          "answer_type": "date",
          "suggested_options": []
        }
      ]
    },
    {
      "candidate_index": 2,
      "title": "Set electricity bill reminder",
      "summary": "User needs a reminder for electricity payment.",
      "readiness_state": "can_start_with_limited_details",
      "confidence": 0.84,
      "missing_questions": [
        {
          "question_id": "q1",
          "importance": "helpful",
          "question_text": "What amount do you expect to pay?",
          "answer_type": "number",
          "suggested_options": []
        }
      ]
    }
  ]
}
```

## 11. UI and UX flows

### Flow 1 — New Task -> Analyze -> Result

- User enters input.
- App shows analyzing state.
- Backend returns `ready`.
- App opens Task Result.

### Flow 2 — New Task -> Analyze -> Clarification

- Backend returns `needs_details`.
- Clarification screen shows required questions first.
- User can answer now or save for later.

### Flow 3 — New Task -> Analyze -> Split Review

- Backend returns `split_review` with candidates.
- User confirms candidates.
- Ready items create tasks.
- Needs-details items persist pending questions.

### Flow 4 — Task Detail -> Answer missing details

- Task card/status shows `Needs details`.
- Task Detail exposes `Answer missing details`.
- User submits answers.
- Result updates or generates.

### Flow 5 — Task Result -> Answer missing details

- For `can_start_with_limited_details` or unresolved clarifications.
- Show `Answer missing details` inline near actions.
- Update result quality when answers are submitted.

### Flow 6 — My Tasks card with Needs details

- Card shows missing details indicator and count.
- Primary action: `Answer missing details`.
- No restart required.

### Flow 7 — Fallback when analysis fails or output is invalid

- If analysis is unavailable or AI output is invalid, preserve the raw input text.
- Show a calm fallback state with `Retry` and `Save for later` actions.
- Keep the request resumable from My Tasks or intake history.
- Avoid technical error copy in user-facing UI.

Suggested user-facing copy:
- "One detail will help"
- "A few details are needed"
- "Answer missing details"
- "Save for later"
- "I can update this when you add details"

Copy to avoid:
- "skip"
- "optional"
- "incomplete"
- "insufficient data"

## 12. Product rules and guardrails

Recommended v1 limits:
- Max needed questions per item: `1-2`
- Max helpful questions per item: `2`
- Max nice-to-have questions per item: `0` in v1
- Max visible questions in first clarification: `3`
- Max task candidates per intake (phase-2 launch): `5`

Guardrails:
- Do not ask for sensitive details unless directly required for task execution guidance.
- Do not over-question the user.
- Keep every question concrete, short, and directly answerable.
- Avoid generic prompts that do not change action quality.
- If confidence is low or output invalid, fall back safely to constrained clarification.
- Do not create tasks from ambiguous multi-task input before split confirmation.
- Ensure idempotency on create/update endpoints.
- Treat `can_start_with_limited_details` as an exception path, not a default path.
- If needed details are blocking, use clarification first.

Confidence handling (draft):
- High confidence + ready -> create and generate.
- Medium confidence + ready -> create and may include one needed verification question.
- Low confidence -> `needs_details` or fallback clarification.

Invalid AI output fallback:
- Reject malformed output.
- Log sanitized diagnostics.
- Return deterministic fallback response with minimal clarification set.
- Preserve raw input and provide retry/save-later UI paths without technical error language.

## 13. Analytics

Suggested events and dimensions:
- `intake_submitted` (single/multiple)
- `intake_tasks_detected_count`
- `intake_questions_generated_count` by importance
- `intake_questions_answered_count`
- `intake_questions_left_pending_count`
- `intake_resumed_later`
- `intake_result_updated_after_details`
- `split_review_shown`
- `split_review_accepted`
- `split_review_item_removed`
- `split_review_item_edited` (future)
- `task_completed_after_intake`

Suggested properties:
- readiness state transitions
- time-to-answer-needed-details
- answer completion rate
- fallback path frequency

Privacy notes:
- Do not store raw sensitive text in analytics events.
- Keep event payloads minimal and aggregated where possible.

## 14. Phased roadmap

### Phase 1

Deliver:
- Single ready flow.
- Single needs-details flow.
- Pending questions persisted.
- Resume and answer later without restart.

Not in Phase 1:
- Multi-task split flow.
- Edit/merge/split candidate tooling.
- Result version UI.
- Learned task archetypes.
- Nice-to-have questions.

### Phase 2

Deliver:
- Multi-task detection.
- Split review UI and confirmation.
- Create ready tasks.
- Preserve details/questions for incomplete items.

### Phase 3

Deliver:
- Result update after added details.
- Explicit result quality states (`provisional`, `complete`).
- My Tasks cards show missing-details count.

### Phase 4

Deliver:
- Learned question patterns.
- Task archetype tuning.
- Candidate edit/merge/split improvements.

## 15. Open questions

Product and technical questions to resolve:
- Should `can_start_with_limited_details` generate a result immediately or save as `needs_details` first?
- How many questions should be shown in v1 before forcing "save and continue later"?
- Should multi-task split create real tasks before user confirmation, or only after confirmation?
- How should result versions be stored and surfaced in UI?
- What is the exact screen pattern for answering questions later from Task Detail vs Task Result?
- Should helpful questions remain visible after a complete result is generated?
- What retry/backoff policy should apply when answer submission triggers generation failures?
