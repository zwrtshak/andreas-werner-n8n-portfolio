**Diagnose Agent: Product Requirements Document (PRD)**\
1. Purpose & Role\
The Diagnose Agent is a specialized sub-agent within the SupervisorAgent
system, designed to support medical decision-making in dermatology
practices. It analyzes patient information and symptoms, performs
semantic searches across the dermarules and learned_rules databases, and
generates accurate, data-based diagnoses for the SupervisorAgent.\
**2. Key Responsibilities**\
Request Handling & Symptom Analysis

- Receive symptom descriptions, images, and contextual information from
  the SupervisorAgent.

- Automatically extract and interpret medically relevant keywords.

- Validate completeness and consistency of patient data.\
  **Semantic Search & RAG Integration**

<!-- -->

- Perform semantic vector search in the primary RAG database dermarules
  via HTTP POST to /rag with embeddings.

- If no sufficient match is found (cosine \< 0.85), query learned_rules
  as fallback.

- Optionally use combined SQL view:\
  SELECT \*, 1 - (embedding \<=\> \'{{ embedding }}\'::vector) AS score\
  FROM (\
  SELECT \* FROM dermarules\
  UNION ALL\
  SELECT \* FROM learned_rules WHERE validated_by_human = true\
  ) AS combined\
  WHERE embedding IS NOT NULL\
  ORDER BY embedding \<=\> \'{{ embedding }}\'::vector\
  LIMIT 3;\
  **Diagnosis Generation**

<!-- -->

- Generate data-driven preliminary or differential diagnoses.

- Propose actionable medical steps, such as treatment options or
  additional tests.

- Return structured diagnosis JSON to the SupervisorAgent.\
  **3. Architecture & Integration**\
  Interfaces

<!-- -->

- Input: JSON structure from the SupervisorAgent containing patient ID
  and symptoms.

- Databases: Access dermarules (primary) and learned_rules (secondary)
  via local API or SQL.

- Output: JSON diagnosis report to the SupervisorAgent.\
  **Example Input Format**\
  {\
  \"patient_id\": \"patient_uuid\",\
  \"symptoms\": \"Description of symptoms\",\
  \"context\": \"Additional context such as age, prior treatment\",\
  \"images\": \[\"local_image_urls\"\]\
  }\
  **Example Output Format**\
  {\
  \"patient_id\": \"patient_uuid\",\
  \"diagnosis\": \[\
  {\
  \"condition\": \"Diagnosis name\",\
  \"confidence\": 0.92,\
  \"recommended_actions\": \[\"Therapy suggestion\", \"Follow-up in 7
  days\"\]\
  }\
  \],\
  \"source\": \"dermarules\" \| \"learned_rules\",\
  \"similar_cases\": \[\"reference_case_ids\"\],\
  \"comments\": \"Additional notes or interpretation\"\
  }\
  **4. Functions & Capabilities**\
  Semantic Similarity Search

<!-- -->

- Use locally generated embeddings (e.g., via sentence-transformers,
  Ollama).

- Dynamically switch between dermarules and learned_rules depending on
  result quality.\
  **Diagnostic Recommendations**

<!-- -->

- Provide diagnoses with confidence scores.

- List alternative diagnoses when appropriate.

- Suggest further diagnostic or therapeutic steps.\
  **5. Error Handling & Monitoring**

<!-- -->

- Validate input data structure from the SupervisorAgent.

- Return structured JSON error message for incomplete or faulty input.

- Log diagnostic processes internally for traceability.\
  **6. Human-in-the-Loop & Feedback Loop**

<!-- -->

- If diagnosis confidence is low: return two ranked options to
  SupervisorAgent.

- Upon human confirmation, store approved reaction in learned_rules.\
  **7. Security & Data Protection**

<!-- -->

- Fully comply with GDPR regulations.

- Process and store all sensitive data locally.

- Encrypt patient information before storage.\
  **8. Continuous Learning**

<!-- -->

- Automatically generate embeddings and store approved diagnoses in
  learned_rules.

- Periodically integrate validated learned_rules into dermarules.\
  **9. Performance & Scalability**

<!-- -->

- RAG and diagnosis generation within 2 seconds.

- Scalable for large volumes of cases.

- Modular architecture to support future diagnostic features.
