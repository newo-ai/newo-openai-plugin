# Integrations, connectors, webhooks and SIP

Use Customer MCP for all actions. Start with `get_integrations`. Without a filter it lists integration types for all customers. With `integration_idn` it returns settings and connector metadata, including required and read-only status, types, options, defaults, groups, dependencies, events and commands. Verify available models and event IDs in the live catalog.

## Connector

1. Read `connector_settings` with `get_integrations(integration_idn)`. `get_connectors(customer_idn)` lists integrations in use. With `integration_idn` it returns connectors, statuses and connection keys. Adding `connector_idn` returns setting values. Use any returned `api_key` only for authorized setup under [operations.md](operations.md). A connector filter requires an integration filter.
2. Match the value to `get_integrations(integration_idn)` metadata. If a NAF attribute controls it, edit that attribute using [configuration.md](configuration.md). Otherwise use `update_connector`.
3. Pass **only changed settings** in `update_connector.settings`. Verify their IDNs, types, constraints and dependencies against `connector_settings`. Read and retain the original values for those fields. Omitted fields keep their values. Exclude read-only fields and masked secrets. Phone and SMS connector updates go to the provider immediately.
4. `create_connector` takes the customer, integration, a new connector IDN and title. Settings are optional. A new connector starts stopped. After a timeout, read the list before retrying.
5. Use `run_connector` with `is_run: true` to start or `is_run: false` to stop. Repeating the current state preserves it. The first start of a phone or SMS connector can purchase a number. Restart only when necessary and in scope. Check state after each step.
6. Partial updates were verified on `api` and WebRTC. On HTTP 400, read back state and check the supplied keys, types and dependencies. If the cause remains unclear, record the request keys and error without secret values.
7. Read back the connector and compare the intended changes and other fields with the baseline. To roll back, first check current state, then restore only the affected fields and verify by reading again.

## Delete a connector

Use `delete_connector` with the exact `customer_idn`, `integration_idn` and `connector_idn` of an authorized deletion. Read the connector status and identify affected webhooks before deleting. Deletion permanently removes the connector and its webhooks. For phone or SMS, it can also release the number. Confirm these effects with the user before proceeding. If the connector is running, stop it with `run_connector(is_run=false)` within the authorized change, then verify `stopped`. After deletion, check that the connector and its webhooks no longer appear in the relevant lists.

## Voice, STT and SIP

`newo_voice` is the integration IDN. `newo_voice_connector` and `newo_voice_connector_web` can be individual connector IDNs. Inspect the actual connector settings to identify phone or WebRTC. The metadata supports `webrtc` as a provider and includes `webrtc_allowed_origins`. Get the actual calling address from a supported client contract. Select model, provider, voice and `v2v_mode_user_stt` from metadata. Use the string `"None"` only when declared. Native recognition and separate STT are distinct settings. Test languages, numbers, names, pauses and interruptions on the real channel.

Settings can originate in `project_attributes_setting_voice_integration_service_voice_mode`, `project_attributes_setting_voice_integration_service_voice_to_voice_provider`, provider-specific voice fields, `project_attributes_setting_common_phone_numbers` and `project_attributes_setting_voice_integration_service_bypass_phone_number`. Read the actual fields and metadata before writing.

Change SIP and its authorization using declared connector settings. Read the agent number, `portal_url` and `chat_urls` with `get_customers(customer_idn=...)`. Forwarding and porting are external procedures. `send_message` tests the sandbox only. For WebRTC read `get_connectors(customer_idn, integration_idn="newo_voice", connector_idn=...)`. Return the actual address or parameters according to the client contract. A connector key alone cannot establish a frontend URL. Use a confirmed handler for automated calls or Assessor. See [telephony.md](telephony.md).

## Outgoing webhooks

`get_outgoing_webhooks` reads all or one by `webhook_idn`. Commands on an `api` connector send arguments to the URL as JSON POST. `create_outgoing_webhook` accepts the customer, a new `idn`, an external HTTPS domain outside Newo, and a `connector_idn` from `get_connectors(integration_idn="api")`. IP addresses are unsupported. `command_idns` and `description` are optional.

`update_outgoing_webhook` changes only supplied fields and preserves the IDN. Supplying `command_idns` replaces the entire list. An empty list stops URL calls. `delete_outgoing_webhook` returns the deleted object. Create, update and delete are unavailable for customers with a signed BAA. Authorization to read does not authorize external data transfer.

## Incoming webhooks

`get_incoming_webhooks` reads all or one. `create_incoming_webhook` requires the customer, a new `idn`, an `api` connector ID and `event_idns`. Optional fields are `allowed_ips`, `description` and `response_payload` up to 4 KB. The webhook service stores `allowed_ips` without enforcing sender filtering. Use the public URL generated by the server. POST raises events with JSON fields as event arguments.

`update_incoming_webhook` preserves the IDN and URL. Passing `event_idns` or `allowed_ips` replaces the full list. Pass `{}` in `response_payload` to clear it. `delete_incoming_webhook` disables the URL. A replacement receives a different URL. Verify delivery and flow execution after creation.

## Agent actions

Use [agent-mcp-client.md](agent-mcp-client.md) for external MCP and OAuth and [business-integrations.md](business-integrations.md) for CRM, calendars, booking and `project_attributes_settings_platform_tools`. Use existing commands and handlers. `send_agent_event` raises an event from `actor_id` on the actor's connector. Use `send_global_agent_event` with integration and connector on explicit request for a global event. Both can cause real calls, SMS or email. Match logs by `external_event_id` and acts from `sent_at`.
