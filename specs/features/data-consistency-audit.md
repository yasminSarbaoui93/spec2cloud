# Feature Requirements Document (FRD)

## Feature: Data Consistency & Audit System

**Feature ID**: FRD-006  
**Phase**: Cross-Phase (All Phases)  
**Priority**: High (Data Integrity & Compliance)  
**Status**: Draft  
**Related PRD Requirements**: [REQ-6], [REQ-10], [REQ-11]

---

## 1. Overview

### **Purpose**
Enforce data consistency rules across all schemas and mappings, prevent duplicate or conflicting property definitions, and maintain comprehensive audit logs of all AI decisions and user actions for traceability and compliance.

### **Business Value**
- Achieves 100% property naming consistency (key PRD goal)
- Prevents data quality issues before they reach Albert
- Provides full audit trail for regulatory compliance
- Enables root cause analysis when issues occur
- Supports continuous improvement through decision tracking

### **User Personas**
- **Schema Owner**: Relies on validation to ensure quality
- **Product Owner**: Reviews audit logs for process improvement
- **Compliance Officer**: Uses audit trail for regulatory reporting
- **Developer**: Debugs issues using audit logs

---

## 2. Functional Requirements

### **Core Capabilities**

#### **2.1 Data Consistency Enforcement**

**Duplicate Property Prevention**
- **Rule**: No two properties can have the same canonical name (case-sensitive)
- **Enforcement Point**: Property Registry creation/update
- **Behavior**: Reject creation with error message: "Property 'Viscosity_PaS' already exists (ID: 12345)"
- **User Action**: View existing property or choose different name

**Similar Name Detection**
- **Rule**: Warn if new property name is similar to existing (Levenshtein distance < 3)
- **Example**: Creating "Viscosity_Pas" when "Viscosity_PaS" exists
- **Behavior**: Show warning: "Similar property found: Viscosity_PaS. Do you want to use existing property?"
- **User Action**: Use existing or confirm new property is distinct

**Unit Consistency**
- **Rule**: Same property across schemas must use same unit
- **Example**: `Tensile_Shear_Strength_MPa` must always be in MPa
- **Enforcement Point**: Schema generation, mapping assistant
- **Behavior**: Flag if schema attempts to use different unit
- **User Action**: Use canonical unit or create new property with different unit suffix

**Schema Property Validation**
- **Rule**: All properties in schema must exist in Property Registry
- **Enforcement Point**: Schema export
- **Behavior**: Reject export if unknown properties found
- **User Action**: Add missing properties to registry first

**Mapping Consistency**
- **Rule**: Raw header → schema property mappings should be consistent across similar files
- **Example**: "Fmax" should always map to `Maximum_Force_N`
- **Enforcement Point**: Mapping Assistant
- **Behavior**: Flag if user maps differently than historical precedent
- **User Action**: Confirm intentional change or correct mapping

**CLP Format Compliance**
- **Rule**: Exported schemas/mappings must validate against CLP JSON Schema spec
- **Enforcement Point**: Export
- **Behavior**: Validate JSON structure before download
- **User Action**: Fix validation errors (unlikely with automated generation)

#### **2.2 Comprehensive Audit Logging**

**Logged Events**

**Property Registry Events**
- Property Created (who, when, all field values)
- Property Updated (who, when, what changed)
- Property Deleted (who, when, property details)
- Property Searched (query, results count)

**Schema Generation Events**
- Files Uploaded (filenames, sizes, test type)
- AI Property Matching Initiated (# headers, test type)
- AI Suggestions Generated (header, suggestion, confidence, justification)
- User Accept/Reject/Edit Action (which suggestion, final decision)
- New Property Created During Schema Workflow (property details)
- Schema Generated (schema ID, # properties, metadata)
- Schema Exported (format, destination)

**Mapping Assistant Events**
- Mapping Generation Initiated (source, target, # fields)
- AI Mapping Suggestions Generated (each suggestion with confidence)
- User Mapping Approval (raw → target, confidence, user decision)
- Unit Conversion Applied (from unit, to unit, formula)
- Mapping Exported (mapping ID, destination)

**Parser Builder Events**
- File Structure Analyzed (file type, detected structure)
- Parser Generated (parser ID, machine type, test type)
- Parser Tested (test results, success/failure)
- Parser Saved to Repository (location, version)

**User Authentication Events** (Phase 2)
- User Login/Logout
- Failed Authentication Attempts
- Permission Changes

**Log Entry Structure**
```json
{
  "log_id": "uuid",
  "timestamp": "2025-11-24T10:30:15.123Z",
  "event_type": "ai_property_matching",
  "user_id": "schema_owner_123",
  "details": {
    "raw_header": "Zugscherfestigkeit",
    "raw_unit": "MPa",
    "ai_suggestion": "Tensile_Shear_Strength_MPa",
    "confidence": 98,
    "justification": "German for tensile shear strength; unit match",
    "user_action": "accepted"
  },
  "metadata": {
    "session_id": "sess_abc123",
    "test_type": "Lap Shear",
    "schema_id": "schema_456"
  }
}
```

#### **2.3 Audit Log Querying & Export**

**Query Interface** (via Frontend UI)
- **Filters**:
  - Date Range
  - Event Type (dropdown)
  - User
  - Entity (property name, schema ID, etc.)
- **Search**: Full-text search across log details
- **Results Table**:
  - Timestamp
  - Event Type
  - User
  - Summary (truncated details)
  - Actions: View Full Details (expandable JSON)

**Export Options**
- CSV (for Excel analysis)
- JSON (for programmatic processing)
- PDF Report (formatted for compliance)

**Retention Policy**
- Phase 1: Retain all logs indefinitely in Azure SQL
- Phase 2: Archive logs >1 year old to Azure Blob Storage (cold tier)
- Phase 3: Implement compliance-driven retention (e.g., 7 years for FDA)

#### **2.4 Real-Time Validation**

**Pre-Save Validation**
- Validate before saving to database
- Display errors inline in forms
- Prevent submission until corrected

**Post-AI Validation**
- After AI generates suggestions, validate:
  - Suggested properties exist in registry
  - Units are compatible
  - Data types match
- Flag issues for user review

**Export Validation**
- Before download, validate:
  - JSON syntax
  - Schema compliance
  - Required fields present
- Block export if validation fails

#### **2.5 Monitoring & Alerting** (Azure Monitor)

**Azure Application Insights Integration**
- Track all API calls (latency, errors, success rate)
- Custom metrics:
  - Property creations per day
  - Schema generations per day
  - AI suggestion accuracy (% accepted)
  - Validation failures per day

**Alerts** (Azure Monitor Alerts)
- **Critical**: API error rate >5%
- **Warning**: Validation failure rate >10%
- **Info**: Daily summary of activity

**Dashboard** (Azure Dashboard or Power BI)
- Real-time system health
- Usage trends (properties, schemas over time)
- AI performance metrics (accuracy, confidence distribution)
- User activity (top users, actions per user)

---

## 3. Technical Architecture

### **Technology Stack (Azure-First)**

#### **Logging: Azure Application Insights**
- **Why**: Native Azure integration, rich querying, alerting, dashboards
- **Log Levels**: Info (user actions), Warning (validation issues), Error (system failures)
- **Custom Events**: Track AI suggestions, user decisions
- **Custom Metrics**: Accuracy rates, consistency violations

**Alternative**: Azure Log Analytics (if Application Insights insufficient)

#### **Audit Storage: Azure SQL Database**
- **Table**: `audit_logs` (structured storage for querying)
- **Indexes**: On timestamp, event_type, user_id
- **Retention**: Keep recent logs in SQL; archive old logs to Blob

#### **Long-Term Archive: Azure Blob Storage**
- **Container**: `audit-logs-archive`
- **Tier**: Cool or Archive (cost-effective for compliance)
- **Format**: JSON files (one per day or week)

#### **Validation Engine: Python**
- **Library**: `jsonschema` for JSON Schema validation
- **Custom Validators**: Unit compatibility, property existence checks
- **Integration**: Called before save/export operations

#### **Monitoring: Azure Monitor**
- **Application Insights**: For logs and telemetry
- **Alerts**: Email/SMS for critical issues
- **Dashboards**: Azure Portal or Power BI

---

## 4. Integration Points

### **Inputs**

| Source | Event | Purpose |
|--------|-------|---------|
| All Features | User actions, AI decisions | Log for audit trail |
| Property Registry API | CRUD operations | Enforce uniqueness, consistency |
| Schema Generation | Property matching | Validate against registry |
| Mapping Assistant | Mapping creation | Validate unit compatibility |

### **Outputs**

| Destination | Format | Purpose |
|-------------|--------|---------|
| Azure Application Insights | JSON | Real-time monitoring |
| Azure SQL Database | SQL Inserts | Queryable audit logs |
| Azure Blob Storage | JSON | Long-term archive |
| Frontend UI | JSON | Display audit logs to users |
| Compliance Reports | PDF | Regulatory reporting |

### **Dependencies**

- **Azure Application Insights**: For logging and monitoring
- **Azure SQL Database**: For audit log storage
- **Azure Blob Storage**: For archive
- **All feature APIs**: Must call audit logging functions

---

## 5. User Workflows

### **Workflow 1: Prevent Duplicate Property**

**Actor**: Schema Owner

1. Attempt to create property: "Viscosity_PaS"
2. System checks registry: Property already exists (ID: 789)
3. Display error: "Property 'Viscosity_PaS' already exists. View existing?"
4. Click "View Existing"
5. Navigate to Property Detail page (ID: 789)
6. User confirms this is the property they need
7. Use existing property in schema

**Acceptance Criteria**:
- Duplicate detection happens before saving
- Error message clear and actionable
- User can view existing property details

### **Workflow 2: Review AI Decisions in Audit Log**

**Actor**: Product Owner

1. Navigate to "Audit Logs" page
2. Filter: Event Type = "AI Property Matching", Date = Last 7 Days
3. See table with 50 matching entries
4. Click on entry: "Zugscherfestigkeit" → "Tensile_Shear_Strength_MPa"
5. Expand details:
   - Confidence: 98%
   - Justification: "German synonym; unit match"
   - User Action: Accepted
   - Timestamp: 2025-11-20 14:32
6. Click "Export to CSV"
7. Download file for analysis

**Acceptance Criteria**:
- Filters work correctly
- All AI decisions logged with full context
- Export generates valid CSV

### **Workflow 3: Validate Schema Before Export**

**Actor**: Schema Owner

1. Generate schema with 10 properties
2. Click "Export Schema"
3. System validates:
   - All properties exist in registry ✓
   - No duplicate property names ✓
   - JSON structure valid ✓
   - Required fields present ✓
4. Validation passes
5. Download JSON file

**Scenario: Validation Failure**
1. Somehow a property "UnknownField" not in registry is in schema
2. Click "Export Schema"
3. Validation fails: "Property 'UnknownField' not found in registry"
4. User must fix before export

**Acceptance Criteria**:
- All validation rules checked before export
- Clear error messages if validation fails
- Export blocked until issues resolved

---

## 6. Acceptance Criteria

### **Functional Acceptance**

- [ ] Duplicate property names rejected at creation
- [ ] Similar name warnings displayed to user
- [ ] All user actions logged to Application Insights
- [ ] All AI decisions logged with confidence and justification
- [ ] Audit log UI displays logs with filters and search
- [ ] Export schemas validates against CLP JSON Schema spec
- [ ] Alerts triggered for high error rates

### **Non-Functional Acceptance**

- [ ] Logging overhead <50ms per operation
- [ ] Audit log queries return results in <2 seconds
- [ ] Azure Application Insights retains logs for 90 days
- [ ] Archived logs accessible within 24 hours (cold storage)
- [ ] Validation completes in <500ms

### **Compliance Acceptance**

- [ ] All logged data includes timestamp, user, action
- [ ] Logs immutable (no deletion except per retention policy)
- [ ] Audit log exports suitable for compliance reporting
- [ ] GDPR compliance: User IDs pseudonymized if needed

---

## 7. Out of Scope

- **Real-time alerting to users**: Email/SMS alerts deferred (Phase 2)
- **Advanced analytics**: ML on audit logs deferred
- **User behavior tracking**: Focus on data actions only; UX analytics deferred
- **Blockchain audit trail**: Immutable ledger deferred (overkill for Phase 1)

---

## 8. Testing Strategy

### **Unit Tests**
- Test duplicate detection logic
- Test validation rules (unit compatibility, property existence)
- Mock audit logging calls

### **Integration Tests**
- Create property, verify logged to Application Insights
- Attempt duplicate creation, verify rejection
- Export schema, verify validation runs

### **Compliance Tests**
- Verify all required events are logged
- Verify log format meets compliance standards
- Test audit log export for completeness

---

## 9. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Logging overhead slows system | Medium | Low | Async logging; batch writes; monitor performance |
| Audit logs grow too large | Medium | Medium | Archive old logs to Blob Storage; implement retention policy |
| Validation too strict blocks legitimate use cases | Medium | Medium | Provide override with justification; log overrides |
| Missing audit events | High | Low | Comprehensive event list; code reviews; testing |

---

## 10. Success Metrics

### **Data Quality Metrics**
- 0 duplicate properties in registry
- 100% of schemas validate before export
- <1% validation failures in production

### **Audit Coverage Metrics**
- 100% of AI decisions logged
- 100% of user actions logged
- 0 compliance gaps in audit trail

### **Performance Metrics**
- Logging overhead <50ms per operation
- Validation <500ms per export
- Audit log queries <2s

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent (spec2cloud workflow)  
**Dependencies**: All features (cross-cutting concern)
