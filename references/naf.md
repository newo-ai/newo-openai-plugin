# NAF and release scope

NAF determines when an agent reads attributes, retrieves knowledge, processes events and calls connectors. Customer attributes save immediately. Initialization depends on the implementation, as described in [versions.md](versions.md). Distinguish confirmed behavior from assumptions about a deployment.

Customer and persona attributes have separate tools. Setting a value preserves metadata. Project attributes live in `project.yaml` or a referenced file. See [project-attributes.md](project-attributes.md) for inspection and targeted changes on explicit technical request.

Customer MCP has no direct Workflow Builder interface. Make supported behavior changes using existing customer attributes and AKB topics. Verify the actual data model before claiming support for canvas, intent types, scenarios or procedures. There is no universal canvas IDN, schema or publish-time rebuild rule.

A registry project can have an active module version ID with an empty list of local versions. Reading its files and forking it require an accessible version. Null or inaccessible IDs leave those operations unavailable. See [versions.md](versions.md).

[newo-external-mcp](../skills/newo-external-mcp/SKILL.md) provides a dedicated NAF project for external MCP OAuth2. Other NAF development, such as programmatic number provisioning, requires a separate request. Knowledge Studio is outside the catalog.
