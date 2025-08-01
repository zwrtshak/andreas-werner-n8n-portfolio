**Contacts Agent: Product Requirements Document (PRD)**\
1. Purpose & Role\
The Contacts Agent is a specialized sub-agent within the SupervisorAgent
system, responsible for managing all contact data in the dermatology
practice. It handles the creation, editing, storage, validation, and
structuring of contact information for patients, medical staff,
pharmacies, laboratories, insurance companies, and other external
partners.\
**2. Key Responsibilities**\
Contact Management

- Create, update, and delete contact records.

- Validate email addresses, phone numbers, and postal addresses.

- Perform automatic duplicate checks during contact creation.\
  **Data Maintenance**

<!-- -->

- Update existing records based on incoming updates (e.g., a patient's
  new phone number).

- Optionally log the history of contact changes.

- Categorize contacts by type (e.g., patient, pharmacy, lab, etc.).\
  **Contextual Queries**

<!-- -->

- Provide specific contact information to other sub-agents (e.g., for
  scheduling or communication).

- Search contacts by name similarity, UUID, or phone number.\
  **3. Architecture & Integration**\
  Interfaces

<!-- -->

- Input: JSON from the SupervisorAgent specifying the contact action and
  data.

- Database: Local Supabase table contacts (assumed, not explicitly
  shown).

- Output: Confirmation or contact data response back to the
  SupervisorAgent.\
  **Input Format**\
  {\
  \"action\": \"create\" \| \"update\" \| \"get\" \| \"delete\",\
  \"data\": {\
  \"uuid\": \"optional_for_update_or_delete\",\
  \"name\": \"Max Mustermann\",\
  \"role\": \"Patient\",\
  \"email\": \"max@example.com\",\
  \"phone\": \"+491701234567\",\
  \"address\": \"Musterstraße 1, 12345 Berlin\"\
  }\
  }\
  **Output Format**\
  {\
  \"status\": \"success\",\
  \"message\": \"Contact updated successfully.\",\
  \"contact_id\": \"uuid\"\
  }\
  **4. Functions & Capabilities**\
  Supported Operations

<!-- -->

- create: Add a new contact.

- update: Modify an existing contact.

- delete: Remove a contact from the database.

- get: Retrieve a contact by UUID, name, or role.\
  **Integration Logic**

<!-- -->

- Frequently triggered by the SupervisorAgent during email or
  appointment processing.

- Other sub-agents access contact data indirectly through the
  SupervisorAgent.\
  **5. Error Handling**

<!-- -->

- Validate required fields during create/update.

- Return clear error messages to the SupervisorAgent when a contact is
  missing (get/delete).

- Always respond in well-structured German (localization assumed).\
  **6. Data Protection & Security**

<!-- -->

- Encrypt all sensitive contact data.

- Ensure full GDPR compliance.

- Do not share contact data with external systems unless explicitly
  requested by the SupervisorAgent.\
  **7. Performance & Reliability**

<!-- -->

- Response time \< 1 second for all CRUD operations.

- Error rate \< 0.1% for data transactions.

- Database operations are transaction-safe with rollback on failure.
