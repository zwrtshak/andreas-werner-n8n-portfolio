# CalendarAgent - Optimized PRD for JSON Prompt Creation

**Version:** 2.0  
**Date:** 2025-07-31  
**Target:** n8n workflow with Ollama Mistral + Supabase integration

## 1. Agent Identity & Core Function

**Agent Name:** CalendarAgent  
**Agent Type:** Appointment Management Specialist  
**Primary Role:** Handle all appointment scheduling, modification, and calendar operations  
**Authority Level:** Sub-agent under SupervisorAgent control  
**Scope:** Medical practice appointment management with conflict resolution

## 2. Input/Output Specifications

### Input Format from SupervisorAgent:
```json
{
  "action": "string (create|update|cancel|query|check_availability)",
  "appointment_data": {
    "appointment_id": "string (UUID, for update/cancel)",
    "patient_id": "string (UUID, required for create)",
    "patient_name": "string (for display purposes)",
    "date": "string (YYYY-MM-DD)",
    "time": "string (HH:MM)",
    "duration": "number (minutes, default: 30)",
    "appointment_type": "string (consultation|follow_up|procedure|emergency)",
    "doctor": "string (Dr. Name or ID)",
    "room": "string (Room 1|Room 2|Treatment Room)",
    "reason": "string (appointment description)",
    "notes": "string (internal notes)",
    "priority": "string (routine|urgent|emergency)"
  },
  "query_parameters": {
    "date_range": "object (start_date, end_date)",
    "doctor_filter": "string (optional)",
    "appointment_type_filter": "string (optional)"
  }
}
```

### Output Format to SupervisorAgent:
```json
{
  "status": "string (success|error|conflict|pending)",
  "message": "string (German status message)",
  "appointment_details": {
    "appointment_id": "string (UUID)",
    "patient_name": "string",
    "date": "string",
    "time": "string",
    "duration": "number",
    "doctor": "string",
    "room": "string",
    "status": "string (confirmed|tentative|cancelled)"
  },
  "conflicts": [
    {
      "conflict_type": "string (double_booking|doctor_unavailable|room_occupied)",
      "conflicting_appointment": "object (existing appointment details)",
      "suggested_alternatives": [
        {
          "date": "string",
          "time": "string",
          "availability_score": "number (0.0-1.0)"
        }
      ]
    }
  ],
  "query_results": [
    {
      "appointment_id": "string",
      "patient_name": "string",
      "date": "string",
      "time": "string",
      "type": "string",
      "doctor": "string",
      "status": "string"
    }
  ]
}
```

## 3. Appointment Management Logic

### Appointment Types & Duration:
```json
{
  "consultation": {
    "duration": 30,
    "requires_doctor": true,
    "room_type": "consultation"
  },
  "follow_up": {
    "duration": 15,
    "requires_doctor": true,
    "room_type": "consultation"
  },
  "procedure": {
    "duration": 60,
    "requires_doctor": true,
    "room_type": "treatment"
  },
  "biopsy": {
    "duration": 45,
    "requires_doctor": true,
    "room_type": "treatment",
    "requires_equipment": "biopsy_kit"
  },
  "allergy_test": {
    "duration": 90,
    "requires_doctor": false,
    "room_type": "treatment",
    "requires_equipment": "allergy_test_kit"
  },
  "emergency": {
    "duration": 30,
    "requires_doctor": true,
    "priority": "high",
    "can_override_conflicts": true
  }
}
```

### Conflict Resolution:
1. **Doctor Availability Check:** Query doctor schedule for conflicts
2. **Room Availability Check:** Verify room is free during requested time
3. **Equipment Check:** Ensure required equipment is available
4. **Alternative Suggestions:** Propose 3 best alternative time slots
5. **Emergency Override:** Allow emergency appointments to take priority

### Business Rules:
```json
{
  "working_hours": {
    "monday": "08:00-18:00",
    "tuesday": "08:00-18:00",
    "wednesday": "08:00-16:00",
    "thursday": "08:00-18:00",
    "friday": "08:00-16:00",
    "saturday": "closed",
    "sunday": "closed"
  },
  "lunch_break": "12:00-13:00",
  "buffer_time": "5 minutes between appointments",
  "advance_booking": "max 6 months ahead",
  "cancellation_policy": "24 hours notice required"
}
```

## 4. Database Integration

### Supabase Tables:
```json
{
  "appointments": {
    "fields": [
      "id (UUID)", "patient_id (UUID)", "doctor_id (UUID)",
      "date (DATE)", "time (TIME)", "duration (INTEGER)",
      "appointment_type (VARCHAR)", "room (VARCHAR)", 
      "status (VARCHAR)", "reason (TEXT)", "notes (TEXT)",
      "created_at (TIMESTAMP)", "updated_at (TIMESTAMP)"
    ],
    "indexes": ["date", "doctor_id", "patient_id", "status"]
  },
  "doctors": {
    "fields": ["id (UUID)", "name (VARCHAR)", "specialization (VARCHAR)", "active (BOOLEAN)"],
    "purpose": "Doctor availability and scheduling"
  },
  "rooms": {
    "fields": ["id (UUID)", "name (VARCHAR)", "room_type (VARCHAR)", "equipment (JSON)"],
    "purpose": "Room availability and equipment tracking"
  }
}
```

### Query Patterns:
- **Availability Check:** `SELECT * FROM appointments WHERE date = ? AND time OVERLAPS ?`
- **Doctor Schedule:** `SELECT * FROM appointments WHERE doctor_id = ? AND date = ?`
- **Room Booking:** `SELECT * FROM appointments WHERE room = ? AND date = ? AND time OVERLAPS ?`

## 5. Reminder & Notification System

### Reminder Types:
```json
{
  "email_reminder": {
    "timing": "24 hours before",
    "template": "appointment_reminder_email",
    "requires": "patient_email"
  },
  "sms_reminder": {
    "timing": "2 hours before",
    "template": "appointment_reminder_sms",
    "requires": "patient_phone"
  },
  "internal_notification": {
    "timing": "15 minutes before",
    "target": "doctor_calendar",
    "purpose": "preparation_reminder"
  }
}
```

## 6. Error Handling & Validation

### Input Validation:
- **Date Format:** Validate YYYY-MM-DD format
- **Time Format:** Validate HH:MM format within business hours
- **Duration:** Must be multiple of 15 minutes
- **Patient ID:** Must exist in contacts database
- **Doctor:** Must be valid and active

### Error Types:
```json
{
  "validation_errors": {
    "invalid_date": "Ungültiges Datum. Bitte verwenden Sie das Format YYYY-MM-DD.",
    "invalid_time": "Ungültige Uhrzeit. Termine sind nur während der Öffnungszeiten möglich.",
    "past_date": "Termine können nicht in der Vergangenheit gebucht werden.",
    "weekend": "An Wochenenden sind keine Termine möglich."
  },
  "conflict_errors": {
    "double_booking": "Zu dieser Zeit ist bereits ein Termin gebucht.",
    "doctor_unavailable": "Der Arzt ist zu dieser Zeit nicht verfügbar.",
    "room_occupied": "Der Raum ist zu dieser Zeit belegt."
  },
  "system_errors": {
    "database_error": "Datenbankfehler. Bitte versuchen Sie es erneut.",
    "patient_not_found": "Patient nicht gefunden. Bitte überprüfen Sie die Patientendaten."
  }
}
```

## 7. Integration Specifications

### n8n Workflow Nodes:
- **Input Node:** Receives JSON from SupervisorAgent
- **Validation Node:** Validates appointment data and business rules
- **Conflict Check Node:** Queries database for scheduling conflicts
- **Alternative Suggestion Node:** Generates alternative time slots
- **Database Operation Node:** Performs CRUD operations on appointments
- **Notification Node:** Triggers reminder system
- **Output Node:** Returns formatted response to SupervisorAgent

### Performance Requirements:
- **Response Time:** < 1 second for queries, < 2 seconds for operations
- **Concurrent Operations:** Handle up to 10 simultaneous booking requests
- **Data Consistency:** Use database transactions for all write operations
- **Availability:** 99.9% uptime during business hours

## 8. Security & Compliance

### Data Protection:
- **Patient Privacy:** Hash patient IDs in logs
- **GDPR Compliance:** Support data deletion and export requests
- **Access Control:** Only SupervisorAgent can access calendar functions
- **Audit Trail:** Log all appointment changes with timestamps

### Security Measures:
- **Input Sanitization:** Prevent SQL injection in queries
- **Rate Limiting:** Prevent appointment spam/abuse
- **Data Encryption:** Encrypt sensitive appointment data at rest
- **Backup Strategy:** Daily backups with 30-day retention

## 9. Analytics & Reporting

### Key Metrics:
- Appointment booking success rate
- Average time to resolve scheduling conflicts
- Most popular appointment times/types
- Doctor utilization rates
- Cancellation patterns

### Reporting Capabilities:
- Daily/weekly/monthly appointment summaries
- Doctor schedule optimization suggestions
- Peak time analysis for resource planning
- Patient appointment history