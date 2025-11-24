# Feature Requirements Document (FRD)

## Feature: Parser Builder

**Feature ID**: FRD-005  
**Phase**: Phase 3 - Automation  
**Priority**: Medium (Efficiency Enhancement)  
**Status**: Draft  
**Related PRD Requirements**: [REQ-4], [REQ-7], [REQ-11]

---

## 1. Overview

### **Purpose**
Automate the creation of file parsers for diverse machine output formats by using AI to analyze file structure and generate parsing code or configuration. This eliminates the manual effort of writing custom parsers for each machine/file type.

### **Business Value**
- Reduces parser development time from days to minutes
- Enables rapid onboarding of new machine types
- Handles format variations (delimiters, encodings, multi-line headers)
- Generates validated, tested parser code
- Supports TXT, CSV, Excel, and PDF formats

### **User Personas**
- **Developer**: Primary user who generates and integrates parsers into CLP
- **Schema Owner**: Uses parsers to extract data for schema generation
- **Product Owner**: Monitors parser coverage across machine types

---

## 2. Functional Requirements

### **Core Capabilities**

#### **2.1 File Structure Analysis**

**Supported File Types**
- **TXT**: Tab, comma, semicolon, pipe delimiters
- **CSV**: Various encodings (UTF-8, Windows-1252, ISO-8859-1)
- **Excel**: XLSX, XLS (multiple sheets)
- **PDF**: Simple tabular layouts (leveraging Azure Document Intelligence)

**Analysis Process**
1. **Upload File**: User provides sample file
2. **Detect Encoding**: Auto-detect character encoding (UTF-8, Windows-1252)
3. **Identify Structure** (real Lap Shear complexity):
   - Locate header row(s) - may be cryptic notation like `a{lo 0};b{lo 0};F{lo max}`
   - Detect delimiter - semicolon (`;`) common in German lab instruments
   - Identify unit rows - often separate row after headers: `mm;mm;mm²;N;MPa`
   - Handle inline units in parentheses: `(N/mm^2)` or `(mm²)`
   - Find data table boundaries
   - Extract metadata blocks: `Auftrags-Nr.;30245;Prüfnorm;ISO 4587;Prüfer;ACC`
   - Handle decimal commas: `"19,59"` instead of `19.59`
   - Handle quoted values: `"12,5";"25,0";"CF"`
4. **Extract Sample Data**: First 5-10 rows for validation
5. **AI Analysis**: Use Azure OpenAI to interpret structure and generate parser

#### **2.2 AI-Powered Parser Generation**

**AI Backend Architecture** (Modular, Customer-Configurable)

**Primary Option: Microsoft Foundry Platform**
- **Model**: GPT-5.1-codex or GPT-4.1 (code generation optimized)
- **Deployment**: Foundry project with code generation prompt flow
- **Benefits**:
  - Optimized for generating Python/YAML parser code
  - Multi-agent coordination for validation + testing
  - Built-in code execution sandbox (secure testing)
  - Integrated with GitHub Copilot for refinement
- **SDK**: `azure-ai-projects` with code generation tools

**Alternative Option: Direct Azure OpenAI Service API**
- **Model**: GPT-4o or GPT-4.1-mini (general-purpose code generation)
- **Deployment**: Standard Azure OpenAI resource
- **Benefits**:
  - Direct control over code generation prompts
  - Simple REST API integration
  - Suitable for customers with existing OpenAI workflows
  - Lower complexity for single-task code generation
- **SDK**: `openai` Python SDK with code-focused system prompts

**Parser Generation Process**:
1. **File Structure Analysis**: Extract metadata, headers, units, data boundaries
2. **AI Prompt Construction**:
   - System: "You are a Python code generator for laboratory data parsers..."
   - Context: File structure analysis, sample rows, target schema
   - Instructions: Generate parser function with error handling, validation, docstrings
3. **Code Generation** (via AI service):
   - Model generates complete Python function or YAML configuration
   - Includes imports, type hints, exception handling
   - Adds unit tests (pytest format)
4. **Validation & Testing**:
   - Syntax check (AST parsing)
   - Run generated parser on sample file
   - Verify output matches expected schema structure
   - Store parser in repository with metadata

**Parser Types**

**Option A: Python Code Generation**
- Generate standalone Python function
- Uses libraries: `pandas`, `openpyxl`, `pdfplumber`
- Includes error handling and validation
- Example output:
```python
def parse_zwick_lap_shear(file_path: str) -> dict:
    """
    Parse Zwick Lap Shear TXT file (real format from AMC samples).
    
    Handles:
    - Semicolon delimiter
    - Metadata header block (Auftrags-Nr., Prüfnorm, etc.)
    - Separate unit row
    - Cryptic column names (a{lo 0}, F{lo max})
    - Decimal commas ("19,59")
    - Quoted values
    
    Args:
        file_path: Path to TXT file
        
    Returns:
        dict with 'metadata', 'headers', 'units', 'data' keys
    """
    import pandas as pd
    import re
    
    # Read first line to extract metadata
    with open(file_path, 'r', encoding='Windows-1252') as f:
        metadata_line = f.readline().strip()
    
    # Parse metadata: Auftrags-Nr.;30245;Prüfnorm;ISO 4587;Werkstoff;...
    metadata_parts = metadata_line.split(';')
    metadata = {metadata_parts[i]: metadata_parts[i+1] 
                for i in range(0, len(metadata_parts)-1, 2)}
    
    # Read main data table (skip metadata line)
    df = pd.read_csv(file_path, sep=';', encoding='Windows-1252', 
                     skiprows=1, quotechar='"', decimal=',')
    
    # Extract headers from first row (after metadata)
    headers = df.columns.tolist()
    
    # Extract units from first data row
    units = df.iloc[0].tolist()
    
    # Actual data starts from second data row
    data = df.iloc[1:].reset_index(drop=True)
    
    return {
        'metadata': metadata,
        'headers': headers,
        'units': units,
        'data': data.to_dict('records')
    }
```

**Option B: YAML Configuration**
- Declarative parser definition
- CLP can interpret YAML without code changes
- Example (real Zwick format):
```yaml
parser_name: "Zwick Lap Shear AMC TXT Parser"
file_format: "txt"
encoding: "Windows-1252"
delimiter: ";"
decimal_separator: ","  # German decimal format
quote_char: "\""
metadata:
  row: 0
  format: "key_value_pairs"  # Auftrags-Nr.;30245;Prüfnorm;ISO 4587
  parse_keys: ["Auftrags-Nr.", "Prüfnorm", "Werkstoff", "Prüfer", "Prüfdatum"]
header_row: 1
unit_row: 2
data_start_row: 3
columns:
  - name: "a{lo 0}"
    unit: "mm"
    type: "float"
    description: "Overlap length"
  - name: "b{lo 0}"
    unit: "mm"
    type: "float"
    description: "Width"
  - name: "S{lo 0}"
    unit: "mm²"
    type: "float"
    description: "Cross-sectional area"
  - name: "Fmax"
    unit: "N"
    type: "float"
    description: "Maximum force"
  - name: "F{lo max}"
    unit: "MPa"
    type: "float"
    description: "Tensile shear strength"
  - name: "dL @Fmax"
    unit: "mm"
    type: "float"
    description: "Displacement at max force"
  - name: "Kommentar"
    unit: null
    type: "string"
    description: "Comment (CF/AF)"
validation:
  - check: "not_null"
    columns: ["Fmax", "F{lo max}"]
  - check: "positive"
    columns: ["a{lo 0}", "b{lo 0}", "S{lo 0}", "Fmax"]
```

#### **2.3 PDF Parsing with Azure Document Intelligence**

**Azure Document Intelligence (Form Recognizer)**
- **Use Case**: Extract tables from PDF reports
- **Models**: Prebuilt "layout" model for table detection
- **Process**:
  1. Upload PDF to Azure Document Intelligence
  2. API returns detected tables, cells, text
  3. AI interprets table structure (headers, units, data)
  4. Generate parser code to extract specific fields

**Fallback for Complex PDFs**
- If Azure Document Intelligence fails, flag for manual parser creation
- Provide extracted text for manual analysis

#### **2.4 Parser Validation & Testing**

**Automated Tests Generation**
- For each parser, generate pytest test cases
- Test cases include:
  - Parse sample file successfully
  - Extract expected number of headers
  - Validate data types
  - Handle malformed rows (missing values, wrong types)
  
**Example Test**:
```python
def test_parse_zwick_lap_shear():
    result = parse_zwick_lap_shear('tests/data/zwick_sample.txt')
    
    assert len(result['headers']) == 7
    assert 'Fmax' in result['headers']
    assert result['units'][result['headers'].index('Fmax')] == 'N'
    assert len(result['data']) > 0
    assert isinstance(result['data'][0]['Fmax'], float)
```

**Validation Checks**
- Ensure headers match expected count
- Verify units are extracted correctly
- Check data types (numeric vs. string)
- Confirm no empty required fields

#### **2.5 Parser Library Management**

**Parser Repository**
- Store all generated parsers in repository
- Organize by machine type and test type
- Version control with Git
- Example structure:
```
parsers/
├── zwick/
│   ├── lap_shear_parser.py
│   ├── tensile_parser.py
│   └── tests/
├── instron/
│   ├── lap_shear_parser.py
│   └── tests/
└── shimadzu/
    └── lap_shear_parser.py
```

**Parser Metadata**
- Each parser has metadata file:
```json
{
  "parser_id": "zwick_lap_shear_v1",
  "machine_manufacturer": "Zwick",
  "test_type": "Lap Shear",
  "supported_formats": ["txt"],
  "created_date": "2025-11-24",
  "last_tested": "2025-11-24",
  "success_rate": 98.5,
  "sample_files": ["zwick_sample_1.txt", "zwick_sample_2.txt"]
}
```

#### **2.6 Parser Execution & Integration**

**Standalone Execution**
- Run parser as Python script: `python parse_zwick_lap_shear.py input.txt`
- Output: JSON with headers, units, data

**CLP Integration**
- CLP calls parser via API: `POST /parse` with file upload
- Returns structured data for schema generation

---

## 3. Technical Architecture

### **Technology Stack (Azure-First)**

#### **AI Code Generation: Azure OpenAI Service**
- **Model**: GPT-4 (strong at code generation)
- **Input**: File structure analysis + sample rows
- **Output**: Python parser code or YAML config

#### **PDF Processing: Azure Document Intelligence**
- **API**: Azure Form Recognizer (Layout Model)
- **Input**: PDF file
- **Output**: Detected tables, cells, bounding boxes, text
- **Pricing**: Pay-per-page analyzed

#### **Backend: Azure Functions (Python)**
- **Function**: `AnalyzeFileStructure` (HTTP POST with file upload)
- **Function**: `GenerateParser` (HTTP POST with analysis results)
- **Function**: `ValidateParser` (HTTP POST with parser code + test file)
- **Function**: `ExecuteParser` (HTTP POST with file + parser ID)

#### **Storage: Azure Blob Storage**
- **Container**: `parser-repository`
  - Store generated parser code
  - Store test files
  - Store parser metadata

#### **Code Repository: Azure DevOps Repos** (or GitHub)
- Version control for all parsers
- CI/CD pipeline for parser testing
- Automated deployment to parser library

---

## 4. Integration Points

### **Inputs**

| Source | Format | Purpose |
|--------|--------|---------|
| User (via UI) | TXT, CSV, Excel, PDF | Sample file for parser generation |
| Azure Document Intelligence | JSON | Extracted PDF table structure |
| Azure OpenAI | Python code or YAML | Generated parser |

### **Outputs**

| Destination | Format | Purpose |
|-------------|--------|---------|
| Parser Repository | Python/.py, YAML | Store parsers |
| Schema Generation | JSON | Extracted headers and data |
| CLP System | JSON API | Structured data for ingestion |
| Azure DevOps | Git commit | Version control |

### **Dependencies**

- **Azure OpenAI Service**: For code generation
- **Azure Document Intelligence**: For PDF parsing
- **Azure Blob Storage**: For parser storage
- **Azure Functions**: For parser execution
- **Optional: Azure DevOps**: For CI/CD

---

## 5. User Workflows

### **Workflow 1: Generate Parser for New Machine Type**

**Actor**: Developer

1. Navigate to "Parser Builder" in UI
2. Upload sample file: `Shimadzu_LapShear.csv`
3. Click "Analyze File"
4. System detects: CSV, UTF-8 encoding, comma delimiter, headers in row 1
5. Click "Generate Parser"
6. Choose output: "Python Code"
7. AI generates `parse_shimadzu_lap_shear.py`
8. Review code in built-in editor
9. Click "Generate Tests"
10. AI generates pytest tests
11. Click "Run Tests" → All pass
12. Click "Save to Repository"
13. Parser stored in `parsers/shimadzu/`

**Acceptance Criteria**:
- File analysis completes in <10 seconds
- Generated parser code is syntactically valid Python
- Tests pass on sample file
- Parser saved to repository successfully

### **Workflow 2: Parse PDF with Azure Document Intelligence**

**Actor**: Developer

1. Upload PDF: `Zwick_Protocol.pdf`
2. Click "Analyze PDF"
3. System calls Azure Document Intelligence
4. API returns detected tables
5. AI interprets table structure
6. Generate parser code to extract specific columns
7. Test parser on sample PDF
8. Save to repository

**Acceptance Criteria**:
- Azure Document Intelligence extracts tables correctly
- Generated parser extracts expected fields from PDF
- Parser handles multi-page PDFs

### **Workflow 3: Use Parser in Schema Generation**

**Actor**: Schema Owner

1. In Schema Generation workflow, upload files
2. System detects machine type: "Zwick"
3. Looks up parser from repository: `parse_zwick_lap_shear.py`
4. Executes parser to extract headers and data
5. Passes structured data to Schema Generation AI
6. Continue with schema generation workflow

**Acceptance Criteria**:
- Parser automatically selected based on file pattern
- Parser executes without errors
- Extracted data used in schema generation

---

## 6. Acceptance Criteria

### **Functional Acceptance**

- [ ] Analyze file structure for TXT, CSV, Excel, PDF
- [ ] Generate valid Python parser code for 90%+ of TXT/CSV files
- [ ] Generate YAML parser config for simple formats
- [ ] Azure Document Intelligence extracts tables from PDFs
- [ ] Generated parsers pass automated tests
- [ ] Parsers stored in repository with metadata
- [ ] Parser execution via API returns structured JSON

### **Non-Functional Acceptance**

- [ ] File analysis completes in <10 seconds
- [ ] Parser generation completes in <30 seconds
- [ ] Azure Document Intelligence processes PDF in <20 seconds per page
- [ ] Generated parser code follows PEP 8 style guidelines
- [ ] All parsers include docstrings and type hints

### **Quality Acceptance**

- [ ] 90%+ of generated parsers work on first attempt
- [ ] Generated tests achieve 80%+ code coverage
- [ ] Parser repository organized and documented
- [ ] No hardcoded file paths in generated code

---

## 7. Out of Scope

- **Binary format parsing**: Proprietary machine formats (e.g., `.raw` files) deferred
- **Real-time parsing**: Batch only; streaming deferred
- **Parser optimization**: Focus on correctness, not performance
- **Multi-format parsers**: One parser per format; combined parsers deferred

---

## 8. Testing Strategy

### **Unit Tests**
- Test file structure analysis logic
- Mock Azure OpenAI for code generation
- Test parser code validation

### **Integration Tests**
- Generate parser for sample files
- Execute generated parser
- Validate output structure

### **AI Code Quality Tests**
- Run linters on generated code (flake8, mypy)
- Test generated code executes without errors
- Validate test coverage of generated tests

---

## 9. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Generated code has bugs | High | Medium | Automated testing; manual code review |
| Azure Document Intelligence fails on complex PDFs | Medium | Medium | Fallback to manual parser creation |
| Parser works on sample but fails on variations | Medium | High | Test with multiple sample files; iterative refinement |
| Generated parsers not maintainable | Medium | Low | Follow coding standards; include comments |

---

## 10. Success Metrics

### **Performance Metrics**
- Parser generation time: <30 seconds
- Parser success rate: 90%+ on first attempt

### **Adoption Metrics**
- 10+ parsers generated in first month
- 5+ machine types supported

### **Quality Metrics**
- Generated parsers pass 95%+ of test cases
- <10% manual correction rate

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent (spec2cloud workflow)  
**Dependencies**: Schema Generation (FRD-002)
