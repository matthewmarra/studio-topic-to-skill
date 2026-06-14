Name: travel-brochure-request

Description: Helps a user request a travel brochure or destination guide. Collects the user's
contact details and preferred delivery method (email or mail), then submits the brochure request.
Use when the user wants to request, order, or receive a travel brochure. Example phrases:
"request a travel brochure", "mail me a travel brochure", "order a travel brochure",
"I'd like a travel brochure", "get information about your destinations".

Instructions:

When this skill is activated:

1. Ask the user for their first and last name.
2. Ask for their telephone number.
3. Ask how they'd like to receive the brochure: Email or Mail.
4. If they choose Email, ask for their email address.
5. If they choose Mail, ask for their street address, then city, then state, then postal code.
6. Confirm the collected details, then submit the brochure request.

## Guidelines

- Only collect the fields needed for the chosen delivery method — never both branches.
- Validate each value against its type (valid email address, phone number, postal code) before submitting.
- If the delivery method is unclear, ask again rather than guessing.

## Examples

**Example 1: Email delivery**
- User request: "Can I get a travel brochure emailed to me?"
- Expected behavior: Collect name and phone number, confirm Email as the method, collect the email address, confirm the details, and submit the request.

**Example 2: Mail delivery**
- User request: "Mail me a travel brochure."
- Expected behavior: Collect name and phone number, confirm Mail as the method, collect street address, city, state, and postal code, confirm the details, and submit the request.

## Notes

- Delivery method is a fixed choice: Email or Mail.
