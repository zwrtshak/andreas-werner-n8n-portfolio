# DiagnoseAgent - Optimized PRD for JSON Prompt Creation

**Version:** 2.0  
**Date:** 2025-07-31  
**Target:** n8n workflow with Ollama Mistral + FastAPI/Supabase RAG integration

## 1. Agent Identity & Core Function

**Agent Name:** DiagnoseAgent  
**Agent Type:** Medical Analysis Specialist  
**Primary Role:** Analyze symptoms and provide evidence-based preliminary diagnoses using RAG knowledge base  
**Authority Level:** Sub-agent under SupervisorAgent control  
**Medical Scope:** Dermatology focus with general medical knowledge support

## 2. Input/Output Specifications

### Input Format from SupervisorAgent:
```json
{
  "patient_id": "string (optional - UUID format)",
  "symptoms": "string (required - German symptom description)",
  "context": "string (optional - age, medical history, duration)",
  "images": "array of strings (optional - local image paths)",
  "severity_level": "string (optional - low|medium|high|urgent)"
}
```

### Output Format to SupervisorAgent:
```json
{
  "patient_id": "string (same as input or generated)",
  "diagnosis": [
    {
      "condition": "string (medical condition name in German)",
      "confidence": "number (0.0-1.0)",
      "icd10_code": "string (optional)",
      "recommended_actions": [
        "string (therapy recommendations in German)"
      ],
      "urgency": "string (routine|urgent|emergency)",
      "follow_up": "string (timeline for follow-up)"
    }
  ],
  "source": "string (dermarules|learned_rules|combined)",
  "rag_matches": [
    {
      "text_snippet": "string",
      "confidence": "number",
      "source_table": "string"
    }
  ],
  "differential_diagnoses": [
    "string (alternative diagnoses if confidence < 0.8)"
  ],
  "additional_questions": [
    "string (questions to clarify diagnosis if needed)"
  ],
  "comments": "string (additional medical context or notes)"
}
```

## 3. Available Tools & Integration

### RAG Tools (via FastAPI localhost:8000):
```json
{
  "RAG_read": {
    "endpoint": "POST /rag/search",
    "purpose": "Query dermatology knowledge base",
    "input": {
      "query": "string (symptom description for embedding)",
      "table": "string (dermarules|learned_rules|both)",
      "limit": "number (default: 5)",
      "min_similarity": "number (default: 0.7)"
    },
    "returns": {
      "matches": [
        {
          "content": "string",
          "similarity": "number",
          "metadata": "object"
        }
      ]
    }
  },
  "RAG_learn": {
    "endpoint": "POST /rag/learn",
    "purpose": "Store validated diagnoses for future learning",
    "input": {
      "text_for_embedding": "string (symptoms + diagnosis)",
      "diagnosis_data": "object (structured diagnosis info)",
      "validated_by_human": "boolean (true for confirmed cases)",
      "tags": "array (dermatology, diagnosis, treatment)",
      "patient_context": "object (anonymized context)",
      "confidence_score": "number"
    },
    "usage": "Call after human validation of diagnosis"
  }
}
```

### Medical Knowledge Sources:
- **Primary:** dermarules table (verified dermatology guidelines)
- **Secondary:** learned_rules table (human-validated cases)
- **Fallback:** General medical knowledge from model training

## 4. Diagnostic Logic & Process

### Step-by-Step Workflow:
1. **Input Validation:** Check for required symptoms field, validate format
2. **Symptom Processing:** Extract key medical terms, normalize language
3. **RAG Query Strategy:**
   - First: Query dermarules with symptom keywords
   - If confidence < 0.8: Query learned_rules
   - If still < 0.7: Query combined view
4. **Analysis & Scoring:** 
   - Weight RAG matches by similarity scores
   - Consider symptom severity and context
   - Generate confidence score for diagnosis
5. **Response Generation:**
   - Primary diagnosis (highest confidence)
   - Alternative diagnoses if confidence < 0.8
   - Specific treatment recommendations
   - Urgency assessment

### Confidence Thresholds:
- **High Confidence (≥0.8):** Single diagnosis with clear recommendations
- **Medium Confidence (0.6-0.79):** Primary diagnosis + 1-2 alternatives
- **Low Confidence (<0.6):** Multiple possibilities + request for more information
- **Emergency Keywords:** "Notfall", "akut", "schwer" → immediate urgent flag

## 5. Medical Decision Support

### Diagnosis Categories:
```json
{
  "dermatological": [
    "Ekzem", "Dermatitis", "Psoriasis", "Akne", "Rosacea", 
    "Pilzinfektion", "Warzen", "Basaliom", "Melanom"
  ],
  "infectious": [
    "Herpes", "Gürtelrose", "Impetigo", "Cellulitis"
  ],
  "allergic": [
    "Kontaktallergie", "Urtikaria", "Angioödem"
  ],
  "autoimmune": [
    "Lupus", "Sklerodermie", "Pemphigus"
  ]
}
```

### Treatment Recommendations:
```json
{
  "topical": [
    "Corticosteroid-Creme", "Antimykotische Salbe", 
    "Feuchtigkeitscreme", "Antibiotische Salbe"
  ],
  "systemic": [
    "Antihistaminika", "Antibiotika", "Immunsuppressiva"
  ],
  "lifestyle": [
    "Hautpflege-Routine", "Allergen-Vermeidung", 
    "UV-Schutz", "Stress-Reduktion"
  ]
}
```

## 6. Error Handling & Quality Assurance

### Input Validation Errors:
- Empty symptoms field → Request symptom description
- Invalid patient_id format → Generate new UUID
- Unsupported image format → Process text symptoms only

### RAG Integration Errors:
- FastAPI connection failure → Use model knowledge only
- No RAG matches found → Provide general recommendations
- Low similarity scores → Request additional symptoms

### Medical Safety Checks:
- Emergency keywords → Always flag as urgent
- Skin cancer indicators → Recommend immediate consultation
- Uncertain diagnoses → Multiple options + specialist referral

## 7. Learning & Knowledge Update

### Feedback Integration:
- Human validates diagnosis → Store via RAG_learn
- Incorrect diagnosis → Update learned_rules with correction
- New treatment outcomes → Add to knowledge base

### Continuous Improvement:
- Track diagnosis accuracy over time
- Identify knowledge gaps in RAG database
- Update confidence thresholds based on performance

## 8. Integration Specifications

### n8n Node Requirements:
- **Input Node:** Receives JSON from SupervisorAgent
- **RAG Query Node:** Calls FastAPI RAG_read endpoint
- **Analysis Node:** Processes RAG results + generates diagnosis
- **Output Node:** Returns structured JSON to SupervisorAgent
- **Error Handling:** Fallback paths for all failure modes

### Performance Requirements:
- **Response Time:** < 3 seconds including RAG queries
- **Concurrent Requests:** Handle up to 5 simultaneous diagnoses
- **Memory Usage:** Efficient embedding processing
- **Accuracy Target:** >85% preliminary diagnosis accuracy

## 9. Security & Compliance

### Patient Data Protection:
- Never log patient symptoms in plain text
- Use hashed patient_ids in all logs
- Anonymize data before RAG_learn storage
- GDPR-compliant data retention policies

### Medical Liability:
- Always include disclaimer about preliminary nature
- Recommend professional consultation for serious symptoms
- Clear documentation of AI-assisted diagnosis process
- Audit trail for all diagnostic decisions