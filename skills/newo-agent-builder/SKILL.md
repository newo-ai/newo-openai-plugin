---
name: newo-agent-builder
description: Create a Newo business and AI agent from a website or Google Maps link, track research and provisioning, then configure and launch the agent. Use a specialized skill for changes to an existing agent.
---

# Create an agent and choose the next step

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md). Use Customer MCP for all Newo operations. For an explicit request to change a project attribute, edit a draft using [project-attributes.md](../../references/project-attributes.md). For agent code or flow changes, use [newo-develop](../newo-develop/SKILL.md).

1. Follow [creation.md](../../references/creation.md). For standard `create_customer`, pass the website or Google Maps link provided by the user as `source`. The server researches the business. Each call creates a new customer and purchases a number in standard mode.
2. Omit `agent_creator_idn` to use the account's default Creator. Pass an override only when the current instructions name a specific Creator. The signed-in account supplies the owner's name and email. Use partner fields only when provided explicitly.
3. If the user explicitly requests an empty customer, pass only `empty: true`. The result contains a default project and omits the agent, phone number and `run_id`.
4. Before creating the customer, retain the latest customer IDN from `get_customers(page=1, per=1)`. Call `create_customer` once. While `running`, poll the same run with `get_run`, passing the last `step_count` as `after_step`, and share new research steps. If the response is lost or inconclusive, compare the latest IDN with the baseline using the [recovery procedure](../../references/creation.md#creation-and-retries). Verify the candidate before proceeding.
5. Return the available agent number and `portal_url`. Verify readiness separately using customer attributes and a new sandbox conversation as described in [testing.md](../../references/testing.md).

| Request | Skill |
| --- | --- |
| OAuth, business selection, accounts and access | [newo-connect](../newo-connect/SKILL.md) |
| Business and persona attributes, health, industries and scenarios | [newo-configure](../newo-configure/SKILL.md) |
| Agent code, event subscriptions, runtime actions and Python migration | [newo-develop](../newo-develop/SKILL.md) |
| AKB topics, fact search, imports and incorrect answers | [newo-knowledge](../newo-knowledge/SKILL.md) |
| Connectors, webhooks, voice, STT, calendars and CRM | [newo-integrations](../newo-integrations/SKILL.md) |
| External MCP servers and OAuth2 | [newo-external-mcp](../newo-external-mcp/SKILL.md) |
| Numbers, SIP, transfers, after-hours and daytime rollout | [newo-launch](../newo-launch/SKILL.md) |
| Sandbox, conversations, logs, statistics and units | [newo-quality](../newo-quality/SKILL.md) |
| Requirements, acceptance, support and branches | [newo-delivery](../newo-delivery/SKILL.md) |

Load the references relevant to the request. Consult the [tool reference](../../references/mcp-tools.md), [capabilities](../../references/capabilities.md), [catalog limitations](../../references/missing-tools.md) and [NAF platform model](../../references/naf.md) as needed.
