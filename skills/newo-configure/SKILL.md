---
name: newo-configure
description: Configure Newo customer and persona attributes, languages, greetings and reports. Use for agent health checks, targeted project attribute edits in a draft, and rerunning initialization on an active version.
---

# Agent configuration and health

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md), [attributes and personas](../../references/configuration.md) and [setup procedure](../../references/setup.md). Use the [attribute catalog](../../references/attribute-catalog.json) for standard IDNs and types. Check availability and metadata on the selected customer.

- To finish setup, confirm services, exceptions, hours, time zone and pricing. Then configure the greeting, languages, transfers and reports. Use the relevant [industry guidance](../../references/industry-guide.md).
- For an agent checkup, follow [health.md](../../references/health.md). Report the evidence, impact and proposed changes. Fix issues within the request.
- For behavior changes or reports, find the supported attributes and structure in [setup.md](../../references/setup.md). Use an attribute designed for the behavior.

Find fields with `search_customer_attributes(query=...)`. Read the full value and metadata with `get_customer_attribute(customer_idn, idn)`, especially when results are `truncated`. Set values with `set_customer_attribute_value`. Pass `value` as a string even for booleans, numbers and JSON. Metadata is preserved, and writes to read-only attributes are rejected. Use `create_customer_attribute` and `delete_customer_attribute` only for a requested change with a known purpose.

Use `get_persona`, `create_persona` and `add_persona_actor` for caller records. Use `search_persona_attributes`, `get_persona_attribute`, `set_persona_attribute_value`, `create_persona_attribute` and `delete_persona_attribute` for their attributes. Get the persona ID from a tool result or an act's `user_persona_id` and the actor ID from `get_persona`. Call `delete_persona` or `delete_persona_actor` only when deletion is requested. Deleting a persona permanently deletes its actors. Persona tools are unavailable for customers with a signed BAA.

Customer values save immediately. Whether initialization is required depends on the NAF implementation. A `latest` registry attachment always runs the most recent published version. Follow [versions.md](../../references/versions.md) for initialization by UUID and for restoring a registry attachment after a local fork.

Project attributes live in `project.yaml` or a referenced file. For an explicit technical request, inspect and edit them in a fork using [project-attributes.md](../../references/project-attributes.md). Customer and project attributes have separate contracts. Workflow Builder itself is unavailable through Customer MCP. Use existing attributes and AKB topics for supported configuration changes. For an explicit code or subscription change, use [newo-develop](../newo-develop/SKILL.md). Verify behavior in a new conversation using [testing.md](../../references/testing.md).

Report changes, read-back, new tests, findings and available rollback steps.
