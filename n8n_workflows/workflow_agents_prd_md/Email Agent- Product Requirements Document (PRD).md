**Email Agent: Product Requirements Document (PRD)**\
1. Purpose & Role\
The Email Agent is a specialized sub-agent within the SupervisorAgent
system, designed for comprehensive email management in a dermatology
practice. It processes all incoming and outgoing emails, structures
communication, and ensures seamless integration and efficient
information flow with the SupervisorAgent.\
**2. Key Responsibilities**\
Incoming Emails

- Full reception of IMAP-triggered emails.

- Automatic extraction of sender, subject, body, and attachments.

- Content categorization based on keywords.

- Optional semantic analysis and assignment to patient records via RAG.

- Detection of critical or urgent messages and immediate escalation to
  the SupervisorAgent.\
  **Outgoing Emails**

<!-- -->

- Compose professional, patient-friendly responses.

- Auto-generate drafts using templates and dynamic RAG-based content.

- Handle and embed attachments (e.g., medical reports, prescriptions).

- Ensure GDPR-compliant labeling and secure archiving of sent
  documents.\
  **Email Organization**

<!-- -->

- Automatic labeling, archiving, and structured storage in local
  Supabase tables (email archive).

- Maintain a status tracking system (e.g., open, answered, escalated).\
  **GDPR Compliance**

<!-- -->

- Automatic pseudonymization of personal data within email content.

- Encrypted local storage of sensitive email data.\
  **3. Architecture & Integration**\
  Interfaces

<!-- -->

- Reception: IMAP trigger directly from the SupervisorAgent.

- SupervisorAgent: Pass all email data as unmodified JSON block.

- RAG System: (optional) Semantic context analysis and response
  generation.

- Supabase: Archive structured email data for audit and history
  tracking.\
  **JSON Format for SupervisorAgent Communication**\
  {\
  \"id\": \"email_uuid\",\
  \"sender\": \"email@address.com\",\
  \"subject\": \"Subject line\",\
  \"body\": \"Email body content\",\
  \"attachments\": \[\
  { \"filename\": \"attachment1.pdf\", \"url\": \"local_storage_url\" }\
  \]\
  }\
  **4. Functions & Capabilities**\
  Classification & Routing

<!-- -->

- Classify emails into categories (e.g., appointment, prescription,
  medical inquiry, admin).

- Automatically route categorized emails to respective tools
  (CalendarAgent, DiagnoseAgent, InventoryAgent).\
  **Response Generation**

<!-- -->

- Use predefined templates.

- Generate dynamic RAG-based response suggestions based on context and
  medical domain knowledge.

- Provide multiple response options for SupervisorAgent approval.\
  **Attachment Handling**

<!-- -->

- Automatically extract and locally store attachments.

- Validate attachments (file type, virus scan).

- Forward relevant medical attachments to DiagnoseAgent for further
  analysis.\
  **5. Error Handling & Monitoring**

<!-- -->

- Validate incoming JSON structure.

- Automatically report malformed or incomplete emails to
  SupervisorAgent.

- Log all activities with timestamps, statuses, and error metadata.\
  **6. Human-in-the-Loop & Feedback Loop**

<!-- -->

- Activate suggestion mode for complex replies requiring human approval.

- Once approved by the SupervisorAgent, responses are stored in the
  learned_rules table to enhance semantic accuracy over time.\
  **7. Security & Data Protection**

<!-- -->

- Handle all content and attachments confidentially in accordance with
  GDPR.

- Encrypt all stored personal data from emails.

- Restrict access via roles and permissions within the practice
  network.\
  **8. Continuous Learning**

<!-- -->

- Automatically create embeddings for confirmed replies.

- Integrate approved replies into the learned_rules database to
  dynamically improve future responses.\
  **9. Performance & Scalability**

<!-- -->

- Processing time: under 2 seconds per incoming email.

- Scalable to hundreds of emails per day without performance
  degradation.

- Modular architecture allows easy expansion and customization.\
