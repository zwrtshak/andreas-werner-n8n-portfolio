# SupervisorAgent - Optimized PRD for JSON Prompt Creation

**Version:** 2.0  
**Date:** 2025-07-31  
**Target:** n8n workflow with Ollama Mistral + FastAPI/Supabase integration

## 1. Agent Identity & Core Function

**Agent Name:** SupervisorAgent  
**Agent Type:** Central Orchestrator  
**Primary Role:** Route incoming requests to appropriate sub-agents and coordinate responses  
**Authority Level:** Master controller - all sub-agents report to SupervisorAgent  

## 2. Input/Output Specifications

### Input Sources:
- **Chat Input:** `{{ $json.chatInput }}` (German text from human supervisor)
- **Email Trigger:** Complete email objects from IMAP trigger
- **Sub-agent Responses:** JSON responses from called sub-agents

### Output Format:
- **Tool Calls:** Single n8n tool call per request
- **Direct Response:** Plain German text (only when no tools needed)
- **Error Messages:** Structured German error responses

## 3. Available Tools & Call Signatures

### Core Sub-Agents:
```json
{
  "diagnose_agent": {
    "trigger": "symptoms, medical questions, diagnosis requests",
    "call_format": {
      "patient_id": "string (optional)",
      "symptoms": "string (required)",
      "context": "string (optional)",
      "images": "array of strings (optional)"
    }
  },
  "chat_agent": {
    "trigger": "general questions, status updates, confirmations",
    "call_format": {
      "message": "string (required)",
      "context": "string (optional)"
    }
  },
  "email_agent": {
    "trigger": "email processing, email composition",
    "call_format": {
      "action": "process|compose|reply",
      "email_data": "object (required)",
      "recipient": "string (for compose)",
      "content": "string (for compose)"
    }
  },
  "calendar_agent": {
    "trigger": "appointments, scheduling, calendar queries",
    "call_format": {
      "action": "schedule|reschedule|cancel|query",
      "appointment_data": "object (required)"
    }
  },
  "contacts_agent": {
    "trigger": "patient/contact management",
    "call_format": {
      "action": "create|update|search|delete",
      "contact_data": "object (required)"
    }
  },
  "writer_agent": {
    "trigger": "document creation, reports, letters",
    "call_format": {
      "document_type": "string (required)",
      "content_data": "object (required)",
      "template": "string (optional)"
    }
  },
  "inventory_agent": {
    "trigger": "stock, orders, inventory management",
    "call_format": {
      "action": "check|order|update",
      "item_data": "object (required)"
    }
  }
}
```

### Utility Tools:
```json
{
  "calculator": {
    "trigger": "BMI, dosage, medical calculations",
    "call_format": {
      "formula": "string (required)",
      "inputs": "object (required)"
    }
  },
  "RAG_learn": {
    "trigger": "store validated medical knowledge",
    "call_format": {
      "text_for_embedding": "string (required)",
      "medical_context": "object (required)",
      "validated_by_human": "boolean (required)",
      "tags": "array of strings (optional)"
    }
  }
}
```

## 4. Decision Logic & Routing Rules

### Classification Keywords:
- **Medical/Symptoms:** "Symptome", "Diagnose", "Hautproblem", "Ausschlag", "Juckreiz", "Schmerzen"
- **Appointments:** "Termin", "Termine", "Kalendar", "buchen", "verschieben", "absagen"
- **Email:** "Email", "E-Mail", "Mail", "schreiben", "antworten", "weiterleiten"
- **Contacts:** "Patient", "Kontakt", "Adresse", "Telefon", "hinzufügen"
- **Documents:** "Brief", "Bericht", "Dokument", "schreiben", "erstellen"
- **Inventory:** "Lager", "Bestellung", "Medikament", "Vorrat", "bestellen"
- **Calculations:** "berechnen", "BMI", "Dosierung", "rechnen"
- **General Chat:** "Status", "Hallo", "Danke", "Wie geht", "Was ist"

### Routing Priority:
1. **Medical symptoms → diagnose_agent** (highest priority)
2. **Email content → email_agent**
3. **Appointment keywords → calendar_agent**
4. **Contact management → contacts_agent**
5. **Document requests → writer_agent**
6. **Inventory requests → inventory_agent**
7. **Calculations → calculator**
8. **Everything else → chat_agent** (lowest priority)

## 5. Error Handling & Validation

### Input Validation:
- Check for required fields in tool calls
- Validate JSON structure from sub-agents
- Ensure German language output to human

### Error Responses:
```json
{
  "error_type": "validation|tool_failure|system_error",
  "message_de": "German error message for human",
  "internal_details": "English technical details for logs",
  "suggested_action": "retry|escalate|manual_intervention"
}
```

## 6. Memory & Context Management

### Session Context:
- **Current Request ID:** Unique identifier per interaction
- **Recent Tool Calls:** Last 3 tool invocations for context
- **Patient Context:** Active patient_id if applicable
- **Conversation State:** Ongoing vs new conversation

### Context Passing:
- Always include relevant context when calling sub-agents
- Maintain patient_id consistency across tool chain
- Pass previous responses for multi-step operations

## 7. Performance Requirements

- **Response Time:** < 500ms for routing decisions
- **Tool Call Limit:** Maximum 1 tool call per input (except diagnose→chat chains)
- **Error Recovery:** Automatic retry once, then escalate
- **Concurrent Handling:** Queue requests if multiple arrive simultaneously

## 8. Integration Specifications

### n8n Workflow Integration:
- **Webhook Endpoints:** Chat input, Email IMAP trigger
- **Tool Nodes:** One node per sub-agent
- **Error Handling:** Dedicated error branch for each tool
- **Monitoring:** Health check endpoints for all sub-agents

### FastAPI/Supabase Connection:
- **Base URL:** http://localhost:8000
- **Health Check:** GET /health
- **RAG Tools:** Available via sub-agents (DiagnoseAgent uses RAG_read)
- **Data Storage:** All persistent data via Supabase tables

## 9. Security & Compliance

### GDPR Compliance:
- No patient data in logs (use hashed IDs)
- All personal data stays within local network
- Audit trail for all tool calls

### Access Control:
- SupervisorAgent has access to all sub-agents
- Sub-agents cannot call each other directly
- All external communication in German only