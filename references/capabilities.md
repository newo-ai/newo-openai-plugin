# Capabilities and verification evidence

The plugin includes ten skills for 57 Customer MCP tools. Contracts draw on the Newo 1.30.0 live schemas and annotations, owner clarifications and `customer_mcp_service` documentation. General integration metadata has been checked for the catalog, `mcp`, `newo_voice`, `email` and `newo_sms`. Customer tests are recorded separately.

The [inventory](tool-inventory.json) records source snapshots, additional sources and runtime checks. The [matrix](use-cases.json) defines 63 scenarios, initially `not_run` pending acceptance on the selected test customer. Verify scope and authorization for each business.

## Scope

The skills cover agent creation and configuration, customer and persona attributes, AKB, connectors and webhooks, telephony, sandbox, diagnostics and rollout. Supported file operations include [Python agent development](agent-development.md), targeted project attributes and a dedicated external MCP OAuth2 project template. The development skill documents 77 [Script Runner actions](python-actions.md) available in the supplied source snapshot. They run inside Python skills and have separate contracts from the Customer MCP tool catalog.

## Before making changes

1. Select the customer and verify your own link and role. Admin status alone grants no customer access. Account-link tools require a direct link from the caller.
2. Use search, get and set for customer attributes and pass string values. Saves are immediate. Initialization depends on how NAF reads the field.
3. For project attributes, flow subscriptions or AKB agents, verify the actual active ID through `list_projects` and read accessible files, including registry versions. Handle null or inaccessible versions using [versions.md](versions.md).
4. Verify connector values against `get_integrations` and read state with `get_connectors`. Use the dedicated project and handler from [agent-mcp-client.md](agent-mcp-client.md) for external MCP OAuth2.
5. Use actual voice settings for WebRTC and acts, logs and provider responses for delivery. Check the live channel and actual outcome separately from sandbox dialogue and request acceptance.
6. Complete supported work and report the specific remaining [catalog limitations](missing-tools.md). Use only confirmed arguments, channels and events.

Read back writes and verify behavior. Observe BAA access limits. Incoming webhook `allowed_ips` is stored without enforcement. Use the connector `api_key` and handle third-party secrets under [operations.md](operations.md). Verify customer execution separately from source schemas, documentation and owner clarifications.
