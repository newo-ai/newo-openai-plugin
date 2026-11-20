---
name: newo-external-mcp
description: Connect an external MCP server to a Newo agent. Configure an mcp connector and, for OAuth2, create a dedicated actor and local project, send init_mcp and verify consent.
---

# Connect an external MCP server to a Newo agent

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md) and [external MCP contract](../../references/agent-mcp-client.md). Perform Newo operations through Customer MCP `newo`.

1. Identify the exact `customer_idn`. Confirm any missing server address and authorization type. For OAuth2, confirm any required scopes and who will provide consent. If the request includes agent behavior, confirm which tools it should use. Have the user enter secrets in the approved secure interface.
2. Check current fields with `get_integrations(integration_idn="mcp")` and existing connectors with `get_connectors`. Before writing, verify that the catalog supports the selected authorization type. Reuse a compatible connector or create one with `create_connector`. For an existing connector, send only the agreed changes in `update_connector.settings`. Read back the settings. When asked to start the connection, start any stopped connector with `run_connector`.
3. For no authorization, headers or JWT, finish after configuring the connector and completing the agreed verification. The following steps apply to OAuth2.
4. Create a dedicated persona using `create_persona` and add an actor using `add_persona_actor` with `integration_idn="mcp"` and the current MCP connector's `connector_idn`. Verify the binding and UUIDs with `get_persona`. Reuse the verified persona and actor for repeated runs of the project.
5. Prepare and activate a local version of the dedicated project using the [Python WASM template](../../references/agent-mcp-client.md#local-project-template). Subscribe `StartOAuth` directly to `init_mcp` on `mcp/<connector_idn>`. OAuth2 must run in the persona and actor context. Never start it via `project_publish_finish` (`project_publish_finished`) or `send_global_agent_event`.
6. Call `send_agent_event` with `customer_idn`, the dedicated `actor_id` and `event_idn="init_mcp"`. `StartOAuth` checks the context and sends `oauth2_start` to the connector. Verify the command in `search_logs` by `external_event_id` and verify the actual `user_actor_id` and `user_persona_id` in `search_conversations`.
7. Check `oauth2_link`, `oauth2_success` and `oauth2_error` for the selected connector. Share the consent link and expiry with the authorized user. After consent, verify `oauth2_success` or catalog access with `newo.mcp.get_tools`.

If the outcome from creation, activation or `send_agent_event` is uncertain, read back state and events before retrying. Use `init_mcp` for reauthorization. Report the connector and project IDNs, the version UUID for OAuth2 and the confirmed authorization status. Include catalog verification if requested.
