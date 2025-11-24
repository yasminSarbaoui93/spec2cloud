# Task: Parser Builder Implementation

**Task ID**: 007  
**Feature**: Parser Builder (FRD-005)  
**Priority**: Medium  
**Estimated Complexity**: High  
**Dependencies**: 001-infrastructure, 002-backend-scaffolding, 005-schema-generation

---

## Task Description

Implement AI-powered parser generation for diverse file formats (TXT, CSV, Excel, PDF) with file structure analysis, Python/YAML code generation, validation, and parser repository management.

---

## Sub-Tasks

### **7.1: File Structure Analysis**
Analyze uploaded sample files to detect:
- Delimiter (semicolon, comma, tab, pipe)
- Encoding (UTF-8, Windows-1252, ISO-8859-1)
- Header row location (may have metadata before headers)
- Unit row (separate vs. inline in parentheses)
- Data table boundaries
- Decimal separator (comma vs. period)
- Quote character

**Acceptance Criteria**:
- [ ] Detect correct structure for real Zwick/Instron files
- [ ] Handle 8 parsing challenges from FRD-005 (metadata blocks, decimal commas, etc.)
- [ ] Return structured analysis report as JSON

### **7.2: AI Parser Code Generation**
Use AI (Foundry GPT-5.1-codex or Azure OpenAI GPT-4.1) to generate:
- **Python parser function** with pandas, openpyxl, error handling
- **YAML configuration** for declarative parsing
- Include docstrings, type hints, validation logic
- Generate pytest test cases for validation

**Acceptance Criteria**:
- [ ] Generated Python code is syntactically valid
- [ ] Generated parser extracts correct headers and units from Zwick sample
- [ ] YAML config includes all 7 columns from real Lap Shear file
- [ ] Generated tests pass on sample file

### **7.3: Azure Document Intelligence Integration**
For PDF files:
- Call Azure Document Intelligence (Form Recognizer) Layout API
- Extract tables, cells, bounding boxes
- Use AI to interpret table structure and generate parser
- Fallback to manual parser creation if AI fails

**Acceptance Criteria**:
- [ ] Document Intelligence extracts tables from sample PDF
- [ ] Generated parser extracts expected fields from PDF
- [ ] Graceful error handling for complex PDFs

### **7.4: Parser Repository Management**
- Store generated parsers in Azure Blob Storage container `parser-repository`
- Organize by machine type and test type (`parsers/zwick/lap_shear_parser.py`)
- Store parser metadata (parser_id, machine, test_type, success_rate)
- Version control integration (Git)

**Acceptance Criteria**:
- [ ] Parsers stored with correct directory structure
- [ ] Metadata includes creation date, success rate, sample files
- [ ] Parser lookup by machine type returns correct parser
- [ ] Version control tracks changes

### **7.5: Frontend Parser Builder UI**
- File upload for sample file
- "Analyze Structure" button showing detected format
- "Generate Parser" button (choose Python or YAML)
- Code preview with syntax highlighting
- "Save to Repository" button

**Acceptance Criteria**:
- [ ] File analysis completes within 10 seconds
- [ ] Parser generation completes within 30 seconds
- [ ] Code preview renders with syntax highlighting
- [ ] Parser saved to repository successfully

---

## Testing Requirements

- [ ] Unit tests for file structure analysis
- [ ] Unit tests for parser code validation
- [ ] Integration tests with Azure Document Intelligence
- [ ] E2E test: Upload file → analyze → generate → save → execute parser
- [ ] Validation: 90%+ of generated parsers work on first attempt

---

**Estimated Effort**: 4-6 days  
**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025
