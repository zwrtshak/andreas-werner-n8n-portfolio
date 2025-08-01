**Writer Agent: Product Requirements Document (PRD)**\
1. Purpose & Role\
The Writer Agent is the content generation specialist within the
SupervisorAgent system. It is responsible for creating, drafting, and
editing structured and unstructured textual content across various
domains, including medical documentation, internal communications,
newsletters, public blog posts, social media content, and informational
emails.\
**2. Key Responsibilities**\
Content Creation

- Generate letters, case notes, reports, and patient communication on
  request.

- Draft blog posts, website content, email newsletters, and
  announcements.

- Support both internal (team communication) and external (patients,
  partners) messaging.\
  **Formatting & Personalization**

<!-- -->

- Customize tone and structure based on target group (e.g., medical
  professionals, patients, social media followers).

- Automatically insert dynamic content such as patient name, date, or
  medical context when available.

- Output formats: plaintext, markdown, HTML (if specified).\
  **Localization & Language Rules**

<!-- -->

- All final output intended for external use must be in fluent,
  professional German.

- Internal drafts or multi-step revisions may use English for internal
  review by the SupervisorAgent.\
  **Compliance & Validation**

<!-- -->

- Ensure that no medical recommendation is output unless validated by
  the Diagnose Agent.

- Avoid personal data in content unless explicitly authorized.

- Add standard GDPR footers or disclaimers when sending to external
  recipients.\
  **3. Architecture & Integration**\
  Input from SupervisorAgent\
  {\
  \"task\": \"generate_report\",\
  \"type\": \"case_note\",\
  \"language\": \"de\",\
  \"context\": {\
  \"patient_id\": \"1234\",\
  \"summary\": \"Follow-up on eczema treatment. Good progress.\"\
  }\
  }\
  **Output Example**\
  {\
  \"status\": \"success\",\
  \"content\": \"Die Patientin zeigt unter der aktuellen Behandlung eine
  deutliche Besserung der Ekzemsymptome. Weitere Kontrolle in 2 Wochen
  empfohlen.\"\
  }\
  **4. Functions & Capabilities**\
  Supported Content Types

<!-- -->

- case_note -- structured clinical observations.

- followup_email -- brief patient-friendly summaries.

- blog_post -- public health awareness articles.

- internal_note -- short staff memos or info updates.

- summary_letter -- formal correspondence.\
  **Workflow Awareness**

<!-- -->

- The Writer Agent may call RAG_read internally if more context is
  needed.

- Optionally tag generated drafts for human review.\
  **Integration with Other Agents**

<!-- -->

- Fetch diagnosis or treatment details from Diagnose Agent when flagged.

- May include calendar-based data if writing appointment follow-ups.\
  **5. Security & GDPR Compliance**

<!-- -->

- Never include PHI (Personal Health Information) unless explicitly
  approved.

- Store generated content locally if needed, linked via case_id.

- Append confidentiality notice to all patient-facing content.\
  **6. Performance & Reliability**

<!-- -->

- Response time: \< 2 seconds for standard text generation.

- Stability across multilingual tasks ensured via system fallback
  (German ↔ English).
