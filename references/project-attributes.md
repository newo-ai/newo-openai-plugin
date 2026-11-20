# Project attributes in version files

Project attributes live in `project.yaml` or a referenced file. Use `read_project_version_files` and `write_project_version_files` to read, add, edit or remove their definitions. Customer attribute tools use a separate scope. A `project_` prefix alone does not establish scope.

Use this procedure for a targeted change to a confirmed project attribute on explicit technical request. Use [agent-development.md](agent-development.md) for explicitly requested edits to flows, skills or dependencies. See [agent-mcp-client.md](agent-mcp-client.md) for a dedicated external MCP OAuth2 project.

## Read the actual definition

1. Select the project in `list_projects(customer_idn)`. Before reading or forking, match the UUID to the intended release using [versions.md](versions.md). Report a missing UUID or inaccessible version.
2. Call `read_project_version_files(customer_idn, project_version_id)` for the tree, then call it with `path="project.yaml"`. If the file references attribute definitions, follow the actual path shown there.
3. Find the IDN, definition, type, current value and active constraints. The [attribute catalog](attribute-catalog.json) provides known examples. Inspect the current NAF for the full set of project attributes.
4. `project_attributes_setting_test_mode_phone_numbers_static` is a project-level JSON list of numbers. Read and edit its actual file representation. Keep it in project scope.

## Edit the agreed attribute

Save the original active version, `active_registry_module`, `registry_item` and version mode. Prepare an exact diff. `create_project_version(customer_idn, project_idn, source_project_version_id=active_id)` forks an accessible version into a draft. Activating a local fork of a registry module changes how the project is deployed. To restore it, use `activate_registry_module` with the saved reference from [versions.md](versions.md). Clarify the specific impact if the switch falls outside the request or the original attachment is unknown.

Read the entire affected draft file. `write_project_version_files(customer_idn, project_version_id=draft_id, files=[{path, content}])` replaces the complete text of each supplied file. If the reply is `truncated` and the full text is unavailable, suspend the write and report the limitation. Leave `delete_paths` unset when changing a single value.

Preserve other attributes, references, code and value formats. Adding or removing a definition requires an explicit request and a known schema. Read the draft back, check the diff and publish or activate as authorized using [versions.md](versions.md). Activating the version applies draft or published changes to the running agent.

After activation, verify the active file, initialization and behavior in a new session. Restore a prior local version with `activate_project_version` or a known module with `activate_registry_module` and its original version mode. Verify with `list_projects`. A version switch cannot undo actions already executed or recover an unknown attachment automatically.
