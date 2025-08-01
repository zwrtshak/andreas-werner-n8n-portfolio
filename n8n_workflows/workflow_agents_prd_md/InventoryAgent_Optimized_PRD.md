# InventoryAgent - Optimized PRD for JSON Prompt Creation

**Version:** 2.0  
**Date:** 2025-07-31  
**Target:** n8n workflow with Ollama Mistral + Supabase integration

## 1. Agent Identity & Core Function

**Agent Name:** InventoryAgent  
**Agent Type:** Medical Inventory Management Specialist  
**Primary Role:** Monitor stock levels, forecast demand, and manage procurement for dermatology practice  
**Authority Level:** Sub-agent under SupervisorAgent control with human approval requirements  
**Scope:** Medical supplies, medications, equipment, and consumables management

## 2. Input/Output Specifications

### Input Format from SupervisorAgent:
```json
{
  "action": "string (check_inventory|forecast_demand|suggest_orders|update_stock|track_usage|generate_report)",
  "inventory_parameters": {
    "item_filter": "string (optional - specific item or category)",
    "urgency_level": "string (routine|urgent|emergency)",
    "forecast_period": "string (1_month|3_months|6_months)",
    "include_expired": "boolean (include expiring items)",
    "minimum_stock_override": "boolean (ignore minimum thresholds)"
  },
  "stock_update": {
    "item_id": "string (UUID)",
    "quantity_change": "number (positive for additions, negative for usage)",
    "transaction_type": "string (delivery|usage|waste|correction)",
    "batch_number": "string (optional)",
    "expiry_date": "string (YYYY-MM-DD, optional)",
    "notes": "string (optional transaction notes)"
  },
  "order_approval": {
    "order_id": "string (UUID)",
    "approval_status": "string (approved|rejected|modified)",
    "modifications": "object (optional - quantity changes)",
    "authorized_by": "string (human supervisor name)"
  }
}
```

### Output Format to SupervisorAgent:
```json
{
  "status": "string (success|warning|critical|error)",
  "inventory_summary": {
    "total_items": "number",
    "low_stock_items": "number",
    "expired_items": "number",
    "expiring_soon": "number (within 30 days)",
    "total_value": "number (estimated inventory value)"
  },
  "critical_alerts": [
    {
      "item_name": "string",
      "current_stock": "number",
      "minimum_required": "number",
      "alert_type": "string (out_of_stock|low_stock|expired|expiring)",
      "urgency": "string (low|medium|high|critical)",
      "recommended_action": "string"
    }
  ],
  "order_recommendations": {
    "requires_human_approval": "boolean",
    "order_priority": "string (routine|urgent|emergency)",
    "estimated_cost": "number",
    "items_to_order": [
      {
        "item_id": "string",
        "item_name": "string",
        "current_stock": "number",
        "minimum_required": "number",
        "recommended_quantity": "number",
        "unit_cost": "number",
        "supplier": "string",
        "justification": "string (reason for order)",
        "urgency_score": "number (0.0-1.0)"
      }
    ]
  },
  "demand_forecast": {
    "forecast_period": "string",
    "methodology": "string (historical|calendar_based|combined)",
    "confidence_level": "number (0.0-1.0)",
    "predicted_usage": [
      {
        "item_name": "string",
        "predicted_monthly_usage": "number",
        "seasonal_adjustment": "number",
        "appointment_based_forecast": "number"
      }
    ]
  },
  "usage_analytics": {
    "top_consumed_items": "array",
    "cost_analysis": "object",
    "waste_analysis": "object",
    "efficiency_metrics": "object"
  }
}
```

## 3. Inventory Categories & Management Rules

### Medical Supply Categories:
```json
{
  "medications": {
    "subcategories": ["topical_treatments", "oral_medications", "injectable_drugs", "anesthetics"],
    "special_handling": "temperature_controlled|controlled_substances",
    "expiry_monitoring": "strict",
    "minimum_stock_days": 30,
    "reorder_threshold": "70% of minimum stock"
  },
  "medical_devices": {
    "subcategories": ["diagnostic_tools", "surgical_instruments", "monitoring_equipment"],
    "maintenance_required": true,
    "calibration_tracking": true,
    "minimum_stock_days": 60
  },
  "consumables": {
    "subcategories": ["syringes", "gloves", "bandages", "sterilization_supplies"],
    "high_turnover": true,
    "bulk_ordering": "preferred",
    "minimum_stock_days": 14
  },
  "diagnostic_supplies": {
    "subcategories": ["biopsy_needles", "culture_media", "test_kits", "reagents"],
    "procedure_specific": true,
    "expiry_sensitive": true,
    "minimum_stock_days": 45
  },
  "office_supplies": {
    "subcategories": ["forms", "labels", "printer_supplies", "cleaning_supplies"],
    "non_medical": true,
    "minimum_stock_days": 30
  }
}
```

### Stock Level Calculations:
```json
{
  "minimum_stock_calculation": {
    "formula": "(average_daily_usage * lead_time_days) + safety_stock",
    "safety_stock_factor": "0.2 (20% buffer)",
    "seasonal_adjustment": "1.1-1.3 multiplier for high seasons",
    "procedure_forecast_factor": "based on calendar_agent data"
  },
  "reorder_point_calculation": {
    "formula": "minimum_stock * reorder_threshold_percentage",
    "urgency_multipliers": {
      "routine": 1.0,
      "urgent": 1.2,
      "emergency": 1.5
    }
  },
  "order_quantity_optimization": {
    "economic_order_quantity": "minimize total ordering and holding costs",
    "supplier_discounts": "consider bulk pricing tiers",
    "storage_capacity": "respect physical storage limits",
    "expiry_considerations": "avoid over-ordering short-shelf-life items"
  }
}
```

## 4. Demand Forecasting Integration

### Calendar-Based Forecasting:
```json
{
  "appointment_type_mapping": {
    "initial_consultation": {
      "typical_items": ["examination_gloves", "antiseptic", "cotton_swabs"],
      "average_quantities": {"examination_gloves": 2, "antiseptic": "5ml", "cotton_swabs": 5}
    },
    "biopsy_procedure": {
      "typical_items": ["biopsy_needle_4mm", "lidocaine_2%", "suture_material", "dressing"],
      "average_quantities": {"biopsy_needle_4mm": 1, "lidocaine_2%": "2ml", "suture_material": 1, "dressing": 1}
    },
    "allergy_testing": {
      "typical_items": ["allergy_test_kit", "antihistamine", "epinephrine"],
      "average_quantities": {"allergy_test_kit": 1, "antihistamine": 1, "epinephrine": "0.3mg"}
    },
    "follow_up": {
      "typical_items": ["examination_gloves", "prescription_pad"],
      "average_quantities": {"examination_gloves": 1, "prescription_pad": 1}
    },
    "laser_treatment": {
      "typical_items": ["laser_consumables", "cooling_gel", "protective_eyewear"],
      "average_quantities": {"laser_consumables": 1, "cooling_gel": "10ml", "protective_eyewear": 2}
    }
  },
  "seasonal_patterns": {
    "summer_peak": {
      "months": ["June", "July", "August"],
      "increased_items": ["sunscreen_products", "allergy_medications", "insect_bite_treatments"],
      "demand_multiplier": 1.3
    },
    "winter_increase": {
      "months": ["December", "January", "February"],
      "increased_items": ["moisturizers", "eczema_treatments", "vitamin_d"],
      "demand_multiplier": 1.2
    }
  }
}
```

### Historical Usage Analysis:
```json
{
  "trend_analysis": {
    "moving_average": "3-month and 12-month averages",
    "seasonal_decomposition": "identify recurring patterns",
    "growth_trends": "detect increasing/decreasing usage",
    "outlier_detection": "identify and adjust for unusual usage spikes"
  },
  "usage_correlation": {
    "appointment_correlation": "correlate usage with appointment volumes",
    "procedure_correlation": "link supply usage to specific procedures",
    "doctor_preferences": "track individual doctor supply preferences",
    "seasonal_correlation": "identify weather/season-dependent usage"
  }
}
```

## 5. Database Integration

### Supabase Tables:
```json
{
  "inventory_items": {
    "fields": [
      "id (UUID PRIMARY KEY)",
      "item_name (VARCHAR)",
      "category (VARCHAR)",
      "subcategory (VARCHAR)",
      "current_stock (INTEGER)",
      "unit_of_measure (VARCHAR)",
      "minimum_stock (INTEGER)",
      "maximum_stock (INTEGER)",
      "reorder_point (INTEGER)",
      "unit_cost (DECIMAL)",
      "supplier_id (UUID)",
      "storage_location (VARCHAR)",
      "requires_refrigeration (BOOLEAN)",
      "controlled_substance (BOOLEAN)",
      "active (BOOLEAN)",
      "created_at (TIMESTAMP)",
      "updated_at (TIMESTAMP)"
    ],
    "indexes": ["category", "current_stock", "reorder_point", "supplier_id"]
  },
  "inventory_batches": {
    "fields": [
      "id (UUID PRIMARY KEY)",
      "item_id (UUID FOREIGN KEY)",
      "batch_number (VARCHAR)",
      "quantity (INTEGER)",
      "expiry_date (DATE)",
      "purchase_date (DATE)",
      "cost_per_unit (DECIMAL)",
      "supplier_batch_ref (VARCHAR)"
    ],
    "purpose": "Track batches for expiry and cost analysis"
  },
  "inventory_transactions": {
    "fields": [
      "id (UUID PRIMARY KEY)",
      "item_id (UUID FOREIGN KEY)",
      "transaction_type (VARCHAR)",
      "quantity_change (INTEGER)",
      "transaction_date (TIMESTAMP)",
      "reference_id (VARCHAR)",
      "notes (TEXT)",
      "user_id (UUID)"
    ],
    "purpose": "Audit trail for all inventory changes"
  },
  "purchase_orders": {
    "fields": [
      "id (UUID PRIMARY KEY)",
      "order_date (DATE)",
      "supplier_id (UUID)",
      "status (VARCHAR)",
      "total_cost (DECIMAL)",
      "approved_by (VARCHAR)",
      "approval_date (TIMESTAMP)",
      "delivery_date (DATE)",
      "notes (TEXT)"
    ],
    "purpose": "Track purchase orders and approvals"
  },
  "suppliers": {
    "fields": [
      "id (UUID PRIMARY KEY)",
      "company_name (VARCHAR)",
      "contact_person (VARCHAR)",
      "email (VARCHAR)",
      "phone (VARCHAR)",
      "address (TEXT)",
      "payment_terms (VARCHAR)",
      "lead_time_days (INTEGER)",
      "minimum_order_value (DECIMAL)",
      "active (BOOLEAN)"
    ],
    "purpose": "Supplier information and terms"
  }
}
```

## 6. Automated Monitoring & Alerts

### Alert Thresholds:
```json
{
  "stock_alerts": {
    "critical_low": "current_stock < 25% of minimum_stock",
    "low_stock": "current_stock < minimum_stock",
    "reorder_point": "current_stock <= reorder_point",
    "overstock": "current_stock > maximum_stock * 1.2"
  },
  "expiry_alerts": {
    "expired": "expiry_date < current_date",
    "expiring_soon": "expiry_date < current_date + 30 days",
    "expiring_this_week": "expiry_date < current_date + 7 days"
  },
  "cost_alerts": {
    "price_increase": "unit_cost increased > 10% from last order",
    "budget_exceeded": "monthly_spending > budget_threshold",
    "high_waste": "expired_value > 5% of category_value"
  }
}
```

### Automated Actions:
```json
{
  "routine_monitoring": {
    "frequency": "daily",
    "actions": [
      "check_stock_levels",
      "identify_expiring_items",
      "update_demand_forecasts",
      "generate_reorder_suggestions"
    ]
  },
  "urgent_monitoring": {
    "frequency": "real_time",
    "triggers": [
      "critical_stock_level",
      "item_expired",
      "failed_delivery",
      "emergency_usage_spike"
    ]
  }
}
```

## 7. Human-in-the-Loop Approval Process

### Approval Requirements:
```json
{
  "automatic_approval": {
    "conditions": [
      "routine_reorder",
      "total_cost < 500 EUR",
      "established_supplier",
      "standard_items_only"
    ]
  },
  "human_approval_required": {
    "conditions": [
      "new_supplier",
      "total_cost > 500 EUR",
      "controlled_substances",
      "emergency_orders",
      "budget_variance > 20%"
    ]
  },
  "approval_workflow": {
    "step_1": "InventoryAgent generates order recommendation",
    "step_2": "SupervisorAgent presents to human supervisor",
    "step_3": "Human reviews and approves/modifies/rejects",
    "step_4": "InventoryAgent executes approved orders",
    "step_5": "Track delivery and update inventory"
  }
}
```

## 8. Performance Requirements

- **Monitoring Frequency:** Real-time for critical items, daily for routine monitoring
- **Forecast Accuracy:** > 85% accuracy for 1-month forecasts, > 70% for 3-month forecasts
- **Stock-out Prevention:** < 2% stock-out rate for critical items
- **Cost Optimization:** Achieve 5-10% cost savings through optimized ordering
- **Response Time:** < 2 seconds for inventory queries, < 5 seconds for demand forecasts
- **Data Accuracy:** > 99.5% accuracy in stock level tracking

## 9. Integration Specifications

### n8n Workflow Integration:
```json
{
  "monitoring_workflow": {
    "schedule": "Daily at 06:00",
    "steps": ["check_inventory", "forecast_demand", "generate_alerts", "create_orders"]
  },
  "real_time_updates": {
    "triggers": ["stock_transaction", "delivery_received", "item_used"],
    "steps": ["update_stock", "check_thresholds", "trigger_alerts"]
  },
  "approval_workflow": {
    "trigger": "order_recommendation_generated",
    "steps": ["format_approval_request", "send_to_supervisor", "await_approval", "execute_order"]
  }
}
```

### External Integrations:
```json
{
  "supplier_apis": {
    "purpose": "Real-time pricing and availability",
    "implementation": "Optional webhook integrations"
  },
  "accounting_system": {
    "purpose": "Cost tracking and budget monitoring",
    "data_export": "Monthly cost reports"
  },
  "calendar_integration": {
    "purpose": "Procedure-based demand forecasting",
    "data_source": "CalendarAgent appointment data"
  }
}
```

## 10. Security & Compliance

### Data Security:
- **Access Control:** Role-based access to inventory data
- **Audit Trail:** Complete transaction history with user tracking
- **Data Encryption:** Encrypt sensitive supplier and cost information
- **Backup Strategy:** Daily backups with 1-year retention

### Regulatory Compliance:
- **Medical Device Tracking:** Comply with medical device regulations
- **Controlled Substance Monitoring:** Special handling for controlled medications
- **Expiry Date Management:** Strict FIFO (First In, First Out) enforcement
- **Supplier Qualification:** Maintain approved supplier list with certifications