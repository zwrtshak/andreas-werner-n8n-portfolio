# ContactsAgent - Optimized PRD for JSON Prompt Creation

**Version:** 2.0  
**Date:** 2025-07-31  
**Target:** n8n workflow with Ollama Mistral + Supabase integration

## 1. Agent Identity & Core Function

**Agent Name:** ContactsAgent  
**Agent Type:** Contact Data Management Specialist  
**Primary Role:** Manage all contact information for patients, staff, and external partners  
**Authority Level:** Sub-agent under SupervisorAgent control  
**Scope:** GDPR-compliant contact database management with validation and deduplication

## 2. Input/Output Specifications

### Input Format from SupervisorAgent:
```json
{
  "action": "string (create|update|search|delete|merge|validate)",
  "contact_data": {
    "contact_id": "string (UUID, for update/delete)",
    "type": "string (patient|doctor|staff|pharmacy|lab|insurance|supplier)",
    "personal_info": {
      "title": "string (Dr.|Herr|Frau|optional)",
      "first_name": "string (required)",
      "last_name": "string (required)",
      "date_of_birth": "string (YYYY-MM-DD, for patients)",
      "gender": "string (m|f|d|optional)"
    },
    "contact_details": {
      "email": "string (validated email format)",
      "phone_primary": "string (validated phone format)",
      "phone_secondary": "string (optional)",
      "fax": "string (optional)",
      "website": "string (optional)"
    },
    "address": {
      "street": "string",
      "house_number": "string",
      "postal_code": "string",
      "city": "string",
      "country": "string (default: Deutschland)"
    },
    "medical_info": {
      "insurance_number": "string (patients only)",
      "insurance_company": "string (patients only)",
      "allergies": "array of strings (optional)",
      "emergency_contact": "object (optional)"
    },
    "professional_info": {
      "company": "string (for business contacts)",
      "department": "string (optional)",
      "position": "string (optional)",
      "specialization": "string (for doctors)"
    },
    "preferences": {
      "communication_language": "string (de|en, default: de)",
      "preferred_contact_method": "string (email|phone|mail)",
      "newsletter_consent": "boolean",
      "data_processing_consent": "boolean (required)"
    },
    "notes": "string (internal notes, optional)"
  },
  "search_criteria": {
    "query": "string (name, email, or phone search)",
    "type_filter": "string (optional contact type filter)",
    "fuzzy_match": "boolean (enable approximate matching)",
    "limit": "number (max results, default: 10)"
  }
}
```

### Output Format to SupervisorAgent:
```json
{
  "status": "string (success|error|warning|duplicate_found)",
  "message": "string (German status message)",
  "contact_id": "string (UUID for created/updated contact)",
  "contact_details": {
    "contact_id": "string",
    "full_name": "string (formatted display name)",
    "type": "string",
    "email": "string",
    "phone_primary": "string",
    "address_formatted": "string (single line address)",
    "last_updated": "string (ISO timestamp)"
  },
  "search_results": [
    {
      "contact_id": "string",
      "full_name": "string",
      "type": "string",
      "email": "string",
      "phone_primary": "string",
      "match_score": "number (0.0-1.0, for fuzzy search)"
    }
  ],
  "duplicates_detected": [
    {
      "existing_contact_id": "string",
      "similarity_score": "number (0.0-1.0)",
      "matching_fields": "array of strings",
      "merge_suggested": "boolean"
    }
  ],
  "validation_errors": [
    {
      "field": "string (field name with error)",
      "error_type": "string (format|required|invalid)",
      "message": "string (German error message)"
    }
  ]
}
```

## 3. Contact Management Logic

### Contact Types & Required Fields:
```json
{
  "patient": {
    "required": ["first_name", "last_name", "date_of_birth", "phone_primary"],
    "optional": ["email", "address", "insurance_number", "allergies"],
    "gdpr_sensitive": true,
    "retention_period": "10 years after last contact"
  },
  "doctor": {
    "required": ["title", "first_name", "last_name", "specialization"],
    "optional": ["email", "phone_primary", "company"],
    "gdpr_sensitive": false,
    "retention_period": "indefinite"
  },
  "staff": {
    "required": ["first_name", "last_name", "email", "position"],
    "optional": ["phone_primary", "department"],
    "gdpr_sensitive": true,
    "retention_period": "2 years after employment end"
  },
  "pharmacy": {
    "required": ["company", "phone_primary", "address"],
    "optional": ["email", "fax", "contact_person"],
    "gdpr_sensitive": false,
    "retention_period": "indefinite"
  },
  "lab": {
    "required": ["company", "email", "address"],
    "optional": ["phone_primary", "fax", "specialization"],
    "gdpr_sensitive": false,
    "retention_period": "indefinite"
  },
  "insurance": {
    "required": ["company", "phone_primary"],
    "optional": ["email", "address", "contact_person"],
    "gdpr_sensitive": false,
    "retention_period": "indefinite"
  }
}
```

### Validation Rules:
```json
{
  "email": {
    "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
    "message": "Bitte geben Sie eine gültige E-Mail-Adresse ein."
  },
  "phone": {
    "pattern": "^(\\+49|0)[1-9][0-9]{1,14}$",
    "message": "Bitte geben Sie eine gültige deutsche Telefonnummer ein."
  },
  "postal_code": {
    "pattern": "^[0-9]{5}$",
    "message": "Postleitzahl muss 5 Ziffern haben."
  },
  "date_of_birth": {
    "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}$",
    "validate_age": "must be between 0 and 120 years",
    "message": "Geburtsdatum im Format YYYY-MM-DD eingeben."
  }
}
```

### Duplicate Detection Algorithm:
1. **Exact Match:** Same email or phone number
2. **Name Similarity:** Levenshtein distance < 2 for full name
3. **Address Match:** Same street + house number + postal code
4. **Combined Score:** Weighted combination of all matches
5. **Threshold:** Similarity > 0.85 triggers duplicate warning

## 4. Database Integration

### Supabase Tables:
```json
{
  "contacts": {
    "fields": [
      "id (UUID PRIMARY KEY)",
      "type (VARCHAR)",
      "title (VARCHAR)",
      "first_name (VARCHAR)",
      "last_name (VARCHAR)",
      "date_of_birth (DATE)",
      "gender (CHAR(1))",
      "email (VARCHAR UNIQUE)",
      "phone_primary (VARCHAR)",
      "phone_secondary (VARCHAR)",
      "address_street (VARCHAR)",
      "address_house_number (VARCHAR)",
      "address_postal_code (VARCHAR)",
      "address_city (VARCHAR)",
      "address_country (VARCHAR DEFAULT 'Deutschland')",
      "company (VARCHAR)",
      "position (VARCHAR)",
      "specialization (VARCHAR)",
      "notes (TEXT)",
      "data_processing_consent (BOOLEAN)",
      "created_at (TIMESTAMP)",
      "updated_at (TIMESTAMP)",
      "last_contact (TIMESTAMP)"
    ],
    "indexes": ["email", "phone_primary", "last_name", "type", "postal_code"]
  },
  "contact_medical_info": {
    "fields": [
      "contact_id (UUID FOREIGN KEY)",
      "insurance_number (VARCHAR)",
      "insurance_company (VARCHAR)",
      "allergies (JSON)",
      "emergency_contact (JSON)"
    ],
    "purpose": "Separate table for sensitive medical data"
  },
  "contact_preferences": {
    "fields": [
      "contact_id (UUID FOREIGN KEY)",
      "communication_language (VARCHAR)",
      "preferred_contact_method (VARCHAR)",
      "newsletter_consent (BOOLEAN)",
      "consent_date (TIMESTAMP)"
    ],
    "purpose": "GDPR compliance tracking"
  }
}
```

### Search Optimization:
```sql
-- Full-text search with ranking
SELECT *, 
  ts_rank(search_vector, plainto_tsquery('german', ?)) as rank
FROM contacts 
WHERE search_vector @@ plainto_tsquery('german', ?)
ORDER BY rank DESC;

-- Fuzzy name matching
SELECT *, 
  similarity(first_name || ' ' || last_name, ?) as name_similarity
FROM contacts 
WHERE similarity(first_name || ' ' || last_name, ?) > 0.3
ORDER BY name_similarity DESC;
```

## 5. GDPR Compliance Features

### Data Protection Measures:
```json
{
  "consent_management": {
    "data_processing_consent": "Required for all personal data",
    "newsletter_consent": "Separate opt-in for marketing",
    "consent_withdrawal": "Easy opt-out mechanism",
    "consent_documentation": "Log all consent changes"
  },
  "data_rights": {
    "right_of_access": "Export all data for contact",
    "right_to_rectification": "Update incorrect data",
    "right_to_erasure": "Delete contact and all related data",
    "right_to_portability": "Export data in structured format"
  },
  "data_minimization": {
    "purpose_limitation": "Only collect necessary data",
    "retention_limits": "Auto-delete after retention period",
    "access_logging": "Log all data access attempts"
  }
}
```

### Anonymization Process:
```json
{
  "patient_anonymization": {
    "trigger": "10 years after last contact",
    "process": [
      "Replace name with 'Anonymisiert + ID'",
      "Clear address and contact details",
      "Retain only medical statistics (anonymized)",
      "Log anonymization process"
    ]
  },
  "staff_anonymization": {
    "trigger": "2 years after employment end",
    "process": [
      "Replace personal details",
      "Retain professional role for auditing",
      "Clear private contact information"
    ]
  }
}
```

## 6. Error Handling & Quality Control

### Input Validation Errors:
```json
{
  "required_field_missing": "Pflichtfeld '{field}' ist erforderlich.",
  "invalid_email_format": "E-Mail-Format ist ungültig.",
  "invalid_phone_format": "Telefonnummer-Format ist ungültig.",
  "invalid_date_format": "Datumsformat muss YYYY-MM-DD sein.",
  "age_out_of_range": "Alter muss zwischen 0 und 120 Jahren liegen.",
  "postal_code_invalid": "Postleitzahl muss 5 Ziffern haben."
}
```

### Data Quality Checks:
- **Email Deliverability:** Basic syntax and domain validation
- **Phone Number:** Format validation and country code check  
- **Address Validation:** Postal code and city consistency
- **Duplicate Prevention:** Check before creating new contacts
- **Data Completeness:** Flag incomplete essential information

## 7. Integration Specifications

### n8n Workflow Nodes:
- **Input Node:** Receives JSON from SupervisorAgent
- **Validation Node:** Validates contact data format and rules
- **Duplicate Check Node:** Searches for potential duplicates
- **Database Operation Node:** Performs CRUD operations
- **Search Node:** Executes contact searches with ranking
- **GDPR Compliance Node:** Handles consent and data rights
- **Output Node:** Returns formatted response

### Performance Requirements:
- **Response Time:** < 500ms for searches, < 1 second for operations
- **Search Accuracy:** > 95% relevant results in top 10
- **Duplicate Detection:** > 90% accuracy in identifying duplicates
- **Concurrent Users:** Handle up to 20 simultaneous operations
- **Data Integrity:** 100% consistency in database operations

## 8. Security Measures

### Access Control:
- **Role-Based Access:** Different permissions for different user types
- **Data Encryption:** AES-256 encryption for sensitive fields
- **Audit Logging:** Log all data access and modifications
- **Session Management:** Secure session handling with timeouts

### Privacy Protection:
- **Data Masking:** Hide sensitive data in logs
- **Secure Deletion:** Cryptographic deletion of sensitive data
- **Backup Encryption:** Encrypted backups with key rotation
- **Network Security:** TLS encryption for all data transmission

## 9. Analytics & Reporting

### Contact Statistics:
- Total contacts by type and status
- Growth trends and demographic analysis
- Data quality metrics and completeness scores
- GDPR compliance status and consent rates
- Search usage patterns and popular queries

### Data Health Monitoring:
- Duplicate detection rates
- Data validation error rates
- Contact update frequency
- Inactive contact identification
- Database performance metrics