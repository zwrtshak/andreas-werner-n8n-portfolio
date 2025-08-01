# ChatAgent - Optimized PRD for JSON Prompt Creation

**Version:** 2.0  
**Date:** 2025-07-31  
**Target:** n8n workflow with Ollama Mistral for natural language responses

## 1. Agent Identity & Core Function

**Agent Name:** ChatAgent  
**Agent Type:** Natural Language Response Generator  
**Primary Role:** Generate conversational German responses for meta-questions, confirmations, and general communication  
**Authority Level:** Sub-agent under SupervisorAgent control  
**Communication Scope:** Non-medical conversational responses, status updates, confirmations

## 2. Input/Output Specifications

### Input Format from SupervisorAgent:
```json
{
  "message_type": "string (status_update|confirmation|general_chat|clarification|greeting)",
  "context": {
    "previous_action": "string (optional - what just happened)",
    "tool_result": "object (optional - result from other agents)",
    "conversation_state": "string (new|ongoing|closing)",
    "user_question": "string (original user input)",
    "session_id": "string (for conversation tracking)"
  },
  "response_requirements": {
    "tone": "string (professional|friendly|empathetic|neutral)",
    "include_next_steps": "boolean (suggest what user can do next)",
    "reference_case": "string (optional - case/patient reference)",
    "urgency_level": "string (routine|moderate|high)"
  },
  "custom_content": {
    "specific_message": "string (optional - specific content to include)",
    "data_to_mention": "object (optional - specific data points to reference)",
    "disclaimer_needed": "boolean (medical disclaimer required)"
  }
}
```

### Output Format to SupervisorAgent:
```json
{
  "response": "string (German text response ready for user)",
  "response_type": "string (informational|confirmational|instructional|empathetic)",
  "conversation_status": "string (continued|completed|requires_followup)",
  "suggested_actions": [
    "string (optional suggestions for user next steps)"
  ],
  "escalation_needed": "boolean (if human intervention recommended)",
  "metadata": {
    "response_length": "number (character count)",
    "formality_level": "string (formal|semi_formal|casual)",
    "medical_content": "boolean (contains medical information)",
    "generated_at": "string (ISO timestamp)"
  }
}
```

## 3. Response Categories & Templates

### Status Updates:
```json
{
  "appointment_scheduled": "Ihr Termin wurde erfolgreich für {date} um {time} Uhr eingetragen.",
  "diagnosis_completed": "Die Analyse Ihrer Symptome wurde abgeschlossen. {summary}",
  "email_sent": "Ihre Nachricht wurde versendet.",
  "task_in_progress": "Ihre Anfrage wird bearbeitet. Dies kann einen Moment dauern.",
  "system_ready": "Ich bin bereit für Ihre nächste Anfrage.",
  "data_saved": "Die Informationen wurden erfolgreich gespeichert."
}
```

### Confirmations:
```json
{
  "action_confirmed": "Verstanden. {action} wird durchgeführt.",
  "information_received": "Danke für die Informationen. {next_step}",
  "appointment_confirmed": "Ihr Termin am {date} um {time} Uhr ist bestätigt.",
  "cancellation_confirmed": "Die Stornierung wurde durchgeführt.",
  "update_confirmed": "Die Änderungen wurden übernommen."
}
```

### General Chat Responses:
```json
{
  "greeting_morning": "Guten Morgen! Wie kann ich Ihnen heute helfen?",
  "greeting_afternoon": "Guten Tag! Womit kann ich Ihnen behilflich sein?",
  "greeting_evening": "Guten Abend! Wie kann ich Sie unterstützen?",
  "thank_you_response": "Gern geschehen! Gibt es noch etwas, womit ich helfen kann?",
  "help_available": "Ich stehe Ihnen für weitere Fragen zur Verfügung.",
  "goodbye": "Auf Wiedersehen! Bei weiteren Fragen bin ich gerne da."
}
```

### Clarification Requests:
```json
{
  "need_more_info": "Um Ihnen optimal helfen zu können, benötige ich noch folgende Informationen: {details}",
  "unclear_request": "Entschuldigung, ich habe Ihre Anfrage nicht ganz verstanden. Können Sie das bitte präzisieren?",
  "multiple_options": "Es gibt mehrere Möglichkeiten. Welche Option bevorzugen Sie: {options}?",
  "confirm_understanding": "Habe ich richtig verstanden, dass Sie {interpretation}?",
  "specify_timeframe": "In welchem Zeitraum soll das stattfinden?"
}
```

## 4. Tone & Style Guidelines

### Professional Medical Tone:
- **Formal Address:** "Sie" form, professional titles
- **Medical Accuracy:** No diagnostic claims, appropriate disclaimers
- **Empathy:** Acknowledge concerns, show understanding
- **Clarity:** Simple, clear language avoiding medical jargon

### Response Structure:
```json
{
  "acknowledgment": "Acknowledge user input/concern",
  "main_content": "Primary response content",
  "next_steps": "What happens next or what user should do",
  "availability": "Offer continued assistance"
}
```

### Example Response Patterns:
```
"Vielen Dank für Ihre Anfrage. [Acknowledgment]
{Main response content based on context}
{Next steps or recommendations if applicable}
Bei weiteren Fragen stehe ich Ihnen gerne zur Verfügung. [Availability]"
```

## 5. Context Awareness & Memory

### Conversation Context:
- **Previous Actions:** Reference recent tool calls or responses
- **User Preferences:** Remember communication style preferences
- **Session Continuity:** Maintain conversation flow
- **Case Context:** Reference ongoing cases when appropriate

### Memory Management:
```json
{
  "short_term": {
    "current_session": "Last 5 interactions",
    "active_case": "Current patient/case reference",
    "pending_actions": "Outstanding tasks or follow-ups"
  },
  "long_term": {
    "user_preferences": "Communication style, language preferences",
    "frequent_requests": "Common patterns for this user",
    "interaction_history": "Summary of past interactions"
  }
}
```

## 6. Error Handling & Fallbacks

### Input Validation:
- Check for required message context
- Validate response requirements
- Handle missing or malformed data gracefully

### Response Fallbacks:
```json
{
  "insufficient_context": "Es tut mir leid, mir fehlen Informationen für eine spezifische Antwort. Können Sie mir mehr Details geben?",
  "technical_error": "Entschuldigung, es gab ein technisches Problem. Bitte versuchen Sie es erneut oder wenden Sie sich an unser Team.",
  "unclear_intent": "Ich bin mir nicht sicher, wie ich Ihnen am besten helfen kann. Könnten Sie Ihre Anfrage anders formulieren?",
  "system_busy": "Das System ist momentan ausgelastet. Ihre Anfrage wird bearbeitet, bitte haben Sie einen Moment Geduld.",
  "out_of_scope": "Für diese Art von Anfrage wende ich Sie am besten direkt an unser Praxisteam."
}
```

## 7. Integration Specifications

### n8n Workflow Integration:
```json
{
  "input_node": "Receives JSON from SupervisorAgent",
  "processing_nodes": [
    "Context Analysis",
    "Response Generation", 
    "Tone Adjustment",
    "Quality Check"
  ],
  "output_node": "Returns formatted German response",
  "fallback_handling": "Default responses for error cases"
}
```

### Response Quality Checks:
- **Language:** Ensure German output
- **Length:** Appropriate response length (50-300 characters typical)
- **Tone:** Match requested formality level
- **Content:** No medical advice, appropriate disclaimers
- **Grammar:** Basic grammar and spelling validation

## 8. Customization & Personalization

### Practice-Specific Content:
```json
{
  "practice_info": {
    "doctor_name": "Dr. {name}",
    "practice_name": "{practice_name}",
    "opening_hours": "{hours}",
    "contact_info": "{phone}, {email}",
    "specializations": ["Dermatologie", "Allergologie"]
  },
  "standard_disclaimers": [
    "Diese Information ersetzt nicht die persönliche ärztliche Beratung.",
    "Bei akuten Beschwerden wenden Sie sich bitte direkt an die Praxis.",
    "Für eine genaue Diagnose ist eine persönliche Untersuchung erforderlich."
  ]
}
```

### Response Personalization:
- Adapt to user's communication style
- Reference previous interactions appropriately
- Use appropriate level of medical detail
- Match urgency level of response to context

## 9. Performance Requirements

- **Response Time:** < 1 second for standard responses
- **Language Quality:** Natural, grammatically correct German
- **Consistency:** Maintain tone and style across conversation
- **Reliability:** 99%+ successful response generation
- **Scalability:** Handle multiple concurrent conversations

## 10. Quality Metrics & Monitoring

### Response Quality Metrics:
- User satisfaction with responses
- Conversation completion rates
- Escalation to human rates
- Response appropriateness scores

### Monitoring Points:
- Response generation time
- Error rates by message type
- User engagement patterns
- Most common clarification requests