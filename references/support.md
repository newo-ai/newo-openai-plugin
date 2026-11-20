# Support and operations requests

Prepare anonymized text in chat and mark it “draft, not sent.” The plugin supports drafting only. Ticket submission, status lookup and tracking are unavailable. Keep requests in the user-approved support workflow.

## Report a defect

Include the customer, channel, UTC time and original time zone, session or event ID, expected and observed behavior, minimal reproduction steps, frequency, impact, checked settings and anonymized log observations. Mark hypotheses and confidence separately. Omit tokens, raw conversations and customer personal data. Use permitted metadata when access to the content is restricted.

## Draft an operations request

Collect the information relevant to the request.

- Number provisioning. Customer, country, type, locality or area code, voice or SMS, demo or production, acceptable cost and desired date. Keep identity and business documents out of chat.
- Porting. Number and country, confirmed authority over the line, responsible contact and preferred window. Treat porting as pending until confirmed.
- SIP or RTP. Provider, signaling and RTP addresses, transport, observed fault, time and test ID. Omit passwords.
- Registration check. Connector, time, symptoms and checks already performed. Use verified registration evidence in addition to running status.
- Assessor. Generalized scenarios, channel, run count, acceptable cost and outcome criteria. Use `send_agent_event` when the established contract requires an actor event and the user requests it. Use `send_global_agent_event` on explicit request for a global event.
- Industry change, platform error or code need. Business requirement, configuration gap, reproduction steps and impact. Prepare a request for the relevant owner.

Provide draft text and a list of missing details. Ticket IDs, status and outcomes require a supported external process. After the user confirms a change, check its observable result in Customer MCP, such as a provisioned number or a new test conversation.

## Propose an improvement or share findings

Describe the original goal, conditions, observed cause, working setting, verification and limits. Remove customer names, data and secrets from material meant for external readers. Return a draft for the user to review.
