# Python Script Runner actions

Use this reference when writing or migrating a Python skill. It describes the supplied Script Runner snapshot. Confirm the deployed runner supports the action before publishing a change that depends on it. See the [development guide](agent-development.md) for project layout, examples and verification.

Call the typed `newo.<module>.<function>()` wrappers with their documented keyword arguments. The runner supplies execution context. The host action column maps older action names to Python and is also useful for reading logs. The Python registry controls which actions can run. Use `newo.actions.call(name, arguments)` only for a verified Python registry entry and its exact payload. Prefer the typed wrapper for encoding and decoding. Validate incoming values before passing them to an action.

Models use attributes, such as `actor.id` and `actor.external_id`. Named UUID parameters take `UUID` values. Keep these values typed within Python and convert them to strings at JSON or string-attribute boundaries. Check optional results for `None`, lists for emptiness and `JsonValue` shapes before indexing. `None` on a mutation means the wrapper has no result payload.

## Runtime helpers

| Helper | Use |
| --- | --- |
| `newo.arguments` | Read-only mapping of the current skill arguments. Use `.get("name", default)` and validate incoming types. |
| `newo.is_global()` | True when there is no current user actor. A persona may still be present. |
| `newo.skills.call(name, arguments=None)` | Call a registered skill and return its JSON result. Use this for an intentional skill boundary, such as distinct model metadata. Call project-local helpers with ordinary imports. |
| `newo.check_interrupt()` | Check for a pending runtime interrupt. Use a checkpoint in longer loops where interruption is safe. |
| `newo.not_interruptible()` | Defer cooperative interruption in a short critical section. A pending interrupt can surface on exit. The block provides no transaction or rollback. |

Source contracts are the public modules under Script Runner `src/newo`, the Python WASM `action_library/library.py` and the runtime contract tests. The migration notes and `actions.md` supplied with the project describe earlier stages.

## Execution context and identity

### `newo.actors`

Use actor UUIDs for routing and persona UUIDs for identity and persistent attributes. `get()` and `update()` default to the current user actor. `get_latest_message()` can resolve a different actor. Scope lists by the known connector and persona before selecting a result.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetActors` | `list(*, persona_id: UUID \| None=None, external_id: str \| None=None, integration_idn: str \| None=None, connector_idn: str \| None=None)` | `list[Actor]` |
| `GetActor` | `get(*, id: UUID \| None=None)` | `Actor` |
| `CreateActor` | `create(*, integration_idn: str, connector_idn: str, external_id: str, persona_id: UUID, timezone: str \| None=None)` | `UUID` |
| `GetLatestMessageActor` | `get_latest_message()` | `Actor` |
| `UpdateActor` | `update(*, name: str, id: UUID \| None=None)` | `None` |

### `newo.acts`

Read an act by UUID or use the act that triggered the execution. `create_message()` records a message act. Use `messages.send()` for channel delivery. Supply `from_person="agent"` or `"user"` and preserve the event payload.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetAct` | `get(*, id: UUID)` | `Act` |
| `GetTriggeredAct` | `get_triggered()` | `Act` |
| `CreateMessageAct` | `create_message(*, text: str, from_person: Literal['agent', 'user'], user_persona_id: UUID \| None=None, user_actor_id: UUID \| None=None, agent_id: UUID \| None=None, agent_persona_id: UUID \| None=None, **arguments: JsonValue)` | `None` |

### `newo.agents`

Use `get()` for agent metadata and `update()` for `title` or `description`. The default IDN is the current agent.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetAgent` | `get(*, idn: str \| None=None)` | `Agent` |
| `UpdateAgent` | `update(*, field: Literal['title', 'description'], value: str, idn: str \| None=None)` | `None` |

### `newo.personas`

Resolve a known persona by UUID or get the current agent persona. Creating a persona returns its UUID. Updates to a persona allow `title`, `description` and `name`.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetAgentPersona` | `get_current_agent()` | `AgentPersona` |
| `UpdateAgentPersona` | `update_current_agent(*, field: Literal['title', 'description', 'name'], value: str)` | `None` |
| `GetPersona` | `get(*, id: UUID)` | `Persona` |
| `CreatePersona` | `create(*, name: str)` | `UUID` |
| `UpdatePersona` | `update(*, id: UUID, field: Literal['title', 'description', 'name'], value: str)` | `None` |

### `newo.users`

Resolve the user persona from the current runtime context. `get_current().id` is a persona UUID. Update `name`, `title` or `description` only for the intended user.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetUser` | `get_current()` | `User` |
| `UpdateUser` | `update_current(*, field: Literal['name', 'title', 'description'], value: str)` | `None` |

### `newo.customers`

These calls address the current customer. Check for an absent company profile before reading its fields. Update only the intended fields of that profile.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetCustomer` | `get_current()` | `Customer` |
| `GetCustomerInfo` | `get_info()` | `CustomerInfo \| None` |
| `SetCustomerInfo` | `update_info(*, organization_name: str \| None=None, number_of_employees: str \| None=None, website: str \| None=None, phone_number: str \| None=None, country: str \| None=None, address: Address \| None=None, industry: list[str] \| None=None, self_description: str \| None=None, logo: str \| None=None, project_stage: str \| None=None, location_count: int \| None=None, comment: str \| None=None, currency: str \| None=None)` | `None` |

## Attributes, state and knowledge

### `newo.attributes`

Use the declared data scope. Customer and project calls resolve the current customer or project. Persona calls take an explicit UUID. Value actions use strings. Handle an empty value and an absent optional metadata object separately. Set or delete the narrowest field. `clear_customer()` deletes customer attributes in bulk.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetCustomerAttribute` | `get_customer(*, name: str)` | `str` |
| `GetCustomerAttributes` | `list_customer()` | `list[AttributeSummary]` |
| `SetCustomerAttribute` | `set_customer(*, name: str, value: str)` | `None` |
| `DeleteCustomerAttribute` | `delete_customer(*, name: str)` | `None` |
| `BulkDeleteCustomerAttributes` | `clear_customer()` | `None` |
| `GetCustomerMetadataAttribute` | `get_customer_metadata(*, name: str)` | `CustomerAttributeMetadata \| None` |
| `SetCustomerMetadataAttribute` | `set_customer_metadata(*, name: str, field: MetadataField, value: MetadataValue)` | `None` |
| `GetPersonaAttribute` | `get_persona(*, persona_id: UUID, name: str)` | `str` |
| `SetPersonaAttribute` | `set_persona(*, persona_id: UUID, name: str, value: str)` | `None` |
| `DeletePersonaAttribute` | `delete_persona(*, persona_id: UUID, name: str)` | `None` |
| `GetProjectAttribute` | `get_project(*, name: str)` | `str` |
| `SetProjectAttribute` | `set_project(*, name: str, value: str)` | `None` |
| `DeleteProjectAttribute` | `delete_project(*, name: str)` | `None` |
| `GetProjectMetadataAttribute` | `get_project_metadata(*, name: str)` | `ProjectAttributeMetadata \| None` |
| `SetProjectMetadataAttribute` | `set_project_metadata(*, name: str, field: MetadataField, value: MetadataValue)` | `None` |

### `newo.states`

Read or write an existing flow state using its declared IDN and scope. The runner needs flow context and an agent persona. User-scoped state also needs a user persona. See the development guide for migration and backfill.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetState` | `get(*, name: str)` | `str` |
| `SetState` | `set(*, name: str, value: str)` | `None` |

### `newo.akb`

Use semantic or fuzzy retrieval with the intended labels and persona filters. Inspect returned topics as models. Fuzzy search defaults to matching facts and summary when no search fields are provided. Use `set_manual()` for known facts and `update()` or `delete()` for verified topic UUIDs.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `SetAkb` | `set(*, context: str, labels: list[str] \| None=None, persona_id: UUID \| None=None)` | `None` |
| `SetManualAkb` | `set_manual(*, persona_id: UUID, summary: str, facts: list[str], name: str \| None=None, confidence: int \| None=None, source: str \| None=None, labels: list[str] \| None=None)` | `None` |
| `UpdateAkb` | `update(*, id: UUID, summary: str \| None=None, facts: list[str] \| None=None, name: str \| None=None, confidence: int \| None=None, source: str \| None=None, labels: list[str] \| None=None)` | `None` |
| `DeleteAkb` | `delete(*, ids: list[UUID])` | `None` |
| `SearchSemanticAkb` | `search_semantic(*, context: str \| None=None, query: str \| None=None, from_person: FromPerson='Both', labels: list[str] \| None=None, number_topics: int \| None=None, priority: Literal['searchFirst', 'filterFirst']='filterFirst', score_threshold: float \| None=None, filter_by_personas: list[UUID] \| None=None)` | `list[AKBTopic]` |
| `SearchFuzzyAkb` | `search_fuzzy(*, query: str, search_fields: list[Literal['facts', 'summary', 'name', 'source']] \| None=None, from_person: FromPerson='Both', number_topics: int=1, score_threshold: float \| None=None, labels: list[str] \| None=None, filter_by_personas: list[UUID] \| None=None)` | `list[AKBTopic]` |

### `newo.memory`

Read a text view of conversation memory. Bound the window and filter by actor, persona or session when isolating a conversation. Pass typed datetimes for explicit time bounds.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetMemory` | `get(*, from_person: Literal['User', 'Agent', 'Both']='Both', offset: int=0, count: int=10, max_len: int=200, as_english_text: bool=False, summarize: bool=False, from_datetime: datetime \| None=None, to_datetime: datetime \| None=None, filter_by_actors: list[UUID] \| None=None, filter_by_user_personas: list[UUID] \| None=None, session_id: UUID \| None=None)` | `str` |

### `newo.sessions`

Resolve the current session only in session context. `persist_session_acts()` invokes persistence for the supplied session UUID or the current session. The supported scope is `"all"`.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetSessionInfo` | `get_current()` | `Session` |
| `PersistSessionActs` | `persist_session_acts(*, session_id: UUID \| None=None, scope: Literal['all']='all')` | `PersistedSessionActs` |

### `newo.similarity`

Choose a strategy explicitly. Supported strategies are `hamming`, `levenshtein`, `semantic` and `symbols`. Preserve the threshold and test boundary values when translating.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `IsSimilar` | `is_similar(*, value1: str, value2: str, strategy: Literal['hamming', 'levenshtein', 'semantic', 'symbols'], threshold: float=0.4)` | `bool` |

## Events, messages and integrations

### `newo.events`

Send a system event from agent code. An explicit `global_=True` selects the global destination. Otherwise use the supplied actor UUIDs or the current user actor. Without either, the call fails. Choose a destination explicitly in reusable code and preserve the event IDN, target project and JSON payload.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `SendSystemEvent` | `send_system(*, event_idn: str, connector_idn: str='system', actor_ids: list[UUID] \| None=None, global_: bool=False, project_idn: str \| None=None, uninterruptible: bool=False, **arguments: JsonValue)` | `None` |

### `newo.messages`

Send a channel message from the current actor and session context. The default target is the current user actor. Even with explicit recipients, a current event, user actor, agent persona and session are required. A returned `None` does not establish recipient delivery.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `SendMessage` | `send(*, message: str, actor_ids: list[UUID] \| None=None, use_filter: bool=False, **arguments: JsonValue)` | `None` |

### `newo.commands`

Invoke a known connector command with its exact payload keys. The handler passes through the original event, actor, persona and session context. Verify the connector-specific contract and check its downstream result. Use the external MCP procedure for the actor-scoped `oauth2_start` command.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `SendCommand` | `send(*, command_idn: str, integration_idn: str, connector_idn: str, **arguments: JsonValue)` | `None` |

### `newo.connectors`

Inspect a connector by integration and connector IDNs. For `set(field="settings", value=...)`, pass only the intended string settings. The host applies them individually and a later failure may leave earlier fields changed. Read back after a write. Creation may start the connector. Deletion and lifecycle changes require scoped authorization.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetConnectorInfo` | `get(*, integration_idn: str, connector_idn: str)` | `Connector \| None` |
| `SetConnectorInfo` | `set(*, integration_idn: str, connector_idn: str, field: str, value: str \| dict[str, str])` | `None` |
| `CreateConnector` | `create(*, integration_idn: str, title: str, connector_idn: str, settings: dict[str, str], start: bool=True)` | `Connector \| None` |
| `DeleteConnector` | `delete(*, integration_idn: str, connector_idn: str)` | `None` |

### `newo.mcp`

Discover tool definitions by server IDN. Omit `tool_names` for unfiltered names, use `[]` for no names and provide a list for an allowlist. Use `query` when searching definitions. Read `.name` and `.input_schema`. Discovering a tool does not invoke it. Structured generation can expose a verified server and tool allowlist using the typed `MCPTool` model.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetMCPTools` | `get_tools(*, mcp_idn: str, tool_names: list[str] \| None=None, query: str \| None=None)` | `list[MCPToolDefinition]` |

### `newo.webhooks`

Use the exact `"incoming"` or `"outgoing"` type and the connector that owns the webhook. Verify permissions, payload contracts and delivery behavior before enabling triggers. Handle URLs and credentials as sensitive configuration.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetWebhook` | `get(*, webhook_idn: str, webhook_type: WebhookType)` | `Webhook \| None` |
| `GetWebhooks` | `list(*, webhook_type: WebhookType)` | `list[Webhook]` |
| `CreateWebhook` | `create(*, idn: str, connector_idn: str, webhook_type: WebhookType, description: str='', url: str \| None=None, allowed_ips: list[str] \| None=None, event_idns: list[str] \| None=None, command_idns: list[str] \| None=None, response_payload: JsonValue \| None=None)` | `Webhook \| None` |
| `UpdateWebhook` | `update(*, webhook_id: UUID, webhook_type: WebhookType, connector_idn: str \| None=None, description: str \| None=None, url: str \| None=None, allowed_ips: list[str] \| None=None, event_idns: list[str] \| None=None, command_idns: list[str] \| None=None, response_payload: JsonValue \| None=None)` | `None` |
| `DeleteWebhook` | `delete(*, webhook_id: UUID, webhook_type: WebhookType)` | `None` |

### `newo.time`

Use the runtime sleep for a delay that follows the runner interruption policy. Use Python date and time operations for local arithmetic and formatting.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `Sleep` | `sleep(*, duration: float, interruptible: bool=False)` | `None` |

## Generation

### `newo.generation`

All three functions require explicit `list[GenerationBlock]`. Plain generation returns text and uses the skill model settings. Streaming sends to the specified actor destination and also returns text. Structured generation returns `GenStructuredResponse` and supports model overrides and tool specifications. Match generation mode, prompt blocks, sampling, schema validation and delivery to the original behavior.

For streaming, `interrupt_mode="interruptWindow"` requires `interrupt_window`. `send_to="currentActor"` requires a current actor, and `send_to="actors"` needs a nonempty list of target UUIDs in `actor_ids`. The SDK exposes `latestMessageActor`, which this snapshot's host rejects as unimplemented. Streaming mode values apply to generation. The subscription's separate `interrupt_mode` accepts `interrupt`, `queue` or `cancel`.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `Gen` | `generate(*, blocks: list[GenerationBlock], temperature: float=0.75, stop: list[str] \| None=None, max_tokens: int \| None=None, top_p: float=1.0, skip_filter: bool=False, skip_checks: bool=False, max_retries: int \| None=None, json_schema: dict[str, JsonValue] \| None=None, validate_schema: bool=False, disable_continuation: bool=False, thinking_budget: int \| None=None, reasoning_effort: ReasoningEffort \| None=None)` | `str` |
| `GenStream` | `generate_stream(*, blocks: list[GenerationBlock], temperature: float=0.75, stop: list[str] \| None=None, max_tokens: int \| None=None, top_p: float=1.0, skip_filter: bool=False, skip_checks: bool=False, max_retries: int \| None=None, interrupt_mode: Literal['interruptWindow', 'interrupt', 'none']='interrupt', interrupt_window: float \| None=None, send_to: Literal['actors', 'currentActor', 'latestMessageActor']='currentActor', actor_ids: list[UUID] \| None=None, disable_continuation: bool=True, thinking_budget: int \| None=None, reasoning_effort: ReasoningEffort \| None=None, **arguments: JsonValue)` | `str` |
| `GenStructured` | `generate_structured(*, blocks: list[GenerationBlock], provider_idn: str \| None=None, model_idn: str \| None=None, stop: list[str] \| None=None, temperature: float=0.75, max_tokens: int \| None=None, top_p: float=1.0, max_retries: int \| None=None, json_schema: dict[str, JsonValue] \| None=None, mcp_idns: list[str] \| None=None, validate_schema: bool=False, thinking_budget: int \| None=None, reasoning_effort: ReasoningEffort \| None=None, tools: list[Tool] \| None=None, tool_choice: Literal['auto', 'required', 'none']='auto')` | `GenStructuredResponse` |

## Projects, registries and telephony

### `newo.projects`

Resolve the current project or list this customer’s projects. Creation and publication are runtime mutations with registry semantics. Use Customer MCP version tools for drafting and applying edits from Codex.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetProject` | `get_current()` | `Project` |
| `GetProjects` | `list()` | `list[Project]` |
| `CreateProject` | `create(*, idn: str, title: str, registry_idn: str, registry_item_idn: str, is_auto_update_enabled: bool, force_update: bool, description: str \| None=None, readme: str \| None=None, registry_item_version: str \| None=None)` | `ProjectMutation` |
| `PublishProject` | `publish(*, idn: str)` | `ProjectMutation` |

### `newo.registries`

List registries or inspect an exact registry/module IDN. Check the customer’s current attachment with Customer MCP before changing a project.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `GetRegistries` | `list()` | `list[Registry]` |
| `GetRegistryModules` | `list_modules(*, registry_idn: str)` | `list[RegistryModule]` |
| `GetRegistryModule` | `get_module(*, registry_idn: str, idn: str)` | `RegistryModuleDetail` |

### `newo.phone_numbers`

Validate the requested number or inspect operation results using typed models. Buying, linking, pooling and releasing numbers affect billing, ownership or routing. Execute these only with the corresponding authorization and verify the provider result.

| Host action | Python signature | Returns |
| --- | --- | --- |
| `BuyPhoneNumber` | `buy(*, country_code: str, area_code: str \| None=None, fallback_area_code: str \| None=None, is_voice: bool=True, is_sms: bool=True, providers: list[str] \| None=None, group: str \| None=None)` | `PhoneNumber \| None` |
| `AddPhoneNumberToPool` | `add_to_pool(*, phone_number: str)` | `None` |
| `AddPhoneNumberToMessagingPool` | `add_to_messaging_pool(*, phone_number: str, async_: bool=False)` | `None` |
| `LinkPhoneNumber` | `link(*, phone_number_id: UUID, connector_idn: str, integration_idn: str)` | `None` |
| `ReleasePhoneNumber` | `release(*, phone_number_id: UUID)` | `PhoneNumberRelease` |
| `ValidatePhoneNumber` | `validate(*, phone_number: str, use_cache: bool=True)` | `PhoneNumberValidation` |
