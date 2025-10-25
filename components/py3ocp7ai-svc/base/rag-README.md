# RAG Base Manifests - Individual Resource Files

This directory contains individual Kubernetes manifest files, with **one resource per file**. This structure makes it easy to apply, modify, or debug specific resources independently.

## 📋 Overview

**Total Resources**: 42 individual YAML files  
**Source**: Generated from `../all-manifests.yaml`  
**Naming Convention**: `{kind}-{name}.yaml`

## 📁 Resource Categories

### 🔐 **Security & RBAC (8 files)**
| File                                                                | Resource Type      | Purpose                       |
| ------------------------------------------------------------------- | ------------------ | ----------------------------- |
| `clusterrole-toolhive-operator-manager-role.yaml`                   | ClusterRole        | Toolhive operator permissions |
| `clusterrolebinding-tssc-app-development-toolhive-operator-anyuid.yaml` | ClusterRoleBinding | SCC binding for toolhive      |
| `clusterrolebinding-toolhive-operator-manager-rolebinding.yaml`     | ClusterRoleBinding | Operator cluster permissions  |
| `role-secret-writer.yaml`                                           | Role               | Secret management permissions |
| `role-toolhive-operator-leader-election-role.yaml`                  | Role               | Leader election permissions   |
| `rolebinding-secret-writer-binding.yaml`                            | RoleBinding        | Secret writer binding         |
| `rolebinding-toolhive-operator-leader-election-rolebinding.yaml`    | RoleBinding        | Leader election binding       |
| `serviceaccount-rag-ingestion-pipeline.yaml`       | ServiceAccount     | Ingestion pipeline SA         |
| `serviceaccount-toolhive-operator.yaml`                             | ServiceAccount     | Toolhive operator SA          |

### 🔑 **Secrets (6 files)**
| File                                        | Purpose                          |
| ------------------------------------------- | -------------------------------- |
| `secret-huggingface-secret.yaml`            | Hugging Face API token           |
| `secret-llama-stack-env.yaml`               | LlamaStack environment variables |
| `secret-minio.yaml`                         | MinIO credentials                |
| `secret-pgvector.yaml`                      | PostgreSQL/PGVector credentials  |
| `secret-rag-ingestion-pipeline-secret.yaml` | Pipeline secrets                 |
| `secret-rag-pipeline-secrets.yaml`          | General pipeline configuration   |

### ⚙️ **Configuration (3 files)**
| File                                 | Purpose                     |
| ------------------------------------ | --------------------------- |
| `configmap-pgvector-initsql.yaml`    | Database initialization SQL |
| `configmap-run-config.yaml`          | LlamaStack configuration    |
| `configmap-vllm-chat-templates.yaml` | vLLM chat templates         |

### 🚀 **Deployments (4 files)**
| File                                                      | Component               |
| --------------------------------------------------------- | ----------------------- |
| `deployment-rag.yaml`                    | Main RAG UI (Streamlit) |
| `deployment-rag-ingestion-pipeline.yaml` | Data ingestion service  |
| `deployment-llamastack.yaml`                              | AI orchestration server |
| `deployment-toolhive-operator.yaml`                       | MCP server operator     |

### 📊 **StatefulSets (2 files)**
| File                        | Component       |
| --------------------------- | --------------- |
| `statefulset-minio.yaml`    | Object storage  |
| `statefulset-pgvector.yaml` | Vector database |

### 🌐 **Services (5 files)**
| File                                                   | Purpose            |
| ------------------------------------------------------ | ------------------ |
| `service-rag.yaml`                    | RAG UI service     |
| `service-rag-ingestion-pipeline.yaml` | Ingestion service  |
| `service-llamastack.yaml`                              | LlamaStack service |
| `service-minio.yaml`                                   | MinIO service      |
| `service-pgvector.yaml`                                | PGVector service   |

### 🌍 **Routes (3 files)**
| File                              | External Access    |
| --------------------------------- | ------------------ |
| `route-rag.yaml` | Main web interface |
| `route-minio-api.yaml`            | MinIO S3 API       |
| `route-minio-webui.yaml`          | MinIO web console  |

### 💾 **Storage (2 files)**
| File                                          | Purpose                       |
| --------------------------------------------- | ----------------------------- |
| `persistentvolumeclaim-llama-stack-data.yaml` | LlamaStack persistent storage |
| `persistentvolumeclaim-pipeline-vol.yaml`     | Pipeline workspace storage    |

### 🔧 **Jobs (2 files)**
| File                                      | Purpose                       |
| ----------------------------------------- | ----------------------------- |
| `job-add-default-ingestion-pipeline.yaml` | Initialize ingestion pipeline |
| `job-upload-sample-docs-job.yaml`         | Upload sample documents       |

### 🤖 **AI/ML Resources (3 files)**
| File                                           | Purpose                        |
| ---------------------------------------------- | ------------------------------ |
| `inferenceservice-llama-3-2-3b-instruct.yaml`  | KServe model serving           |
| `servingruntime-vllm-serving-runtime-gpu.yaml` | vLLM runtime configuration     |
| `notebook-rag-pipeline-notebook.yaml`          | Jupyter notebook for pipelines |

### 🛠️ **MCP Servers (2 files)**
| File                          | Purpose                  |
| ----------------------------- | ------------------------ |
| `mcpserver-weather.yaml`      | Weather tool integration |
| `mcpserver-oracle-sqlcl.yaml` | Oracle SQL client tool   |

### 📊 **Pipeline Resources (1 file)**
| File                                        | Purpose                        |
| ------------------------------------------- | ------------------------------ |
| `datasciencepipelinesapplication-dspa.yaml` | Kubeflow pipelines application |

## 🚀 Deployment Strategies

### **Strategy 1: Deploy by Category**

```bash
# 1. Security and RBAC first
oc apply -f clusterrole-*.yaml
oc apply -f clusterrolebinding-*.yaml
oc apply -f role-*.yaml
oc apply -f rolebinding-*.yaml
oc apply -f serviceaccount-*.yaml

# 2. Secrets and Configuration
oc apply -f secret-*.yaml -n tssc-app-development
oc apply -f configmap-*.yaml -n tssc-app-development

# 3. Storage
oc apply -f persistentvolumeclaim-*.yaml -n tssc-app-development

# 4. Core Services
oc apply -f statefulset-*.yaml -n tssc-app-development
oc apply -f deployment-*.yaml -n tssc-app-development
oc apply -f service-*.yaml -n tssc-app-development

# 5. External Access
oc apply -f route-*.yaml -n tssc-app-development

# 6. AI/ML and Tools
oc apply -f servingruntime-*.yaml -n tssc-app-development
oc apply -f inferenceservice-*.yaml -n tssc-app-development
oc apply -f mcpserver-*.yaml -n tssc-app-development

# 7. Jobs and Pipelines
oc apply -f job-*.yaml -n tssc-app-development
oc apply -f datasciencepipelinesapplication-*.yaml -n tssc-app-development
oc apply -f notebook-*.yaml -n tssc-app-development
```

### **Strategy 2: Deploy Specific Components**

```bash
# Deploy only core RAG functionality
oc apply -f secret-pgvector.yaml -n tssc-app-development
oc apply -f secret-minio.yaml -n tssc-app-development
oc apply -f configmap-run-config.yaml -n tssc-app-development
oc apply -f statefulset-pgvector.yaml -n tssc-app-development
oc apply -f statefulset-minio.yaml -n tssc-app-development
oc apply -f deployment-llamastack.yaml -n tssc-app-development
oc apply -f deployment-rag.yaml -n tssc-app-development
oc apply -f service-*.yaml -n tssc-app-development
oc apply -f route-rag.yaml -n tssc-app-development
```

### **Strategy 3: Deploy Everything**

```bash
# Apply all resources (cluster-level first)
oc apply -f clusterrole-*.yaml
oc apply -f clusterrolebinding-*.yaml

# Apply namespace-scoped resources
oc apply -f . -n tssc-app-development --recursive=false
```

## 🔍 Debugging Individual Resources

### **Check Specific Resource Status**
```bash
# Check a deployment
oc describe -f deployment-llamastack.yaml -n tssc-app-development

# Check a service
oc get -f service-llamastack.yaml -n tssc-app-development -o wide

# Check logs for a specific deployment
oc logs -f deployment/llamastack -n tssc-app-development
```

### **Update Individual Resources**
```bash
# Update a specific deployment
oc apply -f deployment-llamastack.yaml -n tssc-app-development

# Delete and recreate a job
oc delete -f job-upload-sample-docs-job.yaml -n tssc-app-development
oc apply -f job-upload-sample-docs-job.yaml -n tssc-app-development
```

### **Validate Resources Before Apply**
```bash
# Dry run to check for errors
oc apply -f deployment-llamastack.yaml -n tssc-app-development --dry-run=client

# Validate YAML syntax
oc apply -f . --dry-run=client --validate=true
```

## 🧹 Selective Cleanup

### **Remove by Category**
```bash
# Remove jobs only
oc delete -f job-*.yaml -n tssc-app-development

# Remove routes (external access)
oc delete -f route-*.yaml -n tssc-app-development

# Remove AI/ML resources
oc delete -f inferenceservice-*.yaml -n tssc-app-development
oc delete -f servingruntime-*.yaml -n tssc-app-development
```

### **Remove Specific Components**
```bash
# Remove MCP servers
oc delete -f mcpserver-*.yaml -n tssc-app-development

# Remove ingestion pipeline
oc delete -f deployment-rag-ingestion-pipeline.yaml -n tssc-app-development
oc delete -f service-rag-ingestion-pipeline.yaml -n tssc-app-development
```

## 📝 File Naming Convention

**Format**: `{kind}-{name}.yaml`

**Examples**:
- `deployment-llamastack.yaml` → Deployment named "llamastack"
- `secret-pgvector.yaml` → Secret named "pgvector"  
- `route-rag.yaml` → Route named "rag"

## 🔧 Customization

To modify a specific resource:

1. **Edit the individual file**
2. **Apply the changes**: `oc apply -f {filename} -n tssc-app-development`
3. **Verify the update**: `oc get {resource-type} {resource-name} -n tssc-app-development`

## 📚 Related Files

- **`../all-manifests.yaml`**: Combined version of all these files
- **`../README.md`**: Main documentation with architecture overview
- **`../../deploy/helm/rag/values.yaml`**: Source Helm values

---

**Generated from**: `../all-manifests.yaml`  
**Total Individual Resources**: 42 files  
**Namespace**: `tssc-app-development`
