# Task: Mapping Assistant Implementation

**Task ID**: 006  
**Feature**: Intelligent Mapping Assistant (FRD-004)  
**Priority**: High  
**Estimated Complexity**: High  
**Dependencies**: 001-infrastructure, 002-backend-scaffolding, 004-property-registry, 005-schema-generation

---

## Task Description

Implement two-stage AI-powered mapping (raw headers → schema, schema → Albert fields) with confidence scoring, unit conversion, historical learning, and CLP export format.

---

## Sub-Tasks

### **6.1: Two-Stage Mapping Engine**
Implement mapping services:
- **Stage 1**: Raw file headers → canonical schema properties
- **Stage 2**: Schema properties → Albert template fields
- Query historical mappings from `mapping_history` table
- Use AI for synonym detection and confidence scoring

**Acceptance Criteria**:
- [ ] Stage 1 maps 10 real Lap Shear headers with 85%+ accuracy
- [ ] Stage 2 maps schema properties to Albert fields correctly
- [ ] Historical precedents influence AI suggestions
- [ ] Alternative suggestions provided for low-confidence matches (<70%)

### **6.2: Unit Conversion & Validation**
- Implement unit catalog with dimension mappings (pressure, force, length, area, viscosity)
- Handle notation variations: `N/mm²` = `N/mm^2` = `MPa`
- Suggest conversions when compatible units differ
- Validate incompatible unit mappings (e.g., mm → MPa)

**Acceptance Criteria**:
- [ ] Unit compatibility check works for 20+ unit pairs
- [ ] Conversion formulas correct (MPa ↔ psi, mm ↔ cm, etc.)
- [ ] Incompatible mappings flagged with clear error

### **6.3: CLP Export Format**
Generate JSON/YAML mapping configuration:
```json
{
  "mapping_id": "uuid",
  "test_type": "Lap Shear",
  "mappings": [
    {
      "raw_header": "F{lo max}",
      "target_property": "Tensile_Shear_Strength_MPa",
      "confidence": 92,
      "approved_by": "user_id"
    }
  ]
}
```

**Acceptance Criteria**:
- [ ] Export format validates against CLP schema
- [ ] Includes all approved mappings with metadata
- [ ] Import into CLP succeeds without errors

### **6.4: Frontend Mapping UI**
- Mapping generation form (select source, target, test type)
- Review table with confidence colors (green/yellow/red)
- Accept/edit/reject actions per mapping
- Unit conversion display with formula
- Export button

**Acceptance Criteria**:
- [ ] Mapping generation completes within 30 seconds for 50 headers
- [ ] Confidence colors match thresholds (>85% green, 70-85% yellow, <70% red)
- [ ] User can edit mappings via dropdown with type-ahead search
- [ ] Export downloads JSON/YAML file

---

## Testing Requirements

- [ ] Unit tests for mapping engine with mocked AI responses
- [ ] Unit tests for unit conversion logic
- [ ] Integration tests with live Azure OpenAI and SQL database
- [ ] E2E test: Generate mappings → review → approve → export
- [ ] Accuracy test: Validate 85%+ correctness on Lap Shear baseline

---

**Estimated Effort**: 5-6 days  
**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025
