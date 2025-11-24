# Task: Data Consistency & Audit System Implementation

**Task ID**: 008  
**Feature**: Data Consistency & Audit (FRD-006)  
**Priority**: High (Cross-Cutting)  
**Estimated Complexity**: Medium  
**Dependencies**: 001-infrastructure, 002-backend-scaffolding, All feature tasks

---

## Task Description

Implement comprehensive data consistency enforcement and audit logging system with validation rules, duplicate prevention, Application Insights integration, and audit log query interface.

---

## Sub-Tasks

### **8.1: Validation Rules Engine**
Implement validation checks:
- **Duplicate Prevention**: Reject duplicate canonical property names
- **Similar Name Detection**: Warn if Levenshtein distance < 3
- **Unit Consistency**: Same property must use same unit across schemas
- **Schema Property Validation**: All properties must exist in registry
- **CLP Format Compliance**: Validate exported schemas/mappings against spec

**Acceptance Criteria**:
- [ ] Duplicate property creation returns 409 with error message
- [ ] Similar name warning displays with "Use Existing?" option
- [ ] Unit mismatch flagged before schema export
- [ ] Unknown properties blocked at export time
- [ ] JSON Schema validation passes for all exports

### **8.2: Application Insights Logging**
Log all events to Azure Application Insights:
- Property CRUD operations (create, update, delete, search)
- Schema generation (files uploaded, AI suggestions, user decisions)
- Mapping assistant (mapping generation, approval, export)
- Parser builder (parser generated, tested, saved)
- User authentication (login, logout, failed attempts)

**Log Structure**:
```json
{
  "log_id": "uuid",
  "timestamp": "2025-11-24T10:30:15Z",
  "event_type": "ai_property_matching",
  "user_id": "user_123",
  "details": {...},
  "metadata": {...}
}
```

**Acceptance Criteria**:
- [ ] All user actions logged with full context
- [ ] All AI decisions logged with confidence and justification
- [ ] Custom events tracked (AIPropertyMatching, UserMappingDecision, etc.)
- [ ] Logging overhead <50ms per operation
- [ ] Application Insights dashboard displays key metrics

### **8.3: Audit Log Storage & Querying**
- Store audit logs in Azure SQL `audit_logs` table
- Implement query API with filters (date range, event type, user, entity)
- Support full-text search across log details
- Export logs to CSV, JSON, PDF formats
- Archive logs >1 year to Azure Blob Storage (cold tier)

**Acceptance Criteria**:
- [ ] Audit log queries return results within 2 seconds
- [ ] Filters work correctly (event_type, user_id, timestamp range)
- [ ] Full-text search finds relevant logs
- [ ] Export generates valid CSV/JSON/PDF files
- [ ] Archive process runs monthly and moves old logs to Blob

### **8.4: Monitoring & Alerting**
Configure Azure Monitor alerts:
- **Critical**: API error rate >5% for 5 minutes → Email DevOps
- **Warning**: Validation failure rate >10% for 10 minutes → Email DevOps
- **Warning**: Azure SQL DTU >80% for 10 minutes → Email DevOps

**Acceptance Criteria**:
- [ ] Alerts trigger correctly when conditions met
- [ ] Email notifications sent to correct recipients
- [ ] Alert dashboard displays current system health
- [ ] No false positives in 1 week of testing

### **8.5: Frontend Audit Log Viewer**
- Audit Log page with table view
- Filters (date range picker, event type dropdown, user dropdown)
- Search bar for full-text search
- Row expansion to show full JSON details
- Export button (CSV, JSON, PDF)

**Acceptance Criteria**:
- [ ] Table renders with pagination (100 logs per page)
- [ ] Filters update table without page refresh
- [ ] Search returns results within 1 second
- [ ] Row expansion shows formatted JSON
- [ ] Export downloads immediately

---

## Testing Requirements

- [ ] Unit tests for all validation rules
- [ ] Integration tests for Application Insights logging
- [ ] Integration tests for audit log queries
- [ ] E2E test: Perform action → verify logged → query logs → export
- [ ] Performance test: 1000 concurrent log writes complete in <5s

---

**Estimated Effort**: 3-4 days  
**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025
