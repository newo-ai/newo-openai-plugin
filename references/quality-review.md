# Review conversations, errors and metrics

Follow [operations.md](operations.md). Use [testing.md](testing.md) for test execution and [health.md](health.md) for configuration.

## Conversations over a period

Identify the customer, range, time zone and channels. Collect `list_sessions` pages using [session statistics](#session-statistics). Identify test and internal sessions from metadata and confirmed context.

Use server session IDs. One person can have multiple calls and channels. Use `get_session` for the latest 200 messages and `search_conversations` for relevant events and older acts. Acts are chronological within a page and `next_cursor` goes to earlier pages. Defaults to 182 days. Read available material when truncated and mark outcomes unknown when evidence is missing.

For each session, identify intent, outcome (answer, booking, message, transfer or unresolved), fact errors, repeated questions, language, answer length, hours, transfers and knowledge gaps. Verify claimed bookings against criteria and system evidence.

Report reviewed and available counts, period, exclusions, channel and outcome breakdown, a few successful examples, ranked issues, confirmed and suspected causes, and concrete fixes. Limit totals to the observed sample. Omit personal details and full transcripts from team reports.

## Investigate a problem

1. Find time, channel, session ID, expected behavior and actual behavior.
2. Read the conversation and a narrow log window. If errors lack participant context, check the same minutes for the same customer.
3. Test the hypothesis against relevant settings, knowledge source, connector state and active version.
4. Fix the smallest confirmed cause in scope. Re-run the original case and agreed regression suite after applying the change.
5. Add a case to the criteria when agreed. For platform or code issues prepare a [support draft](support.md).

| Symptom | First checks |
| --- | --- |
| Wrong hours or closed status | Each day, exceptions, IANA time zone, setting application and a new session. |
| Incorrect facts or prices | Source attribute, quotation rules, knowledge and publication state, and the actual question and answer. |
| Language change or tone | Language fields, restrictions, style settings and the turn where language changed. |
| Missing report | Session end, full processing, enabled flag, matching content and criteria, recipient, test simulation and delivery outcome. |
| Duplicate booking | Repeated sends or confirmation, lookup of an existing booking and supported idempotency. |
| Missing booking | Request, link or confirmed appointment, inputs, availability, test mode and system response. |
| Missing SMS or email | Permission, recipient, simulation, send event and delivery confirmation. |
| Missing transfer or loop | Criteria, staffed number, destination format, test mode, SIP or REFER and live line. |
| Missing audio, cutoff or pause | Channel, model or STT, state, routing and repeatability on real calls. |
| Setting appears unapplied | Scope, customer, channel, value read-back, source versus derived field, active version and a new session. |

## Session statistics

Use `list_sessions` for counts and `get_session` for details. Set customer, time range, time zone, channels and test-session inclusion. Supported filters include `from_datetime`, `to_datetime`, `channel`, `is_test`, `is_lead` and `query`. Pages start at 1 and `per` goes up to 1000. If a filter is unavailable, use returned fields and qualify results if the required data is missing.

- Record filters and page size. Check `truncated` and records per page. Restart from page 1 with a smaller `per` when results are incomplete. If needed, split the period and check interval boundaries.
- Deduplicate by `session_id` and compare unique sessions with `total` for the same filters. If completeness remains unknown, state the sample size and denominator.
- Count sessions and available breakdowns by channel, time and test mode. Keep unknown mode separate. Verify people counts separately.
- Use `get_session` for outcomes and errors. Distinguish system evidence, agent statements and unknown results. Calculate outcome proportions only for the detailed sample and report the denominator.
- Calculate duration only from valid start and end times or a server duration. Keep unfinished sessions separate. Count leads from an explicit field or agreed observable rule and state the definition.

A spoken booking claim needs booking-system evidence. Official finance aggregates and actual revenue are outside the session report.

## Unit usage and balance

`get_customer_usage(customer_idn)` returns `subscription`, `unit_balance`, `units_spent`, `hard_quota`, `soft_quota`, `secondary_products`, `grace`, `debt`, `period`, `days`, `totals` and `truncated`. For a custom range, pass `start_at` and `end_at` as ISO 8601 values, up to 366 days. Dates without offsets use UTC. `period.source` is `current_billing_cycle` without dates and `provided_range` when supplied.

1. Keep billing-cycle `units_spent` separate from `totals` for the selected range. `days` contains daily voice and text units. `totals` includes voice, text and combined units. Subscription dates do not define the query range. Mark truncated details as incomplete.
2. Report the actual plan name and key, development, trial or paid phase, dates and returned parameters. A null balance outside usage billing is unknown. Do not sum balance snapshots.
3. Report quotas, add-ons, grace and debt from the response. `debt.total` uses minor currency units and `debt.units` uses usage units. Converting durations to units or money requires a verified tariff.
4. Compare quality and usage only over aligned periods. Attribute aggregated usage to individual sessions only with direct evidence. Reading billing does not authorize a payment or plan change.

## Operations

Agree on a review cadence after the first nights, typically weekly and after changes in rollout mode or version. Rerun health and acceptance tests after material changes. Future reports require a configured background mechanism.
