# Task: Backend API Scaffolding (Azure Functions - Python)

**Task ID**: 002  
**Feature**: Backend Foundation  
**Priority**: Critical (Foundational)  
**Estimated Complexity**: High  
**Dependencies**: 001-task-infrastructure-scaffolding

---

## Task Description

Create the foundational Python backend structure for Azure Functions that will host all REST APIs for the spec2cloud platform. This includes project structure, shared libraries, AI service adapters (Microsoft Foundry + Azure OpenAI), middleware, error handling, and configuration management.

---

## Technical Requirements

### **Project Structure**

```
backend/
├── function_app.py                # Main Azure Functions app entry point
├── host.json                      # Azure Functions host configuration
├── requirements.txt               # Python dependencies
├── local.settings.json.template   # Template for local development settings
├── .funcignore                    # Files to exclude from deployment
├── shared/                        # Shared libraries and utilities
│   ├── __init__.py
│   ├── ai_service_adapter.py     # Abstract AI service interface + implementations
│   ├── database.py               # Azure SQL connection and ORM
│   ├── blob_storage.py           # Azure Blob Storage helper
│   ├── cosmos_db.py              # Cosmos DB Gremlin API client
│   ├── validators.py             # Input validation utilities
│   ├── auth.py                   # Authentication middleware
│   ├── logging_utils.py          # Application Insights logging
│   └── config.py                 # Configuration management
├── models/                        # Data models (Pydantic)
│   ├── __init__.py
│   ├── property.py               # Property model
│   ├── schema.py                 # Schema model
│   ├── mapping.py                # Mapping model
│   └── audit_log.py              # Audit log model
├── functions/                     # Azure Functions (one per API endpoint)
│   ├── property_registry/
│   │   ├── __init__.py
│   │   ├── create_property.py    # POST /api/properties
│   │   ├── get_property.py       # GET /api/properties/{id}
│   │   ├── list_properties.py    # GET /api/properties
│   │   ├── update_property.py    # PUT /api/properties/{id}
│   │   ├── delete_property.py    # DELETE /api/properties/{id}
│   │   └── bulk_import.py        # POST /api/properties/bulk-import
│   ├── schema_generation/
│   │   ├── __init__.py
│   │   ├── analyze_files.py      # POST /api/schemas/analyze
│   │   ├── generate_schema.py    # POST /api/schemas/generate
│   │   └── export_schema.py      # GET /api/schemas/{id}/export
│   ├── mapping_assistant/
│   │   ├── __init__.py
│   │   ├── generate_mappings.py  # POST /api/mappings/generate
│   │   └── approve_mappings.py   # POST /api/mappings/approve
│   ├── parser_builder/
│   │   ├── __init__.py
│   │   ├── analyze_structure.py  # POST /api/parsers/analyze
│   │   └── generate_parser.py    # POST /api/parsers/generate
│   └── audit/
│       ├── __init__.py
│       └── query_logs.py         # GET /api/audit/logs
├── tests/                         # Unit and integration tests
│   ├── __init__.py
│   ├── test_ai_service_adapter.py
│   ├── test_property_api.py
│   ├── test_schema_generation.py
│   └── test_validators.py
└── README.md                      # Backend documentation
```

---

## Core Components

### **1. AI Service Adapter Pattern**

**File**: `shared/ai_service_adapter.py`

Implement abstract interface for pluggable AI backends:

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any
from dataclasses import dataclass

@dataclass
class AIMatch:
    """Represents an AI-suggested property match"""
    raw_header: str
    canonical_property: str
    confidence: float  # 0-100
    justification: str
    alternatives: List[Dict[str, Any]] = None

class IAIService(ABC):
    """Abstract interface for AI service implementations"""
    
    @abstractmethod
    def generate_property_matches(
        self,
        headers: List[str],
        registry_properties: List[Dict[str, Any]],
        context: Dict[str, Any]
    ) -> List[AIMatch]:
        """Generate property matching suggestions"""
        pass
    
    @abstractmethod
    def generate_parser_code(
        self,
        file_structure: Dict[str, Any],
        target_format: str
    ) -> str:
        """Generate parser code from file structure analysis"""
        pass

class FoundryAIService(IAIService):
    """Microsoft Foundry implementation"""
    
    def __init__(self, project_name: str, deployment_name: str):
        from azure.ai.projects import AIProjectClient
        from azure.identity import DefaultAzureCredential
        
        self.client = AIProjectClient(
            credential=DefaultAzureCredential(),
            project=project_name
        )
        self.deployment = deployment_name
    
    def generate_property_matches(self, headers, registry_properties, context):
        # Construct prompt for Foundry
        system_prompt = self._build_system_prompt()
        user_prompt = self._build_user_prompt(headers, registry_properties, context)
        
        # Call Foundry API
        response = self.client.inference.chat.completions.create(
            model=self.deployment,
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": user_prompt}
            ],
            temperature=0.3,  # Lower temperature for consistency
            response_format={"type": "json_object"}
        )
        
        # Parse response and return AIMatch objects
        return self._parse_matches(response)
    
    def _build_system_prompt(self) -> str:
        return """You are an expert in laboratory data standardization.
        Match raw column headers to canonical property names.
        Consider multi-language headers, abbreviations, units, and scientific context.
        Provide confidence scores (0-100) and justifications."""
    
    # ... additional helper methods

class AzureOpenAIService(IAIService):
    """Azure OpenAI Service implementation (API Key authentication)"""
    
    def __init__(self, endpoint: str, api_key: str, deployment_name: str):
        from openai import AzureOpenAI
        
        self.client = AzureOpenAI(
            azure_endpoint=endpoint,
            api_key=api_key,
            api_version="2024-10-21"
        )
        self.deployment = deployment_name
    
    def generate_property_matches(self, headers, registry_properties, context):
        # Similar implementation to Foundry but using OpenAI SDK
        system_prompt = self._build_system_prompt()
        user_prompt = self._build_user_prompt(headers, registry_properties, context)
        
        response = self.client.chat.completions.create(
            model=self.deployment,
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": user_prompt}
            ],
            temperature=0.3,
            response_format={"type": "json_object"}
        )
        
        return self._parse_matches(response)
    
    # ... additional helper methods

def create_ai_service(config: Dict[str, Any]) -> IAIService:
    """Factory function to create AI service based on configuration"""
    backend = config.get("ai_backend", "azure_openai")
    
    if backend == "foundry":
        return FoundryAIService(
            project_name=config["foundry_project"],
            deployment_name=config["foundry_deployment"]
        )
    elif backend == "azure_openai":
        return AzureOpenAIService(
            endpoint=config["openai_endpoint"],
            deployment_name=config["openai_deployment"]
        )
    else:
        raise ValueError(f"Unknown AI backend: {backend}")
```

### **2. Database Connection**

**File**: `shared/database.py`

```python
import pyodbc
from azure.identity import DefaultAzureCredential
from typing import Optional, List, Dict, Any
import struct
from contextlib import contextmanager

class DatabaseConnection:
    """Azure SQL Database connection with simple connection string"""
    
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self._connection: Optional[pyodbc.Connection] = None
    
    @contextmanager
    def get_connection(self):
        """Context manager for database connections"""
        conn = None
        try:
            # Simple connection with username/password in connection string
            conn = pyodbc.connect(self.connection_string)
            
            yield conn
            conn.commit()
        except Exception as e:
            if conn:
                conn.rollback()
            raise
        finally:
            if conn:
                conn.close()
    
    def execute_query(self, query: str, params: tuple = None) -> List[Dict[str, Any]]:
        """Execute SELECT query and return results as list of dicts"""
        with self.get_connection() as conn:
            cursor = conn.cursor()
            if params:
                cursor.execute(query, params)
            else:
                cursor.execute(query)
            
            columns = [column[0] for column in cursor.description]
            results = []
            for row in cursor.fetchall():
                results.append(dict(zip(columns, row)))
            
            return results
    
    def execute_non_query(self, query: str, params: tuple = None) -> int:
        """Execute INSERT/UPDATE/DELETE and return affected rows"""
        with self.get_connection() as conn:
            cursor = conn.cursor()
            if params:
                cursor.execute(query, params)
            else:
                cursor.execute(query)
            
            return cursor.rowcount
```

### **3. Configuration Management**

**File**: `shared/config.py`

```python
import os
from typing import Dict, Any
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

class Config:
    """Application configuration from environment variables (demo/sandbox mode)"""
    
    def __init__(self):
        self.environment = os.getenv("ENVIRONMENT", "dev")
    
    def get_secret(self, secret_name: str) -> str:
        """Get configuration value from environment variable"""
        value = os.getenv(secret_name)
        if not value:
            raise ValueError(f"Environment variable {secret_name} not set")
        return value
    
    def get_ai_config(self) -> Dict[str, Any]:
        """Get AI service configuration"""
        # Demo phase: Azure OpenAI only with API key
        return {
            "ai_backend": "azure_openai",
            "openai_endpoint": self.get_secret("AZURE_OPENAI_ENDPOINT"),
            "openai_api_key": self.get_secret("AZURE_OPENAI_API_KEY"),
            "openai_deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT", "gpt-4o-mini")
        }
    
    def get_database_config(self) -> Dict[str, Any]:
        """Get database configuration"""
        return {
            "connection_string": self.get_secret("SQL_CONNECTION_STRING")
        }
    
    def get_blob_storage_config(self) -> Dict[str, Any]:
        """Get Blob Storage configuration"""
        return {
            "connection_string": self.get_secret("BLOB_STORAGE_CONNECTION_STRING"),
            "container_uploads": "uploads",
            "container_outputs": "outputs"
        }

# Global config instance
config = Config()
```

### **4. Logging Utilities**

**File**: `shared/logging_utils.py`

```python
import logging
from opencensus.ext.azure.log_exporter import AzureLogHandler
from opencensus.ext.azure import metrics_exporter
import os
from typing import Dict, Any

class ApplicationInsightsLogger:
    """Azure Application Insights logging wrapper"""
    
    def __init__(self):
        self.logger = logging.getLogger(__name__)
        self.logger.setLevel(logging.INFO)
        
        # Add Azure Application Insights handler
        instrumentation_key = os.getenv("APPINSIGHTS_INSTRUMENTATIONKEY")
        if instrumentation_key:
            handler = AzureLogHandler(connection_string=f"InstrumentationKey={instrumentation_key}")
            self.logger.addHandler(handler)
        
        # Console handler for local development
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.DEBUG)
        self.logger.addHandler(console_handler)
    
    def log_event(self, event_name: str, properties: Dict[str, Any]):
        """Log custom event to Application Insights"""
        self.logger.info(
            event_name,
            extra={
                'custom_dimensions': properties
            }
        )
    
    def log_ai_decision(self, raw_header: str, suggestion: str, confidence: float, justification: str):
        """Log AI property matching decision"""
        self.log_event("AIPropertyMatching", {
            "raw_header": raw_header,
            "suggestion": suggestion,
            "confidence": confidence,
            "justification": justification
        })
    
    def log_user_action(self, user_id: str, action: str, details: Dict[str, Any]):
        """Log user action"""
        self.log_event("UserAction", {
            "user_id": user_id,
            "action": action,
            **details
        })

# Global logger instance
app_logger = ApplicationInsightsLogger()
```

### **5. Validators**

**File**: `shared/validators.py`

```python
from typing import List, Dict, Any
import re
from pydantic import BaseModel, validator, ValidationError

class PropertyValidator(BaseModel):
    """Pydantic model for property validation"""
    canonical_name: str
    description: str
    standard_unit: str
    data_type: str
    
    @validator('canonical_name')
    def validate_canonical_name(cls, v):
        if not re.match(r'^[A-Za-z0-9_]+$', v):
            raise ValueError('Canonical name must contain only alphanumeric characters and underscores')
        return v
    
    @validator('data_type')
    def validate_data_type(cls, v):
        allowed_types = ['float', 'integer', 'string', 'boolean', 'datetime']
        if v not in allowed_types:
            raise ValueError(f'Data type must be one of: {", ".join(allowed_types)}')
        return v

def validate_property_input(data: Dict[str, Any]) -> Dict[str, Any]:
    """Validate property creation/update input"""
    try:
        property_data = PropertyValidator(**data)
        return property_data.dict()
    except ValidationError as e:
        raise ValueError(f"Validation failed: {e.json()}")

def validate_unit_compatibility(source_unit: str, target_unit: str) -> bool:
    """Check if two units are compatible (same dimension)"""
    # Unit catalog with dimension mappings
    unit_dimensions = {
        "MPa": "pressure", "N/mm²": "pressure", "N/mm^2": "pressure", "psi": "pressure",
        "N": "force", "kN": "force",
        "mm": "length", "cm": "length", "m": "length",
        "mm²": "area", "mm^2": "area", "cm²": "area",
        "Pa·s": "viscosity", "cP": "viscosity"
    }
    
    source_dim = unit_dimensions.get(source_unit)
    target_dim = unit_dimensions.get(target_unit)
    
    if not source_dim or not target_dim:
        return False  # Unknown units
    
    return source_dim == target_dim
```

---

## Azure Functions Configuration

### **File**: `host.json`

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond": 20
      }
    }
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  },
  "functionTimeout": "00:10:00"
}
```

### **File**: `requirements.txt`

```
azure-functions==1.18.0
azure-storage-blob==12.19.0
azure-ai-inference==1.0.0b1  # For future Foundry support
openai==1.12.0  # Azure OpenAI SDK
pyodbc==5.0.1
pydantic==2.5.0
pandas==2.1.4
openpyxl==3.1.2  # Excel support
python-multipart==0.0.6  # File upload support
jsonschema==4.20.0  # JSON Schema validation
opencensus-ext-azure==1.1.11  # Application Insights
```

**Note**: Removed `azure-identity` and `azure-keyvault-secrets` for demo phase simplicity.

### **File**: `local.settings.json.template`

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "ENVIRONMENT": "dev",
    "AZURE_OPENAI_ENDPOINT": "https://your-openai.openai.azure.com",
    "AZURE_OPENAI_API_KEY": "your-api-key-here",
    "AZURE_OPENAI_DEPLOYMENT": "gpt-4o-mini",
    "SQL_CONNECTION_STRING": "Driver={ODBC Driver 18 for SQL Server};Server=tcp:your-server.database.windows.net,1433;Database=spec2cloud;Uid=sqladmin;Pwd=YourPassword123!;Encrypt=yes;TrustServerCertificate=no;",
    "BLOB_STORAGE_CONNECTION_STRING": "DefaultEndpointsProtocol=https;AccountName=yourblob;AccountKey=...;EndpointSuffix=core.windows.net",
    "APPINSIGHTS_INSTRUMENTATIONKEY": "your-instrumentation-key"
  }
}
```

**Key Changes for Demo**:
- Added `AZURE_OPENAI_API_KEY` for API key authentication
- Full SQL connection string with username/password (`Uid=sqladmin;Pwd=...`)
- Removed `USE_KEY_VAULT` flag
- Database name changed to `spec2cloud`

---

## Sample Function Implementation

### **File**: `functions/property_registry/create_property.py`

```python
import azure.functions as func
import json
from shared.database import DatabaseConnection
from shared.config import config
from shared.validators import validate_property_input
from shared.logging_utils import app_logger

def main(req: func.HttpRequest) -> func.HttpResponse:
    """
    Create new property in registry
    POST /api/properties
    """
    try:
        # Parse request body
        req_body = req.get_json()
        
        # Validate input
        validated_data = validate_property_input(req_body)
        
        # Get database connection
        db_config = config.get_database_config()
        db = DatabaseConnection(db_config["connection_string"])
        
        # Check for duplicate canonical name
        existing = db.execute_query(
            "SELECT property_id FROM properties WHERE canonical_name = ?",
            (validated_data["canonical_name"],)
        )
        
        if existing:
            return func.HttpResponse(
                json.dumps({
                    "error": f"Property '{validated_data['canonical_name']}' already exists",
                    "existing_id": str(existing[0]["property_id"])
                }),
                status_code=409,
                mimetype="application/json"
            )
        
        # Insert new property
        query = """
        INSERT INTO properties (canonical_name, description, standard_unit, data_type)
        OUTPUT INSERTED.property_id
        VALUES (?, ?, ?, ?)
        """
        
        result = db.execute_query(
            query,
            (
                validated_data["canonical_name"],
                validated_data["description"],
                validated_data["standard_unit"],
                validated_data["data_type"]
            )
        )
        
        property_id = result[0]["property_id"]
        
        # Log action
        app_logger.log_user_action(
            user_id=req.headers.get("X-User-Id", "anonymous"),
            action="PropertyCreated",
            details={"property_id": str(property_id), "canonical_name": validated_data["canonical_name"]}
        )
        
        return func.HttpResponse(
            json.dumps({
                "property_id": str(property_id),
                "canonical_name": validated_data["canonical_name"],
                "message": "Property created successfully"
            }),
            status_code=201,
            mimetype="application/json"
        )
    
    except ValueError as e:
        return func.HttpResponse(
            json.dumps({"error": str(e)}),
            status_code=400,
            mimetype="application/json"
        )
    except Exception as e:
        app_logger.logger.error(f"Error creating property: {str(e)}")
        return func.HttpResponse(
            json.dumps({"error": "Internal server error"}),
            status_code=500,
            mimetype="application/json"
        )
```

---

## Acceptance Criteria

### **Project Structure**
- [ ] All directories and files created as specified
- [ ] `requirements.txt` includes all necessary dependencies
- [ ] `host.json` configured with proper timeout and logging
- [ ] `.funcignore` excludes unnecessary files from deployment

### **Shared Libraries**
- [ ] `ai_service_adapter.py` implements abstract interface and both Foundry + Azure OpenAI
- [ ] `database.py` supports Managed Identity authentication
- [ ] `config.py` retrieves secrets from Key Vault or environment variables
- [ ] `validators.py` validates all input models
- [ ] `logging_utils.py` logs to Application Insights

### **Sample Functions**
- [ ] `create_property.py` successfully creates property and returns 201
- [ ] Duplicate property creation returns 409 with clear error
- [ ] Invalid input returns 400 with validation errors
- [ ] All functions log actions to Application Insights

### **Testing**
- [ ] Unit tests cover ≥85% of shared library code
- [ ] Integration tests validate database connectivity with connection string
- [ ] AI service works with Azure OpenAI using API key
- [ ] All tests pass locally

### **Documentation**
- [ ] `README.md` explains project structure and setup
- [ ] Each shared module has docstrings
- [ ] API endpoint functions have clear docstrings
- [ ] Configuration template documented with demo values

---

## Testing Requirements (Demo Phase)

### **Unit Tests**
- [ ] Test `validate_property_input` with valid and invalid data
- [ ] Test `validate_unit_compatibility` with various unit pairs
- [ ] Mock database calls and test query execution
- [ ] Mock Azure OpenAI responses

### **Integration Tests (Simplified)**
- [ ] Deploy to local Azure Functions runtime (`func start`)
- [ ] Test database connection with actual Azure SQL using connection string
- [ ] Test AI service with live Azure OpenAI endpoint using API key
- [ ] Test file upload to Blob Storage with connection string
- [ ] Test Application Insights logging

**Note**: No Foundry testing required for demo phase.

---

## Estimated Effort

**Demo Phase**: **2-3 days**
- 1 day: Project structure and shared libraries (database, config, validators, logging)
- 0.5 day: Azure OpenAI service implementation (API key auth only)
- 0.5 day: Sample function implementation
- 0.5 day: Unit tests for shared libraries
- 0.5 day: Integration testing and configuration

**Comparison to Original**: Reduced from 3-4 days by removing:
- Key Vault integration
- Managed Identity setup
- Microsoft Foundry adapter
- Complex authentication flows

---

## Success Metrics

- Backend deploys locally with `func start` without errors
- All shared libraries pass unit tests with ≥85% coverage
- Sample function successfully creates property via API call
- Azure OpenAI integration works with API key authentication
- Application Insights receives telemetry
- Local development environment fully functional

---

**Document Status**: Updated v2.0 (Demo/Sandbox Mode)  
**Last Updated**: November 24, 2025  
**Owner**: Developer (spec2cloud workflow)  
**Dependencies**: 001-task-infrastructure-scaffolding
**Note**: Simplified for rapid demo/sandbox development
