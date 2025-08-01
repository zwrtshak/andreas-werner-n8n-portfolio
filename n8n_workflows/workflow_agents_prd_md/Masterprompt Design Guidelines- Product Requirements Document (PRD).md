**Masterprompt Design Guidelines: Product Requirements Document (PRD)**\
1. Purpose & Scope\
This document outlines the best practices for designing effective and
interoperable masterprompts for AI agents within the SupervisorAgent
system. It provides structure, tone, formatting, and content-level
recommendations to ensure clarity, efficiency, and modular agent
orchestration.\
The goal is to create masterprompts that:

- Are precise yet flexible.

- Provide enough domain context for the agent to act meaningfully.

- Are not overloaded with irrelevant rules that could cause confusion.

- Ensure every sub-agent works in harmony with others under the
  coordination of the SupervisorAgent.\
  \
  **2. Prompt Structure**\
  Each masterprompt must be structured into the following standard
  sections:\
  **2.1 Role Definition**\
  Clearly define what this agent is, what it is not, and under whose
  authority it acts.\
  You are \[agent_name\], a sub-agent within the SupervisorAgent
  framework. Your domain is \[e.g., email communication / diagnosis /
  calendar management\].\
  **2.2 Communication Rules**\
  Define how the agent communicates (language, tone, formal/informal
  style, structured/unstructured output).\
  - Internal communication: English\
  - External responses: German\
  - Format: \[JSON \| plain text \| Markdown\] depending on the tool
  chain\
  - Tone: Professional, efficient, empathetic (adjustable per agent)\
  **2.3 Available Tools (Optional)**\
  If the agent uses tools or APIs, include brief descriptions without
  deep implementation details.\
  You may call the following tools if needed:\
  - CalendarAgent → schedule, reschedule, cancel appointments\
  - WriterAgent → create structured written content\
  **2.4 Trigger Conditions & Behaviors**\
  Describe under what conditions the agent acts and how it decides what
  to do.\
  When you receive a message related to a medical symptom, always first
  perform a RAG-based semantic lookup before responding.\
  If no confident match is found, ask the SupervisorAgent for further
  instructions.\
  **2.5 Memory / State Awareness (If applicable)**\
  - Always consider the last 2--3 interactions.\
  - Discard irrelevant memory or summarize long threads.\
  **2.6 Response Format**\
  Explain how the agent must return output (this ensures smooth
  interoperability).\
  Always respond with a single tool call or a plain-text explanation.\
  Do not mix formats. Do not use markdown unless explicitly requested.\
  **2.7 Coordination with SupervisorAgent**\
  Reinforce the chain of command:\
  You never act independently. You always respond to input or delegation
  from the SupervisorAgent.\
  Never override other agents\' decisions. Instead, refer back when
  uncertain.\
  \
  **3. Content Guidelines**\
  3.1 Sufficient Context\
  Include:

<!-- -->

- What the agent is responsible for

- What input format it receives

- What kind of decisions it is expected to make

- What tools (if any) it is allowed to use\
  **3.2 Avoid Prompt Overload**\
  Too much specificity (e.g., repeating tool syntax rules, embedding SQL
  details, long disclaimers) will:

<!-- -->

- Make the agent brittle

- Cause hallucinations or slowdowns\
  **3.3 Inter-Agent Interoperability**\
  All prompts must:

<!-- -->

- Respect the boundaries of other agents

- Rely on SupervisorAgent for coordination

- Use shared schemas and formats (especially for JSON)

- Never communicate directly with each other outside the SupervisorAgent
  pipeline\
  \
  **4. Example Prompt Shell**\
  {\
  \"role\": \"system\",\
  \"communication\": {\
  \"internal\": \"English\",\
  \"external\": \"German\"\
  },\
  \"content\": \"You are the DiagnoseAgent. You analyze symptoms and
  propose diagnoses via RAG lookup. Use dermarules first, fallback to
  learned_rules. Confirm confidence. Never respond without
  SupervisorAgent instruction. Output must be JSON format.\"\
  }\
  \
  **5. Versioning & Maintenance**

<!-- -->

- All prompts must be version-controlled

- Changes to shared schemas or memory behavior must be communicated
  across the agent framework

- Updates must be tested in simulation mode before rollout\
  \
  **6. JSON Prompt Schema für Sub-Agenten**\
  Die optimale Prompt-Struktur im **reinen JSON-Format** für deine
  Sub-Agents (z. B. DiagnoseAgent, CalendarAgent etc.) sollte so
  gestaltet sein, dass sie:

<!-- -->

- **klar getrennt zwischen Systemrolle, Kommunikationsregeln und
  eigentlichem Content** ist,

- **vom SupervisorAgent oder einem LLM-System automatisch geparst und
  analysiert** werden kann,

- **nicht zu tief verschachtelt ist**, um Missverständnisse bei Modellen
  zu vermeiden,

- **nur \"system\"-Rolle** nutzt (kein \"user\" oder \"assistant\" in
  der Initialisierung),

- **keine unnötigen Steuerzeichen oder Markdown-Formatierungen**
  enthält.\
  {\
  \"role\": \"system\",\
  \"agent_name\": \"diagnose_agent\",\
  \"communication\": {\
  \"input_language\": \"German\",\
  \"output_language\": \"German\",\
  \"internal_language\": \"English\",\
  \"output_format\": \"JSON\"\
  },\
  \"permissions\": {\
  \"can_call_tools\": true,\
  \"tool_list\": \[\
  \"RAG_read\",\
  \"RAG_learn\"\
  \],\
  \"can_access_memory\": false\
  },\
  \"behavior\": {\
  \"description\": \"You analyze patient symptom input and return a
  structured diagnosis. Always use dermarules first, fallback to
  learned_rules if no result. If confidence is low, return two possible
  options and wait for human approval.\",\
  \"trigger_condition\": \"Input contains symptom-related
  information\",\
  \"coordination\": \"Only respond to direct instruction from
  SupervisorAgent. Never execute actions independently.\"\
  },\
  \"output_schema\": {\
  \"type\": \"object\",\
  \"properties\": {\
  \"patient_id\": {\
  \"type\": \"string\"\
  },\
  \"diagnosis\": {\
  \"type\": \"array\",\
  \"items\": {\
  \"type\": \"object\",\
  \"properties\": {\
  \"condition\": { \"type\": \"string\" },\
  \"confidence\": { \"type\": \"number\" },\
  \"recommended_actions\": {\
  \"type\": \"array\",\
  \"items\": { \"type\": \"string\" }\
  }\
  },\
  \"required\": \[\"condition\", \"confidence\"\]\
  }\
  },\
  \"source\": {\
  \"type\": \"string\",\
  \"enum\": \[\"dermarules\", \"learned_rules\"\]\
  },\
  \"comments\": {\
  \"type\": \"string\"\
  }\
  },\
  \"required\": \[\"patient_id\", \"diagnosis\", \"source\"\]\
  }\
  }\
  **Warum diese Struktur?**

<!-- -->

- Trennscharf: Jedes Element hat eine eindeutige Semantik.

- **Erweiterbar:** Du kannst tools, output_schema, examples, etc.
  ergänzen.

- **LLM-freundlich:** Viele Modelle verstehen \"role\": \"system\" als
  Initialkontext perfekt.

- **Validierbar:** Optional kannst du dieses JSON-Schema gegen ein
  JSON-Schema-Validator-Tool prüfen.
