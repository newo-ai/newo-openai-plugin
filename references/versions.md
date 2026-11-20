# Apply settings, publish versions and initialize projects

Use Customer MCP for all operations. Verify customer attribute writes and initialization separately. See [agent-development.md](agent-development.md) for Python agent code, [project-attributes.md](project-attributes.md) for targeted project attribute edits and [agent-mcp-client.md](agent-mcp-client.md) for the dedicated external MCP OAuth2 project.

## Save and apply

`set_customer_attribute_value(customer_idn, idn, value)` saves immediately. A flow that reads it at runtime sees the new value on its next read. If a handler reads it at initialization, run the appropriate initialization separately. NAF implementation determines which behaviors need initialization. Check the subscription and attribute use in the accessible active version, review permitted logs and test a fresh conversation. Avoid publishing after routine writes unless needed.

To initialize, call `activate_project_version(customer_idn, project_version_id)` with the project's active version. For a registry UUID, verify the specific version as described below. Check subscriptions before initializing several projects. External MCP OAuth2 starts separately through `init_mcp` in persona and actor context.

Reactivation emits `project_publish_finish` with `triggered_by: manual`. Use the activation tool and verify the actual event and subscription in the logs. An action requires a matching subscription.

Use the returned `external_event_id` with `search_logs(customer_idn, external_event_id=...)`. Verify initialization in logs, affected settings and a new session. Activation alone confirms the selected version.

## Active registry module version

`list_projects(customer_idn)` returns `active_registry_module`, `registry_item`, `active_project_version_id` and local versions. A registry reference without `@version` uses `latest`. The agent always runs the most recently published module version and follows later releases automatically.

For registry projects, `active_project_version_id` holds the version UUID recorded at the most recent activation, when available. Match the UUID to the required release for version-specific calls.

| Observation | Action |
| --- | --- |
| The running version has a verified, accessible UUID | Pass it to `activate_project_version` for initialization or `read_project_version_files` for inspection. For an authorized fork pass it as `source_project_version_id` to `create_project_version`. |
| The registry UUID is absent from local `versions` | This is normal for a module version. Check access by reading its files and report any server refusal. |
| The ID is null | `activate_project_version` requires a version ID. `activate_registry_module` can attach a known module on request. Check the impact of changing the attachment before using that tool. |
| The list is truncated | Account for incomplete results when checking whether a version exists. |

Save `active_registry_module`, `registry_item` and `active_project_version_id` before switching. Restore the original reference with its unpinned or pinned version mode. Leave the attachment in place during reads and verification.

## Attach or restore a registry module

`activate_registry_module(customer_idn, project_idn, registry, module, version?)` attaches a public registry module, including when replacing a local version. The registry is the part before `/`, and the module is the part after `/` before any `@version`. Use the known reference from `list_projects` or the user.

For `latest`, omit `version`. To pin a release, pass the version string after `@`. Use the original version mode when restoring an attachment.

Use `activate_registry_module` to restore a module after a local fork. `activate_project_version` with a registry UUID can only reinitialize the version while the registry version is active. The registry tool returns `active_registry_module`, `module_version`, `active_project_version_id`, `previous_registry_module`, `previous_project_version_id` and `external_event_id`.

If the previous project ran a registry module, restore its previous reference and version mode. If `previous_registry_module=null` and a local version was active, use `activate_project_version(previous_project_version_id)`. Read back the attachment and version with `list_projects`. Verify initialization by `external_event_id` and test a new conversation.

## Publish and activate

`create_project_version(customer_idn, project_idn, source_project_version_id, version_prefix?)` copies files from the specified version into a draft. Omitting the source creates an empty `project.yaml`. Customer attributes retain their scope when a project is forked.

`publish_project_version(customer_idn, project_version_id)` validates and publishes without activating. Validation failure leaves the state unchanged. Publication makes the version read-only. Publishing an already published version returns it as is. Activate only when requested.

`activate_project_version` validates and publishes a draft before switching the running agent. The switch can affect callers. Record the previous active version and check `list_projects` afterward. A separate publish call is optional for draft validation.

After timeout, read back state before retrying. Reactivating the active version reruns initialization. When acting on several projects, record partial outcomes and handle each operation independently.

## Roll back

Identify whether the original project used a local version or registry module. To restore a local version, call `activate_project_version` with its prior ID. To restore a module, call `activate_registry_module` with the saved reference and version mode. When switching from registry to local, `previous_project_version_id` can be null, so save the original registry reference first. A UUID alone cannot reconstruct an unknown registry attachment. Restore customer attributes separately with `set_customer_attribute_value`. A version switch cannot undo knowledge changes, webhooks, external writes or connector changes. Test again after rollback.
