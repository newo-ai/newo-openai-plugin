# Customer MCP tools

The Newo 1.30.0 catalog contains 57 tools, including `delete_connector`. `send_agent_event` requires `actor_id`, and `send_global_agent_event` has a separate connector-scoped signature. See [tool-inventory.json](tool-inventory.json) for schemas and annotations. Check the current schema and access before calling.

The inventory records exact parameters, types, defaults, enums and numeric constraints. Tables list required arguments first and optional arguments after the divider. Pass the selected customer wherever the signature requires it. There is no universal scope parameter.

## Customers and accounts

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `get_customers` | None required; optional: `customer_idn`, `page`, `per`, `query`, `filters` | List accessible businesses. With `customer_idn`, return one business with its number, chat links, balance and Portal dashboard link, and ignore other filters. Several `newo_chat` connectors can produce several chat links. |
| `create_customer` | None required; optional: `source`, `agent_creator_idn`, `industry`, `referral`, `account_managers`, `technical_owners`, `linked_emails`, `external_owner_account_id`, `hash_external_account_id`, `partner_idn`, `partner_member_idn`, `external_customer_id`, `promo_code`, `empty` | Start creating the business and agent from `source` and purchase a number in standard mode. Save the latest customer IDN first. Standard creation returns a running `run_id`. Follow progress with `get_run`. Use only `empty` when explicitly asked for an empty business with a default project. |
| `get_customer_usage` | `customer_idn`; optional: `start_at`, `end_at` | Subscription, period or phase, balance, usage, allowances, add-ons, daily voice and text figures and totals. Supply both dates for a range of up to 366 days. Defaults to the current billing cycle. |
| `list_customer_accounts` | `customer_idn` | Linked accounts with email, name, role and `is_you`. Requires your own customer link. |
| `unlink_account` | `customer_idn`; optional: `email` | Remove access for the selected account. Omitting email targets the caller. Unlinking someone else requires the owner role. |

## Sessions, conversations and logs

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `list_sessions` | `customer_idn`; optional: `from_datetime`, `to_datetime`, `channel`, `is_test`, `is_lead`, `query`, `page`, `per` | Sessions newest first with time, channel, duration, outcome and summary. `per` supports up to 1000. |
| `get_session` | `customer_idn`, `session_id` | One session and the most recent 200 user and agent messages. A message count gives no evidence about audio. |
| `search_conversations` | `customer_idn`; optional: `session_id`, `from_datetime`, `to_datetime`, `query`, `flow_idn`, `skill_idn`, `event_names`, `turn_type`, `connector`, `user_persona_ids`, `user_actor_ids`, `cursor`, `limit` | Acts with events, arguments, actors and personas. Each page is oldest first and `next_cursor` goes back in time. Defaults to 182 days. `query` is case-sensitive. |
| `search_logs` | `customer_idn`; optional: `from_datetime`, `to_datetime`, `levels`, `flow_idn`, `skill_idn`, `session_id`, `external_event_id`, `text`, `page`, `per` | Flow and skill logs, function calls, parameters and results. Usually newest first, or oldest first with `session_id`. Use `external_event_id` to trace an asynchronous run. |

## Projects

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `list_projects` | `customer_idn` | Projects, versions, active version ID and registry attachment. For registry projects, `active_project_version_id` records the last activation. See [versions.md](versions.md). |
| `create_project` | `customer_idn`, `idn`, `title`; optional: `description` | Create an empty project for an existing customer, such as the [dedicated external MCP OAuth2 project](agent-mcp-client.md). |
| `update_project` | `customer_idn`, `project_id`; optional: `title`, `description` | Update title and description. The IDN is stable. Behavior is configured separately. |
| `delete_project` | `customer_idn`, `project_id` | Delete a project and its versions. Active projects and projects with published versions are rejected. Outside plugin scope. |

## Project versions

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `read_project_version_files` | `customer_idn`, `project_version_id`; optional: `path` | Without `path`, return the file tree. With `path`, return text that can be truncated. Find project attributes in `project.yaml` or its references. Inspect accessible active or registry files for flow subscriptions and AKB agent IDs. |
| `create_project_version` | `customer_idn`, `project_idn`; optional: `source_project_version_id`, `version_prefix` | Fork an accessible source version into a draft, including an accessible registry version. Omitting the source creates an empty `project.yaml`. Use a verified source when editing an existing agent. |
| `write_project_version_files` | `customer_idn`, `project_version_id`; optional: `files`, `delete_paths` | Replace entire draft files. Use for requested [agent code](agent-development.md), targeted project attributes and the [dedicated external MCP OAuth2 project](agent-mcp-client.md). Published files can be read. |
| `delete_project_version` | `customer_idn`, `project_version_id` | Delete a version and its files. Active versions and registry versions in use are rejected. Outside plugin scope. |
| `publish_project_version` | `customer_idn`, `project_version_id` | Validate and publish without activation. A validation error leaves state unchanged. Publication makes the version read-only. |
| `activate_project_version` | `customer_idn`, `project_version_id` | Activate the exact version UUID. Repeating the active version runs `project_publish_finish`. Subscriptions determine effects. The tool requires a non-null version ID and has no all-project or `project_idn` mode. Use `external_event_id` to verify and `previous_project_version_id` to roll back. |
| `activate_registry_module` | `customer_idn`, `project_idn`, `registry`, `module`; optional: `version` | Attach a public registry module, including after a local fork. Omit `version` to always follow the latest published version, or provide it to pin a release. The previous registry reference and local version ID are returned separately. Verify initialization by `external_event_id`. |

## Integrations and connectors

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `get_integrations` | None required; optional: `integration_idn` | General catalog. With `integration_idn`, return settings, types, required and read-only status, options, defaults, dependencies, events and commands. |
| `get_connectors` | `customer_idn`; optional: `integration_idn`, `connector_idn` | An integration filter returns status and `api_key`. Adding a connector filter also returns settings. Use both filters for WebRTC settings on the voice connector. The returned `api_key` is usable as described in [operations.md](operations.md). |
| `create_connector` | `customer_idn`, `integration_idn`, `connector_idn`, `title`; optional: `settings` | Create a stopped connector. Start it separately after checking for an equivalent existing connector. |
| `update_connector` | `customer_idn`, `integration_idn`, `connector_idn`, `settings` | Send only changed fields. Omitted settings are preserved. Partial updates and rollback were verified on `api` and WebRTC. Phone and SMS connector writes go to their service immediately. See [integrations.md](integrations.md). |
| `run_connector` | `customer_idn`, `integration_idn`, `connector_idn`; optional: `is_run` | `is_run=true` starts and `is_run=false` stops. Repeating the same state preserves it. The first start of a phone or SMS connector can purchase a number. |
| `delete_connector` | `customer_idn`, `integration_idn`, `connector_idn` | Permanently delete a stopped connector and its webhooks and return the prior connector. Deleting a phone or SMS connector can release its number. See [connector deletion](integrations.md#delete-a-connector). |

## Events and messages

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `send_agent_event` | `customer_idn`, `actor_id`, `event_idn`; optional: `arguments`, `external_event_id` | Raise an event from an actor on their connector and in their session. Get the `actor_id` UUID from `get_persona`, `send_message` or an act's `user_actor_id`. Returns `external_event_id` and `sent_at` before flows execute. Can trigger real calls, SMS or email. |
| `send_global_agent_event` | `customer_idn`, `integration_idn`, `connector_idn`, `event_idn`; optional: `arguments`, `external_event_id` | Raise a connector event without a caller or session on explicit request for a global event. Returns `external_event_id` and `sent_at` before flows execute. Can trigger real calls, SMS or email. |
| `send_message` | `customer_idn`, `connector_idn`, `text`; optional: `actor_external_id`, `external_event_id` | Sandbox only. Omit `actor_external_id` for a new conversation or pass it to continue. Returns actor ID, actor external ID, event ID and `sent_at` before the reply. Read replies with `search_conversations`. |

## Outgoing webhooks

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `get_outgoing_webhooks` | `customer_idn`; optional: `webhook_idn` | List webhooks whose `api` connector sends commands to a URL as JSON POST. Optionally select a `webhook_idn`. |
| `create_outgoing_webhook` | `customer_idn`, `idn`, `url`, `connector_idn`; optional: `command_idns`, `description` | Create a webhook on an `api` connector for selected commands. Use a permitted external HTTPS domain outside Newo. IP addresses and customers with BAA are rejected. |
| `update_outgoing_webhook` | `customer_idn`, `webhook_idn`; optional: `url`, `connector_idn`, `command_idns`, `description` | Change supplied fields. The IDN is stable, and supplying `command_idns` replaces the full list. Unavailable for customers with BAA. |
| `delete_outgoing_webhook` | `customer_idn`, `webhook_idn` | Delete the outgoing webhook and return its previous representation. Unavailable for customers with BAA. |

## Incoming webhooks

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `get_incoming_webhooks` | `customer_idn`; optional: `webhook_idn` | List incoming URLs that raise `api` connector events using POST body fields as arguments. Optionally select `webhook_idn`. |
| `create_incoming_webhook` | `customer_idn`, `idn`, `connector_idn`, `event_idns`; optional: `allowed_ips`, `response_payload`, `description` | Generate a public URL for `event_idns` on an `api` connector and an optional response up to 4 KB. `allowed_ips` is stored without enforcement. |
| `update_incoming_webhook` | `customer_idn`, `webhook_idn`; optional: `connector_idn`, `event_idns`, `allowed_ips`, `response_payload`, `description` | Change supplied fields. The IDN and URL remain stable. Supplying `event_idns` or `allowed_ips` replaces the respective list. Pass `{}` to clear `response_payload`. |
| `delete_incoming_webhook` | `customer_idn`, `webhook_idn` | Delete the incoming webhook and disable its URL. Creating a replacement generates a different URL. |

## Customer attributes

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `search_customer_attributes` | `customer_idn`; optional: `query` | Case-insensitive search by IDN, group or title. Omit `query` to list all. Returns string values and metadata. Long values may be truncated. |
| `get_customer_attribute` | `customer_idn`, `idn` | Read one customer attribute's full value and metadata. |
| `set_customer_attribute_value` | `customer_idn`, `idn`, `value` | Immediately save the string value of an existing field and preserve metadata. Read-only and missing fields are rejected. Initialization effects depend on NAF. |
| `create_customer_attribute` | `customer_idn`, `idn`, `value`; optional: `title`, `description`, `group`, `value_type`, `possible_values`, `is_hidden` | Create a field with a string value and type `string`, `enum`, `bool`, `number` or `json`. Duplicate IDNs are rejected. |
| `delete_customer_attribute` | `customer_idn`, `idn` | Delete a field on explicit request and return the previous field. Read-only fields are rejected. |

## Personas and callers

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `get_persona` | `customer_idn`, `persona_id` | Read the caller's user persona with actors, contact details and time zone. An agent persona or a persona from another customer cannot be retrieved. Unavailable for BAA. |
| `create_persona` | `customer_idn`, `name`; optional: `title`, `description` | Create a named user persona without actors. Unavailable for BAA. |
| `delete_persona` | `customer_idn`, `persona_id` | Permanently delete the persona and its actors. Later contact creates a new persona. Unavailable for BAA. |
| `add_persona_actor` | `customer_idn`, `persona_id`, `integration_idn`, `connector_idn`, `external_id`, `name`; optional: `contact_information`, `time_zone_identifier` | Add a connector identity with a unique `external_id` on that connector. Phone numbers usually use E.164. Contact details default to the external ID and the time zone to UTC. Unavailable for BAA. |
| `delete_persona_actor` | `customer_idn`, `persona_id`, `actor_id` | Delete the actor from this persona and return the previous actor. The persona is retained. Unavailable for BAA. |

## Persona attributes

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `search_persona_attributes` | `customer_idn`, `persona_id`; optional: `query` | Search user persona attributes by IDN, group or title. Long values may be truncated. Unavailable for BAA. |
| `get_persona_attribute` | `customer_idn`, `persona_id`, `idn` | Read the full value and metadata of a user persona attribute. Unavailable for BAA. |
| `set_persona_attribute_value` | `customer_idn`, `persona_id`, `idn`, `value` | Overwrite the string value of an existing persona attribute and retain metadata. Unavailable for BAA. |
| `create_persona_attribute` | `customer_idn`, `persona_id`, `idn`, `value`; optional: `title`, `description`, `group`, `value_type`, `possible_values`, `is_hidden` | Create a user persona attribute with a string value. Duplicate IDNs are rejected. Unavailable for BAA. |
| `delete_persona_attribute` | `customer_idn`, `persona_id`, `idn` | Delete the persona attribute and return the previous representation. Unavailable for BAA. |

## Agent knowledge base (AKB)

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `list_knowledge_base_topics` | `customer_idn`, `project_idn`, `agent_idn`; optional: `labels`, `page`, `per` | List AKB topics newest first without text query. Label filters match case-insensitive substrings within a single label. Reduce `per` when results are truncated. |
| `search_knowledge_base_topics` | `customer_idn`, `project_idn`, `agent_idn`, `query`; optional: `search_fields`, `score_threshold`, `topics_number`, `labels` | Semantic search for agent topics only. Returns `fields_score` with best results first. All labels must match exactly. MCP defaults to four fields and five topics. `SearchFuzzyAkb` normally uses facts and one topic, and can include caller topics. |
| `create_knowledge_base_topic` | `customer_idn`, `project_idn`, `agent_idn`, `topic_summary`, `topic_facts`; optional: `topic_name`, `source`, `labels` | Create a summary and single-line facts. A matching name leaves the old topic in place. Available from the next conversation without publication. |
| `delete_knowledge_base_topic` | `customer_idn`, `project_idn`, `agent_idn`, `topic_id` | Permanently delete the exact agent topic by `topic_id` and return the previous topic. In-place topic updates are unavailable. |

## Runs

| Tool | Parameters | Contract and limits |
| --- | --- | --- |
| `get_run` | `run_id`; optional: `after_step` | Get creation status and ordered research steps. Pass the last `step_count` as `after_step` to fetch only new steps. The cursor defaults to `0`. |

## Verify results and handle errors

ISO 8601 dates without offsets are treated as UTC. Use the date range arguments for each conversation, logging or usage tool. Handle `isError: true` even when the transport succeeds. If a write has an uncertain result, read back state first. Recreating a customer, topic or webhook can produce a duplicate.

The server checks access on each request. Under `phi_restricted`, observe restrictions on conversations, logs and persona data. Creating, modifying or deleting outgoing webhooks is unavailable for customers with BAA. Keep third-party secrets and full conversations out of output. Use a returned connector `api_key` only for agreed setup under [operations.md](operations.md).

For procedures, see [creation](creation.md), [attributes and personas](configuration.md), [AKB](knowledge.md), [connectors and webhooks](integrations.md), [sandbox and diagnostics](testing.md), [versions](versions.md) and [statistics and units](quality-review.md). See [catalog limitations](missing-tools.md) for unavailable operations.

See [project attributes](project-attributes.md), [agent development](agent-development.md) and [external MCP for an agent](agent-mcp-client.md). Customer MCP has no direct interface for Workflow Builder, Studio, general-purpose `send_command`, standalone number purchases or provisioning a test customer on demand. The [Python action reference](python-actions.md) describes the separate runtime contracts.

Access requires an account link or eligible contractor record. Admin status alone does not grant access. Writes require an eligible role and account-link tools require your own link. Results around 70,000 characters can be truncated. Check `truncated`, list length and file completeness before overwriting. Agent AKB remains available for customers with BAA. Respect restrictions on caller data.
