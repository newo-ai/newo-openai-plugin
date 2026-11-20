---
name: newo-quality
description: Test Newo agents in the sandbox, investigate sessions, events and logs, and verify AKB retrieval and unit usage. Use for regressions, conversation reviews and live-call outcome checks.
---

# Testing, conversations and metrics

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md), [testing.md](../../references/testing.md) and [quality-review.md](../../references/quality-review.md). Use [health.md](../../references/health.md) for settings and [knowledge.md](../../references/knowledge.md) for AKB.

- Sandbox test. Choose a connector with `get_connectors(integration_idn="sandbox")`. Omit `actor_external_id` from `send_message(customer_idn, connector_idn, text)` to start a new conversation. Pass the returned actor external ID to continue. Read the asynchronous reply with `search_conversations` from `sent_at` using `connector="sandbox/<connector_idn>"`. Match the actor and use the act's session ID with `get_session`. Correlate logs by `external_event_id`. Test the voice line separately.
- Review. Set the time range, time zone and `list_sessions` filters. If results are truncated, follow the [session statistics procedure](../../references/quality-review.md#session-statistics). `get_session` returns the most recent 200 messages. Read earlier acts with `search_conversations` and flow or skill logs with `search_logs`.
- Act pagination. Each page is oldest first and `next_cursor` goes to earlier acts. The default range is 182 days. Logs are generally newest first and oldest first when filtered by `session_id`.
- Units. `get_customer_usage(customer_idn)` returns subscription, phase or period, balance, usage, allowances, add-ons and daily voice and text totals. Defaults to the current billing cycle. For a custom range, pass both `start_at` and `end_at` as ISO 8601 values, up to 366 days. Treat missing values as unknown. Use usage data for consumption figures.
- SMS or email delivery. Read `search_conversations(session_id=...)` and `search_logs(session_id=...)`, then match actual provider events and results to the NAF behavior. Email metadata includes `result_success` and `result_error`. Neither is a universal delivery receipt. SMS has no predefined delivery event. Track sending, provider acceptance and recipient delivery separately.
- Errors and regressions. Correlate session, acts, logs, attributes, connector and `search_knowledge_base_topics` when relevant. Trace the active version and subscription for a code regression, then use [newo-develop](../newo-develop/SKILL.md) for an authorized fix. Re-run the original trigger and look for the expected outcome. If the outcome is absence of an effect, verify a positive marker from the same execution window.

Use `send_message` for sandbox dialogue. `send_agent_event` runs subscriptions in the supplied `actor_id` context. `send_global_agent_event` runs subscriptions without a caller or session on an explicit request for a global event. Both may trigger real actions. Respect `phi_restricted` restrictions on conversations, logs and persona data.

Use a test customer explicitly identified by the user and actually accessible. Read access alone does not authorize write scenarios. Draft support requests using the [support procedure](../../references/support.md). Report evidence, hypotheses, sample completeness, changes and verified results. Keep session statistics and unit usage tied to their own sources and periods.
