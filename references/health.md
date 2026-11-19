# Agent health check

Review settings and observed behavior. This procedure produces findings rather than an official quality score. Follow [operations.md](operations.md). Make changes only when requested.

## Gather enough evidence

Read the selected customer with `get_customers(customer_idn=...)`. Find fields with `search_customer_attributes` and the [catalog](attribute-catalog.json). Read full values and metadata with `get_customer_attribute`, especially when `truncated`. Check connectors and active versions. Report the scope actually checked. Verify the schema of any new JSON scenario. Distinguish missing, empty and inaccessible fields. Persona data is restricted under a signed BAA. Read project attributes via version files.

## Critical findings

- Routing loops where escalation or bypass points to the agent number or a public number that forwards to it. Compare normalized numbers and the known route.
- Transfers enabled with no staffed line in the selected window, missing destinations, or the wrong `full_number` or `short_code` format.
- Voice enabled with a missing greeting or spoken template placeholders.
- Booking or availability enabled without a destination, required schema or working connection.
- A primary language on the restricted list or inconsistent business names in greetings.
- Global test mode enabled for live service. Verify test number lists and the session's `session_test_mode` when persona access permits.
- A scenario points to an unavailable action, loops indefinitely or confirms an appointment before a system result.

## Business fit

- Check industry fit. Use the support workflow for an incorrect industry.
- Verify daily opening hours, breaks, exceptions and IANA time zone. Keep 24/7 intake separate from office hours.
- Verify actual services, exclusions, prices, quotation policy and payment methods.
- Confirm the calendar, appointment duration, staff and locations. Google requires a working connector and the correct calendar owner.
- Check industry fields using [industry-guide.md](industry-guide.md).
- `project_attributes_setting_hipaa_mode_enabled` configures supported sensitive-data behavior. Verify owner requirements before changing it. Server access restrictions apply independently of the flag, which alone establishes no compliance result.

## Content and reporting quality

Look for `N/A`, `TBD`, string `null`, other businesses' text and contradictions across FAQs, catalogs and hours. Keep limited-time offers distinct from permanent prices. Use style for delivery, the business story for description, terms for restrictions and FAQ for answers. Move large documents to the appropriate knowledge source.

Check matching report content and criteria types, valid destinations, full session-end processing, the minimum required personal data and test-mode effects. Verify scenario destinations and trigger conditions independently.

## Report findings

For each finding, report severity, observed evidence, user impact, suspected cause and confidence, targeted change and verification method. Lead with critical issues, then significant issues and improvements. Mark unread fields as unverified.

After an authorized fix, read back values, apply initialization as needed, test again and repeat affected health checks. Full acceptance needs the agreed regression suite from [testing.md](testing.md). Use [support.md](support.md) for code and platform issues.
