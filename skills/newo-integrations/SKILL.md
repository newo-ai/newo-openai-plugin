---
name: newo-integrations
description: Manage Newo connectors, voice, STT and webhooks, including explicitly requested connector deletion. Use existing settings and handlers to connect CRM, calendars, booking and notifications.
---

# Connectors, webhooks and business integrations

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md). Use [integrations.md](../../references/integrations.md) for connectors and webhooks and [business-integrations.md](../../references/business-integrations.md) for CRM, calendars, booking and actions.

Identify the system, action, access, test environment and existing connection.

- `get_integrations` lists integration types shared by customers. With `integration_idn`, it returns setting types, required and read-only status, choices, dependencies, events and commands.
- Pass only changed fields in `update_connector.settings`. Check IDs and allowed values with `get_integrations`. Read the connector before the write and verify it afterward. To undo a change, pass only the affected fields. Follow the [connector verification procedure](../../references/integrations.md#connector). Updates to phone or SMS connectors are sent immediately to their service.
- Start or stop a connector with `run_connector(is_run=true/false)`. A first start can purchase a number for a phone or SMS connector. Restart only when needed.
- Use `delete_connector` only for an authorized deletion. Follow the [connector deletion procedure](../../references/integrations.md#delete-a-connector). Deletion also removes the connector’s webhooks and can release a phone or SMS number.
- Use `get_incoming_webhooks` and `get_outgoing_webhooks` and the corresponding `create_*`, `update_*` and `delete_*` tools. The server creates the incoming URL. POST raises events on the `api` connector. Outgoing webhooks send command arguments to the agreed HTTPS domain. Updating event, command or IP lists replaces the list. The service stores `allowed_ips` without enforcing it. Delete webhooks only on request. A replacement incoming webhook gets a different URL. Outgoing webhook creation, updates and deletion are unavailable for customers with a BAA.
- Use [newo-external-mcp](../newo-external-mcp/SKILL.md) to connect an external MCP server, including the local project and handler for OAuth2.
- Select voice and STT using `newo_voice` metadata. Read WebRTC settings from the specific voice connector. If a write fails, read back its state and capture the error using the [connector procedure](../../references/integrations.md#connector).
- Edit attribute-controlled settings at the source attribute. CRM, calendars and actions require known formats and handlers. A webhook alone cannot implement new agent behavior.

`send_message` operates in the sandbox. `send_agent_event` raises an event from the supplied `actor_id`. Use `send_global_agent_event` on an explicit request for a connector event without a user or session. Events can trigger real calls, SMS or email. Verify the result in `search_logs` by `external_event_id` and in `search_conversations` from `sent_at`.

Read back writes and test the scenario. Confirm booking or delivery using system results available through Customer MCP or an observation attributed to the user. Leave direct calls to the external system to its operator.
