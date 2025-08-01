**Calendar Agent: Product Requirements Document (PRD)**\
1. Purpose & Role\
The Calendar Agent is a central sub-agent in the SupervisorAgent system,
responsible for comprehensive appointment management within the
dermatology practice. Its duties include creating, modifying,
confirming, and canceling appointments, as well as managing appointment
types, reminders, and escalations.\
**2. Key Responsibilities**\
Appointment Management

- Create new appointments based on structured requests from the
  SupervisorAgent.

- Modify existing appointments (date, time, reason, participants).

- Cancel or delete appointments with an optional reason.\
  **Appointment Types & Categories**

<!-- -->

- Support various types of appointments: initial consultation,
  follow-up, diagnostics, call, callback, pre-op, etc.

- Assign appointments to categories for reporting and resource planning
  purposes.\
  **Reminder Logic**

<!-- -->

- Trigger reminders (e.g., via email or internal notification).

- Support configurable reminder timings (e.g., 24h / 1h before the
  appointment).\
  **Prioritization & Escalation**

<!-- -->

- Flag urgent or high-priority requests (e.g., emergency cases).

- Escalate scheduling conflicts or missing resources (e.g., room,
  doctor) to the SupervisorAgent.\
  **3. Architecture & Integration**\
  Interfaces

<!-- -->

- Input: JSON containing the action type (create, update, cancel, query)
  and appointment details.

- Database: Local appointment storage (e.g., Supabase appointments
  table).

- Output: Status confirmation or structured response to the
  SupervisorAgent.\
  **Example Input Format**\
  {\
  \"action\": \"create\",\
  \"patient_id\": \"uuid\",\
  \"date\": \"2025-08-01\",\
  \"time\": \"09:30\",\
  \"type\": \"Initial Consultation\",\
  \"location\": \"Room 2\",\
  \"doctor\": \"Dr. Müller\",\
  \"notes\": \"Patient presenting with first-time eczema symptoms.\"\
  }\
  **Example Output Format**\
  {\
  \"status\": \"success\",\
  \"message\": \"The appointment was successfully created.\",\
  \"appointment_id\": \"uuid\"\
  }\
  **4. Functions & Capabilities**\
  Supported Actions

<!-- -->

- create: Add a new appointment.

- update: Modify an existing appointment.

- cancel: Cancel or delete an appointment.

- query: Query upcoming appointments (e.g., "What's scheduled for
  tomorrow?").\
  **Time Logic & Conflict Management**

<!-- -->

- Check for overlapping appointments.

- Validate time format and duration.

- Confirm physician availability.\
  **Feedback & Escalation**

<!-- -->

- Notify of conflicts (e.g., "Dr. Müller is not available at this
  time.").

- Request clarification in case of ambiguous input (e.g., "Please
  specify a date.").\
  **5. Security & Data Protection**

<!-- -->

- No sharing of patient data unless linked to a confirmed case.

- All data is stored locally in a GDPR-compliant manner.

- Access allowed only via the SupervisorAgent.\
  **6. Performance & Stability**

<!-- -->

- Response time: \< 1 second for standard operations.

- High availability and transaction-safe appointment logging.

- Protection from double bookings via lock or transaction handling.
