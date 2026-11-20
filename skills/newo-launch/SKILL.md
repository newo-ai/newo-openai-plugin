---
name: newo-launch
description: Configure available Newo phone connector settings, SIP and transfers, start or stop a channel, and prepare after-hours or daytime rollout. Use for live calls, external forwarding and number porting plans.
---

# Telephony and launch

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md), [telephony plan](../../references/telephony.md) and [connector reference](../../references/integrations.md).

1. Identify the business number, the agent number and staff lines. Read the agent number and links with `get_customers(customer_idn=...)`. Establish whether the request is about an existing line, SIP, an additional number, transfers or after-hours or daytime operation.
2. Check hours, time zone, reports, test mode, supported lines and transfer loops. For full readiness, use [health.md](../../references/health.md) and the acceptance steps in [delivery.md](../../references/delivery.md).
3. Read integration settings through `get_integrations(integration_idn)` and the connector through `get_connectors` with both IDNs. For SIP, send only changed fields in `update_connector.settings`. Start or stop with `run_connector(is_run=true/false)`. The first start of a phone or SMS connector may purchase a number. Prepare a change window and rollback steps.
4. For an existing agent, read `portal_url` and `chat_urls` from customer details and WebRTC settings from the specific voice connector. Automated calls or Assessor require a confirmed existing handler and event contract. There is no universal event signature. Agent code can purchase numbers, but this plugin provides neither a standalone purchase tool nor new code for this workflow. Have the line owner apply any external forwarding rule. Describe the exact rule and mark it “not applied.” For number provisioning, porting or network permissions outside connector settings, prepare an [operations request](../../references/support.md) marked “not sent.” Leave pricing, timing and inventory to the provider.
5. For an agreed after-hours rollout, verify the live line and initial results. Moving to overflow or AI-first requires a separate user decision.

Use `send_message` for sandbox checks. Verify audio, transfers and Caller ID on the phone channel. `send_agent_event` and `send_global_agent_event` may trigger real actions. Use known events in scope and send global events only on explicit request. Background monitoring or timed shutdown requires a configured mechanism.

Report applied settings, verification, external dependencies and available rollback steps.
