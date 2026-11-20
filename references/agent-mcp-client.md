# External MCP for a Newo agent

[newo-external-mcp](../skills/newo-external-mcp/SKILL.md) connects an external server through a Newo customer's connector. For OAuth2 it sets up a dedicated persona and actor and a local project with handlers for starting authorization and processing results.

This procedure draws on the Confluence pages “MCP Usage” (323944449, version 4), “MCP: tools & OAuth2” (615940100, version 5) and “Projects v2: Overview and Key Components” (506101766, version 2). Use the current Customer MCP schemas for creation and updates.

For runtime tool discovery and generation after setup, use the [Python action reference](python-actions.md#events-messages-and-integrations) and [development guide](agent-development.md#generate-text-structured-results-or-a-stream).

## Inputs

Identify the `customer_idn`, MCP server address and authorization method. Supported methods include no authorization, headers, JWT and OAuth2. Confirm the intended use, existing connection and required tools. For OAuth2, find out which scopes the provider needs and who will open the consent link. A preconfigured `client_id` may be optional when the provider supports discovery and client registration.

Use a unique connector IDN containing ASCII letters, digits and underscores. Start with a letter or underscore. A customer connector with the same IDN takes precedence over a global AvailableMCP. The top-level `title` names the connector. Use `settings.title` to name the MCP server if the field is supported and needed.

## Create or update the connector

1. Read `get_integrations(integration_idn="mcp")` and `get_connectors(customer_idn, integration_idn="mcp")`. Verify field type, availability and write support before creation. Record the current state of a connector you plan to reuse.
2. Set `settings.url` to the requested address. For a new connector without authorization, leave authorization settings empty. The header and JWT formats from “MCP Usage” apply when the metadata declares a string field named `auth`. Serialize the corresponding object as a JSON string once.

   ```json
   {"type":"headers","data":{"HeaderName":"VALUE_FROM_SECURE_INPUT"}}
   ```

   ```json
   {"type":"jwt","data":"VALUE_FROM_SECURE_INPUT"}
   ```

   Have the user enter secrets through the approved secure interface. Report only whether they are configured.

3. For interactive OAuth2, use `auth_type="oauth2"` and optional `oauth2_scopes` when the current metadata declares these IDs. The scope list defaults to empty. Use the representation supported by the field and tool. Customer MCP `settings` accepts strings and numbers. If scopes must be entered as a list and string encoding is unconfirmed, use a supported connector settings interface. Use the default when the request requires no specific scopes.

   If the catalog only shows the string field `auth`, verify how interactive OAuth2 is enabled in this deployment. The older `auth.type="oauth2"` JSON format alone does not establish support for `oauth2_start`. If the OAuth2 mode cannot be set through a supported interface, stop before creating the project and report the specific gap.

4. Create the connector with `create_connector(customer_idn, integration_idn="mcp", connector_idn, title, settings)`. For an existing connector, pass only agreed fields in `update_connector.settings`. Read back the address, authorization mode and scopes with `get_connectors`. New connectors start stopped. When asked to start the connection, use `run_connector(customer_idn, integration_idn="mcp", connector_idn, is_run=true)` and verify its status.
5. For a connection without OAuth2, finish with connector configuration and the agreed verification. The rest of this procedure applies to OAuth2.

## Prepare the persona, actor and local OAuth2 project

1. Check access and project IDs with `list_projects(customer_idn)`. Use a dedicated project clearly associated with the connector, such as `mcp_oauth_<connector_idn>`. For a repeat request, read the existing project and verify its subscriptions, connector and actor binding.
2. Call `create_persona(customer_idn, name)`. Pass `customer_idn`, the returned `persona_id`, `integration_idn="mcp"`, the current `connector_idn`, `name` and a service `external_id` such as `oauth2_init_<connector_idn>` to `add_persona_actor`. Use `get_persona` to verify the UUIDs and the actor's connector. Reuse the verified persona and actor UUIDs from the project on subsequent runs.
3. For a new project, call `create_project(customer_idn, idn, title)` and `create_project_version(customer_idn, project_idn)` without a source version. For a published project, fork its active local version with `source_project_version_id`. Remove any old global OAuth2 subscription to `project_publish_finish` or `project_publish_finished`. If migrating a project with `RouteOAuth`, remove that skill, its file using `delete_paths` and its `system/system:init_mcp` subscription.
4. Write the template files with `write_project_version_files`. Substitute actual values for `MCP_CONNECTOR_IDN`, `MCP_PERSONA_ID` and `MCP_ACTOR_ID`. Read the files back. Record the previous version UUID and activate the draft UUID with `activate_project_version`. Verify the active version with `list_projects`.

## Send init_mcp

1. Confirm the connector is running and the `send_agent_event` schema requires `actor_id`. Call `send_agent_event` with `customer_idn`, the dedicated `actor_id` and `event_idn="init_mcp"`. The tool gets the integration and connector from the actor. Record `external_event_id` and `sent_at`. Refresh the client catalog before calling if it still shows the older schema.
2. `StartOAuth` receives `init_mcp` directly from the actor on `mcp/<connector_idn>`. It verifies the actual actor and persona, then sends `oauth2_start` with `newo.commands.send`. OAuth2 requires persona and actor context. Never start it with `send_global_agent_event` or `project_publish_finish`.
3. Verify `StartOAuth` and the command with `search_logs(external_event_id=...)`. Query `search_conversations` from `sent_at` with `connector="mcp/<connector_idn>"`, `event_names=["init_mcp"]`, `turn_type="custom"` and `user_actor_ids=[...]`. Match `external_event_id`, `user_actor_id` and `user_persona_id`. On timeout, check acts and logs before retrying. For reauthorization, send `init_mcp` from the same actor.

## Local project template

`project.yaml`

```yaml
name: External MCP OAuth setup
agents:
  ExternalMcpOAuth:
    flows:
      OAuthSetup:
        skills:
          StartOAuth:
            execution_path: skills/StartOAuth.py
            runner_type: python_wasm
            parameters: []
          ObserveOAuth:
            execution_path: skills/ObserveOAuth.py
            runner_type: python_wasm
            parameters: []
        subscriptions:
          - idn: init_mcp
            integration_idn: mcp
            connector_idn: "MCP_CONNECTOR_IDN"
            interrupt_mode: queue
            skill_selector: skill_idn
            skill_idn: StartOAuth
          - idn: oauth2_link
            integration_idn: mcp
            connector_idn: "MCP_CONNECTOR_IDN"
            interrupt_mode: queue
            skill_selector: skill_idn
            skill_idn: ObserveOAuth
          - idn: oauth2_success
            integration_idn: mcp
            connector_idn: "MCP_CONNECTOR_IDN"
            interrupt_mode: queue
            skill_selector: skill_idn
            skill_idn: ObserveOAuth
          - idn: oauth2_error
            integration_idn: mcp
            connector_idn: "MCP_CONNECTOR_IDN"
            interrupt_mode: queue
            skill_selector: skill_idn
            skill_idn: ObserveOAuth
        state_fields: []
```

`skills/StartOAuth.py`

```python
from uuid import UUID

import newo


def main():
    actor = newo.actors.get()
    if (
        actor.id != UUID("MCP_ACTOR_ID")
        or actor.persona_id != UUID("MCP_PERSONA_ID")
        or actor.integration_idn != "mcp"
        or actor.connector_idn != "MCP_CONNECTOR_IDN"
    ):
        raise ValueError("Unexpected actor for MCP OAuth2 initialization")

    newo.commands.send(
        integration_idn="mcp",
        connector_idn="MCP_CONNECTOR_IDN",
        command_idn="oauth2_start",
    )
```

`skills/ObserveOAuth.py`

```python
def main():
    return "mcp_oauth_event_observed"
```

`ObserveOAuth` returns an execution marker. The event contains the link and the other result arguments.

## Retrieve the link and verify authorization

`newo.commands.send` starts authorization asynchronously. Query `search_conversations` from the `init_mcp` event's `sent_at` with `connector="mcp/<connector_idn>"`, `turn_type="custom"` and `event_names=["oauth2_link","oauth2_success","oauth2_error"]`. Match the connector, time and logs for the attempt. A result event can have its own `external_event_id`. If acts are unavailable, say which confirmation is missing.

| Event | Action |
| --- | --- |
| `oauth2_link` | Read `authorization_url` and `expires_at` from the arguments. Share the link and expiration with the authorized user. Record the status as awaiting consent. |
| `oauth2_success` | Record that the connector is authorized. The event has no arguments. If requested, verify catalog access separately. |
| `oauth2_error` | Report `code`, `error_type` and a safe summary of `reason`. For `409`, check the authorization type. For `410`, check the attempt expiry. For `403`, check the user's or provider's decision. For `502`, check discovery and registration. Retry after resolving the cause or on an agreed request. |

If the server reports unsupported CIMD or DCR, register an OAuth2 client with the provider. Confirm the `client_id` and direct the owner to the approved secure interface for `oauth2_client_secret`. Use these fields only as supported by the current deployment. Keep secrets out of project code and output. After configuration, send `init_mcp` again for the same connector and verify the new link.

## Verify the tool catalog

`newo.mcp.get_tools(mcp_idn=...)` in Python WASM reads a connector's tool catalog for the current customer. Use `tool_names` for exact names and `query` to search names and descriptions. Omit the names list to request all tools. An empty list returns no tools. Authorization or external server errors can prevent catalog reads. A successful call confirms access to the tool definitions.

To connect tools to a Python scenario with `newo.generation.generate_structured`, pass a `newo.models.MCPTool` in `tools` with the connector's `server_idn` and the agreed `allowed_tools`. An empty list disables tools for that server. Omitting the list allows all of them. For the `platform_tools` attribute, verify the actual scenario's contract separately. Catalog inspection does not invoke external actions.
