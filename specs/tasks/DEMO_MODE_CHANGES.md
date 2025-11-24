# Demo/Sandbox Mode Simplifications

**Date**: November 24, 2025  
**Purpose**: Simplify infrastructure and development approach for rapid prototyping and demo purposes  
**Status**: Updated Tasks 001, 002, and TASK_SUMMARY

---

## Overview

The task breakdown has been updated to support a **demo/sandbox development mode** that prioritizes:
- ✅ **Speed**: Faster setup and iteration
- ✅ **Simplicity**: Minimal complexity and learning curve
- ✅ **Ease of Development**: Direct configuration without abstraction layers
- ✅ **Cost Efficiency**: Fewer resources, cheaper tiers

**Trade-offs Accepted**:
- ⚠️ Lower security (acceptable for sandbox/demo)
- ⚠️ Manual configuration (no IaC initially)
- ⚠️ Single AI backend (Azure OpenAI only)
- ⚠️ No enterprise features (alerting, VNet, geo-redundancy)

---

## Key Changes

### **Task 001: Infrastructure Scaffolding**

#### **REMOVED** (Demo Phase):
- ❌ **Azure Key Vault**: Secrets now in environment variables (`local.settings.json` or App Settings)
- ❌ **Managed Identity**: Using connection strings and API keys instead
- ❌ **Azure Cosmos DB**: Ontology feature deferred to future phase
- ❌ **Azure Monitor Alerts**: Using Application Insights portal for manual monitoring
- ❌ **Complex Bicep IaC**: Simplified to single-file template OR manual Portal setup
- ❌ **Private Endpoints**: Public endpoints with basic firewall rules
- ❌ **Multiple SQL Databases**: Consolidated to single database `spec2cloud`
- ❌ **Blob Lifecycle Policies**: Simple Hot tier with 2 containers

#### **SIMPLIFIED** (Demo Phase):
- ✅ **Azure SQL**: Basic tier with SQL username/password auth
- ✅ **Blob Storage**: Connection string-based access, 2 containers (`uploads`, `outputs`)
- ✅ **Azure OpenAI**: Single deployment (GPT-4o-mini) with API key authentication
- ✅ **Firewall Rules**: Open to Azure services + developer IPs
- ✅ **Deployment**: Manual Azure Portal OR simple CLI script (no CI/CD yet)

#### **Resource Count**:
- Original: 10 resources (including Key Vault, Cosmos DB, Azure Monitor)
- Demo Mode: **6 core resources** (Functions, SQL, Blob, OpenAI, Static Web Apps, App Insights)

#### **Effort Reduction**:
- Original: 4-5 days
- **Demo Mode: 1-2 days** (3 days saved)

---

### **Task 002: Backend Scaffolding**

#### **REMOVED** (Demo Phase):
- ❌ **Microsoft Foundry Adapter**: Azure OpenAI only for demo
- ❌ **Key Vault Client**: No `azure-keyvault-secrets` dependency
- ❌ **Managed Identity Auth**: No `azure-identity` for database/OpenAI
- ❌ **Complex Config Class**: Simplified to environment variable lookup only

#### **SIMPLIFIED** (Demo Phase):
- ✅ **DatabaseConnection**: Simple `pyodbc.connect()` with connection string (no Managed Identity token logic)
- ✅ **Config**: Direct `os.getenv()` lookups, no Key Vault fallback
- ✅ **AI Service**: Single `AzureOpenAIService` with API key (no adapter pattern for demo)
- ✅ **Connection Strings**: Full credentials included (e.g., `Uid=sqladmin;Pwd=...`)

#### **Dependencies Removed**:
```python
# No longer needed for demo:
# azure-identity
# azure-keyvault-secrets
# azure-ai-projects (Foundry SDK)
```

#### **Configuration Example** (`local.settings.json`):
```json
{
  "AZURE_OPENAI_ENDPOINT": "https://your-openai.openai.azure.com",
  "AZURE_OPENAI_API_KEY": "sk-...",
  "SQL_CONNECTION_STRING": "Driver={...};Server=...;Database=spec2cloud;Uid=sqladmin;Pwd=Password123!;",
  "BLOB_STORAGE_CONNECTION_STRING": "DefaultEndpointsProtocol=https;AccountName=...;AccountKey=...;"
}
```

#### **Effort Reduction**:
- Original: 3-4 days
- **Demo Mode: 2-3 days** (1 day saved)

---

### **Task Summary: Timeline Impact**

#### **Phase 1: Foundation**
- Original: 10-13 days
- **Demo Mode: 6-9 days**
- **Realistic with Parallelization: 5-6 days** (vs 7-8 days original)

#### **Total Project Timeline**
- Original: 38-40 days (~8 weeks)
- **Demo Mode: 32-34 days (~7 weeks)**
- **Time Savings: 4-6 days** in foundation phase

#### **Updated Critical Path**
```
001 (1-2 days) → 002 (2-3 days) → 004 (3-5 days) → 005 (5-7 days) → 006 (5-6 days) → 008 (3-4 days) → 009 (4-5 days)
```
**Duration**: 24-32 days (vs 28-36 days original)

---

## Security Considerations

### **What's Less Secure in Demo Mode**:
1. **Secrets in Plain Text**: Environment variables instead of Key Vault
2. **SQL Authentication**: Username/password instead of Managed Identity
3. **Open Firewalls**: Azure services + developer IPs allowed
4. **API Keys**: Stored in configuration files
5. **No Private Endpoints**: Public internet access to resources

### **Why It's Acceptable**:
- ✅ **Sandbox Environment**: Not handling production/sensitive data
- ✅ **Temporary Setup**: Demo and iteration phase only
- ✅ **Clear Documentation**: Security limitations documented
- ✅ **Hardening Plan**: Production phase will re-introduce security features

### **Production Hardening Roadmap**:
When moving to production, re-introduce:
1. **Azure Key Vault** for secret management
2. **Managed Identity** for service-to-service authentication
3. **VNet + Private Endpoints** for network isolation
4. **Azure AD Authentication** for user access
5. **Network Security Groups** and **IP restrictions**
6. **Azure Monitor Alerts** for proactive monitoring
7. **IaC with Bicep** for repeatable, auditable deployments

---

## Migration Path: Demo → Production

### **Step 1: Validate Demo** (Current Phase)
- Build all features with simplified infrastructure
- Validate AI accuracy and user workflows
- Gather stakeholder feedback
- Estimated: 32-34 days

### **Step 2: Production Hardening** (Future Phase)
- Convert to Bicep IaC templates
- Deploy Key Vault and migrate secrets
- Enable Managed Identity for all services
- Add network security (VNet, Private Endpoints)
- Implement CI/CD pipeline (GitHub Actions)
- Set up monitoring alerts
- Estimated: 2-3 days additional effort

### **Step 3: Production Deployment** (Future Phase)
- Deploy to production subscription with hardened configuration
- Enable geo-redundancy and disaster recovery
- Implement Azure AD authentication
- Add compliance and auditing features
- Estimated: 1-2 days additional effort

**Total Production-Ready Timeline**: Demo (32-34 days) + Hardening (2-3 days) + Deployment (1-2 days) = **35-39 days**

---

## Developer Quick Start (Demo Mode)

### **1. Provision Azure Resources** (1-2 hours)
**Option A: Azure Portal** (Recommended for beginners)
- Create Resource Group: `rg-spec2cloud-demo`
- Create resources manually, document connection strings

**Option B: Azure CLI Script**
```bash
bash infra/deploy-demo.sh
```

### **2. Configure Local Development** (30 minutes)
1. Copy `backend/local.settings.json.template` → `backend/local.settings.json`
2. Fill in connection strings from Azure Portal
3. Run `pip install -r requirements.txt`
4. Start Functions: `func start`

### **3. Verify Setup** (15 minutes)
- Test database connection
- Call Azure OpenAI API
- Upload file to Blob Storage
- Check Application Insights for telemetry

**Total Setup Time: 2-3 hours** (vs 1-2 days with production setup)

---

## Cost Comparison

### **Demo Mode** (~$50-100/month):
- Azure Functions: Consumption plan (first 1M executions free)
- Azure SQL: Basic tier (~$5/month)
- Blob Storage: Standard LRS (~$2-5/month)
- Azure OpenAI: GPT-4o-mini (~$20-50/month for demo usage)
- Application Insights: First 5GB free
- Static Web Apps: Free tier

### **Production Mode** (~$300-500/month):
- Add Key Vault (~$5/month)
- Upgrade SQL to Standard S2 (~$150/month)
- Add Cosmos DB (~$25-50/month)
- Add Azure Monitor alerts (~$10/month)
- Enable geo-redundancy for Blob/SQL (~$50-100/month)
- Higher Azure OpenAI usage (~$200-300/month)

**Cost Savings in Demo: 70-80% reduction**

---

## FAQ

### **Q: Can I use this in production?**
**A**: No. Demo mode is for sandbox/development only. Security is intentionally relaxed for ease of use.

### **Q: When should I migrate to production mode?**
**A**: After demo validation, before handling real customer data or launching publicly.

### **Q: Can I switch back to production-ready config?**
**A**: Yes. Tasks 001 and 002 include commented-out sections for Key Vault and Managed Identity. Re-enable those and follow production hardening steps.

### **Q: What if I need Microsoft Foundry now?**
**A**: Uncomment the Foundry adapter code in Task 002. The AI Service Adapter pattern supports both backends.

### **Q: Is CI/CD completely removed?**
**A**: No, just deferred. GitHub Actions workflows are documented in Task 009 but not required for demo phase.

---

## Affected Files

### **Updated**:
- ✅ `specs/tasks/001-task-infrastructure-scaffolding.md` (Demo mode sections)
- ✅ `specs/tasks/002-task-backend-scaffolding.md` (Simplified auth and config)
- ✅ `specs/tasks/TASK_SUMMARY.md` (Updated timelines and effort estimates)

### **Created**:
- ✅ `specs/tasks/DEMO_MODE_CHANGES.md` (This document)

### **Unchanged** (Feature tasks remain the same):
- ➖ `003-task-frontend-scaffolding.md`
- ➖ `004-task-property-registry.md`
- ➖ `005-task-schema-generation.md`
- ➖ `006-task-mapping-assistant.md`
- ➖ `007-task-parser-builder.md`
- ➖ `008-task-data-consistency-audit.md`
- ➖ `009-task-testing-suite.md`

---

## Summary

**Demo mode prioritizes rapid iteration and ease of development** by removing enterprise security features and using direct configuration. This allows the team to:

1. ✅ **Start coding faster** (1-2 days setup vs 4-5 days)
2. ✅ **Iterate quickly** (no IaC updates, direct config changes)
3. ✅ **Reduce costs** (fewer resources, cheaper tiers)
4. ✅ **Focus on features** (less infrastructure complexity)

When ready for production, follow the hardening roadmap to re-introduce Key Vault, Managed Identity, network security, and IaC automation.

---

**Status**: ✅ All changes implemented and documented  
**Ready for**: Demo/sandbox development phase  
**Next Steps**: Begin Task 001 (Infrastructure setup via Azure Portal or CLI script)
