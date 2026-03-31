# Medical-Spa-AI-Chatbot-Using-GHL-N8N

## Retell Chatbot Checking Availability

A lightweight **n8n appointment availability checking workflow** that receives a requested appointment datetime through a **Webhook**, checks available calendar slots from **GoHighLevel (GHL)**, compares the requested time against actual availability, and returns either:

- the requested slot if available
- or the nearest available alternatives

This workflow is designed for **Retell AI voice agents**, chatbots, booking assistants, and appointment automation systems.

---

## Features

- Receives booking requests through a **Webhook**
- Accepts appointment datetime input from an external assistant
- Checks live availability from **GoHighLevel calendar**
- Supports date window search using:
  - start date
  - end date
- Detects whether the requested slot is available
- Returns:
  - exact availability confirmation
  - or closest available slots
- Designed for **Retell AI / chatbot booking assistants**
- Fast and simple webhook-based availability checker

---

## Workflow Overview

This workflow follows the process below:

1. A chatbot or voice AI sends a requested appointment datetime
2. The workflow receives the request via **Webhook**
3. It normalizes the year and date formatting
4. Generates:
   - start timestamp
   - end timestamp
5. Calls **GoHighLevel free slots API**
6. Retrieves all available slots for that date window
7. Compares the requested appointment time against available slots
8. Returns:
   - confirmation if available
   - or nearest available alternatives

---

## Main Use Case

This workflow is built for **AI booking assistants** that need to quickly check whether a requested appointment time is available before confirming it to the user.

This is especially useful for:

- Retell AI voice bots
- AI receptionists
- appointment setter chatbots
- GHL-based booking systems
- AI SDR appointment flows

---

## Nodes Used

## 1. Input Layer

### 1. Webhook
The workflow starts with a **Webhook** node.

### Webhook configuration:
- **Method:** `POST`
- **Response Mode:** `responseNode`

This allows an external chatbot, Retell voice agent, or automation system to send booking requests directly into n8n.

---

## 2. Date Normalization Layer

### 2. Code in JavaScript1
This node updates the incoming appointment year to the **current system year**.

### What it does:
- reads:
```json
body.args.appointment_datetime
```

- converts the incoming datetime into a JavaScript date
- replaces the year with the current year
- writes the updated value back into the request payload

### Why this matters
This helps fix situations where an external bot or user request might provide a date with the wrong year.

### Example behavior:
If the incoming request contains an outdated year, the workflow corrects it automatically before checking availability.

---

## 3. Start / End Date Preparation Layer

### 3. Get Start Date
This node converts the requested appointment date into a **Unix timestamp**.

### Input used:
```json
body.args.appointment_date
```

### Output:
- formatted timestamp in milliseconds

This becomes the **startDate** for the availability request.

---

### 4. Get Next Date
This node adds **1 day** to the formatted appointment date.

This creates the upper bound for the slot search range.

### Purpose:
- search availability for the requested day window
- limit API search to a manageable range

---

### 5. Get End Date
This node formats the calculated next-day date into a **Unix timestamp**.

### Output:
- `endDate`

This becomes the **endDate** for the availability request.

---

## 4. Availability Check Layer

### 6. Get free slots from ghl
This node calls the **GoHighLevel free slots API** to fetch available booking slots.

### API endpoint used:
```text
https://services.leadconnectorhq.com/calendars/ecr2X6XjI6k7ljIiSd96/free-slots
```

### Query parameters sent:
- `startDate`
- `endDate`

### Headers include:
- `Accept: application/json`
- `Authorization: Bearer ...`
- `Version: 2021-04-15`
- `timezone: America/New_York`

This returns all free booking slots available for the requested day range.

---

## 5. Slot Matching Logic Layer

### 7. Code in JavaScript
This is the **core availability decision engine** of the workflow.

It compares the requested datetime against the free slots returned by GHL.

### What it does:
- flattens all returned slot arrays into one list
- checks if the exact requested datetime exists
- if exact match exists:
  - returns a human-readable availability confirmation
- if exact match does not exist:
  - finds the nearest available slot before
  - finds the nearest available slot after

---

## Availability Decision Logic

### Case 1: Requested Slot Exists
If the requested appointment time is found in the available slot list:

### Example output:
```text
Tuesday at 2:30 PM is available.
```

This allows the chatbot or voice assistant to confirm the slot instantly.

---

### Case 2: Requested Slot Is Not Available
If the requested slot is not found:

The workflow returns nearby alternatives such as:

- closest slot before
- closest slot after

### Example behavior:
```json
{
  "closestBefore": "2026-04-03T17:00:00.000Z",
  "closestAfter": "2026-04-03T18:00:00.000Z"
}
```

This allows the AI assistant to suggest nearby times instead of failing the booking request.

---

## 6. Response Layer

### 8. Respond to Webhook
This node sends the final result back to the original caller.

The response is returned directly from the webhook flow.

This makes the workflow ideal for real-time integrations such as:

- Retell AI function calling
- chatbot tool usage
- AI receptionist booking logic
- custom scheduling assistants

---

## Example Input Payload

This workflow expects a webhook payload containing a requested appointment datetime.

### Example request:
```json
{
  "body": {
    "args": {
      "appointment_date": "2026-04-03T14:30:00.000Z",
      "appointment_datetime": "2025-04-03T14:30:00.000Z"
    }
  }
}
```

---

## Example Workflow Behavior

### Scenario 1: Exact Slot Available
Input:
```json
{
  "appointment_date": "2026-04-03T14:30:00.000Z"
}
```

### Possible response:
```text
Friday at 7:30 AM is available.
```

---

### Scenario 2: Exact Slot Not Available
Input:
```json
{
  "appointment_date": "2026-04-03T14:30:00.000Z"
}
```

### Possible response:
```json
{
  "closestBefore": "2026-04-03T13:00:00.000Z",
  "closestAfter": "2026-04-03T15:00:00.000Z"
}
```

This makes the workflow useful for intelligent scheduling conversations.

---

## Timezone Handling

This workflow uses multiple timezone assumptions.

### GHL request timezone:
```text
America/New_York
```

### Slot formatting in JavaScript:
```text
America/Los_Angeles
```

### Important note
This means the workflow currently has **mixed timezone logic** and may require adjustment depending on your business timezone or assistant deployment region.

---

## Requirements

To run this workflow successfully, you need:

- **n8n**
- a **GoHighLevel account**
- access to a valid **GHL calendar**
- a valid **LeadConnector / GHL API token**
- a chatbot or voice assistant capable of calling this webhook

---

## External Services Used

This workflow integrates with:

- **GoHighLevel / LeadConnector**
- **Retell AI** or any external chatbot via webhook

---

## Setup Instructions

1. Import the JSON workflow into n8n
2. Copy your webhook URL from the **Webhook** node
3. Connect your chatbot or Retell tool call to that webhook
4. Replace the hardcoded **GHL calendar ID** with your own
5. Replace the hardcoded **Authorization Bearer token** with your own
6. Confirm your target calendar has active availability
7. Activate the workflow
8. Send a test appointment request
9. Verify that:
   - requested dates are parsed correctly
   - GHL free slots are returned
   - exact availability is detected properly
   - fallback slots are returned if needed

---

## Important Notes

- The workflow is currently **active**
- It is designed as a **real-time availability checker**
- Best used as a helper tool inside a larger booking assistant system
- The response behavior is currently minimal and lightweight
- The workflow is optimized for **fast slot lookup**

---

## Limitations

At the moment, this workflow has a few limitations:

- No actual appointment booking step
- No timezone normalization consistency
- No natural language date parsing
- No support for multi-day search ranges
- No validation for malformed webhook payloads
- No user-friendly formatting for alternate slots
- Current response node may only return one field depending on branch behavior

---

## Suggested Improvements

You can improve this workflow further by adding:

- automatic booking after availability confirmation
- timezone auto-detection
- natural language date parsing
- formatted alternate slot suggestions
- support for multi-day fallback search
- CRM contact creation
- appointment confirmation SMS/email
- chatbot-friendly structured JSON response
- retry logic for API failures
- support for multiple calendars

---

## Recommended Workflow Pairing

This workflow works very well when paired with:

- appointment booking workflows
- Retell AI voice agent workflows
- AI receptionist systems
- GHL lead intake workflows
- outbound calling assistants

A complete system could work like this:

1. AI chatbot asks for preferred appointment time
2. This workflow checks availability
3. AI confirms or suggests alternatives
4. another workflow books the appointment
5. confirmation message is sent automatically

This creates a complete **AI booking assistant system**.

---

## Example Real-World Use Cases

This workflow is ideal for:

- AI receptionists
- medical office scheduling bots
- sales appointment setters
- coaching / consultation booking assistants
- home service call center bots
- GHL-based scheduling automations

---

## Author

Built as an **n8n availability-checking workflow** for Retell AI, booking assistants, and GoHighLevel calendar automation systems.

---

## License

You can add your preferred license here, such as:

- MIT
- Apache-2.0
- Proprietary


# Retell Chatbot Training Book Appointment

A practical **n8n AI appointment booking workflow** built for **Retell AI chatbots / voice agents** that can search for an existing contact in **GoHighLevel CRM**, create a new contact if needed, and instantly book an appointment into a **GoHighLevel calendar**.

This workflow is ideal for AI receptionists, training bots, booking assistants, and appointment automation systems that need a simple **search → create → book** flow.

---

## Features

- Receives booking requests through a **Webhook**
- Built for **Retell AI chatbot / voice assistant integrations**
- Searches existing contacts in **GoHighLevel CRM**
- Detects whether a contact already exists
- Creates a new contact if not found
- Supports:
  - client name
  - phone number
  - email
  - service name
  - appointment date / datetime
- Books confirmed appointments directly in **GoHighLevel Calendar**
- Includes fallback handling for contact lookup errors
- Lightweight and suitable for real-time AI booking flows

---

## Workflow Overview

This workflow follows the process below:

1. A chatbot or voice assistant sends booking details through a **Webhook**
2. The workflow normalizes the incoming appointment year
3. It searches **GoHighLevel CRM** using the phone number
4. The workflow checks the contact search result:
   - **If contact exists** → directly books the appointment
   - **If contact does not exist** → creates a new contact first
   - **If something goes wrong** → returns an error response
5. If the contact does not exist:
   - it checks whether a valid email is available
   - creates the contact accordingly
6. The workflow books the appointment into the configured **GoHighLevel calendar**

---

## Main Use Case

This workflow is designed for **Retell AI appointment booking flows** where an AI agent collects booking information during a live call or chat and then sends the structured data into n8n for automated booking.

It works especially well for:

- AI receptionist systems
- service business booking assistants
- clinic / consultation booking bots
- support scheduling assistants
- training/demo chatbot flows

---

## Nodes Used

## 1. Input Layer

### 1. Webhook
The workflow begins with a **Webhook** node.

### Webhook configuration:
- **Method:** `POST`

This allows external tools such as:

- Retell AI
- chatbots
- voice agents
- custom booking forms

to send appointment data directly into the workflow.

---

## 2. Date Normalization Layer

### 2. Code in JavaScript1
This node updates the incoming appointment date to the **current system year**.

### What it does:
- reads:
```json
body.args.appointment_date
```

- converts it into a JavaScript date
- replaces the year with the current year
- writes the updated value back into the request payload

### Why this matters
This helps prevent booking issues when an assistant sends a relative or outdated year value.

---

## 3. Contact Search Layer

### 3. Search contact
This node searches **GoHighLevel CRM** for an existing contact using the provided phone number.

### API endpoint used:
```text
https://services.leadconnectorhq.com/contacts/search
```

### What it sends:
- `locationId`
- phone-based search filter
- page and sorting parameters

### Search purpose:
To avoid creating duplicate contacts before booking an appointment.

---

## 4. Contact Decision Layer

### 4. Contact Status
This is a **Switch** node that determines what happens after contact search.

### Routing logic:

#### Not Found
If no matching contact exists:
- the workflow moves into the **contact creation path**

#### Found
If a matching contact is found:
- the workflow skips contact creation
- and directly books the appointment

#### Error
If the result is unexpected:
- the workflow moves to an error response path

This acts as the main decision engine for the workflow.

---

## 5. Error Handling Layer

### 5. Return Error
If the contact search result is not usable, this node returns a fallback response.

### Returned message:
```text
I apologize; it seems we are having an issue.
```

This helps the chatbot fail gracefully instead of breaking the conversation.

---

## 6. Contact Creation Logic

If the contact is not found, the workflow decides how to create the contact.

---

### 6. If
This node checks whether the provided email looks valid.

### Validation rule:
It checks if the incoming email contains:

```text
@
```

### Routing logic:
- **If email looks valid** → create contact with email
- **If email is missing or invalid** → create contact without email

This makes the booking flow more flexible for phone-based chatbot interactions.

---

### 7. Create a contact
This node creates a new **GoHighLevel contact** when a valid email is available.

### API endpoint used:
```text
https://services.leadconnectorhq.com/contacts
```

### Fields sent:
- name
- email
- location ID
- phone
- tag: `ai_agent`

This creates a CRM contact before booking the appointment.

---

### 8. Create a contact1
This node also creates a new **GoHighLevel contact**, but without relying on email.

### Used when:
- the user did not provide a valid email
- or the chatbot only captured phone + name

This keeps the booking workflow usable even in low-data conversations.

---

## 7. Appointment Booking Layer

The workflow supports **three booking paths** depending on whether the contact was found or newly created.

---

### 9. Book Appointment1
This node books the appointment when the contact already exists in CRM.

### API endpoint used:
```text
https://services.leadconnectorhq.com/calendars/events/appointments
```

### It uses:
- existing contact ID
- existing location ID
- requested service
- requested appointment datetime

This is the fastest booking path.

---

### 10. Book Appointment
This node books the appointment after a **new contact with email** has been created.

### Booking payload includes:
- service title
- meeting location type
- appointment status
- assigned user ID
- calendar ID
- location ID
- contact ID
- appointment start time

This creates a confirmed appointment directly into GoHighLevel.

---

### 11. Book Appointment2
This node books the appointment after a **new contact without email** has been created.

This allows the workflow to still complete the booking even if email was not captured.

---

## Booking Configuration

The appointment booking nodes use a shared appointment structure.

### Booking details include:
- **Title**:
```text
Service - {{ service }}
```

- **Appointment Status**:
```text
confirmed
```

- **Meeting Location Type**:
```text
custom
```

- **Calendar ID**
- **Assigned User ID**
- **Contact ID**
- **Location ID**
- **Start Time**

### Important behavior:
The workflow currently uses:

```text
ignoreFreeSlotValidation: true
```

This means it can create appointments **without strict availability validation**, so it is best used **after** a separate availability-checking workflow confirms the slot.

---

## Example Input Payload

This workflow expects a webhook payload containing booking details.

### Example request:
```json
{
  "body": {
    "args": {
      "client_name": "John Doe",
      "client_phone": "+19295893118",
      "email": "john@example.com",
      "service": "Training Session",
      "appointment_date": "2026-04-10T15:00:00.000Z",
      "appointment_datetime": "2026-04-10T15:00:00.000Z"
    }
  }
}
```

---

## Example Workflow Paths

### Scenario 1: Existing Contact Found
1. Retell sends booking request
2. Workflow searches CRM
3. Contact is found
4. Appointment is booked immediately

---

### Scenario 2: Contact Not Found + Valid Email
1. Retell sends booking request
2. Workflow searches CRM
3. Contact is not found
4. Valid email is detected
5. Contact is created with email
6. Appointment is booked

---

### Scenario 3: Contact Not Found + No Valid Email
1. Retell sends booking request
2. Workflow searches CRM
3. Contact is not found
4. Email is missing or invalid
5. Contact is created without email
6. Appointment is booked

---

## Requirements

To run this workflow successfully, you need:

- **n8n**
- a **GoHighLevel account**
- access to **GoHighLevel CRM**
- access to a valid **GoHighLevel calendar**
- a valid **LeadConnector / GHL API token**
- a chatbot or voice assistant that can send webhook requests

---

## External Services Used

This workflow integrates with:

- **GoHighLevel / LeadConnector**
- **Retell AI** or any external chatbot via webhook

---

## Setup Instructions

1. Import the JSON workflow into n8n
2. Copy your webhook URL from the **Webhook** node
3. Connect your Retell chatbot / AI assistant to that webhook
4. Replace the hardcoded **GHL API token** with your own
5. Replace:
   - `locationId`
   - `calendarId`
   - `assignedUserId`
   with your own GoHighLevel values
6. Activate the workflow
7. Send a test booking request
8. Verify that:
   - contact search works correctly
   - new contacts are created when needed
   - appointments are booked successfully
   - fallback error handling works if needed

---

## Important Notes

- The workflow is currently **active**
- It is optimized for **real-time AI booking**
- The workflow is simple and effective for **Retell-based scheduling flows**
- It is best used after availability has already been confirmed
- It supports both:
  - known returning contacts
  - brand new contacts

---

## Limitations

At the moment, this workflow has a few limitations:

- No built-in availability checking
- No timezone normalization layer
- No appointment conflict prevention
- No SMS / email confirmation after booking
- No webhook response formatting for rich chatbot output
- No reschedule / cancel logic
- No CRM note or booking reason logging

---

## Suggested Improvements

You can improve this workflow further by adding:

- availability checking before booking
- timezone auto-detection
- booking confirmation SMS/email
- duplicate appointment detection
- CRM notes for booking reason
- user-friendly chatbot response formatting
- cancellation / rescheduling support
- service-specific routing
- booking confirmation webhook response
- internal staff notifications

---

## Recommended Workflow Pairing

This workflow works best when paired with:

- availability checker workflows
- Retell AI voice bot workflows
- chatbot qualification flows
- CRM lead intake workflows
- outbound callback assistants

A complete system could work like this:

1. AI chatbot asks for preferred time
2. availability workflow checks slot
3. this workflow books the appointment
4. confirmation workflow sends follow-up message

This creates a complete **AI receptionist booking system**.

---

## Example Real-World Use Cases

This workflow is ideal for:

- AI receptionist booking systems
- medical consultation booking bots
- coaching call schedulers
- clinic intake assistants
- sales appointment booking bots
- home service booking assistants

---

## Author

Built as an **n8n Retell chatbot appointment booking workflow** for AI receptionists, support assistants, and GoHighLevel automation systems.

---

## License

You can add your preferred license here, such as:

- MIT
- Apache-2.0
- Proprietary
