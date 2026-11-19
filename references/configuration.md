# Customer and persona attributes

Use Customer MCP. Newo 1.30.0 has separate customer and persona tools. Their signatures have no universal `scope` or `project_idn` argument. Read project attributes in `project.yaml` or a referenced file and edit them through version files using [project-attributes.md](project-attributes.md).

## Find and read

`search_customer_attributes(customer_idn, query=...)` matches part of the IDN, group or title case-insensitively. Omit the query to list all. Values are returned as strings with metadata. For `truncated` results, read the full value and metadata with `get_customer_attribute(customer_idn, idn)`.

Use the [standard field catalog](attribute-catalog.json) to identify candidates. Verify actual availability, type, `possible_values` and read-only status on the customer. A `project_` prefix alone does not set the scope. Use version files for project-scoped definitions. Verify the actual nested JSON contract for complex values.

## Change a value

1. Identify the exact IDN and read the original value and metadata. Find the relevant procedure in [setup.md](setup.md).
2. Prepare a diff and retain the old value in the current context. Read the field again before writing to account for concurrent changes. Separate changes can succeed or fail independently.
3. `set_customer_attribute_value(customer_idn, idn, value)` updates only the value of an existing field. Always pass a string, including string-encoded numbers, JSON serialized once and the field's supported boolean format. Match enum values and case. Leave metadata out of the call.
4. Read-only and missing fields are rejected. Verify the intended field rather than creating a substitute. Read back with `get_customer_attribute`.
5. Customer values save immediately. Verify whether the flow reads the value at runtime or during initialization. Test a new session. If initialization is required, use the active version and its confirmed subscription as described in [versions.md](versions.md).

For ordinary values, check for concurrent changes, write the original value and verify. For append values, writing the old value can re-run the import. Follow [knowledge.md](knowledge.md).

## Create and delete fields

On request, `create_customer_attribute` takes the customer, `idn` and a string `value`. It supports `title`, `description`, `group`, `value_type` (`string`, `enum`, `bool`, `number` or `json`), `possible_values` and `is_hidden`. Duplicate IDNs are rejected. Confirm that the flow reads the new field before expecting behavior changes.

`delete_customer_attribute` deletes a field and returns its previous representation. Read-only fields are rejected. Use it only for an authorized deletion. External side effects have their own recovery procedures.

## Callers, personas and actors

`get_persona(customer_idn, persona_id)` returns a user persona and its connector identities, contact details and time zone. Use the `user_persona_id` returned in a `search_conversations` act or the ID from `create_persona`. Agent personas and personas from other customers are unavailable. Use the persona ID rather than an account or session ID.

`create_persona` requires a name and creates a persona without actors. `add_persona_actor` requires the customer, persona, integration, connector, `external_id` and name. Phone external IDs usually use E.164. Contact details default to the external ID and time zone to UTC unless an IANA zone is supplied. The external ID must be unique on its connector.

Use `search_persona_attributes`, `get_persona_attribute`, `set_persona_attribute_value`, `create_persona_attribute` and `delete_persona_attribute` with `persona_id`. Values are strings. Search and full reads follow the same pattern as customer attributes. Verify any read-only restriction in the persona tool schema.

`delete_persona_actor` requires the actor ID from `get_persona` and retains the persona. `delete_persona` permanently deletes the persona and all its actors. A subsequent contact creates a new persona. Use either deletion only on request.

Persona tools, including attribute tools, are unavailable for customers with a signed BAA. Observe this restriction across attributes and other channels.
