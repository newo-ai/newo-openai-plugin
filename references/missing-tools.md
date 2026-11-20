# Catalog limits and open contracts

Customer MCP contains 57 tools. See the [inventory](tool-inventory.json) for sources and [tool reference](mcp-tools.md) for procedures.

## Supported by existing tools

- Customer and persona attributes have separate search, get, set, create and delete tools. Customer values save immediately. Initialization effects depend on NAF.
- Read project attributes in `project.yaml` or a referenced file. Use `read_project_version_files` and `write_project_version_files` on a fork under [project-attributes.md](project-attributes.md).
- Rerun initialization by activating the running version of the relevant project. Verify registry UUIDs using [versions.md](versions.md).
- Attach or restore a registry module with `activate_registry_module`, the registry and module and saved version mode. Active UUID reactivation reruns initialization under [versions.md](versions.md).
- Read a specific connector with `get_connectors` using both `integration_idn` and `connector_idn`. Use returned `api_key` only as described in [operations.md](operations.md).
- Send only changed settings in `update_connector`. Partial updates and rollback were verified on `api` and WebRTC. See [integrations.md](integrations.md).
- `delete_connector` permanently removes a stopped connector and its webhooks. Phone or SMS deletion can release the number. See [connector deletion](integrations.md#delete-a-connector).
- External MCP uses a customer connector. For OAuth2, use the dedicated project in [agent-mcp-client.md](agent-mcp-client.md).
- Read WebRTC settings from the voice connector and `portal_url` and `chat_urls` from detailed `get_customers`. Verify voice separately from chat.
- Investigate sends with `search_conversations` and `search_logs`. Delivery evidence depends on implementation and provider data.
- AKB supports reading, searching, creating and deleting topics. A duplicate name creates a second topic. No in-place update or separate publication is available.

## Work that needs a confirmed contract

- `activate_project_version` requires a valid version UUID. Reading or forking also requires access to that version. `project_idn`, `all` and empty versions provide no confirmed workaround.
- Workflow Builder has no direct operations. Use confirmed customer attributes and AKB topics for supported behavior changes.
- OAuth2 connector fields depend on deployment metadata. Python `newo.generation.generate_structured` and the `platform_tools` attribute have separate contracts.
- WebRTC URL format and the integration or event for automated calls or Assessor need a confirmed deployment contract. Voice settings and actual calls can be verified independently.
- Knowledge Studio is outside Customer MCP. The plugin uses one server.
- Agent code can provision numbers. There is no standalone purchase tool or purchase code in this plugin. The first start of a phone or SMS connector can purchase a number as a side effect.
- Use the [development guide](agent-development.md) for requested agent code changes and confirm runtime action support in the deployed runner. Support submission, porting, external carrier changes, general invitations or role changes and automatic test customer allocation lack dedicated tools.

The incoming webhook service stores `allowed_ips` without enforcement. A BAA limits conversations, logs, personas and outgoing webhook changes. Agent AKB remains available. Respect these restrictions across channels.
