---
name: newo-delivery
description: Plan and verify Newo rollouts for customers and branches using settings, AKB, integrations and Customer MCP results. Use for requirements, acceptance, access handoff, quality reviews and draft support requests.
---

# Rollout and support

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md) and [rollout procedure](../../references/delivery.md). For support, numbering, porting, SIP or Assessor, read [support.md](../../references/support.md).

Identify the rollout stage and use the existing requirements. Keep small changes scoped to the request.

1. Identify the business goals and observable criteria, including required answers and actions, restrictions, escalation to a person and how to verify outcomes.
2. Map the criteria to attributes, AKB topics, connectors and webhooks. Use [setup.md](../../references/setup.md), [industry-guide.md](../../references/industry-guide.md) and the relevant skills.
3. Collect results using [testing.md](../../references/testing.md) and identify critical criteria still unverified. Test the sandbox and live line separately.
4. Record the customer's actual acceptance and the remaining limitations. Leave acceptance to the customer. Follow [telephony.md](../../references/telephony.md) for launch.
5. Identify who owns [quality reviews](../../references/quality-review.md) and rollback. Confirm automation separately from the agreed review schedule.
6. For branches, use `get_customers` and the agreed pilot. Read current attributes, AKB topics and connectors for each customer. Obtain explicit direction before copying numbers, secrets, personas or branch-specific rules.
7. For live acceptance, use the agreed test customer and scenarios authorized to have side effects. Update `not_run` only after an actual test. General integration metadata does not complete customer UAT. A missing test environment does not authorize creating a customer or changing access.
8. For access handoff, verify `list_customer_accounts`, roles and `is_you`. Customer MCP has no general invitation or role-change tool for an existing customer. Use `unlink_account` only on an explicit unlink request. Omit the email for the signed-in account. Unlinking another account requires the owner role.

Prepare requirements and templates in chat and keep business data out of the plugin. Prepare anonymized support drafts marked “not sent.” Ticket submission and tracking are unavailable.
