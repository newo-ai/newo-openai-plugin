# Phone lines and phased rollout

Perform supported Newo operations through Customer MCP. Configure SIP on the connector using [integrations.md](integrations.md). External telephone systems remain under their operator's control.

## Numbers and routing

| Number | Source and purpose |
| --- | --- |
| Public business number | `project_business_public_phone` and owner confirmation. Customers call this number. |
| Agent number | `get_customers(customer_idn=...)` or `project_representative_agent_voice_phone_number`. Used to reach the agent. Assignment occurs through provisioning. |
| Staff number | Business-hours or after-hours escalation line or additional destination. A person must be reachable there. |

Forward inbound calls from the business number to the agent number. Verify that transfers back to a person and bypass numbers do not route into the agent again. Normalize full numbers to international format. Use short extensions only in the appropriate PBX mode.

## Existing line

Confirm the carrier or PBX, line type, rollout window and rule for scheduled, no-answer, busy or unconditional forwarding. If Customer MCP cannot manage it, prepare a specification for the owner with the source line, agent number, time zone, condition, timer in seconds, start and end and exact rollback. Carrier codes depend on the line and plan. Mark external work as pending until the owner or administrator confirms the change.

After changing routing, call the business number from a different phone. Check the new session and Caller ID. If callers all appear as the business number, report a routing issue. Restore the forwarding rule separately from stopping the connector to avoid leaving the line unanswered.

## SIP

Gather nonsecret host and port, transport, account name or extension, Caller ID, outbound dialing or REFER, and signaling and RTP addresses when needed. Use secure password input. Choose registration-free mode only when supported by the connector schema and confirmed by the provider.

Find the connector with `get_connectors` and confirm SIP fields with `get_integrations(integration_idn="newo_voice")`. Send only changed fields in `update_connector.settings`. Verify and roll back using [integrations.md](integrations.md). Use `run_connector(is_run=true)` or `run_connector(is_run=false)`. The first start of a phone or SMS connector can purchase a number. Check registration and audio separately from the `running` state. If registration state is unavailable, verify a call or draft a support request using actual observations.

Have the line owner verify PBX routing, REFER permissions, allowed networks and outbound dialing. Check inbound calls, outbound calls when needed, two-way audio and transfers separately. Missing calls or audio may require network access on the Newo side. Prepare a [draft operations request](support.md).

## Transfer to a person

Use `project_attributes_setting_transfer_call_enabled`, `project_attributes_setting_transfer_call_criteria`, `project_attributes_setting_transfer_call_phone_numbers`, `project_attributes_setting_transfer_call_destination_format`, `project_business_working_hour_escalation_phone` and `project_business_non_working_hour_escalation_phone`. Choose the required `full_number` or `short_code` format. Verify staff availability, fallback, loop prevention and availability under current connection and billing. Paid or international routing requires approval.

Verify every agreed route with a live call and an available staff member. Treat test-mode simulations as separate from live transfers.

## Additional numbers and porting

Collect the customer, country, number type (local, national, mobile or toll-free), locality or area code, voice and SMS needs, demo or production, current line retention and acceptable recurring cost. Use the current Newo response for availability, documents and timing. Keep identity or business documents out of chat.

Prepare a draft provisioning or porting request marked “not sent.” Submission, tracking and number assignment are external. The owner confirmed that agent code can purchase a number for an existing customer. A standalone purchase tool and new purchase code are outside this workflow. A connector's first start can provision a number as a side effect. Provisioning follows service rules and cannot be done by overwriting a read-only attribute. After the user confirms assignment, verify the number and requested voice or SMS behavior. Deleting a phone or SMS connector can release its number. Follow the [connector deletion procedure](integrations.md#delete-a-connector) for that separately authorized action.

## Rollout

1. **Readiness.** Verify critical criteria, customer acceptance, hours and time zone, destinations and reports. Check live environment settings and test-mode overrides.
2. **After-hours pilot.** Confirm the agent states the office is closed and performs available actions. Route to on-call staff or take a message. Check report delivery. If bookings are simulated, disclose that the appointment is pending.
3. **Live line.** Confirm the external route, test call, rollback and who ends the change window. Arrange rollback with the line owner.
4. **Initial review.** Consider reviewing all calls from the first two or three nights. Fix issues and repeat tests. Scheduling automation requires its own setup.
5. **Daytime.** On the owner's decision, use overflow from staff on no answer or busy, or AI-first with transfers. Verify the new routing and review the first calls.

Have the line owner restore the external rule for rollback. Restore connector or attribute values separately after reading current state. Verify the next inbound call. Custom audio bridges and new telephony backends are outside this workflow. Configure existing connectors through their supported schema.

## Browser calls

Detailed `get_customers(customer_idn=...)` returns `portal_url` and `chat_urls` when available. For WebRTC, read `get_integrations(integration_idn="newo_voice")` and `get_connectors(customer_idn, integration_idn="newo_voice", connector_idn=...)`. Settings are on the voice connector. The provider can be `webrtc` and `webrtc_allowed_origins` specifies origins. Return the actual URL or parameters under a verified client contract. An `api_key` does not determine the frontend address. Use an existing confirmed flow for automated calls or Assessor, or prepare a developer request.
