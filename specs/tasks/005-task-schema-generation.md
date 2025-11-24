# Task: Schema Generation Implementation

**Task ID**: 005  
**Feature**: AI-Powered Schema Generation (FRD-002)  
**Priority**: Critical  
**Estimated Complexity**: High  
**Dependencies**: 001-infrastructure, 002-backend-scaffolding, 004-property-registry

---

## Task Description

Implement AI-powered schema generation with file parsing, property matching using Microsoft Foundry/Azure OpenAI, confidence scoring, JSON Schema output, and user review workflow.

---

## Sub-Tasks

### **5.1: File Parsing Library**
Create parsers for TXT, CSV, Excel formats handling real Lap Shear complexity:
- Semicolon delimiter, decimal commas (`"19,59"`), quoted values
- Cryptic notation (`a{lo 0}`, `F{lo max}`, `S{lo 0}`)
- Metadata blocks (`Auftrags-Nr.;30245;Prüfnorm;ISO 4587`)
- Separate unit rows vs. inline units `(N/mm^2)`
- Multi-encoding support (UTF-8, Windows-1252)

**Acceptance Criteria**:
- [ ] Parse real Zwick and Instron sample files successfully
- [ ] Extract 7+ properties with units from Lap Shear files
- [ ] Handle German text and cryptic notation
- [ ] Return structured data (headers, units, metadata, rows)

### **5.2: AI Property Matching Service**
Implement AI service adapter calling Foundry or Azure OpenAI:
- Build system prompt for property matching task
- Construct user prompt with headers, registry properties, context
- Parse AI response into `AIMatch` objects with confidence scores
- Handle 10 real examples from FRD-002 (German + cryptic notation)
- Return justifications for each match

**Acceptance Criteria**:
- [ ] AI correctly matches 9/10 Lap Shear headers (≥90% accuracy)
- [ ] Confidence scores calibrated (high confidence = high accuracy)
- [ ] Justifications include unit match, synonym detection, context
- [ ] Supports both Foundry (GPT-5.1/o4-mini) and Azure OpenAI (GPT-4o)

### **5.3: JSON Schema Generation**
Generate CLP-compatible JSON Schema from matched properties:
- Include `$henkel.property-class` ontology URIs
- Add `source_headers` array for traceability
- Infer required vs. optional based on presence in sample files
- Include metadata (test_type, standard, source_files, created_date)
- Validate against JSON Schema Draft 2020-12

**Acceptance Criteria**:
- [ ] Generated schema validates against JSON Schema spec
- [ ] All properties reference Property Registry canonical names
- [ ] Schema includes all real Lap Shear properties from samples
- [ ] Export to JSON, YAML, Excel formats

### **5.4: Frontend Schema Generation Wizard**
Three-step wizard from FRD-003:
- Step 1: File upload with test type selection
- Step 2: Review AI suggestions table with confidence colors
- Step 3: Preview and export JSON Schema

**Acceptance Criteria**:
- [ ] File upload handles 20 files, 50MB each
- [ ] AI matching completes within 60 seconds
- [ ] User can accept, edit, reject, or create new property
- [ ] Schema preview renders formatted JSON
- [ ] Export downloads immediately

---

## Testing Requirements

- [ ] Unit tests for file parsers with real Zwick/Instron samples
- [ ] Unit tests for AI matching with mocked responses
- [ ] Integration tests with live Azure OpenAI endpoint
- [ ] E2E test: Upload files → review suggestions → export schema
- [ ] Accuracy test: Validate 9/10 matches for Lap Shear baseline

---

**Estimated Effort**: 5-7 days  
**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025
