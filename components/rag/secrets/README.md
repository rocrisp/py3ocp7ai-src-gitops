# RAG Secrets with Vault and External Secrets Operator

This directory contains External Secrets configuration for managing ALL RAG application secrets securely.

## Overview

- **HashiCorp Vault**: Stores all secrets securely
- **External Secrets Operator**: Syncs secrets from Vault to Kubernetes
- **ClusterSecretStore**: Single cluster-wide Vault connection
- **ExternalSecrets**: 7 separate secrets for different components

## Quick Start

### 1. Deploy ClusterSecretStore (once, cluster-wide)

```bash
oc apply -f clustersecretstore.yaml
```

### 2. Deploy All ExternalSecrets

The `externalsecret.yaml` file contains all 7 ExternalSecret resources.

```bash
# Development
oc apply -f externalsecret.yaml

# Stage  
sed 's/namespace: tssc-app-development/namespace: tssc-app-stage/g' \
  externalsecret.yaml | oc apply -f -

# Production
sed 's/namespace: tssc-app-development/namespace: tssc-app-prod/g' \
  externalsecret.yaml | oc apply -f -
```

### 3. Verify Secrets

```bash
# Check ExternalSecrets status
oc get externalsecrets -n tssc-app-development

# Verify all Kubernetes secrets were created
oc get secrets -n tssc-app-development | grep -E "huggingface|llama|tavily|pgvector|minio|pipeline"
```

## Secrets Managed

| Secret Name                     | Vault Path             | Keys                                   | Used By               |
| ------------------------------- | ---------------------- | -------------------------------------- | --------------------- |
| `huggingface-secret`            | `secret/rag`           | HF_TOKEN                               | vLLM ServingRuntime   |
| `llama-stack-env`               | `secret/rag`           | TAVILY_SEARCH_API_KEY                  | llamastack deployment |
| `tavily-secret`                 | `secret/rag`           | TAVILY_API_KEY                         | MCP servers           |
| `pgvector`                      | `secret/rag/pgvector`  | user, password, host, port, dbname     | llamastack & pgvector |
| `minio`                         | `secret/rag/minio`     | user, password, host, port             | minio & upload jobs   |
| `rag-pipeline-secrets`          | `secret/rag/pipeline`  | MINIO_*, LLAMASTACK_*, DS_PIPELINE_URL | Kubeflow pipelines    |
| `rag-ingestion-pipeline-secret` | `secret/rag/ingestion` | SOURCE, EMBEDDING_MODEL, S3 config     | Ingestion pipeline    |

## Vault Configuration

### Authentication
- **Vault Role**: `rag`
- **Policy**: `rag-policy` (read access to `secret/rag/*`)
- **TTL**: Infinite (never expires - ttl=0, max_ttl=0)
- **Namespaces**: `*` (any namespace)
- **Method**: Token auth via `vault-token` secret

### Managing Secrets in Vault

```bash
# View all RAG secrets
oc exec -n vault vault-0 -- vault kv list secret/

# View main secrets
oc exec -n vault vault-0 -- vault kv get secret/rag

# View pgvector credentials
oc exec -n vault vault-0 -- vault kv get secret/rag/pgvector

# View minio credentials
oc exec -n vault vault-0 -- vault kv get secret/rag/minio

# View pipeline config
oc exec -n vault vault-0 -- vault kv get secret/rag/pipeline

# View ingestion config
oc exec -n vault vault-0 -- vault kv get secret/rag/ingestion

# Update a secret
oc exec -n vault vault-0 -- vault kv patch secret/rag hf_token="new_token"
```

## Security Features

- ✅ **All secrets in Vault** - Never stored in Git
- ✅ **Never expire** - Vault auth ttl=0, ExternalSecret refreshInterval=0
- ✅ **Standardized consumption** - All use `secretKeyRef` (explicit key mapping)
- ✅ **Cluster-wide access** - ClusterSecretStore works in any namespace
- ✅ **Automatic sync** - External Secrets Operator manages lifecycle

## Architecture

```
Vault (secret/rag/*)
  ↓
ClusterSecretStore (vault-backend)
  ↓
ExternalSecrets (7 resources)
  ↓
Kubernetes Secrets (7 secrets in namespace)
  ↓
Workloads (Deployments, StatefulSets, Jobs)
```

See `../py3ocp7ai-svc/README.md` for detailed workload → secret mapping.

