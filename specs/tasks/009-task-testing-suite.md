# Task: Comprehensive Testing Suite

**Task ID**: 009  
**Feature**: Testing & Quality Assurance  
**Priority**: High  
**Estimated Complexity**: Medium  
**Dependencies**: All implementation tasks (004-008)

---

## Task Description

Implement comprehensive testing strategy covering unit tests (≥85% coverage), integration tests, E2E tests, AI accuracy validation, and performance tests to ensure system quality and reliability.

---

## Test Categories

### **9.1: Backend Unit Tests** (pytest)
Test all Python modules with ≥85% coverage:
- `shared/ai_service_adapter.py` - Mock AI responses, test both Foundry and Azure OpenAI
- `shared/database.py` - Mock database connections, test query execution
- `shared/validators.py` - Test all validation rules with valid/invalid inputs
- `functions/property_registry/*` - Test CRUD operations with mocked database
- `functions/schema_generation/*` - Test file parsing, AI matching logic
- `functions/mapping_assistant/*` - Test mapping engine, unit conversion
- `functions/parser_builder/*` - Test structure analysis, code generation

**Acceptance Criteria**:
- [ ] All unit tests pass
- [ ] Coverage ≥85% for all shared modules
- [ ] Coverage ≥85% for all function modules
- [ ] Tests run in <2 minutes
- [ ] No flaky tests (consistent results)

### **9.2: Frontend Unit Tests** (Vitest + React Testing Library)
Test all React components:
- Common components (Button, Input, Table, Modal, FileUpload)
- Layout components (Header, Sidebar, MainLayout)
- Feature components (PropertyList, SchemaWizard, MappingTable, AuditLogViewer)
- Hooks (useApi, useAuth, useFileUpload)
- Services (apiClient, propertyService, schemaService)

**Acceptance Criteria**:
- [ ] All unit tests pass
- [ ] Coverage ≥85% for all components
- [ ] Coverage ≥85% for all hooks and services
- [ ] Tests run in <1 minute
- [ ] Accessibility tests pass (axe-core)

### **9.3: Integration Tests**
Test end-to-end flows with live Azure services:
- **Property Registry**: Create → read → update → delete property with Azure SQL
- **Schema Generation**: Upload files → call Azure OpenAI → generate schema
- **Mapping Assistant**: Generate mappings → store in SQL → export
- **Parser Builder**: Analyze file → generate parser → store in Blob Storage
- **Audit Logging**: Perform action → verify logged in Application Insights

**Acceptance Criteria**:
- [ ] All integration tests pass against dev environment
- [ ] Tests can run in CI/CD pipeline
- [ ] Tests clean up resources after execution
- [ ] No hardcoded credentials (use environment variables)

### **9.4: AI Accuracy Validation**
Measure AI performance against manual baseline:
- **Schema Generation**: 10 real Lap Shear files → 9/10 correct matches (90% accuracy)
- **Mapping Assistant**: 10 mapping sets → 8.5/10 correct mappings (85% accuracy)
- **Parser Builder**: 10 file formats → 9/10 working parsers (90% success rate)
- **Confidence Calibration**: High confidence (>85%) correlates with high accuracy

**Baseline Dataset**:
- 3 Lap Shear sample files (Zwick AMC, Zwick AMC 2, Instron ACC)
- Manual annotations by domain scientist
- 10 properties with expected matches

**Acceptance Criteria**:
- [ ] Schema generation achieves ≥90% accuracy on baseline
- [ ] Mapping assistant achieves ≥85% accuracy on baseline
- [ ] Parser builder achieves ≥90% success rate
- [ ] Confidence scores calibrated (precision/recall metrics calculated)

### **9.5: E2E Tests** (Playwright)
Test critical user workflows:
- **Property Registry**: Create property → search → edit → delete
- **Schema Generation**: Upload files → review suggestions → approve → export schema
- **Mapping Assistant**: Generate mappings → review → approve → export config
- **Audit Log**: Perform actions → view audit logs → filter → export

**Acceptance Criteria**:
- [ ] All E2E tests pass on localhost
- [ ] All E2E tests pass on dev environment
- [ ] Tests include screenshots on failure for debugging
- [ ] Tests run in <10 minutes

### **9.6: Performance Tests**
Load testing with Azure Load Testing:
- **Property Registry API**: 100 concurrent searches → all complete in <2s
- **Schema Generation API**: 10 concurrent schema generations → all complete in <2min
- **Mapping Assistant API**: 50 concurrent mapping requests → all complete in <30s
- **Audit Log Queries**: 1000 concurrent queries → all complete in <2s

**Acceptance Criteria**:
- [ ] All APIs meet performance targets under load
- [ ] No memory leaks detected (monitoring over 1 hour)
- [ ] Azure SQL DTU usage <80% under load
- [ ] Application Insights tracks all requests correctly

### **9.7: Security Tests**
- **Authentication**: Verify unauthorized requests return 401
- **Input Validation**: Test SQL injection, XSS, file upload attacks
- **API Rate Limiting**: Test excessive requests are blocked
- **Secrets Management**: Verify no secrets in code or logs

**Acceptance Criteria**:
- [ ] All security tests pass
- [ ] No SQL injection vulnerabilities
- [ ] No XSS vulnerabilities
- [ ] Rate limiting works correctly
- [ ] Secrets retrieved only from Key Vault

---

## Test Infrastructure

### **CI/CD Integration** (GitHub Actions)

**File**: `.github/workflows/test.yml`

```yaml
name: Run Tests

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          cd backend
          pip install -r requirements.txt
          pip install pytest pytest-cov
      - name: Run unit tests
        run: |
          cd backend
          pytest --cov=. --cov-report=xml --cov-report=term
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./backend/coverage.xml

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install dependencies
        run: |
          cd frontend
          npm ci
      - name: Run unit tests
        run: |
          cd frontend
          npm run test:coverage
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./frontend/coverage/coverage-final.json

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Playwright
        run: npx playwright install --with-deps
      - name: Run E2E tests
        run: |
          cd frontend
          npm run test:e2e
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-results
          path: frontend/test-results/
```

---

## Acceptance Criteria (Overall)

- [ ] Backend unit tests: ≥85% coverage, all pass
- [ ] Frontend unit tests: ≥85% coverage, all pass
- [ ] Integration tests: All pass against dev environment
- [ ] AI accuracy tests: Meet targets (90%, 85%, 90%)
- [ ] E2E tests: All pass on localhost and dev
- [ ] Performance tests: All APIs meet performance targets
- [ ] Security tests: All pass with no vulnerabilities
- [ ] CI/CD pipeline runs all tests automatically
- [ ] Code coverage reports uploaded to Codecov

---

**Estimated Effort**: 4-5 days  
**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025
