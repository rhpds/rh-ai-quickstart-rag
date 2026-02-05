# RAG Quickstart - Field Sourced Content Deployment

Deploy RAG (Retrieval Augmented Generation) chatbot with LiteMaaS integration via RHDP Field Sourced Content catalog.

## Quick Start

1. Order **Field Sourced Content - OpenShift** from RHDP catalog
2. Fill in the form:
   ```
   GitOps Repo: https://github.com/rhpds/rh-ai-quickstart-rag
   GitOps Revision: main
   GitOps Path: deploy/helm

   ☑ Enable LiteMaaS/MaaS API Keys
     ├─ Model: granite-3-2-8b-instruct (or your choice)
     └─ Duration: 7d
   ```
3. Wait for deployment (~15-20 minutes)
4. Access RAG UI from the provided URL

## What Gets Deployed

- **PostgreSQL pgvector** - Vector database for embeddings
- **Llama Stack Server** - LLM orchestration layer
- **RAG UI** - Chat interface
- **Ingestion Pipelines** - Document loading (HR, legal, sales, procurement, tech support)
- **Jupyter Notebook** - For pipeline development

## Architecture

```
Field Content Workload
  ↓ (reads LiteMaaS credentials)
  ↓
This Helm Chart (deploy/helm)
  ↓ (creates ArgoCD Application)
  ↓
RAG Helm Chart (from https://rh-ai-quickstart.github.io/ai-architecture-charts)
  ↓
RAG Stack Deployment
```

## Configuration

### LiteMaaS Integration

When you enable LiteMaaS in the catalog order:
- API key automatically provisioned
- Credentials injected into `values.yaml`
- RAG stack configured to use the LLM

**values.yaml structure:**
```yaml
litemaas:
  enabled: true  # Auto-set by field content workload
  apiUrl: "https://..."  # Injected from agnosticd_user_data
  apiKey: "sk-xxxxx"     # Injected from agnosticd_user_data
  model: "granite-3-2-8b-instruct"
```

### Vector Database Agents

Enabled by default:
- **HR Agent** - Employee benefits, policies
- **Legal Agent** - Contracts, compliance
- **Sales Agent** - Sales processes, objections
- **Procurement Agent** - Purchase requests, vendors
- **Tech Support Agent** - Product documentation

### Websearch Agent

**Disabled** - Requires Tavily API key (not provided by default)

## Customization

### Change Ingestion Sources

Edit `values.yaml`:
```yaml
rag:
  sampleData:
    sources:
      - hr
      - legal
      # Remove others if not needed
```

### Use Different LLM Model

When ordering catalog, select from:
- granite-3-2-8b-instruct
- llama-scout-17b
- qwen3-14b
- deepseek-r1-distill-qwen-14b

### Database Configuration

Edit `values.yaml`:
```yaml
database:
  user: "postgres"
  password: "rag_password"  # Change for production
  dbname: "rag_blueprint"
```

## Testing Locally

```bash
# Lint the chart
helm lint .

# Template with mock values
helm template rag-test . \
  --set deployer.domain=apps.example.com \
  --set litemaas.enabled=true \
  --set litemaas.apiUrl=https://test.example.com \
  --set litemaas.apiKey=sk-test
```

## Troubleshooting

### No LLM Model Available
- Verify LiteMaaS checkbox was enabled in catalog order
- Check if field content workload ran successfully
- Look for `litemaas.enabled: true` in ArgoCD Application

### Ingestion Pipelines Not Running
- Check if RHOAI is installed (pipelines require it)
- Verify namespace has proper service accounts
- Check pipeline run status in RHOAI console

### RAG UI Not Accessible
- Verify Route was created: `oc get route rag -n rag`
- Check pod status: `oc get pods -n rag`
- Review logs: `oc logs -n rag -l app=rag-ui`

## Support

- **Issues**: https://github.com/rhpds/rh-ai-quickstart-rag/issues
- **Slack**: #forum-rhdp
- **Documentation**: https://github.com/rh-ai-quickstart/RAG
