---
title: SMS Crisis Referral System - Plan
type: feat
date: 2026-09-10
topic: sms-crisis-referral
artifact_contract: ce-unified-plan/v1
artifact_readiness: implementation-ready
product_contract_source: ce-brainstorm
execution: code
---

# SMS Crisis Referral System - Plan

## Goal Capsule

- **Objective:** Clients of Moreneta Initiative seeking crisis or safety-sensitive help (domestic violence, homelessness, addiction, child safety) can, over SMS, reach the service centre best suited to their need without already knowing where to look.
- **Means:** An SMS-based conversational AI agent handles intake and consent; a service-fit/distance/cost heuristic ranks candidate service centres to surface the best matches.
- **Product authority:** Established via `ce-brainstorm` dialogue on 2026-09-10.

---

## Product Contract

### Summary

An SMS-based intake and referral system connects Moreneta Initiative clients seeking crisis and safety-sensitive help with the most appropriate service centre. A conversational AI agent gathers consent and required details over SMS, and the backend ranks up to five service centres by service fit, then distance, then cost, offering to notify the client's chosen centre on their behalf.

### Problem Frame

Moreneta Initiative currently has no structured way to connect clients to the right crisis or safety-sensitive service — clients rely on word of mouth or self-directed search. For people facing domestic violence, homelessness, addiction, or child-safety crises, a delay or a mismatched referral has real costs: a service that doesn't fit the need, is too far to reach, or is unaffordable wastes time that safety-critical situations may not allow. Moreneta wants a single, low-friction channel — SMS — through which a client can describe what they need and be pointed to the most appropriate, currently-operating service centre nearby.

### Requirements

**Intake & Consent**
- R1. The agent requests explicit consent to collect and store the client's information before asking for any personal details.
- R2. The agent conversationally collects the client's stated need, name, age or date of birth, current address (at minimum suburb), and an emergency/safe contact.
- R3. When a required field is missing, the agent tells the client specifically what's still needed rather than proceeding with an incomplete record.

**Safety Handling**
- R4. When the agent detects language indicating immediate danger, it includes crisis-line contact information in its reply while continuing the normal intake flow.
- R5. The client can send a designated safe-word at any point to immediately end the conversation and clear/obscure recent message content.

**Data Handling**
- R6. The client's exact address is used only to calculate distance to candidate service centres and is not persisted; only suburb-level location is stored on the client record.
- R7. Service centre records carry an operating-status field (e.g. active, at-capacity, closed), maintained directly in the database by Moreneta staff.
- R8. Once the conversation resolves — the client has received their ranked list and any centre notification has been sent or explicitly declined — the client's stored record is deleted.

**Matching & Ranking**
- R9. Each service centre is tagged with one or more categories from a defined need-taxonomy (e.g. domestic violence, housing, addiction, child safety, mental health).
- R10. The agent classifies the client's stated need into the same taxonomy and ranks candidate centres first by category match, using the client's free-text description to break ties among same-category centres.
- R11. Centres not in active operating status are excluded or deprioritized from ranking. Governs R7.
- R12. Among equally-matched centres, ranking then orders by distance from the client's address, then by cost to the client, with free services ranking above paid where cost is known.
- R13. The agent returns up to 5 ranked service centres to the client via SMS.

**Handoff**
- R14. After presenting the ranked list, the agent asks the client whether they want their details sent to the centre they choose.
- R15. Only on the client's explicit yes does the system send the client's stored details to the chosen centre.

### Key Decisions

- **AI-only safety handling, no human escalation** — the agent auto-surfaces crisis-line info on detected danger signals rather than handing off to a human responder, keeping the system self-contained for this build while still surfacing emergency resources. Governs R4.
- **Address collected transiently, never stored** — trades minor per-conversation friction for removing the largest privacy liability from the client record; an abuser or data breach exposing a full home address is the most severe realistic harm in this population. Governs R6.
- **Consent captured upfront** — the agent asks for consent before collecting any personal detail, favoring trust and compliance over a frictionless start. Governs R1.
- **Client-triggered safe-word over discreet-by-default wording** — puts a fast exit in the client's control rather than hedging every message's wording. Governs R5.
- **Service directory curated directly in the database** — Moreneta staff maintain centre records with no admin UI in this build. Governs R7.
- **Records deleted once the conversation resolves** — no retention window and no indefinite keep; minimizes data at rest for a population where a leaked record carries acute risk, at the cost of no history if the same client contacts again later. Governs R8.
- **Hybrid category + free-text matching** — centres are tagged into a fixed need-taxonomy for filtering, and the client's free-text situation breaks ties within a category. Governs R9, R10.
- **Notify-the-centre handoff is opt-in per conversation** — the system asks the client each time rather than always notifying or never notifying, and only acts on an explicit yes. Governs R14, R15.

### Actors

- A1. Client — a person in crisis or needing safety-sensitive help, contacting the system via SMS.
- A2. AI Agent — the conversational system that gathers consent and details, applies safety handling, and returns ranked matches.
- A3. Moreneta Staff — maintain the service centre directory directly in the database.
- A4. Service Centre — the referral destination; may be notified with client details at the client's request.

### Key Flows

- F1. Standard intake and match
  - **Trigger:** Client sends an initial SMS to the Moreneta Initiative number.
  - **Actors:** A1, A2
  - **Steps:** Agent requests consent; on consent, it asks for stated need, name, age/DOB, address/suburb, and emergency contact, prompting for anything missing; it computes distance from the provided address; it classifies the need against the centre taxonomy; it ranks active centres by category match (using free text to break ties), then distance, then cost.
  - **Outcome:** Client receives a ranked list of up to 5 service centres suited to their need. If they don't go on to a notification handoff (F4), the record is then deleted per R8.
  - **Covers:** R1, R2, R3, R6, R9, R10, R11, R12, R13

- F2. Danger-signal handling
  - **Trigger:** Agent detects language indicating immediate danger, at any point in the conversation.
  - **Actors:** A1, A2
  - **Steps:** Agent includes crisis-line contact info in its next reply; normal intake flow continues.
  - **Outcome:** Client has emergency resources without the conversation stopping.
  - **Covers:** R4

- F3. Safe-word exit
  - **Trigger:** Client sends the designated safe-word, at any point in the conversation.
  - **Actors:** A1, A2
  - **Steps:** Agent ends the conversation immediately and clears/obscures recent message content.
  - **Outcome:** Client can discreetly exit if their phone is being monitored.
  - **Covers:** R5

- F4. Centre notification handoff
  - **Trigger:** Client has received the ranked list and indicates which centre they want.
  - **Actors:** A1, A2, A4
  - **Steps:** Agent asks whether to notify that centre; on yes, the system sends the client's stored details to the chosen centre.
  - **Outcome:** The chosen centre is expecting the client, only with the client's explicit go-ahead. The client's record is then deleted per R8.
  - **Covers:** R8, R14, R15

### Acceptance Examples

- AE1. Missing field blocks matching
  - **Covers:** R3
  - **Given:** A client has stated their need and name but not an address.
  - **When:** The agent processes the message.
  - **Then:** It replies asking specifically for the address (and any other missing required fields) rather than generating a match.
- AE2. Danger signal surfaces crisis-line info
  - **Covers:** R4
  - **Given:** A client's message contains language indicating immediate danger.
  - **When:** The agent replies.
  - **Then:** The reply includes crisis-line contact information alongside its normal response.
- AE3. Safe-word ends the conversation
  - **Covers:** R5
  - **Given:** A client sends the safe-word at any point.
  - **When:** The agent receives it.
  - **Then:** The conversation ends immediately and recent message content is cleared/obscured, regardless of how much intake was completed.
- AE4. Inactive centres rank below active ones
  - **Covers:** R11, R12
  - **Given:** Two service centres match the client's need category equally well, one active and one at-capacity.
  - **When:** The agent ranks candidates.
  - **Then:** The active centre ranks above the at-capacity one regardless of distance or cost.
- AE5. Declining notification withholds client details
  - **Covers:** R14, R15
  - **Given:** The client has received the ranked list.
  - **When:** They decline to have a centre notified.
  - **Then:** No client details are sent to any centre.
- AE6. Record deleted once the conversation resolves
  - **Covers:** R8
  - **Given:** A client has received their ranked list and either declined notification or a notification was sent.
  - **When:** The conversation is complete.
  - **Then:** The client's stored record is deleted.

### Scope Boundaries

- No human-in-the-loop escalation for detected danger signals in this build — the automated crisis-line surfacing in R4 is the full response.
- No admin interface for staff to manage the service directory — centres are entered and edited directly in the database. Governs R7.
- No appointment booking or scheduling — the handoff ends at optionally notifying the chosen centre. Governs R14, R15.
- Full street address is never persisted to the client record. Governs R6.
- No long-term retention or client history across separate conversations — each record is deleted once its conversation resolves. Governs R8.

### Dependencies / Assumptions

- Assumes an SMS gateway/provider capable of receiving and sending messages is available for the agent to use; exact vendor is a planning decision.
- Assumes a geocoding or distance-calculation capability exists to convert the client's address and centre locations into a distance for ranking; exact approach is a planning decision.
- Assumes Moreneta staff have direct database access or tooling to maintain service centre records.

### Outstanding Questions

All items below were deferred to planning and are now resolved in the Planning Contract; each line keeps its original wording with a resolution pointer.

- **Resolved: KTD1.** Backend framework and hosting (FastAPI plus the Supabase Python client was discussed favorably during brainstorming, but the final call belongs to planning).
- **Resolved: KTD5.** The specific need-taxonomy category list behind R9/R10.
- **Resolved: KTD4.** The danger-signal detection mechanism behind R4 (keyword-based, model-based, or otherwise).
- **Resolved: KTD6.** The exact safe-word value and the technical mechanism for "clearing/obscuring" recent SMS content behind R5 — SMS-channel constraints may limit what's actually achievable.
- **Resolved: KTD7.** The exact deletion mechanism and timing tolerance behind R8 (immediate hard delete vs. a short grace period).

**Product Contract preservation:** unchanged — no R/A/F/AE was altered; every item above is resolved without touching product scope.

---

## Planning Contract

### Key Technical Decisions

- KTD1. **FastAPI app on Fly.io, Supabase (Postgres) for all persistent storage via `supabase-py`.** Matches the stack discussed favorably during brainstorming: Fly.io gives a cheap container host with a public HTTPS endpoint for the SMS gateway's webhook, and Supabase's Python client removes the need for a separate ORM. Governs backend/hosting from Outstanding Questions.
- KTD2. **Twilio Programmable Messaging for the SMS gateway** — inbound webhook to a FastAPI route, outbound replies via Twilio's REST client. Mature two-way SMS provider with a first-party Python SDK; no meaningful advantage from alternatives (Vonage, MessageBird) at this scope.
- KTD3. **Mapbox Geocoding API converts the client's address to coordinates transiently; each service centre stores a fixed lat/long; distance is a server-side haversine calculation.** Keeps the geocoded coordinate out of storage entirely (R6) — only the derived suburb persists on the client record. Nominatim (OSM) is a viable free drop-in if Mapbox cost or its usage terms become a concern at higher volume.
- KTD4. **One LLM-backed conversational service drives intake, and the same per-turn call also classifies danger-signal language**, returned as a structured field alongside the reply. Reuses the call that already reads every inbound message instead of adding a second detection system; keyword lists undercatch the varied phrasing a crisis-intake channel sees. Governs R4.
- KTD5. **Fixed 5-category need-taxonomy** — `domestic_violence`, `housing`, `addiction`, `child_safety`, `mental_health` — stored as an enum column on `service_centres` (one or more per centre) and reused as the classification output space for the client's stated need. Lifted directly from the brainstorm's own example list. Governs R9, R10.
- KTD6. **Safe-word is a single configurable value** (application config/environment, not hardcoded) compared case-insensitively and trimmed against each inbound message before any other processing. On match: the conversation ends, no further reply is sent (a monitored phone should see nothing further, matching F3's discreet-exit intent), and the client's record plus that conversation's transcript rows are hard-deleted immediately via the KTD7 mechanism rather than merely flagged. SMS gives no control over the client's own handset, so the guarantee is scoped to our stored copy. Governs R5.
- KTD7. **Deletion is a synchronous hard delete with no grace period**, executed in the same request/transaction as the terminal action (notification sent, notification declined, or safe-word triggered) — no scheduled purge job, no retention-flag column. Governs R8.

### Assumptions

- No service centre currently matches the client's active/eligible category (an edge case the original requirements don't address): the agent sends a clear "no matching centre found" message rather than an empty ranked list.
- A service centre's notification channel (R14/R15 handoff) is SMS to the centre's stored contact number, keeping a single delivery channel rather than adding email infrastructure. Centres wanting email can be added as a later decision.
- Tie-breaking same-category centres on free-text description (R10) reuses the same LLM call rather than standing up a separate embedding/similarity pipeline.

### Sequencing

U1 (data model) is the foundation every other unit reads or writes against. U2 (SMS webhook) can proceed in parallel with U1. U3 (intake agent) and U4 (danger-signal detection) share one LLM call and land together once U1+U2 exist. U5 (safe-word) and U6 (geocoding) depend only on U1+U2. U7 (matching) depends on U1, U3, and U6. U8 (results delivery) depends on U7. U9 (notification handoff) depends on U8. U10 (deletion) is a shared helper invoked by U5 and U9, so it lands before either calls it.

---

## Implementation Units

| U-ID | Title | Files touched | Depends on |
|---|---|---|---|
| U1 | Data model & Supabase schema | `supabase/migrations/0001_init.sql`, `app/db/models.py` | — |
| U2 | SMS gateway webhook integration | `app/routers/sms_webhook.py`, `app/services/sms_client.py` | U1 |
| U3 | Conversational intake agent | `app/services/agent.py`, `app/services/prompts/intake.py` | U1, U2 |
| U4 | Danger-signal detection & crisis-line surfacing | `app/services/agent.py`, `app/data/crisis_lines.py` | U3 |
| U5 | Safe-word exit | `app/services/safeword.py`, `app/config.py` | U1, U2, U10 |
| U6 | Geocoding & distance calculation | `app/services/geocoding.py` | U1 |
| U7 | Need classification & matching/ranking engine | `app/services/matching.py` | U1, U3, U6 |
| U8 | Ranked results delivery via SMS | `app/services/results_formatter.py` | U7 |
| U9 | Centre notification handoff | `app/services/handoff.py` | U8, U1, U10 |
| U10 | Client record deletion lifecycle | `app/services/cleanup.py` | U1 |

### U1. Data model & Supabase schema

- **Goal:** Stand up the Postgres schema behind `service_centres`, `client_records`, and `conversation_state`.
- **Requirements:** R6, R7, R8, R9, R11
- **Files:** `supabase/migrations/0001_init.sql`, `app/db/models.py`
- **Approach:** `service_centres(id, name, categories <need-taxonomy enum>[], operating_status enum[active|at_capacity|closed], cost_type enum[free|paid|unknown], lat, long, contact_phone)`; `client_records(id, phone_number, name, age_or_dob, suburb, emergency_contact, stated_need_text, need_category, consent_at, created_at)`; `conversation_state(id, client_record_id, transcript jsonb[], ended_at, safeword_triggered bool)`. Use `supabase-py` for CRUD from the app layer.
- **Test Scenarios:** migration applies cleanly on a fresh Supabase project; `service_centres` insert with a multi-value `categories` array round-trips; `client_records` insert then delete leaves no row.
- **Verification:** `pytest tests/db/` against a disposable test schema (Supabase local dev stack or a scratch project).

### U2. SMS gateway webhook integration

- **Goal:** Receive inbound SMS from Twilio and send outbound replies.
- **Requirements:** transport for F1-F4
- **Files:** `app/routers/sms_webhook.py`, `app/services/sms_client.py`
- **Approach:** `POST /webhook/sms` validates the Twilio request signature, resolves or creates `conversation_state` keyed by the `From` number, hands the message to U3's orchestrator, and sends the reply back via Twilio's REST client.
- **Test Scenarios:** valid Twilio signature is accepted; invalid signature returns 403; an unseen `From` number creates a new conversation; a known number resumes its existing one.
- **Verification:** `pytest tests/routers/test_sms_webhook.py` with a mocked Twilio client and a fixture-signed/unsigned request pair.

### U3. Conversational intake agent (consent + field collection)

- **Goal:** Gather consent, then the required intake fields, prompting specifically for whatever is still missing.
- **Requirements:** R1, R2, R3
- **Files:** `app/services/agent.py`, `app/services/prompts/intake.py`
- **Approach:** LLM-backed turn handler that asks for consent before any personal-detail question (R1); once consented, collects stated need, name, age/DOB, suburb, and emergency contact; each turn recomputes which required fields are still missing and, if any remain, names them in the reply (R3) instead of proceeding to matching.
- **Test Scenarios:** AE1 (address missing blocks matching, reply names the address); no personal-detail question appears before consent is given; a fully-populated turn sequence proceeds to matching.
- **Verification:** `pytest tests/services/test_agent.py` driving `agent.py` with scripted message sequences, asserting the reply text against the missing-field set.

### U4. Danger-signal detection & crisis-line surfacing

- **Goal:** Surface crisis-line contact info whenever a message reads as an immediate-danger signal, without stopping intake.
- **Requirements:** R4
- **Files:** `app/services/agent.py` (per KTD4, same call as U3), `app/data/crisis_lines.py`
- **Approach:** extend the per-turn LLM call's structured output with a `danger_detected` boolean; when true, append crisis-line contact info to that turn's reply while the normal intake flow (F2) continues unchanged.
- **Test Scenarios:** AE2 (a danger-signal message's reply includes crisis-line info); a neutral message's reply omits it; a danger-signal message that also has a missing field includes both the crisis-line info and the missing-field prompt.
- **Verification:** `pytest tests/services/test_danger_detection.py` with representative danger-signal and neutral fixture messages.

### U5. Safe-word exit

- **Goal:** End the conversation immediately and remove the client's stored data the moment the safe-word is sent.
- **Requirements:** R5
- **Files:** `app/services/safeword.py`, `app/config.py`
- **Approach:** per KTD6, check every inbound message against the configured safe-word before any other processing; on match, mark the conversation ended, send no further reply, and call U10's `delete_client_record` to hard-delete the client record and that conversation's transcript rows immediately.
- **Test Scenarios:** AE3 (safe-word ends the conversation and clears the record regardless of how much intake had completed, mid-intake or post-matching); no reply is sent after a safe-word match.
- **Verification:** `pytest tests/services/test_safeword.py` asserting DB rows are absent immediately after a safe-word message, across different intake-progress starting states.

### U6. Geocoding & distance calculation

- **Goal:** Turn the client's address into a distance to each candidate service centre without persisting the address itself.
- **Requirements:** R6, R12
- **Files:** `app/services/geocoding.py`
- **Approach:** per KTD3, call the Mapbox Geocoding API with the client's collected address to get coordinates transiently; compute haversine distance from those coordinates to each active service centre's stored lat/long; discard the geocoded coordinate once ranking completes, persisting only the suburb on `client_records`.
- **Test Scenarios:** distance is computed correctly for known coordinate-pair fixtures; a geocoding failure (address not recognized) prompts the client to re-send the address rather than crashing; `client_records` never gains a full-address column or value.
- **Verification:** `pytest tests/services/test_geocoding.py` with a mocked geocoding client and fixed coordinate fixtures asserting the haversine math.

### U7. Need classification & matching/ranking engine

- **Goal:** Classify the client's stated need and rank eligible centres by category match, then distance, then cost.
- **Requirements:** R9, R10, R11, R12, R13
- **Files:** `app/services/matching.py`
- **Approach:** classify the client's free-text stated need into the KTD5 taxonomy; query centres excluding non-active operating status (R11); rank surviving candidates first by category match, breaking same-category ties with the free-text description via the same LLM call (no separate embedding pipeline, per Assumptions); then order by distance, then cost with free ranking above paid where cost is known (R12); truncate to the top 5 (R13).
- **Test Scenarios:** AE4 (an equally-matched at-capacity centre ranks below an active one regardless of distance/cost); same-category centres tie-break as expected on free text; a free centre ranks above an equal-distance paid centre; more than 5 matches truncates to exactly 5.
- **Verification:** `pytest tests/services/test_matching.py` with a fixture set of centres covering active/inactive, varying category, distance, and cost.

### U8. Ranked results delivery via SMS

- **Goal:** Deliver the ranked list to the client over SMS, handling the no-match case.
- **Requirements:** R13
- **Files:** `app/services/results_formatter.py`
- **Approach:** format up to 5 ranked centres (name, category, contact) into SMS-appropriate message(s), relying on Twilio's multi-segment handling for lists that exceed a single SMS segment; when no active centre matches, send the Assumptions-defined "no matching centre found" message instead of an empty list.
- **Test Scenarios:** a 5-result list renders and sends correctly across multiple segments; the 0-result case sends the fallback message, not an empty list.
- **Verification:** `pytest tests/services/test_results_formatter.py` asserting message formatting and the empty-result fallback path.

### U9. Centre notification handoff

- **Goal:** Ask which centre the client wants notified and only notify on explicit yes.
- **Requirements:** R14, R15
- **Files:** `app/services/handoff.py`
- **Approach:** after the ranked list is delivered, ask which centre (by list position) the client wants notified; on an explicit yes, send the client's stored details via SMS to that centre's `contact_phone` (per Assumptions); on decline, send nothing; an ambiguous reply re-prompts rather than guessing. On completion of either branch, call U10's `delete_client_record`.
- **Test Scenarios:** AE5 (decline sends no details to any centre); an explicit yes sends details only to the chosen centre; a reply that isn't a valid choice or yes/no re-prompts instead of guessing.
- **Verification:** `pytest tests/services/test_handoff.py` with a mocked Twilio client, asserting outbound payload contents and that decline never triggers a send.

### U10. Client record deletion lifecycle

- **Goal:** Provide the single deletion path every terminal branch calls.
- **Requirements:** R8
- **Files:** `app/services/cleanup.py`
- **Approach:** per KTD7, implement `delete_client_record(client_id)` as one DB transaction removing the `client_records` row and its `conversation_state` transcript rows; called from U9's notification-sent and decline paths and from U5's safe-word path.
- **Test Scenarios:** AE6 (record is deleted after a decline and after a notification send); record is deleted immediately on safe-word trigger, not only at natural resolution.
- **Verification:** `pytest tests/services/test_cleanup.py` asserting row absence post-completion across all three trigger paths.

---

## Verification Contract

| Command | Applies to | Notes |
|---|---|---|
| `pytest -q` | All units | Full suite; Twilio, Mapbox, Supabase, and the LLM client are mocked — no live external calls in CI |
| `pytest tests/db/` | U1 | Runs against a disposable Supabase schema, not the production project |
| `pytest tests/services/ tests/routers/` | U2-U10 | Unit- and route-level tests per the Test Scenarios above |

No repo tooling exists yet (greenfield); this unit's scaffolding (`pyproject.toml`, `pytest` as a dev dependency, a `.env.example` documenting `TWILIO_*`, `MAPBOX_TOKEN` or equivalent, `SUPABASE_URL`/`SUPABASE_KEY`, the LLM provider key, and `SAFEWORD`) is part of U1.

## Definition of Done

- Every requirement R1-R15 is covered by at least one automated test scenario listed above, and `pytest -q` passes.
- A manual end-to-end smoke test — a real SMS conversation through a Twilio sandbox number completing F1 (standard intake and match) followed by F4 (notification handoff) — succeeds before the build is considered demo-ready.
- No dead-end or experimental code from approaches that didn't pan out remains in the diff.
- `artifact_readiness: implementation-ready` holds: no launch-blocking open question remains (confirmed above — all five deferred items are resolved by KTD1, KTD4, KTD5, KTD6, KTD7).

