# Finish setup and configure scenarios

Follow [operations.md](operations.md) and use [configuration.md](configuration.md) for reading and writing values. The [attribute catalog](attribute-catalog.json) lists standard IDNs, scopes and types. Verify availability with `search_customer_attributes`, which looks up IDN, group and title case-insensitively. Read full values and metadata with `get_customer_attribute`. Confirm the contract for any nested JSON structure. Use [project-attributes.md](project-attributes.md) for version-level definitions.

## Business and hours

Confirm services, exclusions, hours, time zone and prices first. Then check the public name, address, website, phone, payment methods and escalation contacts. Use `project_business_story` for a short business description, `project_attributes_products_and_services` for the catalog, `project_attributes_products_and_services_not_offering` for exclusions, and `project_attributes_current_specials` for dated offers. Cancellation, late arrival and deposit rules belong in `project_attributes_terms_and_conditions`. Questions and answers belong in `project_attributes_faq`. Store prices with services and use `project_representative_agent_providing_prices_explicit_constraint` to control when they can be quoted.

Treat `project_business_industry` and the assigned `project_representative_agent_voice_phone_number` as provisioned. If the industry is wrong, prepare a support request. See [industry-guide.md](industry-guide.md).

`project_business_working_hours` uses text for every day, separate opening windows and exceptions. The following example shows the format.

```text
Working days: Monday, Tuesday, Wednesday, Thursday, Friday
Working hours:
1) Monday: open from 9AM to 1PM and from 2PM to 6PM
2) Tuesday: open from 9AM to 6PM
3) Wednesday: open from 9AM to 6PM
4) Thursday: open from 9AM to 6PM
5) Friday: open from 9AM to 5PM
6) Saturday: closed
7) Sunday: closed
```

Add confirmed holiday exceptions and distinguish office hours from around-the-clock intake. Use an IANA zone in `project_business_time_zone`. Test midnight, breaks and the closed message using supported test time or the actual time window.

## Agent, speech and languages

Use `project_representative_agent_name` and `project_representative_agent_title` for identity. Primary language, additional languages and multilingual mode have separate attributes. Use `project_business_language_restricted_list` for business language restrictions.

Configure voice and chat greetings separately. `project_representative_agent_voice_disclaimer_phrase` holds the AI or recording disclosure. Agree on the copy with the owner. The catalog includes separate recent and later returning-caller greetings. Use `project_business_name_phonetic` for pronunciation and make sure the greeting contains the business name it replaces.

`project_representative_agent_voice_speaking_style` and `project_representative_agent_chat_speaking_style` govern tone and pace. Use separate formatting and verbosity fields for layout and answer length. Use the relevant business attributes for menus, prices and booking rules. See [integrations.md](integrations.md) for voice, model and recognition.

## Staff reports

The following fields use `customer` scope in the standard deployment.

| Attribute | Purpose |
| --- | --- |
| `project_attributes_setting_session_ended_report_enabled` | Enables report sending. |
| `project_attributes_setting_session_ended_full_processing_enabled` | Enables full session-end processing required for reports. |
| `project_attributes_setting_session_ended_report_content` | Defines report type names, extracted fields, subject lines and SMS copy. |
| `project_attributes_setting_session_ended_report_criteria_template` | Defines conditions and recipients for those report types. |
| `project_attributes_setting_session_ended_report_include_transcript` | Includes the full transcript when required. Use the smallest necessary payload. |
| `project_business_manager_email` | Primary contact email. |

Define report types from the actual business requirements. The following example shows the content format.

```text
## Report Type: **CALLBACK REQUEST**
### Fields to be included in the report:
Reason for callback
Preferred callback time
Short summary
### Subject line format:
CALLBACK REQUEST: [6-word additional summary]
```

Use the same report type in its criteria.

```text
**Criteria for Email Report:**
- If it is a CALLBACK REQUEST, send an email to [[project_business_manager_email]]
```

Adapt the examples within the authorized scope. Use `[[attribute_idn]]` references only in fields that support substitution. Keep unresolved placeholders out of spoken text. Several email addresses can represent the primary recipient and CC. SMS goes to each specified number. Use approved recipients. Staff reporting, customer promotions and CRM writes need separate permission for their respective actions.

Verify a new session's end, selected report type, route and actual delivery. Test-mode simulation can pass without delivering an email or SMS. Configure both content and the agreed criteria.

## Workflow Builder and conversation scenarios

Customer MCP has no direct Workflow Builder operations for reading or writing canvas, intent types, scenarios or procedures. Supported changes use existing customer attributes and AKB topics as described in [configuration.md](configuration.md) and [knowledge.md](knowledge.md).

When necessary, read logic and subscriptions from accessible active files, including registry versions, using [versions.md](versions.md). Verify NAF's actual contract before promising export or a canvas rebuild after publication.

Describe the caller's intent, required data, action, confirmation, result and fallback. Use existing fields and confirmed formats. A new handler needs a separate development request. After saving a customer value, test a fresh session. If behavior depends on initialization, activate the relevant current version with a confirmed subscription. Verify the save and behavior separately.
