# Task: Property Registry Implementation

**Task ID**: 004  
**Feature**: Property Registry (FRD-001)  
**Priority**: Critical  
**Estimated Complexity**: Medium  
**Dependencies**: 001-infrastructure, 002-backend-scaffolding

---

## Task Description

Implement the Property Registry feature with Azure SQL backend, REST API endpoints, unit catalog, CRUD operations, bulk import/export, and search functionality.

---

## Sub-Tasks

### **4.1: Database Schema & Seed Data**
- Execute `001_create_properties_table.sql` from infrastructure task
- Create unit catalog table with real units from Lap Shear samples (MPa, N, mm, mm², etc.)
- Seed initial properties from Albert Lap Shear template (50+ properties)
- Create indexes for performance optimization

**Acceptance Criteria**:
- [ ] Properties table created with all constraints
- [ ] Unit catalog populated with 20+ units and dimension mappings
- [ ] Seed data contains 50+ Lap Shear properties
- [ ] Queries execute in <500ms for 1000 properties

### **4.2: REST API Endpoints**
Implement Azure Functions for all CRUD operations:
- `POST /api/properties` - Create property
- `GET /api/properties/{id}` - Get property by ID
- `GET /api/properties` - List/search properties (with filters)
- `PUT /api/properties/{id}` - Update property
- `DELETE /api/properties/{id}` - Delete property (with usage check)
- `POST /api/properties/bulk-import` - Bulk import from CSV/JSON
- `GET /api/properties/export` - Export to JSON/CSV/Excel

**Acceptance Criteria**:
- [ ] All endpoints implement error handling and validation
- [ ] Duplicate canonical names rejected with 409 status
- [ ] Search supports filters (test_type, unit, data_type)
- [ ] Bulk import validates and reports errors per row
- [ ] All operations logged to Application Insights

### **4.3: Frontend UI Components**
Implement React components from FRD-003:
- Property List view with table, search, and filters
- Property Detail view with all fields
- Create/Edit Property form with validation
- Bulk Import modal with file upload
- Export functionality

**Acceptance Criteria**:
- [ ] Property list renders with pagination (50 per page)
- [ ] Search returns results within 1 second
- [ ] Form validates all inputs before submission
- [ ] Bulk import shows progress and error summary
- [ ] Export downloads file immediately

---

## Testing Requirements

- [ ] Unit tests for all API endpoints (≥85% coverage)
- [ ] Integration tests with live Azure SQL database
- [ ] E2E tests for create, search, edit, delete workflows
- [ ] Performance tests: 1000 concurrent searches complete in <2s

---

**Estimated Effort**: 3-5 days  
**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025
