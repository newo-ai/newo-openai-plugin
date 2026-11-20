# Sandbox, live channels and diagnostics

Perform Newo actions through Customer MCP and follow [operations.md](operations.md). Use a new run to verify agent behavior. Historical reads and local text checks provide separate evidence.

## Sandbox text test

1. Establish the customer, project, agent and active version with `list_projects`. A sandbox request goes to the active agent.
2. Read `get_integrations(integration_idn="sandbox")` and `get_connectors(customer_idn, integration_idn="sandbox")` to select a connector. If creation of a test environment is requested and no suitable connector exists, use `create_connector` and the start procedure in [integrations.md](integrations.md).
3. Use `send_message(customer_idn, connector_idn, text)` without `actor_external_id` for a fresh Builder sandbox conversation. No phone, SMS or email is contacted. Pass the returned `actor_external_id` to continue or omit it for an independent test.
4. Keep `actor_id`, `actor_external_id`, `external_event_id` and `sent_at`. The send result arrives before the agent reply and omits the session ID.
5. Query `search_conversations` with `customer_idn`, `connector="sandbox/<connector_idn>"` and `from_datetime=sent_at`. Filter by `user_actor_ids` as needed and match acts to the send and conversation. Poll for a bounded period instead of resending. Follow-ups can have their own event ID.
6. Take the session ID from an act and call `get_session`. Use `search_logs(external_event_id=...)` for send and flow logs. If no reply arrives, report timeout or an unknown outcome.

Specify the acceptance criterion, conditions, messages, expected fact, forbidden actions and actual result. Compare facts and meaning. After changes, repeat the original and adjacent cases in new conversations. Use independent runs for critical cases where useful.

## Events and real actions

`send_agent_event` takes `customer_idn`, `actor_id` and `event_idn` and raises a caller event on the actor's connector and in their session. On an explicit request for a global event, use `send_global_agent_event` with `customer_idn`, `integration_idn`, `connector_idn` and `event_idn`. A global event has no caller or session. Both can trigger real calls, SMS or email. Check events and arguments in `get_integrations` metadata and actual subscriptions.

Both tools return `external_event_id` and `sent_at` before flow execution. Find logs by `external_event_id` and acts from `sent_at`. For actor events, match `user_actor_id`, `user_persona_id` and connector. Use a supplied `external_event_id` for correlation. It does not guarantee deduplication.

## Voice verification

Use a supported live voice channel to verify audio, Caller ID, transfer or SIP. Use a user call or a confirmed flow authorized to trigger a call. For WebRTC, read settings with `get_connectors` using the integration and voice connector IDs. Use the returned `api_key` with a verified client contract before giving a calling URL. Read Portal and chat links from detailed `get_customers`. Attribute observations supplied by the user. If Assessor has no confirmed contract, prepare a draft [operations request](support.md).

## Standard deployment test mode

Find `project_attributes_setting_test_mode` and `project_attributes_setting_test_mode_phone_numbers` through `search_customer_attributes`. Read full values and check their formats. Global test mode can simulate standard booking, calendar, email, SMS and transfer branches. Inspect the test environment separately for custom actions that may still have real effects.

Read the project-level `project_attributes_setting_test_mode_phone_numbers_static` list in the active `project.yaml` or its referenced file. Use the [project attribute procedure](project-attributes.md) for edits. Read persona-level `session_test_mode` with `get_persona_attribute` and the actual persona ID when authorized. A global change may leave an existing conversation in its prior mode. Report actual state.

Before live use, verify customer and persona overrides and the project list. Report an inaccessible or null active version as a verification gap. An empty local version list may coexist with a registry attachment. Simulation cannot verify real booking, delivery or transfer.

## Inspect sessions and paginate

- `list_sessions` accepts `from_datetime`, `to_datetime`, `channel`, `is_test`, `is_lead` and `query`. Pages start at 1, `per` goes up to 1000 and sessions are newest first. On truncation, restart at page 1 with a smaller page size. Verify completeness using [session statistics](quality-review.md#session-statistics).
- `get_session` returns session details and the latest 200 user and agent messages. Recording counts do not establish access to audio files.
- `search_conversations` returns messages and other acts with arguments, flows, skills, actors and personas. The default range is 182 days and `query` is case-sensitive. Acts are oldest first on each page and `next_cursor` goes back in time. Merge pages in time order and deduplicate by ID.
- `search_logs` returns flow and skill calls, parameters and results. Default order is newest first, or oldest first with `session_id`. Prefer the relevant `external_event_id` or `session_id` to a broad customer log export.

Under `phi_restricted`, observe restrictions on conversations, logs and persona data across tools, attributes and exports. Check allowed metadata, versions and aggregates separately.

## Report results

Distinguish written, read back, published, activated, sandbox passed and live voice passed. Report the customer, version, scenario, returned IDs, expected and actual results. Missing access, responses or contracts leave the result untested or unknown. Keep secrets and full personal conversations out of output.

## Subscriptions and delivery

Use `get_integrations` for event definitions and `read_project_version_files` for project subscriptions. Read `project.yaml` and referenced flows and skills using the accessible active ID, including registry versions. Null or inaccessible IDs leave subscriptions unverified. Confirm event IDs for test calls, Assessor and OAuth from actual contracts.

For SMS or email, read the relevant session's acts and logs through `search_conversations` and `search_logs`. Match flow, skill, connector, external event and delivery identifier as needed. NAF and the provider define acceptance, rejection and delivery. Email metadata includes `result_success` and `result_error`. `newo_sms` has no declared delivery event. Where only acceptance is known, report delivery as unconfirmed. Avoid duplicate sends for verification.

Live acceptance cases remain `not_run` until executed on the agreed test customer. Catalog access and metadata reads do not authorize purchases, deletions, sends or production project switches.
