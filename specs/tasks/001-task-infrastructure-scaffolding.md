# Task: Azure Infrastructure Scaffolding

**Task ID**: 001  
**Feature**: Infrastructure Foundation  
**Priority**: Critical (Foundational)  
**Estimated Complexity**: High  
**Dependencies**: None (foundational task)

---

## Task Description

Set up the complete Azure infrastructure foundation for the spec2cloud platform, including all cloud resources, Infrastructure as Code (IaC) templates, deployment pipelines, and monitoring configuration. This task establishes the technical foundation that all other features will build upon.

---

## Technical Requirements

### **Azure Resources to Provision**

#### **1. Resource Group**
- Name: `rg-spec2cloud-{environment}` (dev, staging, prod)
- Region: Select based on Henkel's compliance requirements (e.g., `westeurope` for GDPR)

#### **2. Azure Functions (Backend API)**
- Plan: Consumption (serverless) - simple and cost-effective
- Runtime: Python 3.11
- Authentication: API Keys (no Managed Identity for demo phase)
- Functions:
  - Property Registry CRUD operations
  - Schema Generation orchestration
  - Mapping Assistant services
  - Parser Builder operations
  - Audit logging endpoints

#### **3. Azure SQL Database**
- Tier: Basic (cheapest, sufficient for demo)
- Single Database: `spec2cloud` (consolidate all tables)
- Tables:
  - `properties` (Property Registry)
  - `audit_logs` (Data Consistency & Audit)
  - `mapping_history` (Mapping Assistant)
- Firewall: **Allow all Azure services + your IP** (open for demo)
- Authentication: SQL username/password (stored in environment variables)
- Backup: Default automated backups

#### **4. ~~Azure Cosmos DB (Gremlin API)~~ - DEFERRED**
- **Status**: Not needed for Phase 1 demo
- **Rationale**: Ontology feature (FRD-007) is future/advanced phase
- **Alternative**: Use Azure SQL tables for simple synonym storage if needed
- **Action**: Skip Cosmos DB provisioning entirely

#### **5. Azure Blob Storage**
- Account Type: StorageV2 (general-purpose v2)
- Tier: Hot (simple, no lifecycle policies for demo)
- Containers:
  - `uploads` (all file uploads)
  - `outputs` (generated schemas, parsers, exports)
- Access: **Connection string-based** (no SAS tokens for demo)
- Public Access: Disabled (but relaxed CORS for development)

#### **6. Azure OpenAI Service** (Simplified)
- Resource: Single Azure OpenAI account
- Deployment: **GPT-4o-mini only** (cost-effective for demo)
- Authentication: **API Key** (store in environment variable)
- Rate Limits: Default (sufficient for demo usage)
- Region: Choose nearest region with GPT-4o-mini availability
- **Note**: Microsoft Foundry deferred to production phase

#### **7. Azure Application Insights**
- Instrumentation: Connect to Azure Functions, web apps
- Retention: 90 days (standard)
- Features:
  - API performance monitoring
  - Error tracking and alerting
  - Custom events for AI decisions
  - Usage analytics

#### **8. Azure Static Web Apps**
- Tier: Free or Standard
- Framework: React (built with Vite)
- Features:
  - Global CDN distribution
  - Automatic HTTPS
  - GitHub Actions CI/CD integration
  - Custom domain support (future)

#### **9. ~~Azure Monitor & Alerts~~ - DEFERRED**
- **Status**: Not needed for demo phase
- **Rationale**: Application Insights provides sufficient monitoring
- **Action**: Skip alert configuration, use Application Insights portal for monitoring

#### **10. ~~Azure Key Vault~~ - NOT NEEDED FOR DEMO**
- **Status**: Removed for demo phase simplicity
- **Rationale**: Secrets stored in environment variables (local.settings.json for local, App Settings for Azure)
- **Security Trade-off**: Acceptable for sandbox/demo environment
- **Production Note**: Re-introduce Key Vault before production deployment

---

## Infrastructure as Code (IaC)

### **Technology Choice: Azure Bicep** (Simplified for Demo)

**Create Minimal Bicep Template:**

1. **`main.bicep`** - Single file with all resources
   - Parameters: location, appName, sqlAdminPassword, openaiApiKey
   - All resources defined inline (no modules for simplicity)
   - Outputs: Connection strings, endpoints, resource names
   
2. **Resources Included**:
   - Azure Functions (Consumption plan with Python 3.11)
   - Azure SQL Database (Basic tier, single database `spec2cloud`)
   - Azure Blob Storage (StorageV2, 2 containers)
   - Azure OpenAI (GPT-4o-mini deployment)
   - Application Insights
   - Azure Static Web Apps

3. **Simplified Configuration**:
   - No Key Vault
   - No Cosmos DB
   - No Azure Monitor alerts
   - No Managed Identity (API keys/connection strings)
   - No VNet/Private endpoints
   - Open firewall rules for demo access
   - Connection strings as outputs for easy copy-paste

**Alternative: Azure Portal Manual Setup**
- For fastest demo setup, provision resources manually via Azure Portal
- Document resource names and connection strings in a `config.txt` file
- Use Bicep template as optional automation once architecture stabilizes

---

## Deployment Approach (Demo Phase)

### **Recommended: Manual Azure Portal Setup**

**Step-by-Step:**
1. Create Resource Group: `rg-spec2cloud-demo`
2. Create Azure SQL Database (Basic tier)
3. Create Storage Account (StorageV2)
4. Create Azure OpenAI resource with GPT-4o-mini deployment
5. Create Function App (Consumption, Python 3.11)
6. Create Application Insights
7. Create Static Web App (linked to GitHub repo)
8. Document all connection strings in `config.txt`
9. Add connection strings to Function App settings in Azure Portal

**Benefits:**
- Fastest setup (no Bicep learning curve)
- Easy to troubleshoot
- Good for iterating on architecture
- Can migrate to IaC later

### **Alternative: Azure CLI Script** (Optional)

**File**: `infra/deploy-demo.sh`

```bash
#!/bin/bash
set -e

RESOURCE_GROUP="rg-spec2cloud-demo"
LOCATION="eastus"
APP_NAME="spec2cloud-demo"
SQL_ADMIN_PASSWORD="<YourSecurePassword>"

echo "Creating resource group..."
az group create --name $RESOURCE_GROUP --location $LOCATION

echo "Creating SQL Server and Database..."
az sql server create --name "${APP_NAME}-sql" --resource-group $RESOURCE_GROUP \
  --location $LOCATION --admin-user sqladmin --admin-password $SQL_ADMIN_PASSWORD
  
az sql db create --name spec2cloud --resource-group $RESOURCE_GROUP \
  --server "${APP_NAME}-sql" --service-objective Basic
  
az sql server firewall-rule create --resource-group $RESOURCE_GROUP \
  --server "${APP_NAME}-sql" --name AllowAzureServices \
  --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

echo "Creating Storage Account..."
az storage account create --name "${APP_NAME//-/}storage" \
  --resource-group $RESOURCE_GROUP --location $LOCATION --sku Standard_LRS
  
az storage container create --name uploads \
  --account-name "${APP_NAME//-/}storage"
az storage container create --name outputs \
  --account-name "${APP_NAME//-/}storage"

echo "Creating Azure OpenAI..."
az cognitiveservices account create --name "${APP_NAME}-openai" \
  --resource-group $RESOURCE_GROUP --kind OpenAI --sku S0 --location $LOCATION
  
# Deploy GPT-4o-mini model (manual step required via portal)

echo "Creating Application Insights..."
az monitor app-insights component create --app "${APP_NAME}-insights" \
  --location $LOCATION --resource-group $RESOURCE_GROUP --kind web

echo "Creating Function App..."
az functionapp create --name "${APP_NAME}-func" \
  --resource-group $RESOURCE_GROUP \
  --storage-account "${APP_NAME//-/}storage" \
  --consumption-plan-location $LOCATION \
  --runtime python --runtime-version 3.11 --functions-version 4

echo "===== SETUP COMPLETE ====="
echo "Next steps:"
echo "1. Get SQL connection string:"
echo "   az sql db show-connection-string --client ado.net --server ${APP_NAME}-sql --name spec2cloud"
echo "2. Get Storage connection string:"
echo "   az storage account show-connection-string --name ${APP_NAME//-/}storage"
echo "3. Get OpenAI API key from Azure Portal"
echo "4. Add these to Function App settings or local.settings.json"
```

### **GitHub Actions** (Deferred)

**File**: `.github/workflows/deploy-infrastructure.yml`

```yaml
name: Deploy Infrastructure

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Deploy Bicep
        run: |
          az deployment group create \
            --resource-group rg-spec2cloud-${{ inputs.environment }} \
            --template-file infra/main.bicep \
            --parameters @infra/parameters.${{ inputs.environment }}.json
      
      - name: Store Connection Strings in Key Vault
        run: |
          # Script to extract outputs and store in Key Vault
          # Example: SQL connection string, Blob Storage key, etc.
```

---

## Database Schema Initialization

### **Azure SQL Database Schema Scripts**

**File**: `infra/sql/001_create_properties_table.sql`

```sql
CREATE TABLE properties (
    property_id         UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    canonical_name      NVARCHAR(255) NOT NULL UNIQUE,
    description         NVARCHAR(1000),
    standard_unit       NVARCHAR(50) NOT NULL,
    data_type           NVARCHAR(50) NOT NULL CHECK (data_type IN ('float', 'integer', 'string', 'boolean', 'datetime')),
    test_types          NVARCHAR(500),
    albert_template_ref NVARCHAR(255),
    is_required         BIT DEFAULT 0,
    validation_rules    NVARCHAR(MAX),
    created_date        DATETIME2 DEFAULT GETUTCDATE(),
    last_modified       DATETIME2 DEFAULT GETUTCDATE(),
    created_by          NVARCHAR(255),
    is_active           BIT DEFAULT 1,
    usage_count         INT DEFAULT 0
);

CREATE INDEX idx_canonical_name ON properties(canonical_name);
CREATE INDEX idx_test_types ON properties(test_types);
CREATE INDEX idx_unit ON properties(standard_unit);
```

**File**: `infra/sql/002_create_audit_logs_table.sql`

```sql
CREATE TABLE audit_logs (
    log_id       UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    timestamp    DATETIME2 DEFAULT GETUTCDATE(),
    event_type   NVARCHAR(100) NOT NULL,
    user_id      NVARCHAR(255),
    details      NVARCHAR(MAX),  -- JSON
    metadata     NVARCHAR(MAX),  -- JSON
    session_id   NVARCHAR(255)
);

CREATE INDEX idx_timestamp ON audit_logs(timestamp DESC);
CREATE INDEX idx_event_type ON audit_logs(event_type);
CREATE INDEX idx_user_id ON audit_logs(user_id);
```

**File**: `infra/sql/003_create_mapping_history_table.sql`

```sql
CREATE TABLE mapping_history (
    mapping_id      UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    raw_header      NVARCHAR(255) NOT NULL,
    raw_unit        NVARCHAR(50),
    target_field    NVARCHAR(255) NOT NULL,
    target_unit     NVARCHAR(50),
    confidence      FLOAT,
    approved_by     NVARCHAR(255),
    timestamp       DATETIME2 DEFAULT GETUTCDATE(),
    test_type       NVARCHAR(100),
    machine_type    NVARCHAR(100)
);

CREATE INDEX idx_raw_header ON mapping_history(raw_header);
CREATE INDEX idx_target_field ON mapping_history(target_field);
```

### **Deployment Script**

**File**: `infra/scripts/init-database.sh`

```bash
#!/bin/bash
# Initialize Azure SQL databases with schemas

ENVIRONMENT=$1
SQL_SERVER="sql-spec2cloud-${ENVIRONMENT}.database.windows.net"
SQL_ADMIN=$2
SQL_PASSWORD=$3

# Apply schema scripts
for sql_file in infra/sql/*.sql; do
    echo "Applying $sql_file..."
    sqlcmd -S $SQL_SERVER -U $SQL_ADMIN -P $SQL_PASSWORD \
           -d property_registry -i $sql_file
done

echo "Database initialization complete"
```

---

## Monitoring Configuration (Simplified for Demo)

### **Application Insights Custom Events**

Define custom events to track:
- `AIPropertyMatching` - AI suggestion generation
- `UserMappingDecision` - User accepts/rejects mapping
- `SchemaGenerated` - New schema created
- `PropertyCreated` - New property added to registry
- `ParserGenerated` - New parser created

**Note**: No alerts configured for demo phase. Use Application Insights portal for manual monitoring.

3. **Blob Storage Capacity Alert**
   - Condition: Storage > 90% of quota
   - Action: Email DevOps team
   - Severity: Warning

---

## Security Configuration (Relaxed for Demo)

### **Authentication Approach**
- **Demo Phase**: Connection strings and API keys in environment variables
- **Storage**: Local `local.settings.json` (local dev), Azure Functions App Settings (cloud)
- **No Key Vault**: Secrets in configuration files (acceptable for sandbox)
- **No Managed Identity**: Direct authentication with keys/passwords
- **SQL Authentication**: SQL username and password (not Azure AD)

### **Network Security (Open for Development)**
- **Azure SQL**: Firewall open to Azure services + developer IPs
- **Blob Storage**: No private endpoints, CORS enabled for development
- **Azure Functions**: Public endpoints (no VNet, no IP restrictions)
- **Static Web Apps**: Public access (no authentication required for demo)

### **Future Production Hardening**
- Phase 2: Introduce Key Vault for secret management
- Phase 3: Enable Managed Identity for service-to-service auth
- Phase 4: Restrict network access with VNet/Private endpoints
- Phase 5: Enable Azure AD authentication for users

---

## Acceptance Criteria

### **Infrastructure Provisioning**
- [ ] All Azure resources created successfully (Portal or CLI)
- [ ] Resource group `rg-spec2cloud-demo` contains 6 resources:
  - Azure Functions (Consumption plan, Python 3.11)
  - Azure SQL Database (Basic tier, database `spec2cloud`)
  - Azure Blob Storage (2 containers: `uploads`, `outputs`)
  - Azure OpenAI (GPT-4o-mini deployment)
  - Application Insights
  - Azure Static Web Apps
- [ ] Azure Functions app is running and accessible
- [ ] Azure SQL database created with schema applied (3 tables)
- [ ] Blob Storage containers created
- [ ] Application Insights collecting telemetry
- [ ] Azure Static Web Apps deployed

### **Configuration Management**
- [ ] All connection strings documented in `config.txt` or `local.settings.json`
- [ ] Azure Functions App Settings configured with all required values
- [ ] OpenAI API key accessible from environment variables
- [ ] SQL connection string includes username/password

### **Database Initialization**
- [ ] All SQL schema scripts execute without errors
- [ ] Tables created with correct indexes
- [ ] Foreign key constraints (if any) are valid
- [ ] Sample data can be inserted successfully

### **Security**
- [ ] Managed Identity enabled for Azure Functions
- [ ] Key Vault contains all secrets
- [ ] Azure SQL firewall restricts public access
- [ ] Blob Storage containers use private access
- [ ] No secrets committed to Git repository

### **Monitoring**
- [ ] Application Insights receiving telemetry
- [ ] Custom events defined and tracked
- [ ] Alerts configured and tested
- [ ] Dashboard displays key metrics
- [ ] Log retention policies configured

### **Documentation**
- [ ] README in `infra/` directory explains deployment process
- [ ] Parameter files documented with descriptions
- [ ] Connection string retrieval process documented
- [ ] Troubleshooting guide for common deployment issues

---

## Testing Requirements (Demo Phase)

### **Infrastructure Tests**
- [ ] Deploy to Azure successfully (Portal or CLI)
- [ ] Verify all 6 resources accessible
- [ ] Test connection strings work from local development environment
- [ ] Validate Azure Functions can connect to SQL with username/password
- [ ] Test firewall rules (Azure services + your IP allowed)
- [ ] Verify OpenAI API key works for GPT-4o-mini calls

### **Database Tests**
- [ ] Insert sample property record using SQL connection string
- [ ] Query property by canonical name
- [ ] Validate constraints (unique canonical_name, valid data_type)

### **Monitoring Tests**
- [ ] Trigger custom event, verify appears in Application Insights portal
- [ ] Query audit logs from Application Insights

---

## Out of Scope (Demo Phase)

- **Key Vault**: Secrets in environment variables
- **Managed Identity**: Direct API key/connection string auth
- **Cosmos DB**: Ontology feature deferred to future phase
- **Azure Monitor Alerts**: Use Application Insights portal manually
- **CI/CD Pipeline**: Manual deployment for demo
- **Multi-region deployment**: Single region only
- **Private endpoints**: Public endpoints with basic firewall rules
- **VNet integration**: Not needed for demo

---

## Implementation Phases

### **Current: Demo/Sandbox Phase**
- **Goal**: Quick setup, easy iteration, minimal complexity
- **Approach**: Manual Azure Portal OR simple CLI script
- **Security**: Relaxed (connection strings, API keys, open firewalls)
- **Resources**: 6 core services only
- **Timeline**: 1-2 days for setup
- **Cost**: ~$50-100/month for demo usage

### **Future: Production Phase**
- Introduce Key Vault and Managed Identity
- Add network restrictions and private endpoints
- Implement CI/CD with GitHub Actions or Azure DevOps
- Enable monitoring alerts and auto-scaling
- Add Cosmos DB if ontology feature needed
- Harden security and compliance

---

## Estimated Effort

**Demo Phase**: **1-2 days**
- 0.5 days: Manual Azure Portal resource creation (or 2 hours with CLI script)
- 0.5 days: SQL schema scripts and initial setup
- 0.5 days: Configuration and connection string documentation
- 0.5 days: Testing and validation

**Production Migration** (Future): **2-3 days**
- Convert to Bicep IaC
- Implement Key Vault and Managed Identity
- Add network security and private endpoints
- Set up CI/CD pipeline

---

## Success Metrics

- Infrastructure deployment completes in <1 hour (Portal) or <15 minutes (CLI)
- All connection strings documented and working
- Local development environment connects to Azure resources successfully
- Application Insights receiving telemetry
- Zero P0 blockers for backend development to begin

---

**Document Status**: Updated v2.0 (Demo/Sandbox Mode)  
**Last Updated**: November 24, 2025  
**Owner**: Developer (spec2cloud workflow)  
**Dependencies**: None (foundational task)
**Note**: Simplified for rapid demo/sandbox development
