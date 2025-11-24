# Task Breakdown Summary

**Project**: spec2cloud - AI-Enhanced Laboratory Data Harmonization Platform  
**Created**: November 24, 2025  
**Updated**: November 24, 2025 (Demo/Sandbox Mode)  
**Phase**: Planning → Implementation Ready  
**Total Tasks**: 9 (3 scaffolding + 6 feature implementation)  
**Mode**: **Demo/Sandbox** (Simplified infrastructure for rapid prototyping)

---

## Task Execution Sequence

### **Phase 1: Foundation (Must Complete First)**

#### Task 001: Azure Infrastructure Scaffolding (SIMPLIFIED)
- **Priority**: Critical
- **Complexity**: Low (Demo Mode)
- **Estimated Effort**: **1-2 days** (reduced from 4-5)
- **Dependencies**: None
- **Deliverables**:
  - 6 core Azure resources (Portal or CLI deployment)
  - Azure SQL (Basic tier, connection string auth)
  - Blob Storage (2 containers, connection string)
  - Azure OpenAI (GPT-4o-mini, API key)
  - Functions, Static Web Apps, Application Insights
  - Database schema scripts
  - **NO Key Vault, NO Managed Identity, NO Cosmos DB, NO Alerts**

#### Task 002: Backend API Scaffolding (SIMPLIFIED)
- **Priority**: Critical
- **Complexity**: Medium (Demo Mode)
- **Estimated Effort**: **2-3 days** (reduced from 3-4)
- **Dependencies**: 001
- **Deliverables**:
  - Python Azure Functions project structure
  - Azure OpenAI integration with API key (no Foundry for demo)
  - Database connection with connection string (no Managed Identity)
  - Simple config from environment variables (no Key Vault)
  - Logging utilities with Application Insights
  - Sample CRUD function implementation
  - **NO Foundry adapter, NO Key Vault client, NO Managed Identity**

#### Task 003: Frontend UI Scaffolding
- **Priority**: Critical
- **Complexity**: High
- **Estimated Effort**: 3-4 days (unchanged)
- **Dependencies**: 001
- **Deliverables**:
  - React/TypeScript project with Vite
  - Azure Fluent UI component library
  - API client with React Query
  - Main layout (header, sidebar, content)
  - File upload component
  - Routing configuration
  - Azure Static Web Apps deployment

**Phase 1 Total**: **6-9 days** (reduced from 10-13 days)  
**Parallelization**: After 001 completes (1-2 days), run 002 (2-3 days) || 003 (3-4 days)  
**Realistic Timeline**: 5-6 days with parallelization

---

### **Phase 2: Core Features (Implement in Order)**

#### Task 004: Property Registry Implementation
- **Priority**: Critical
- **Complexity**: Medium
- **Estimated Effort**: 3-5 days (unchanged)
- **Dependencies**: 001, 002
- **Sub-Tasks**:
  - 4.1: Database schema & seed data (50+ Lap Shear properties)
  - 4.2: REST API endpoints (CRUD, bulk import/export, search)
  - 4.3: Frontend UI components (list, detail, create/edit forms)
- **Acceptance**: ≥85% test coverage, all CRUD operations working, bulk import tested

#### Task 005: Schema Generation Implementation
- **Priority**: Critical
- **Complexity**: High
- **Estimated Effort**: 5-7 days (unchanged)
- **Dependencies**: 001, 002, 004
- **Sub-Tasks**:
  - 5.1: File parsing library (TXT, CSV, Excel - handle Zwick/Instron formats)
  - 5.2: AI property matching (Azure OpenAI GPT-4o-mini, confidence scoring)
  - 5.3: JSON Schema generation (CLP-compatible with ontology URIs)
  - 5.4: Frontend wizard (3-step: upload → review → export)
- **Acceptance**: 9/10 Lap Shear headers matched correctly (≥90% accuracy)

#### Task 006: Mapping Assistant Implementation
- **Priority**: High
- **Complexity**: High
- **Estimated Effort**: 5-6 days (unchanged)
- **Dependencies**: 001, 002, 004, 005
- **Sub-Tasks**:
  - 6.1: Two-stage mapping engine (raw → schema, schema → Albert)
  - 6.2: Unit conversion & validation
  - 6.3: CLP export format
  - 6.4: Frontend mapping UI (review table, confidence colors)
- **Acceptance**: ≥85% mapping accuracy on Lap Shear baseline

#### Task 007: Parser Builder Implementation
- **Priority**: Medium
- **Complexity**: High
- **Estimated Effort**: 4-6 days (unchanged)
- **Dependencies**: 001, 002, 005
- **Sub-Tasks**:
  - 7.1: File structure analysis (detect delimiter, encoding, units, metadata)
  - 7.2: AI parser code generation (Python/YAML)
  - 7.3: Azure Document Intelligence integration (PDF parsing)
  - 7.4: Parser repository management (Blob Storage + metadata)
  - 7.5: Frontend Parser Builder UI
- **Acceptance**: 9/10 generated parsers work on first attempt (≥90% success)

#### Task 008: Data Consistency & Audit System
- **Priority**: High (Cross-Cutting)
- **Complexity**: Medium
- **Estimated Effort**: 3-4 days
- **Dependencies**: 001, 002, All feature tasks
- **Sub-Tasks**:
  - 8.1: Validation rules engine (duplicate prevention, unit consistency)
  - 8.2: Application Insights logging (all user actions and AI decisions)
  - 8.3: Audit log storage & querying (Azure SQL only, no Blob archive for demo)
  - 8.4: Frontend Audit Log Viewer
- **Acceptance**: 100% of actions logged, audit queries <2s
- **Note**: No Azure Monitor alerts for demo phase

**Phase 2 Total**: 20-28 days (unchanged, sequential with some parallelization possible)

---

### **Phase 3: Quality Assurance**

#### Task 009: Comprehensive Testing Suite
- **Priority**: High
- **Complexity**: Medium
- **Estimated Effort**: 4-5 days (unchanged)
- **Dependencies**: All implementation tasks (004-008)
- **Test Categories**:
  - 9.1: Backend unit tests (pytest, ≥85% coverage)
  - 9.2: Frontend unit tests (Vitest, ≥85% coverage)
  - 9.3: Integration tests (live Azure services)
  - 9.4: AI accuracy validation (90%/85%/90% targets)
  - 9.5: E2E tests (Playwright)
  - 9.6: Performance tests (Azure Load Testing)
  - 9.7: Security tests (authentication, input validation)
- **Acceptance**: All test categories pass, CI/CD pipeline green

**Phase 3 Total**: 4-5 days

---

## Total Project Timeline (Demo/Sandbox Mode)

### **Original Estimate** (Production-Ready):
- Minimum: 34 days (10 + 20 + 4)
- Maximum: 46 days (13 + 28 + 5)
- Realistic: 38-40 days (~8 weeks)

### **Demo Mode Estimate** (Simplified):
- **Minimum**: 30 days (6 + 20 + 4)
- **Maximum**: 42 days (9 + 28 + 5)
- **Realistic with parallelization**: **32-34 days (~7 weeks)**

**Time Savings**: 4-6 days reduction in foundation phase by eliminating:
- Key Vault setup and integration
- Managed Identity configuration
- Microsoft Foundry adapter development
- Complex Bicep IaC (manual Portal setup instead)

---

## Critical Path (Updated)

```
001 (Infrastructure - 1-2 days SIMPLIFIED)
  ├─→ 002 (Backend - 2-3 days SIMPLIFIED) ─→ 004 (Property) ─→ 005 (Schema) ─→ 006 (Mapping)
  │                                                                ↓
  └─→ 003 (Frontend - 3-4 days) ──────────────────────────────────┴──→ 007 (Parser) ─→ 008 (Audit) ─→ 009 (Testing)
```

**Critical Path**: 001 → 002 → 004 → 005 → 006 → 008 → 009  
**Duration**: 24-32 days minimum (reduced from 28-36 days)

---

## Parallelization Opportunities

### **Week 1**: Foundation (Days 1-6)
- Days 1-2: Task 001 Infrastructure (solo, Azure Portal setup)
- Days 3-5: 
  - Task 002 Backend (Team Member A) - 2-3 days
  - Task 003 Frontend (Team Member B) - 3-4 days
- Result: Foundation complete in 5-6 days (vs 10-13 days original)

### **Week 2-3**: Core Features
- Task 004: Property Registry (Team Member A - depends on 002)
- Frontend components for Property Registry (Team Member B - depends on 003)

### **Week 4-5**: AI-Powered Features
- Task 005: Schema Generation (Team Member A - depends on 004)
- Task 006: Mapping Assistant (Team Member B - starts after 005 completes)
- Task 007: Parser Builder (Team Member C - can start after 005, parallel with 006)

### **Week 6-7**: Quality & Integration
- Task 008: Data Consistency & Audit (Team Member A)
- Task 009: Testing (All team members - integration and E2E)

---

## Key Success Criteria

### **Technical**
- [ ] All 9 tasks completed with acceptance criteria met
- [ ] ≥85% test coverage (backend and frontend)
- [ ] AI accuracy targets met (90% schema, 85% mapping, 90% parser)
- [ ] All APIs respond within performance targets
- [ ] No critical security vulnerabilities (demo environment acceptable)

### **Functional**
- [ ] Property Registry: 50+ properties seeded, CRUD operations working
- [ ] Schema Generation: Generate Lap Shear schema from real Zwick/Instron files
- [ ] Mapping Assistant: Generate and export CLP-compatible mappings
- [ ] Parser Builder: Generate working parsers for 3+ machine types
- [ ] Audit System: 100% of actions logged with queryable audit trail

### **User Experience**
- [ ] Schema generation time: <4 hours (vs. weeks currently)
- [ ] AI suggestions reviewed in <30 minutes
- [ ] All workflows intuitive (<30 min training)
- [ ] UI responsive and accessible (WCAG AA)

---

## Risk Management

### **High-Priority Risks**
1. **AI Accuracy Below Targets**
   - Mitigation: Iterative prompt engineering, few-shot examples, user review workflow
   - Contingency: Manual override always available

2. **Azure OpenAI API Costs Exceed Budget**
   - Mitigation: Caching, batch processing, use GPT-4o-mini (cost-effective)
   - Contingency: Rate limiting, monitoring usage alerts

3. **Property Registry Becomes Inconsistent**
   - Mitigation: Strict validation rules, duplicate detection, audit logging
   - Contingency: Database rollback, property merge tools

4. **Performance Degrades with Large Datasets**
   - Mitigation: Database indexing, pagination, limit demo dataset sizes
   - Contingency: Optimize queries, upgrade Azure SQL tier if needed

### **Medium-Priority Risks**
5. **Team Capacity Constraints**
   - Mitigation: Prioritize critical path tasks, defer nice-to-have features
   - Contingency: Extend timeline, focus on core features only

6. **Azure Service Outages**
   - Mitigation: Implement retry logic, graceful error messages
   - Contingency: Queue operations for retry when service recovers

### **Demo-Specific Risks**
7. **Security Concerns with Open Configuration**
   - Mitigation: Clear documentation that this is demo/sandbox mode only
   - Action: Plan production hardening phase (Key Vault, Managed Identity, VNet)
   - Timeline: Address before any customer/public deployment

8. **Configuration Management Complexity**
   - Mitigation: Document all connection strings in single config file
   - Contingency: Use `.env` files, provide setup scripts

---

## Next Steps

1. **Review** task breakdown with CLP team and Henkel stakeholders
2. **Prioritize** any adjustments based on business constraints
3. **Assign** tasks to team members based on expertise
4. **Kick off** Task 001 (Infrastructure) immediately
5. **Schedule** weekly reviews to track progress and adjust plan
6. **Prepare** sample data (Lap Shear files, Albert templates) for testing

---

**Document Status**: Final v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent & Developer Agent (spec2cloud workflow)  
**Approval Needed**: CLP Product Owner, Henkel Technical Lead
