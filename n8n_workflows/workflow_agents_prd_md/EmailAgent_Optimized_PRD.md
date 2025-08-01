# EmailAgent - Optimized PRD for JSON Prompt Creation

**Version:** 2.0  
**Date:** 2025-07-31  
**Target:** n8n workflow with Ollama Mistral + IMAP integration

## 1. Agent Identity & Core Function

**Agent Name:** EmailAgent  
**Agent Type:** Communication Specialist  
**Primary Role:** Process incoming emails and compose professional responses for dermatology practice  
**Authority Level:** Sub-agent under SupervisorAgent control  
**Communication Scope:** GDPR-compliant email management with medical context awareness

## 2. Input/Output Specifications

### Input Format from SupervisorAgent:
```json
{
  "action": "string (process|compose|reply|forward)",
  "email_data": {
    "id": "string (email UUID)",
    "sender": "string (email address)",
    "recipient": "string (for compose action)",
    "subject": "string",
    "body": "string (full email content)",
    "attachments": [
      {
        "filename": "string",
        "path": "string (local storage path)",
        "mime_type": "string",
        "size": "number (bytes)"
      }
    ],
    "received_at": "string (ISO timestamp)",
    "priority": "string (low|normal|high|urgent)"
  },
  "context": {
    "patient_id": "string (optional)",
    "appointment_reference": "string (optional)",
    "case_type": "string (appointment|prescription|inquiry|complaint|admin)"
  },
  "compose_instructions": {
    "template_type": "string (appointment_confirmation|prescription_ready|general_inquiry)",
    "custom_content": "string (specific content to include)",
    "tone": "string (formal|friendly|medical)"
  }
}
```

### Output Format to SupervisorAgent:
```json
{
  "action_completed": "string (process|compose|reply|forward)",
  "email_id": "string (processed email ID)",
  "classification": {
    "category": "string (appointment|medical_inquiry|prescription|complaint|admin|spam)",
    "urgency": "string (low|normal|high|urgent)",
    "requires_doctor_attention": "boolean",
    "contains_attachments": "boolean",
    "language": "string (de|en|other)"
  },
  "extracted_information": {
    "patient_name": "string (if identifiable)",
    "appointment_request": "object (if applicable)",
    "medical_concern": "string (if applicable)",
    "contact_info": "object (phone, address if provided)",
    "requested_action": "string (what the sender wants)"
  },
  "response_draft": {
    "subject": "string (German response subject)",
    "body": "string (German response body)",
    "attachments_needed": [
      "string (list of attachments to include)"
    ],
    "send_immediately": "boolean",
    "requires_approval": "boolean"
  },
  "routing_recommendations": [
    {
      "agent": "string (calendar_agent|diagnose_agent|contacts_agent)",
      "reason": "string",
      "data_to_pass": "object"
    }
  ],
  "compliance_notes": {
    "gdpr_relevant": "boolean",
    "patient_data_present": "boolean",
    "retention_category": "string (medical|admin|marketing)"
  }
}
```

## 3. Email Processing Logic

### Classification Algorithm:
```json
{
  "appointment_keywords": [
    "Termin", "Terminbuchung", "Sprechstunde", "Vereinbarung",
    "buchen", "verschieben", "absagen", "stornieren"
  ],
  "medical_keywords": [
    "Symptom", "Beschwerden", "Hautproblem", "Ausschlag", 
    "Juckreiz", "Schmerzen", "Behandlung", "Medikament"
  ],
  "prescription_keywords": [
    "Rezept", "Medikament", "Salbe", "Creme", "Verschreibung",
    "abholen", "bereit", "Apotheke"
  ],
  "urgent_keywords": [
    "dringend", "Notfall", "akut", "sofort", "Schmerzen",
    "Blutung", "Schwellung", "Entzündung"
  ],
  "complaint_keywords": [
    "Beschwerde", "unzufrieden", "Problem", "Fehler",
    "schlecht", "Kritik", "beanstanden"
  ]
}
```

### Response Templates:
```json
{
  "appointment_confirmation": {
    "subject": "Terminbestätigung - Praxis Dr. {doctor_name}",
    "body_template": "Sehr geehrte/r {title} {name},\n\nwir bestätigen Ihren Termin am {date} um {time} Uhr.\n\nBitte bringen Sie folgende Unterlagen mit:\n- Versichertenkarte\n- Überweisungsschein (falls vorhanden)\n- Liste aktueller Medikamente\n\nBei Fragen stehen wir Ihnen gerne zur Verfügung.\n\nMit freundlichen Grüßen\nIhr Praxisteam"
  },
  "prescription_ready": {
    "subject": "Rezept abholbereit - Praxis Dr. {doctor_name}",
    "body_template": "Sehr geehrte/r {title} {name},\n\nIhr Rezept ist abholbereit. Sie können es zu unseren Öffnungszeiten in der Praxis abholen:\n\n{opening_hours}\n\nBitte bringen Sie Ihre Versichertenkarte mit.\n\nMit freundlichen Grüßen\nIhr Praxisteam"
  },
  "medical_inquiry_response": {
    "subject": "Re: {original_subject}",
    "body_template": "Sehr geehrte/r {title} {name},\n\nvielen Dank für Ihre Anfrage. {custom_response}\n\nFür eine genaue Diagnose und Behandlung empfehlen wir Ihnen, einen Termin zu vereinbaren.\n\nHinweis: Diese Nachricht ersetzt nicht die persönliche ärztliche Beratung.\n\nMit freundlichen Grüßen\nIhr Praxisteam"
  },
  "general_inquiry": {
    "subject": "Re: {original_subject}",
    "body_template": "Sehr geehrte/r {title} {name},\n\nvielen Dank für Ihre Nachricht. {custom_response}\n\nBei weiteren Fragen stehen wir Ihnen gerne zur Verfügung.\n\nMit freundlichen Grüßen\nIhr Praxisteam"
  }
}
```

## 4. Attachment Handling

### Supported File Types:
```json
{
  "medical_images": [".jpg", ".jpeg", ".png", ".tiff", ".bmp"],
  "documents": [".pdf", ".doc", ".docx", ".txt"],
  "lab_results": [".pdf", ".xml", ".hl7"],
  "max_size": "10MB per attachment",
  "virus_scan": "required for all attachments",
  "storage_path": "/secure/attachments/{email_id}/"
}
```

### Processing Rules:
- **Medical Images:** Forward to DiagnoseAgent for analysis
- **Lab Results:** Store securely, flag for doctor review
- **Insurance Documents:** Forward to admin processing
- **Suspicious Files:** Quarantine and alert supervisor

## 5. GDPR Compliance & Security

### Data Protection Measures:
```json
{
  "patient_data_handling": {
    "encryption": "AES-256 for stored emails",
    "anonymization": "Remove/hash personal identifiers in logs",
    "retention": "7 years for medical, 3 years for admin",
    "access_control": "Role-based, logged access only"
  },
  "privacy_controls": {
    "data_minimization": "Only process necessary information",
    "consent_tracking": "Log patient communication preferences",
    "right_to_erasure": "Support deletion requests",
    "data_portability": "Export patient communication history"
  }
}
```

### Compliance Checks:
- Scan for personal health information (PHI)
- Flag emails requiring special handling
- Auto-apply retention policies
- Generate audit trails for all actions

## 6. Error Handling & Quality Control

### Input Validation:
- Verify email format and structure
- Check attachment security and format
- Validate sender authenticity (SPF/DKIM)
- Detect spam and phishing attempts

### Response Quality Checks:
- Grammar and spelling validation
- Medical accuracy review for clinical content
- Tone appropriateness for medical practice
- Compliance with practice communication guidelines

### Fallback Procedures:
- Malformed emails → Forward to manual review
- Spam detection → Quarantine with notification
- System errors → Log and retry with exponential backoff
- Unknown classification → Route to ChatAgent for human review

## 7. Integration Specifications

### n8n Workflow Integration:
```json
{
  "input_nodes": [
    "IMAP Email Trigger",
    "SupervisorAgent Tool Call"
  ],
  "processing_nodes": [
    "Email Classification",
    "Content Extraction", 
    "Response Generation",
    "Attachment Processing"
  ],
  "output_nodes": [
    "SMTP Send Email",
    "Database Storage",
    "SupervisorAgent Response"
  ],
  "error_handling": [
    "Retry Logic",
    "Dead Letter Queue",
    "Manual Review Trigger"
  ]
}
```

### Database Integration:
```json
{
  "email_archive": {
    "table": "emails",
    "fields": ["id", "sender", "subject", "body_hash", "classification", "status", "created_at"],
    "indexes": ["sender", "classification", "created_at"]
  },
  "attachments": {
    "table": "email_attachments", 
    "fields": ["id", "email_id", "filename", "file_path", "mime_type", "virus_scan_result"],
    "storage": "secure local filesystem"
  },
  "templates": {
    "table": "email_templates",
    "fields": ["id", "name", "subject_template", "body_template", "language", "active"],
    "versioning": "track template changes"
  }
}
```

## 8. Performance Requirements

- **Processing Time:** < 2 seconds per email
- **Concurrent Emails:** Handle up to 20 simultaneous emails
- **Template Generation:** < 500ms for standard responses
- **Attachment Processing:** < 5 seconds for images, < 10 seconds for documents
- **Storage Efficiency:** Compress and deduplicate attachments

## 9. Monitoring & Analytics

### Key Metrics:
- Email classification accuracy
- Response time distribution  
- Template usage statistics
- Error rates by email type
- Patient satisfaction scores

### Alerting:
- High volume of unclassified emails
- Repeated processing failures
- Potential security threats detected
- GDPR compliance violations