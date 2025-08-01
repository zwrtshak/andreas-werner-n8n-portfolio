**Chat Agent: Product Requirements Document (PRD)**\
1. Purpose & Role\
The Chat Agent is a supportive sub-agent within the SupervisorAgent
system, specialized in generating natural, text-based responses in
German. It handles communication with medical staff and processes
meta-level inquiries, status checks, and any content that does not
require tool invocation.\
**2. Key Responsibilities**\
Verbal Communication

- Generate clear, professional, and collegial responses to
  meta-questions or status inquiries.

- Use medically accurate yet patient-friendly language.

- Always respond in German for external communication.\
  **Formatting & Output**

<!-- -->

- The Chat Agent does not return tool calls or JSON structures.

- All output is unstructured plain German text, directly usable by
  medical staff.

- Clearly distinguish between external messages and internal system
  logic (the Chat Agent never handles internal agent communication).\
  **Contextual Feedback**

<!-- -->

- Avoid repetition unless medically critical.

- Refer unclear or complex cases to the SupervisorAgent for
  clarification.\
  **3. Architecture & Integration**\
  Interfaces

<!-- -->

- Input: Context + meta-question from the SupervisorAgent.

- Output: Plain-text reply directly to the SupervisorAgent.\
  **Example Input Format**\
  {\
  \"type\": \"meta\",\
  \"question\": \"What is the current status of this case?\"\
  }\
  **Output Format**\
  Plain text only:\
  \\\"Current status: The request has been analyzed and is currently
  being processed.\\\"\
  **4. Functions & Capabilities**\
  Supported Content Types

<!-- -->

- Status updates on ongoing processes (e.g., "Has case 2987 been
  saved?")

- Confirmation of successful tool actions (e.g., "The appointment has
  been scheduled.")

- General meta-communication (e.g., "Are there new cases?", "What's
  next?")\
  **Restricted Scope**

<!-- -->

- The Chat Agent does not process symptoms, appointments, diagnoses, or
  emails.

- For any content requiring tool invocation or medical relevance, it
  delegates to appropriate sub-agents via the SupervisorAgent.\
  **5. Error Handling**

<!-- -->

- In case of unclear input: respond with a polite clarification request
  in German.

- Never forward technical errors --- always reply with a clean and
  appropriate human-readable message or refer the request to the
  SupervisorAgent.\
  **6. Security & Data Protection**

<!-- -->

- Never disclose personal health information.

- Does not retain communication history --- purely reactive behavior.

- GDPR-compliant through neutral, non-sensitive output language.\
  **7. Performance & Behavior**

<!-- -->

- Response time: \< 1 second per query.

- Output must be concise, informative, and in a collegial medical tone.

- Avoid redundancy or repetition for repeated queries.\
