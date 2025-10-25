# RAG Application - GitOps Configuration

## Overview

This directory contains Kustomize-based GitOps configuration for the RAG (Retrieval-Augmented Generation) application.

## Structure

- **base/** - Base Kubernetes manifests (43+ resources)
- **overlays/** - Environment-specific customizations (dev/stage/prod)
- **edit/** - Helper scripts for image and replica management

## TAD GitOps Annotations

The deployment patches use **TAD (Trusted Application Pipeline) GitOps** annotations for automated image updates:

### What TAD GitOps Annotations Do:

#### `tad.gitops.set/image`
- Tells CI/CD pipeline **WHERE to update** the image in the manifest
- Uses JSONPath to locate the image field
- Example: `".spec.template.spec.containers[0].image"`

#### `tad.gitops.get/image`
- Tells the pipeline **WHERE to read** the current image from
- Usually the same path as `set`

#### `tad.gitops.set/replicas` & `tad.gitops.get/replicas`
- Same concept but for replica counts
- Allows automated scaling changes

### How It Works in CI/CD:

```
1. CI builds new image → quay.io/org4rong/my-app:abc123
2. CI reads annotation tad.gitops.set/image
3. Gets JSONPath: .spec.template.spec.containers[0].image
4. CI updates the YAML at that path with new image tag
5. CI commits to GitOps repo
6. ArgoCD detects change and syncs new image
```

### Benefits:
- ✅ Automated image updates from CI pipelines
- ✅ No manual editing of manifests
- ✅ Audit trail in git commits
- ✅ Works with GitOps workflow
- ✅ Part of Red Hat Trusted Secure Supply Chain (TSSC)

## Supported Day-2 Operations

The following operations are supported via TAD or manual updates:
- `set/get image` - Updates the container image
- `set/get replicas` - Updates replica count

## Example Usage

```bash
# Add component
tad add-component rag http 

# Update replicas
tad set rag replicas 3     

# Update image
tad set rag image quay.io/org4rong/rag-ui:v1.2.3
```

## Secret Management

All secrets are managed by **HashiCorp Vault** and synchronized using **External Secrets Operator**.

### Workload → Secret Mapping

| Workload Type  | Workload Name                    | Secret Name              | Keys Used                          | How Consumed           | Vault Path            |
| -------------- | -------------------------------- | ------------------------ | ---------------------------------- | ---------------------- | --------------------- |
| Deployment     | `rag` (UI)                       | _None_                   | -                                  | -                      | -                     |
| Deployment     | `rag-ingestion-pipeline`         | _None_                   | -                                  | -                      | -                     |
| Deployment     | `llamastack`                     | `rag-pgvector`           | user, password, host, port, dbname | `env` → `secretKeyRef` | `secret/rag/pgvector` |
| Deployment     | `llamastack`                     | `rag-llama-stack-env`    | TAVILY_SEARCH_API_KEY              | `env` → `secretKeyRef` | `secret/rag`          |
| Deployment     | `toolhive-operator`              | _None_                   | -                                  | -                      | -                     |
| StatefulSet    | `pgvector`                       | `rag-pgvector`           | user, password, port, dbname       | `env` → `secretKeyRef` | `secret/rag/pgvector` |
| StatefulSet    | `minio`                          | `rag-minio`              | user, password                     | `env` → `secretKeyRef` | `secret/rag/minio`    |
| Job            | `upload-sample-docs-job`         | `rag-minio`              | user, password                     | `env` → `secretKeyRef` | `secret/rag/minio`    |
| Job            | `add-default-ingestion-pipeline` | _None_                   | -                                  | -                      | -                     |
| ServingRuntime | `vllm-serving-runtime-gpu`       | `rag-huggingface-secret` | HF_TOKEN                           | `env` → `secretKeyRef` | `secret/rag`          |

### Secret Consumption Method

**All secrets use explicit `secretKeyRef` for security and clarity:**

```yaml
env:
  - name: POSTGRES_USER
    valueFrom:
      secretKeyRef:
        name: rag-pgvector
        key: user
  - name: TAVILY_SEARCH_API_KEY
    valueFrom:
      secretKeyRef:
        name: rag-llama-stack-env
        key: TAVILY_SEARCH_API_KEY
```

**Benefits of this approach:**
- ✅ Explicit - Clear what each env var contains
- ✅ Secure - Only loads the keys you actually need
- ✅ Auditable - Easy to see what secrets are used where
- ✅ Standard - Industry best practice for Kubernetes

### All Secrets and Their Sources

| Secret Name                     | Keys                                   | Vault Path                             | ExternalSecret                  | Managed By       |
| ------------------------------- | -------------------------------------- | -------------------------------------- | ------------------------------- | ---------------- |
| `rag-huggingface-secret`        | HF_TOKEN                               | `secret/rag` → `hf_token`              | `external-hftoken`              | External Secrets |
| `rag-llama-stack-env`           | TAVILY_SEARCH_API_KEY                  | `secret/rag` → `TAVILY_SEARCH_API_KEY` | `llama-stack-env-from-vault`    | External Secrets |
| `rag-tavily-secret`             | TAVILY_API_KEY                         | `secret/rag` → `TAVILY_API_KEY`        | `tavily-secret-from-vault`      | External Secrets |
| `rag-pgvector`                  | user, password, host, port, dbname     | `secret/rag/pgvector`                  | `pgvector-from-vault`           | External Secrets |
| `rag-minio`                     | user, password, host, port             | `secret/rag/minio`                     | `minio-from-vault`              | External Secrets |
| `rag-pipeline-secrets`          | MINIO_*, LLAMASTACK_*, DS_PIPELINE_URL | `secret/rag/pipeline`                  | `pipeline-secrets-from-vault`   | External Secrets |
| `rag-ingestion-pipeline-secret` | SOURCE, EMBEDDING_MODEL, S3 config     | `secret/rag/ingestion`                 | `ingestion-pipeline-from-vault` | External Secrets |

### Security Features

- ✅ **All secrets stored in Vault** - Never in Git
- ✅ **Never expire** - Vault auth ttl=0, ExternalSecret refreshInterval=0
- ✅ **Automatic sync** - External Secrets Operator manages lifecycle
- ✅ **Namespace flexible** - ClusterSecretStore works in any namespace

### Accessing Secrets in Vault

```bash
# View all secret paths
oc exec -n vault vault-0 -- vault kv list secret/

# View main RAG secrets
oc exec -n vault vault-0 -- vault kv get secret/rag

# View pgvector credentials
oc exec -n vault vault-0 -- vault kv get secret/rag/pgvector

# View minio credentials
oc exec -n vault vault-0 -- vault kv get secret/rag/minio

# View pipeline configuration
oc exec -n vault vault-0 -- vault kv get secret/rag/pipeline

# View ingestion configuration
oc exec -n vault vault-0 -- vault kv get secret/rag/ingestion
```

## Deployment

See parent directory README for full deployment instructions.
