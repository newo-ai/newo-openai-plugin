# Create a customer and agent

Perform all actions through Customer MCP. Keep the latest accessible customer IDN before creating another customer. After `create_customer`, verify the result and readiness using [testing.md](testing.md).

## Inputs

Use the business website or Google Maps link supplied by the user as `source`. Ask for it if missing. Server-side Agent Creator researches the business. `industry` is an optional hint. The Creator determines the industry when omitted.

Omit `agent_creator_idn` to use the account's default Creator. Pass only a specific Creator named in the current instructions. Keep overrides out of the plugin. If the requested Creator fails, report its failure without substituting another. The signed-in account supplies the owner's name and email.

Pass `referral`, `account_managers`, `technical_owners`, `linked_emails`, `external_owner_account_id`, `hash_external_account_id`, `partner_idn`, `partner_member_idn`, `external_customer_id` and `promo_code` only when explicitly supplied. Additional owners gain access, so use only the supplied account data.

## Creation and retries

Each call creates another customer. Standard creation purchases a phone number. Check authorization for creation and related costs. Demo mode alone does not establish that a purchase is skipped. Respect any user instruction that rules out a number purchase.

Before calling, read `get_customers(page=1, per=1)` without filters and retain only `items[0].idn` in the current context. Record a successful empty result as the baseline. A failed read leaves the baseline unknown.

Call `create_customer` once. It can take about three minutes to respond. Keep the `run_id`, `state` and returned results. While `running`, poll the same UUID with `get_run(run_id)` at reasonable intervals and keep the user updated.

| State | Action |
| --- | --- |
| `running` | Poll the same run and avoid a duplicate creation. |
| `created` | Verify `customer_idn` with `get_customers` and share the available number and `portal_url`. |
| `error` | Read the error. If creation could have started, check the latest customer IDN. |
| `unknown` or lost response | Check the latest customer IDN to recover the outcome. |

If the response contains `customer_idn`, read that customer's details first. Otherwise call `get_customers(page=1, per=1)` and compare the first IDN with the baseline. Treat a new IDN as a candidate. Use `get_customers(customer_idn=...)` to verify the business name and other available details against the request. Proceed when the match is confirmed. If the IDN is unchanged, the baseline is unknown or the match is ambiguous, the outcome remains uncertain. Resolve it before retrying.

`get_run` takes only the run ID returned by `create_customer`. Use the agreed Creator.

## Empty customer on explicit request

Use `empty: true` only when the user explicitly requests an empty customer. Pass no other arguments. A `created` result includes `customer_idn`, `portal_url` and a default project. There is no agent, phone number or `run_id` to poll.

## Verify the result

Report only returned values. Detailed `get_customers(customer_idn=...)` includes the available number, chat links, balance and Portal and billing links. Treat null as unknown. Multiple `chat_urls` are valid for multiple `newo_chat` connectors. Verify readiness after creation through customer attributes, a fresh sandbox response and, when requested, a real call. The sandbox cannot verify audio. If a test is unavailable, report that the customer was created and the test is pending.
