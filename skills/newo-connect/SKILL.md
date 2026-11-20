---
name: newo-connect
description: Connect Newo through OAuth, select an accessible business, and retrieve agent links, phone numbers and linked accounts. Use for initial connection, access errors or an explicit request to unlink an account.
---

# Connect Newo and verify business access

Use the user's language, defaulting to English. Read the [operating rules](../../references/operations.md) and [connection guide](../../references/connection.md). Perform Newo operations through Customer MCP `newo`.

1. Find the server's tools in the host catalog and read the current schemas. Use the built-in OAuth connection when needed. Keep passwords, cookies, tokens and secrets out of chat.
2. Compare the available tools with the [catalog and contracts](../../references/mcp-tools.md). Verify access to the selected customer's operations separately.
3. Find the business using `get_customers` with search, filters or pagination. Clarify the choice when multiple businesses match. Read details with `get_customers(customer_idn=...)`. This mode ignores pagination, search and filters and returns the phone number, chat links, balance and Portal dashboard link. Multiple `chat_urls` are expected when several `newo_chat` connectors exist. Show returned links as provided and verify chat availability separately. Creating a business does not require selecting an existing one.
4. Check individual access using `list_customer_accounts` and inspect email, name, role and `is_you`. On an explicit unlink request, pass the exact email to `unlink_account`. Omitting the email unlinks the signed-in account. Unlinking another account requires the owner role.
5. Access requires a customer linked to the account or an eligible contractor record. Admin status alone grants no customer access. Listing linked accounts requires a link to the customer. Report the selected business and supported actions. Customer MCP has no general invitation tool or tool for issuing test customers on demand.

For a 401, use the built-in reconnection flow. Reconnecting does not resolve `forbidden` or `phi_restricted`. Use Customer MCP as the sole route for these actions.

For creation or setup, continue with [newo-agent-builder](../newo-agent-builder/SKILL.md).
