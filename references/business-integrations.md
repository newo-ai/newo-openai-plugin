# Calendars, CRM, booking and notifications

Configure and verify integrations through Customer MCP under [operations.md](operations.md). These attributes use customer scope except where the [catalog](attribute-catalog.json) says otherwise. See [integrations.md](integrations.md) for connectors and [agent-mcp-client.md](agent-mcp-client.md) for external MCP, HubSpot and OAuth.

## Choose the integration

Identify the system, agent action, supported connection, limitations, access owner and test environment. Use the schema supplied by the server or user. A visual-only service without a supported connection cannot be written to automatically by this plugin.

| Task | Configuration |
| --- | --- |
| Staff reports | Report content and criteria from [setup.md](setup.md). |
| CRM, spreadsheet or post-call handler | Session-ended webhook. |
| Google Calendar or Cal.com | Calendar connector and meeting attributes. |
| Availability, booking or cancellation | Booking attributes and an agreed scenario. |
| In-conversation reads or actions | `project_attributes_settings_platform_tools`. |
| Customer email or SMS | Separate permissions, allowed content and templates. |
| Agent use of external MCP | `mcp` connector. For OAuth2 use the [local project template](agent-mcp-client.md). |

## Send conversation results to a CRM

Configure general incoming and outgoing webhooks with `get_incoming_webhooks`, `get_outgoing_webhooks` and the corresponding write tools in [integrations.md](integrations.md). They operate on `api` connector events and commands. The standard session-ended webhook is a separate mechanism. Match the agent's actual flow and confirm the subscription and handler.

Use `project_attributes_setting_session_ended_webhook_url` for the agreed destination, `project_attributes_setting_session_ended_webhook_schema` for result fields and `project_attributes_setting_session_ended_webhook_auth_header` for secure authorization. Verify full session-end processing and the payload before enabling it. Limit fields to the agreed needs. Verify through Newo's session logs and results.

Run a fresh conversation and wait for it to end. Check the send and response in available logs. Confirm delivery or record creation through a result accessible to Customer MCP or a user observation in the destination system. Attribute the source of the evidence. A spoken confirmation alone is insufficient.

## Calendars and booking

`project_attributes_setting_calendar_event_integration` selects `disabled`, `calcom_integration` or `google_integration` where supported. Verify the working connector, calendar owner, time zone, duration and permitted schedule. Configure authorization on the connector.

Agree on availability lookup and booking before enabling writes. Main fields include `project_attributes_setting_booking_enabled`, `project_attributes_setting_booking_type`, `project_attributes_setting_booking_url`, `project_attributes_setting_booking_payload_schema_template` and `project_attributes_setting_booking_request_headers`. Availability uses the catalog's check_availability fields. Confirm the request schema and destination. The `project` mode requires an existing handler.

Identify the caller as required, confirm the service, location and specialist, and fetch real availability. Repeat the selected details, obtain confirmation, book once and verify the outcome. Cancellation and rescheduling require locating the booking and verifying permission to change it. Treat repeated confirmation and timeouts carefully. Check for an existing booking before retrying an uncertain result.

If booking is unavailable, collect a request for staff and say that the appointment is pending confirmation. Use a callback, message or transfer fallback according to the business rules.

## In-conversation actions

`project_attributes_settings_platform_tools` takes JSON with a top-level `tools` object keyed by tool name. A matching key overrides the built-in tool. Each tool requires `conditions` and `useConversationHistory`.

- `conditions` describes verifiable trigger criteria. A write requires explicit confirmation of the details.
- `useConversationHistory` controls history use. When `false`, evaluation uses the last agent message only.
- `action.type` supports `send_webhook` and `send_event`. An omitted action can default to an event. Set the type explicitly for a network call.
- For `send_webhook`, set `options.method` and `options.url` and optional `query`, `headers` and `body` schemas. Use actual destinations and fields supplied by the system or user.
- A top-level schema property with `runtime_value` equal to `session_id` or `customer_idn` can use execution context. Add it when the request needs it.
- `after_actions` run in order for a nonempty webhook response. `set_prompt_section` requires `section_name`, `set_persona_attribute` requires `attribute_name` and `send_urgent_message` forwards the response immediately. Keep unverified external content out of system instructions.
- `conflictPolicy.tag` makes tools with the same tag mutually exclusive within one selection. Deduplication across turns or retries requires separate handling.

Test one read tool before enabling a write tool. Preserve other keys in `tools`. Validate JSON and current schema before `set_customer_attribute_value`. Confirm the exact handler contract. The NAF configuration action `send_event` uses a separate schema from the Customer MCP tool `send_agent_event`. New code or handlers require a separate request.

## Customer email and SMS

`project_attributes_setting_email_send_information_enabled` and `project_attributes_setting_sms_send_information_enabled` control separate permissions. Their allowed_content or allowed_items fields specify permitted content. `project_attributes_setting_session_ended_sms_enabled` and `project_attributes_setting_session_ended_sms_content` configure a distinct post-call message. Staff reports do not enable it.

Verify recipient, content and authorization against the request and business rules. The owner confirms consent and unsubscribe requirements. An enabled flag alone does not establish legal permission. Verify actual delivery separately from acceptance and test simulation.

## Sensitive data and tests

For medical appointment records, verify identity before reading or changing customer data. Collect the minimum necessary and omit sensitive material from support reports. Honor `phi_restricted` across integrations.

See [testing.md](testing.md) for test mode. Only supported branches are simulated, and custom actions can have real effects. Verify results in the agreed test environment. Connector presence, saved JSON and a spoken success message require actual integration evidence.

## Post-call delivery

Use `search_conversations(customer_idn, session_id, turn_type="all")` and `search_logs(customer_idn, session_id)` for an existing send. Get event names and provider response mapping from active NAF and logs. Email metadata declares `result_success` and `result_error`. Receipt depends on how the provider and flow use these. `newo_sms` declares an integration command `send_message` without a delivery event. The sandbox tool has its own contract. Report invocation, acceptance, delivery, rejection or unknown outcome.
