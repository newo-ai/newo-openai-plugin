# Industry setup

Confirm services and exclusions first, then hours and time zone, prices and quotation policy, greeting, transfers and reports. See [configuration.md](configuration.md) for addresses and values and the [catalog](attribute-catalog.json) for standard scopes and types. Use the support workflow for an incorrect industry on an existing agent.

## Restaurants, cafés and bars

Verify dishes, drinks, menu and prices, kitchen hours versus venue hours, breaks and holidays, and owner-confirmed ingredients and allergens. Allergen claims require verified facts.

Use `project_attributes_restaurant_menu` and `project_attributes_restaurant_signature_dishes`. One attribute is usually sufficient for menus around 30,000 characters. This is a storage guideline. Put larger supplemental material in knowledge when needed. `project_attributes_restaurant_menu_overview` derives from the source menu.

Confirm the booking link, party size, deposit and cancellation policy and actual availability. Check the booking system before confirming a table. `project_attributes_restaurant_large_group_reservation_minimum_party_size`, `project_attributes_restaurant_large_group_reservation_questions` and `project_attributes_restaurant_large_group_reservation_email` handle large groups. Use the catalog's order, private_dining and catering fields for those cases.

## Catering

Collect date, time, location, party size, menu, restrictions, budget and a contact. Use `project_attributes_catering_email`, supplied booking links and current policies. Verify minimums, estimated pricing and availability with the owner. Distinguish estimates from invoices and verify handoff to staff.

## Dental and appointment-based services

Confirm services, providers, new and existing client requirements, hours by provider and location, pricing, accepted insurance, required questions and business-approved urgent routing. Use verified urgent instructions.

Dental fields include `project_attributes_dental_insurances`, `project_attributes_dental_appointment_schedule_questions_and_info`, `project_attributes_dental_appointment_url`, `project_attributes_dental_appointment_scenario_link_guidance_instruction` and `project_attributes_dental_appointment_check_in_notes`. Verify identity by the agreed procedure before reading or changing a record. Collect the minimum. If booking is unavailable, relay the request for staff confirmation.

## Beauty and wellness

Check service, duration, specialist, location, new and returning customers, cancellation and late arrival. Use `project_attributes_beauty_appointment_schedule_questions_and_info`, `project_attributes_beauty_appointment_url`, `project_attributes_beauty_appointment_scenario_link_guidance_instruction` and `project_attributes_beauty_appointment_check_in_notes`. Cancellation and rescheduling require identity verification and the existing appointment.

## Cleaning

Check services, exclusions, address and area, frequency, service area and surcharges. Use `project_attributes_cleaning_frequency`, `project_attributes_products_and_services_served_area`, `project_attributes_products_and_services_served_zip_codes`, `project_attributes_cleaning_appointment_url` and `project_attributes_cleaning_estimate_url`. Track estimates and confirmed jobs separately. Verify address type and service availability.

## Field services

Check services and exclusions, cities and postal codes, dispatch and intake hours, urgency, on-call line and estimate policy. Use `project_attributes_products_and_services_served_zip_codes` and catalog flags for background location and availability checks. Confirm the recognized address and quote arrival windows only from actual system data.

## Hotels and lodging

Confirm standard and group bookings, minimum party size, questions, check-in rules and where to send requests. Use `project_attributes_hospitality_reservation_url`, `project_attributes_hospitality_regular_reservation_questions`, `project_attributes_hospitality_regular_reservation_check_in_notes`, `project_attributes_hospitality_large_group_reservation_minimum_party_size`, `project_attributes_hospitality_large_group_reservation_questions` and `project_attributes_hospitality_large_group_reservation_email`. `bill_*` values are estimation inputs. Verify confirmed price or payment separately.

## Sales

Use `project_attributes_sales_pre_sale_questions`, `project_attributes_sales_qualifying_questions`, `project_attributes_sales_qualifying_criteria`, `project_attributes_sales_qualifying_criteria_fallback_action`, `project_attributes_sales_appointment_url` and `project_attributes_sales_after_appointment_notes`. Define qualification criteria and a fallback for unqualified requests. Use this workflow when it matches the business objective.

## General business and support

`project_business_type` identifies the business domain. `project_attributes_general_modality` supports `booking` for self-service appointment links, `transactional` for purchase or registration links and `followup` for staff handoff. Booking uses `project_attributes_general_appointment_url` and the catalog's general_appointment_scenario_link_guidance_instruction and general_appointment_check_in_notes fields. Verify booking or payment separately from link delivery.

## Multiple branches

A branch typically has its own customer. If an agent already covers several addresses, inspect `project_business_locations` and actual routing. Preserve individual hours and numbers. Check the destination, time zone and branch in every test. Start with a pilot and apply agreed changes one customer at a time.
