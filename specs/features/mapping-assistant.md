# Feature Requirements Document (FRD)

## Feature: Intelligent Mapping Assistant

**Feature ID**: FRD-004  
**Phase**: Phase 2 - Intelligent Mapping  
**Priority**: Critical (Core Value Proposition)  
**Status**: Draft  
**Related PRD Requirements**: [REQ-3], [REQ-6], [REQ-8]

---

## 1. Overview

### **Purpose**
Provide AI-assisted mapping between raw file headers, canonical schema properties, and Albert template fields. This feature eliminates the manual, error-prone process of creating CLP data mappings, reducing mapping time from weeks to minutes.

### **Business Value**
- Reduces mapping creation time by 90%
- Achieves 85%+ mapping accuracy through AI
- Supports multi-language synonym resolution (German, English)
- Enables rapid onboarding of new machine types
- Generates CLP-compatible mapping configurations automatically

###**User Personas**
- **Schema Owner**: Creates and approves mappings for test types
- **Developer**: Imports mapping configs into CLP system
- **Domain Scientist**: Validates scientific accuracy of mappings

---

## 2. Functional Requirements

### **Core Capabilities**

#### **2.1 Two-Stage Mapping Process**

**Stage 1: Raw Headers → Schema Properties**
- Input: Raw column headers from machine files (with units)
- Output: Mapping to canonical schema properties
- Use case: Transform diverse machine outputs to standardized schema

**Stage 2: Schema Properties → Albert Template Fields**
- Input: Canonical schema properties
- Output: Mapping to Albert LIMS field names
- Use case: Prepare data for Albert ingestion

#### **2.2 AI-Powered Mapping Suggestions**

**Azure OpenAI Integration**
- **Model**: GPT-4 Turbo (faster, cheaper than GPT-4)
- **Prompt Strategy**: Few-shot learning with examples
- **Context Provided**:
  - Source headers (raw or schema properties)
  - Target field list (schema properties or Albert fields)
  - Unit information
  - Test type and standard
  - Historical mappings (if available)

**Confidence Scoring**
- **High Confidence (85-100%)**: Strong match, likely correct
  - Same name or clear synonym
  - Unit match
  - Appears in similar contexts
- **Medium Confidence (70-84%)**: Probable match, needs review
  - Partial name match
  - Unit convertible
  - Found in related test types
- **Low Confidence (<70%)**: Uncertain, manual review required
  - Ambiguous or unknown term
  - Unit mismatch
  - No historical precedent

**Justification Generation**
- For each mapping, AI provides rationale:
  - Name similarity (e.g., "German synonym for 'Maximum Force'")
  - Unit match/conversion (e.g., "Both use MPa")
  - Contextual evidence (e.g., "Common field in adhesion tests")
  - Historical precedent (e.g., "Mapped same way in Tensile schema")

#### **2.3 Multi-Language Support**

**Supported Languages** (Phase 2)
- German (primary)
- English (primary)

**Extensibility** (Phase 3+)
- French, Spanish, Chinese (deferred)

**Synonym Detection** (with real Lap Shear examples)
- AI recognizes common translations:
  - "Zugscherfestigkeit" (German) → "Tensile Shear Strength" (English) → `Tensile_Shear_Strength_MPa`
  - "Maximalkraft" (German) → "Maximum Force" (English) → `Maximum_Force_N`
  - "Fläche" (German) → "Area" (English) → `Cross_Section_Area_mm2`
- Handles abbreviations and cryptic notation:
  - "Fmax" → "Maximum Force"
  - "F{lo max}" (Zwick notation) → "Tensile Shear Strength"
  - "a{lo 0}" (Zwick notation) → "Overlap Length"
  - "b{lo 0}" (Zwick notation) → "Width"
  - "S{lo 0}" (Zwick notation) → "Cross-Sectional Area"
  - "dL @Fmax" → "Displacement at Maximum Force"

#### **2.4 Unit Normalization & Conversion**

**Unit Catalog Integration**
- Maintain catalog of standard units and conversions
- Handle unit notation variations from real samples:
  - `N/mm²` = `N/mm^2` = `MPa` (pressure/strength)
  - `mm²` = `mm^2` (area)
- Example conversions: 1 MPa = 1 N/mm² = 145.038 psi

**Conversion Suggestions**
- If raw header has "psi" and target has "MPa", suggest conversion
- If raw header has "N/mm²" or "N/mm^2", recognize as equivalent to "MPa"
- Include conversion formula in mapping metadata
- Flag when units cannot be converted (different dimensions)

**Unit Validation**
- Warn if mapping units to incompatible targets
- Example: Cannot map "mm" (length) to "MPa" (pressure)

#### **2.5 Alternative Suggestions**

**Multiple Candidates**
- When confidence <85%, provide top 3 alternatives
- Display with confidence scores
- Allow user to select preferred mapping

**Example**:
| Rank | Target Property | Confidence | Justification |
|------|----------------|------------|---------------|
| 1 | `Displacement_mm` | 72% | Partial match; same unit |
| 2 | `Elongation_mm` | 68% | Similar concept; same unit |
| 3 | `Length_mm` | 45% | Generic length measure |

#### **2.6 Batch Mapping**

**Process Multiple Files**
- Upload multiple machine files simultaneously
- Extract headers from all files
- Deduplicate headers
- Generate mappings for all unique headers in one batch

**Bulk Actions**
- "Accept All High Confidence" (≥85%)
- "Flag All Low Confidence for Review" (<70%)
- "Export Mapping Configuration"

#### **2.7 Historical Learning**

**Mapping History Tracking**
- Store all user-approved mappings in database
- Use as context for future AI suggestions
- Example: If user previously mapped "Fmax" → `Maximum_Force_N`, AI prioritizes this for future "Fmax" headers

**Continuous Improvement**
- As more mappings approved, accuracy improves
- Re-train prompts with successful mappings as few-shot examples

---

## 3. Technical Architecture

### **Technology Stack (Azure-First)**

#### **AI/ML: Pluggable AI Backend (Modular Architecture)**

**Primary Option: Microsoft Foundry Platform** (Recommended for Development)
- **Model**: GPT-5.1-mini or o4-mini (cost-effective reasoning models)
- **Deployment**: Foundry project with serverless or managed compute
- **Features**:
  - Prompt flow orchestration for two-stage mapping
  - Built-in evaluation metrics for accuracy tracking
  - Foundry Agent Service for agentic mapping workflows
  - Integrated tracing and monitoring
- **SDK**: `azure-ai-projects`, `azure-ai-inference`

**Alternative Option: Direct Azure OpenAI Service API** (For Enterprise Customers)
- **Model**: GPT-4o-mini or GPT-4.1-mini (fast, cost-efficient)
- **Deployment**: Standard Azure OpenAI resource
- **Features**:
  - Direct API access with full control
  - Simple authentication (Managed Identity or API Key)
  - Lower latency for simple mapping tasks
  - Proven enterprise reliability
- **SDK**: `openai` Python SDK (v1.0+)

**Implementation Pattern**:
```python
# AI Service Adapter (defined in shared library)
class IAIMappingService:
    def generate_mappings(self, source_headers, target_fields, context) -> List[Mapping]:
        pass
    
    def calculate_confidence(self, source, target, historical_data) -> float:
        pass

# Concrete implementations
class FoundryMappingService(IAIMappingService):
    # Uses Microsoft Foundry with prompt flow
    pass

class AzureOpenAIMappingService(IAIMappingService):
    # Uses Azure OpenAI Chat Completions API
    pass

# Configuration-driven (environment variable or config file)
mapping_service = create_mapping_service(os.getenv("AI_BACKEND", "azure_openai"))
```

**Model Comparison**:

| Aspect | Microsoft Foundry (GPT-5.1-mini) | Azure OpenAI (GPT-4o-mini) |
|--------|----------------------------------|----------------------------|
| **Context Window** | 400K tokens | 128K tokens |
| **Reasoning** | Advanced (built-in reasoning) | Standard (chat completion) |
| **Cost** | $$ (higher for reasoning) | $ (cost-optimized) |
| **Orchestration** | Prompt flow, agents | Manual SDK calls |
| **Best For** | Complex multi-stage workflows | Simple, direct API access |
| **Customer Preference** | Internal platforms | Enterprise direct deployment |

#### **Semantic Search: Azure AI Search** (Optional Enhancement)
- **Use Case**: Find similar properties/fields by description
- **Index**: Property registry + Albert templates
- **Query**: Semantic search for ambiguous headers

#### **Backend: Azure Functions (Python)**
- **Function**: `GenerateMapping` (HTTP POST)
- **Inputs**: Raw headers + target list (schema or Albert)
- **Processing**:
  1. Query historical mappings
  2. Call Azure OpenAI for suggestions
  3. Calculate confidence scores
  4. Return suggestions with justifications
- **Function**: `ApproveMapping` (HTTP POST)
- **Inputs**: User-approved mappings
- **Processing**: Store in database for future learning

#### **Storage: Azure SQL Database**
- **Table**: `mapping_history`
  - `raw_header`, `target_field`, `confidence`, `approved_by`, `timestamp`
- **Table**: `unit_catalog`
  - `unit_name`, `unit_symbol`, `dimension`, `conversion_factor_to_si`

#### **Export: Mapping Configuration Files**
- **Format**: JSON or YAML (CLP-compatible)
- **Structure**:
```json
{
  "mapping_id": "uuid",
  "test_type": "Lap Shear",
  "machine_type": "Zwick",
  "created_date": "2025-11-24",
  "mappings": [
    {
      "raw_header": "Fmax",
      "raw_unit": "N",
      "target_property": "Maximum_Force_N",
      "target_unit": "N",
      "conversion": null,
      "confidence": 95,
      "approved_by": "schema_owner"
    },
    {
      "raw_header": "Zugscherfestigkeit",
      "raw_unit": "MPa",
      "target_property": "Tensile_Shear_Strength_MPa",
      "target_unit": "MPa",
      "conversion": null,
      "confidence": 98,
      "approved_by": "schema_owner"
    }
  ]
}
```

---

## 4. Integration Points

### **Inputs**

| Source | Format | Purpose |
|--------|--------|---------|
| Schema Generation | JSON | Raw headers extracted from files |
| Property Registry | JSON API | Target schema properties |
| Albert Template Exports | Excel/CSV | Target Albert fields |
| Historical Mappings | SQL Database | Context for AI suggestions |

### **Outputs**

| Destination | Format | Purpose |
|-------------|--------|---------|
| Frontend UI | JSON | Display mapping suggestions |
| CLP System | JSON/YAML | Import mapping configuration |
| Mapping History DB | SQL Insert | Store approved mappings |
| Audit Log | JSON | Record all mapping decisions |

### **Dependencies**

- **Property Registry API**: For schema property lists
- **Schema Generation API**: For raw header extraction
- **Azure OpenAI Service**: For AI-powered suggestions
- **Azure SQL Database**: For mapping history
- **Optional: Azure AI Search**: For semantic property search

---

## 5. User Workflows

### **Workflow 1: Create Raw → Schema Mapping**

**Actor**: Schema Owner

1. In Schema Generation workflow, complete Step 2 (all properties mapped)
2. Click "Create Mapping for CLP"
3. System extracts raw headers from uploaded files
4. System loads canonical properties from generated schema
5. AI generates mappings: raw → schema
6. Display mapping table with confidence scores
7. Review:
   - 8 high confidence (green) → auto-accept
   - 2 medium confidence (yellow) → manually confirm
   - 0 low confidence
8. Click "Approve Mappings"
9. Export mapping config JSON
10. Download file for CLP import

**Acceptance Criteria**:
- Mapping generation completes in <30 seconds
- AI achieves 85%+ accuracy for Lap Shear
- Export file validates against CLP schema

### **Workflow 2: Create Schema → Albert Mapping**

**Actor**: Schema Owner

1. Navigate to Schema Library
2. Select "Lap Shear Schema v1.0"
3. Click "Create Albert Mapping"
4. Upload Albert Lap Shear template (Excel export)
5. System extracts Albert field names
6. AI generates mappings: schema → Albert
7. Review mapping table
8. For field "Tensile_Shear_Strength_MPa" → Albert "Result_Strength_MPa" (95% confidence)
9. Click "Accept"
10. Export mapping config
11. Import into CLP

**Acceptance Criteria**:
- Albert field extraction handles Excel format
- AI correctly maps schema properties to Albert fields
- Confidence scores accurately reflect match quality

### **Workflow 3: Handle Unit Conversion**

**Actor**: Schema Owner

1. Upload file with header "Shear Strength" in psi
2. Schema has property `Tensile_Shear_Strength_MPa` (unit: MPa)
3. AI suggests mapping with conversion:
   - Raw: "Shear Strength" (psi)
   - Target: `Tensile_Shear_Strength_MPa` (MPa)
   - Conversion: multiply by 0.00689476
   - Confidence: 90%
4. Review conversion formula
5. Accept mapping
6. Export config includes conversion metadata

**Acceptance Criteria**:
- Unit conversion detected automatically
- Conversion formula mathematically correct
- CLP can apply conversion during data ingestion

---

## 6. Acceptance Criteria

### **Functional Acceptance**

- [ ] AI generates mappings with 85%+ accuracy for Lap Shear test case
- [ ] Confidence scores correlate with actual correctness
- [ ] Multi-language synonyms detected (German/English)
- [ ] Unit conversions suggested correctly
- [ ] Alternative suggestions provided for low-confidence mappings
- [ ] User can accept, edit, or reject each mapping
- [ ] Historical mappings improve future accuracy
- [ ] Export generates CLP-compatible JSON/YAML

### **Non-Functional Acceptance**

- [ ] Mapping generation completes within 30 seconds for 50 headers
- [ ] Azure OpenAI API calls <$0.20 per mapping session
- [ ] Historical mapping queries return results in <500ms
- [ ] Confidence scores calibrated: high confidence = high accuracy

### **Quality Acceptance**

- [ ] No mappings with incompatible units (e.g., mm → MPa)
- [ ] All approved mappings stored in history database
- [ ] Exported configs successfully imported into CLP without errors
- [ ] Domain scientist validates mappings are scientifically correct

---

## 7. Out of Scope

- **Automatic mapping without review**: Always require user approval
- **Complex transformation logic**: Simple unit conversions only; custom formulas deferred
- **Real-time mapping updates**: Batch processing only
- **Multi-target mappings**: One source field → one target field; many-to-one deferred

---

## 8. Testing Strategy

### **Unit Tests**
- Test AI prompt construction
- Test confidence score calculation
- Mock Azure OpenAI responses

### **Integration Tests**
- End-to-end mapping generation
- Test with real Lap Shear files
- Validate exported configs

### **AI Accuracy Tests**
- Baseline: Manual mappings by domain scientist
- Measure: % of AI suggestions matching baseline
- Target: 85%+ accuracy

### **User Acceptance Testing**
- Schema Owner creates mappings for 3 machine types
- Validate exported configs import into CLP successfully

---

## 9. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| AI accuracy <85% | High | Medium | Iterative prompt tuning; expand few-shot examples |
| Unit conversions incorrect | High | Low | Validate against physics; manual review for critical fields |
| Historical data biases AI | Medium | Medium | Periodic review of mapping history; remove outliers |
| CLP import fails | High | Low | Validate export format against CLP schema; integration testing |

---

## 10. Success Metrics

### **Performance Metrics**
- Mapping generation time: <30 seconds for 50 headers
- AI accuracy: 85%+ for Lap Shear, 80%+ for new test types

### **Adoption Metrics**
- 100% of new schemas have associated mappings created via tool
- 3+ machine types mapped in first month

### **Quality Metrics**
- <5% user override rate for high-confidence suggestions
- 0 unit conversion errors in production

### **Cost Metrics**
- Azure OpenAI cost: <$0.20 per mapping session
- Total cost for 20 test types: <$50

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent (spec2cloud workflow)  
**Dependencies**: Property Registry (FRD-001), Schema Generation (FRD-002)
