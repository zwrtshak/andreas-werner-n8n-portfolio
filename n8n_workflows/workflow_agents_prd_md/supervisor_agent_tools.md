# Supervisor Agent Specifications

| Tool / Agent    | Purpose                                                                      | Calling Signature                                                                                    |
|:----------------|:-----------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------|
| call_agent      | Handle inbound/outbound calls                                                | <<tool:call_agent { "call_id":"…", "action":"dial" }>>                                               |
| email_agent     | Forward & process complete emails (sender, subject, body, attachments)       | <<tool:email_agent { "email_id":"…", "action":"reply" }>>                                            |
| contacts_agent  | Manage contact records                                                       | <<tool:contacts_agent { "contact":{"name":"…","type":"patient"}}>>                                   |
| calendar_agent  | Schedule, reschedule, cancel appointments                                    | <<tool:calendar_agent { "appointment_request":{…} }>>                                                |
| writer_agent    | Draft structured letters, notes, reports                                     | <<tool:writer_agent { "document_type":"report", "context":{…} }>>                                    |
| diagnose_agent  | Propose differential diagnoses & therapy plans (includes internal RAG_read)  | <<tool:diagnose_agent { "patient_id":"…", "symptoms":"…", "rag_snippets":[] }>>                      |
| inventory_agent | Monitor stock levels, place orders, track reorders                           | <<tool:inventory_agent { "product_id":"…","action":"order","quantity":…}>>                           |
| calculator      | Perform medical/math calculations (BMI, dosages, intervals)                  | <<tool:calculator { "formula":"BMI","inputs":{…} }>>                                                 |
| RAG_read        | Retrieve top-K relevant snippets from vector DB (dermarules + learned_rules) | _invoked by DiagnoseAgent_                                                                           |
| RAG_learn       | Store human-validated rule entries into learned_rules for future RAG lookups | <<tool:RAG_learn { "text_for_embedding":"…","reaction":{…},"tags":["…"],"validated_by_human":true}>> |
| chat_agent      | Produce free-text German responses for the human supervisor                  | <<tool:chat_agent { "message":"<German text>" }>>                                                    |