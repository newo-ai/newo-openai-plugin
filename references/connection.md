# Connect Customer MCP

The plugin uses the `newo` server configured in `.mcp.json` at the package root. Perform all Newo operations through this Customer MCP. The bundled instructions are self-contained.

## Sign in

The host handles OAuth. The current configuration uses a preregistered public client with `mode: provided`, `tokenEndpointAuthMethod: none` and scopes `openid offline_access`. The authentication method applies to the application's secret. The user still signs in to Newo. Keep passwords, cookies, tokens and secrets out of chat and tool arguments.

For a first connection, choose Connect and sign in on the page opened by the host. The application administrator must allow the exact callback displayed for that connection. A different local client's callback does not apply. If the redirect fails, have the allowlist corrected with the exact callback from the host and keep OAuth enabled.

## Verify access

1. Find the `newo` tools in the host's catalog. Names can be prefixed. Read the current descriptions and schemas.
2. List accessible businesses with `get_customers`. If several match, clarify which one and read details with `get_customers(customer_idn=...)`. Multiple `chat_urls` are valid for multiple `newo_chat` connectors. Preserve the server's link format. If needed, list `get_connectors(integration_idn="newo_chat")` and verify chat availability independently of business selection.
3. Check tools required by the request against the [tool reference](mcp-tools.md). See [catalog limitations](missing-tools.md) and use the exact names from the current schema.
4. Report the specific limitation when the catalog, authorization or required operation is unavailable. Use Customer MCP as the sole route.

A user account and a business customer are separate entities. Most operations require a customer link or contractor record. Writes require an eligible role on that link. Admin status alone does not grant a link. `list_customer_accounts` and `unlink_account` require a direct link from the caller. A contractor record alone is insufficient. The server checks customer access on each request. Authorization to connect does not extend to changes across all businesses.

## Errors

| Signal | Action |
| --- | --- |
| 401 or sign-in required | Use the host's reconnection or authorization refresh flow. |
| `forbidden` | Verify the current customer's link and role. |
| `phi_restricted` | Observe the restrictions on conversations and logs. Reconnecting does not change them. |
| `invalid_input` | Correct the rejected field according to the tool schema and preserve the request's target. |
| `not_found` | Verify the customer and IDs before taking further action. |
| Write transport timeout | Check actual state and results before deciding whether to retry. |

Catalog discovery confirmed 57 tools. General integration metadata has been checked. Verify access and scenarios for each customer separately.

Use `list_customer_accounts` to inspect accounts. On an explicit unlink request, pass the target email to `unlink_account`. Omit email to unlink the current account. Unlinking another account requires the owner role.

Local Codex uses the package's public OAuth client and a callback on localhost, port 8765, path /callback. When adding the server locally, set both the callback address and listener port. On subsequent sign-in, retain the port in server settings or pass the override again. See the repository README for the command. For other hosts, allow the exact callback their own connection interface displays.
