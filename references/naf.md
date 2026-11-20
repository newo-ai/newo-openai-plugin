# NAF and release scope

NAF determines when an agent reads attributes, retrieves knowledge, processes events and calls connectors. Customer attributes save immediately. Initialization depends on the implementation, as described in [versions.md](versions.md). Distinguish confirmed behavior from assumptions about a deployment.

Customer and persona attributes have separate tools. Setting a value preserves metadata. Project attributes live in `project.yaml` or a referenced file. See [project-attributes.md](project-attributes.md) for inspection and targeted changes on explicit technical request.

Customer MCP has no direct Workflow Builder interface. Make supported behavior changes using existing customer attributes and AKB topics. Verify the actual data model before claiming support for canvas, intent types, scenarios or procedures. There is no universal canvas IDN, schema or publish-time rebuild rule.

A registry project can have an active module version ID with an empty list of local versions. Reading its files and forking it require an accessible version. Null or inaccessible IDs leave those operations unavailable. See [versions.md](versions.md).

SuperAgent arranges core NAF, extensions and integrations as distinct projects. Trace the owning project, agent, flow, subscription and skill in the active version. For requested code changes, use the [development guide](agent-development.md) and [Python action reference](python-actions.md). Runtime actions are called inside a skill. Customer MCP provides the separate customer configuration, file and version tools.

[newo-external-mcp](../skills/newo-external-mcp/SKILL.md) provides a dedicated NAF project for external MCP OAuth2. Knowledge Studio is outside the catalog.
