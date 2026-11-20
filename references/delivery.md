# Rollout, acceptance and support

Use the full procedure for a new rollout and the relevant shorter procedure for a single fix. Follow [operations.md](operations.md) throughout. Route explicitly requested agent code changes to [newo-develop](../skills/newo-develop/SKILL.md).

## Requirements

Use known inputs and confirm missing details about the business and branches, industry, hours and time zone, languages and channels, services and prices, restrictions, escalation, systems and access, test environment, knowledge sources, report recipients, acceptance owner and desired launch date. Have the owner confirm sensitive-data and messaging requirements. A flag alone does not establish compliance.

Each criterion identifies observable behavior with an ID, `critical`, `important` or `nice-to-have` priority, condition, expected result, forbidden outcome, channel and verification method.

| ID | Criterion | Verification |
| --- | --- | --- |
| AC-001 | After hours, says the office is closed and takes a message. | Start a fresh conversation during a confirmed closed period and verify report receipt. |
| AC-002 | Books only available times after caller confirmation. | Check the transcript and test-system result. Repeating “yes” must not duplicate a booking. |
| AC-003 | Transfers on the agreed condition to a staffed line. | Place a live call and verify that a staff member answers. |

Identify unsupported criteria before committing to delivery. Agree on requirements, access ownership, test data, exclusions and acceptance steps with the customer. Record only actual agreements and sign-offs.

## Configure and verify

For each criterion, record the corresponding attribute, knowledge topic, integration or scenario, the customer, the old and new values without secrets, how to apply the change and how to roll it back. Run the health check, followed by a quick check for each critical case. Then run the agreed text, voice and integration tests in [testing.md](testing.md).

Record the criterion ID, environment and channel, attempts, expected result, actual observation, session or system result, and `passed`, `failed`, `not_tested` or `blocked`. After a fix, run the agreed regression cases. Close critical verification gaps before marking the rollout ready.

## Customer acceptance

The customer decides whether the agreed criteria are met. For each round, record the date, checked IDs, failures, decision and causes across requirements, configuration, tests, integrations, telephony and platform. Obtain agreement again for revised criteria. Base the acceptance status on observed results and the customer's decision.

Follow [telephony.md](telephony.md) for live launch. Verify the production environment, customer destinations and numbers, test-mode settings, routing, rollback and owner. Authorization for a live test appointment and its subsequent cancellation are separate.

## Handoff and operations

Summarize what the customer accepted, remaining limits, launch mode, how to restore routing, who reviews conversations and when, who maintains content and where to report issues. Use [quality-review.md](quality-review.md). For repeatable cases, retain an anonymized description and criterion in the agreed workspace. Keep customer data out of the plugin.

Draft requirements, acceptance criteria, change logs, test reports, sign-off records and weekly reviews in chat. Saving objects to Newo requires a supported tool. Support tickets can be drafted for the user. Submission and tracking are unavailable.

## Partners and multiple branches

Confirm the exact customers and exceptions. Start with a pilot. Recheck identity, current values, branch hours, numbers and sources before changing each customer. One customer's authorization does not extend to other customers. Report completed, partial, skipped and unverified changes per branch. On partial failure, preserve successful writes and retry only the affected work.
