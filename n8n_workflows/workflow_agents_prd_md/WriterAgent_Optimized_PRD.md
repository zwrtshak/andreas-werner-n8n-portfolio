# WriterAgent - Optimized PRD for JSON Prompt Creation

**Version:** 2.0  
**Date:** 2025-07-31  
**Target:** n8n workflow with Ollama Mistral + optional RAG integration

## 1. Agent Identity & Core Function

**Agent Name:** WriterAgent  
**Agent Type:** Medical Content Generation Specialist  
**Primary Role:** Create structured medical documents, correspondence, and reports  
**Authority Level:** Sub-agent under SupervisorAgent control  
**Content Scope:** Medical documentation, patient communication, administrative documents

## 2. Input/Output Specifications

### Input Format from SupervisorAgent:
```json
{
  "task_type": "string (generate|edit|format|translate)",
  "document_type": "string (case_note|follow_up_letter|discharge_summary|referral_letter|patient_info|blog_post|newsletter|internal_memo)",
  "content_requirements": {
    "language": "string (de|en, default: de)",
    "tone": "string (formal|professional|patient_friendly|clinical)",
    "format": "string (plain_text|markdown|html|pdf_ready)",
    "length": "string (brief|standard|detailed)",
    "target_audience": "string (patient|doctor|staff|public)"
  },
  "content_data": {
    "patient_info": {
      "patient_id": "string (UUID, optional)",
      "name": "string (for document addressing)",
      "age": "number (optional)",
      "gender": "string (optional)",
      "case_number": "string (optional)"
    },
    "medical_context": {
      "diagnosis": "string (primary diagnosis)",
      "symptoms": "string (symptom description)",
      "treatment": "string (treatment plan)",
      "medications": "array of strings (prescribed medications)",
      "follow_up": "string (follow-up instructions)",
      "prognosis": "string (expected outcome)"
    },
    "administrative_context": {
      "doctor_name": "string (treating physician)",
      "practice_info": "object (practice details)",
      "date": "string (document date)",
      "reference_number": "string (internal reference)"
    }
  },
  "template_preferences": {
    "use_template": "string (template_name or 'custom')",
    "custom_sections": "array of strings (custom section requirements)",
    "include_disclaimer": "boolean (medical disclaimer needed)",
    "letterhead": "boolean (include practice letterhead)"
  },
  "special_instructions": {
    "emphasis_points": "array of strings (important points to highlight)",
    "avoid_terms": "array of strings (terms to avoid)",
    "required_phrases": "array of strings (must include phrases)",
    "confidentiality_level": "string (standard|high|patient_sensitive)"
  }
}
```

### Output Format to SupervisorAgent:
```json
{
  "status": "string (success|error|needs_review)",
  "document_content": {
    "title": "string (document title)",
    "content": "string (main document content)",
    "format": "string (actual output format)",
    "word_count": "number (content length)",
    "sections": [
      {
        "section_name": "string",
        "content": "string",
        "order": "number"
      }
    ]
  },
  "metadata": {
    "document_id": "string (generated UUID)",
    "template_used": "string (template reference)",
    "language": "string (actual output language)",
    "creation_date": "string (ISO timestamp)",
    "estimated_reading_time": "number (minutes)"
  },
  "quality_checks": {
    "medical_accuracy": "string (verified|needs_review|not_applicable)",
    "language_quality": "string (excellent|good|needs_improvement)",
    "completeness": "string (complete|missing_info|partial)",
    "compliance": "string (gdpr_compliant|needs_review)"
  },
  "suggestions": [
    {
      "type": "string (improvement|addition|correction)",
      "section": "string (which section)",
      "suggestion": "string (specific suggestion)"
    }
  ],
  "requires_approval": "boolean (if human review needed)"
}
```

## 3. Document Types & Templates

### Medical Documentation:
```json
{
  "case_note": {
    "structure": ["patient_info", "chief_complaint", "examination", "diagnosis", "treatment_plan", "follow_up"],
    "tone": "clinical",
    "audience": "medical_staff",
    "typical_length": "200-500 words"
  },
  "discharge_summary": {
    "structure": ["patient_info", "admission_reason", "course_of_treatment", "discharge_condition", "medications", "follow_up_care"],
    "tone": "formal",
    "audience": "referring_doctor",
    "typical_length": "300-800 words"
  },
  "referral_letter": {
    "structure": ["patient_info", "reason_for_referral", "relevant_history", "current_findings", "requested_consultation"],
    "tone": "professional",
    "audience": "specialist",
    "typical_length": "200-400 words"
  },
  "follow_up_letter": {
    "structure": ["greeting", "appointment_summary", "treatment_progress", "next_steps", "contact_info"],
    "tone": "patient_friendly",
    "audience": "patient",
    "typical_length": "150-300 words"
  }
}
```

### Patient Communication:
```json
{
  "patient_information_sheet": {
    "structure": ["condition_explanation", "treatment_options", "risks_benefits", "aftercare", "contact_info"],
    "tone": "patient_friendly",
    "language_level": "accessible",
    "include_illustrations": "optional"
  },
  "appointment_confirmation": {
    "structure": ["greeting", "appointment_details", "preparation_instructions", "contact_info"],
    "tone": "professional_friendly",
    "format": "email_template"
  },
  "treatment_instructions": {
    "structure": ["medication_schedule", "application_instructions", "side_effects", "when_to_contact"],
    "tone": "clear_instructional",
    "format": "structured_list"
  }
}
```

### Administrative Documents:
```json
{
  "internal_memo": {
    "structure": ["subject", "background", "key_points", "action_items", "deadline"],
    "tone": "professional",
    "audience": "staff",
    "format": "structured"
  },
  "newsletter_article": {
    "structure": ["headline", "introduction", "main_content", "call_to_action"],
    "tone": "engaging",
    "audience": "patients",
    "include_seo": "optional"
  },
  "blog_post": {
    "structure": ["title", "introduction", "main_sections", "conclusion", "references"],
    "tone": "informative_accessible",
    "audience": "general_public",
    "seo_optimized": true
  }
}
```

## 4. Content Generation Logic

### Medical Accuracy Safeguards:
```json
{
  "diagnosis_statements": {
    "rule": "Only include diagnoses from DiagnoseAgent or verified sources",
    "disclaimer": "Always include preliminary/educational disclaimers",
    "verification": "Cross-reference with medical knowledge base"
  },
  "treatment_recommendations": {
    "rule": "Only suggest treatments from approved protocols",
    "consultation_note": "Include 'consult healthcare provider' for medical advice",
    "contraindications": "Mention when professional consultation is required"
  },
  "medication_information": {
    "dosage": "Never specify dosages without physician approval",
    "side_effects": "Include standard side effect information",
    "interactions": "Warn about potential drug interactions"
  }
}
```

### Language & Style Guidelines:
```json
{
  "german_medical_terminology": {
    "prefer_german_terms": "Use German medical terms when appropriate",
    "explain_technical_terms": "Define complex medical terms for patients",
    "formal_address": "Use 'Sie' form for patient communication"
  },
  "tone_adaptation": {
    "patient_friendly": "Empathetic, reassuring, clear explanations",
    "professional": "Precise, clinical, evidence-based",
    "clinical": "Objective, detailed, technical accuracy",
    "formal": "Official, structured, comprehensive"
  },
  "readability_optimization": {
    "patient_documents": "Max 8th grade reading level",
    "professional_documents": "Medical professional level",
    "sentence_length": "Vary sentence length, average 15-20 words",
    "paragraph_structure": "Clear topic sentences, logical flow"
  }
}
```

## 5. Quality Assurance & Compliance

### Content Review Process:
```json
{
  "automated_checks": {
    "spell_check": "German spell checking",
    "grammar_check": "Basic grammar validation",
    "formatting_check": "Consistent formatting standards",
    "completeness_check": "All required sections present"
  },
  "medical_review_triggers": {
    "diagnosis_mentioned": "Flag for medical review",
    "treatment_specified": "Require physician approval",
    "medication_named": "Verify dosage and contraindications",
    "emergency_symptoms": "Include emergency contact info"
  },
  "compliance_checks": {
    "gdpr_compliance": "No unnecessary personal data",
    "medical_disclaimers": "Appropriate disclaimers included",
    "practice_branding": "Consistent with practice standards",
    "legal_requirements": "Meets medical documentation standards"
  }
}
```

### Error Prevention:
```json
{
  "sensitive_content_detection": {
    "personal_health_data": "Flag PHI in wrong context",
    "confidential_information": "Prevent accidental disclosure",
    "incorrect_patient_data": "Verify patient-document matching"
  },
  "factual_accuracy": {
    "medical_facts": "Cross-reference with knowledge base",
    "practice_information": "Use current practice details",
    "contact_information": "Verify current contact details"
  }
}
```

## 6. Template System

### Template Structure:
```json
{
  "template_components": {
    "header": {
      "practice_letterhead": "boolean",
      "document_title": "string",
      "date": "auto_generated",
      "reference_number": "optional"
    },
    "content_sections": {
      "variable_sections": "array (customizable content areas)",
      "required_sections": "array (mandatory sections)",
      "optional_sections": "array (conditional sections)"
    },
    "footer": {
      "practice_info": "contact details",
      "disclaimers": "legal/medical disclaimers",
      "confidentiality_notice": "privacy statements"
    }
  },
  "variable_substitution": {
    "patient_variables": "{{patient_name}}, {{patient_age}}, {{case_number}}",
    "practice_variables": "{{doctor_name}}, {{practice_name}}, {{phone}}",
    "date_variables": "{{current_date}}, {{appointment_date}}",
    "medical_variables": "{{diagnosis}}, {{treatment}}, {{medications}}"
  }
}
```

## 7. Integration Specifications

### n8n Workflow Integration:
```json
{
  "input_processing": {
    "content_analysis": "Analyze requirements and select appropriate template",
    "data_validation": "Verify all required data is present",
    "template_selection": "Choose or customize template based on document type"
  },
  "content_generation": {
    "text_generation": "Generate content using Ollama Mistral",
    "template_rendering": "Apply template structure and formatting",
    "variable_substitution": "Replace template variables with actual data"
  },
  "quality_control": {
    "automated_review": "Run quality checks and compliance validation",
    "human_review_flagging": "Identify documents requiring human approval",
    "version_control": "Track document versions and changes"
  }
}
```

### RAG Integration (Optional):
```json
{
  "knowledge_enhancement": {
    "medical_context": "Query RAG_read for relevant medical information",
    "template_improvement": "Use learned patterns from RAG_learn",
    "content_validation": "Cross-reference generated content with knowledge base"
  },
  "continuous_learning": {
    "approved_documents": "Store successful documents in RAG_learn",
    "correction_patterns": "Learn from human corrections",
    "style_preferences": "Adapt to practice writing style preferences"
  }
}
```

## 8. Performance Requirements

- **Generation Time:** < 5 seconds for standard documents, < 10 seconds for complex documents
- **Quality Score:** > 90% accuracy for automated quality checks
- **Template Coverage:** Support for 15+ common document types
- **Language Quality:** Native-level German with medical terminology accuracy
- **Compliance Rate:** 100% GDPR compliance for patient-facing documents
- **Revision Rate:** < 20% of documents require human revision

## 9. Security & Privacy

### Data Protection:
- **Content Encryption:** Encrypt generated documents containing PHI
- **Access Logging:** Log all document generation and access
- **Secure Storage:** Temporary files securely deleted after processing
- **Audit Trail:** Complete traceability of document creation process

### Content Security:
- **Input Sanitization:** Prevent injection attacks in template processing
- **Output Validation:** Ensure no malicious content in generated documents
- **Template Security:** Secure template storage and version control
- **PHI Protection:** Automatic detection and handling of sensitive data