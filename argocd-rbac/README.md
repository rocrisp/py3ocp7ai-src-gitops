# ArgoCD RBAC for MCPServer CRDs

## Overview

This directory contains RBAC resources that grant ArgoCD Application Controller permission to manage MCPServer custom resources.

## Why This is Needed

ArgoCD's Application Controller (`tssc-gitops-argocd-application-controller`) needs explicit permission to deploy custom resources like MCPServers. Without this, ArgoCD sync will fail with:

```
mcpservers.toolhive.stacklok.dev is forbidden: 
User "system:serviceaccount:tssc-gitops:tssc-gitops-argocd-application-controller" 
cannot create resource "mcpservers" in API group "toolhive.stacklok.dev"
```

## Installation

**These must be applied MANUALLY before ArgoCD can sync applications with MCPServers:**

```bash
# Apply from repository root
oc apply -f argocd-rbac/
```

Or individually:

```bash
oc apply -f argocd-rbac/clusterrole-argocd-mcpserver.yaml
oc apply -f argocd-rbac/clusterrolebinding-argocd-mcpserver.yaml
```

## What's Included

### `clusterrole-argocd-mcpserver.yaml`
- Grants permissions to manage MCPServer CRDs
- Allows: create, read, update, delete, list, watch

### `clusterrolebinding-argocd-mcpserver.yaml`
- Binds the ClusterRole to ArgoCD Application Controller ServiceAccount
- ServiceAccount: `tssc-gitops-argocd-application-controller`
- Namespace: `tssc-gitops`

## Lifecycle

- **Applied**: Manually, one time
- **Managed**: NOT by ArgoCD (prevents circular dependency)
- **Scope**: Cluster-wide
- **When to update**: If ArgoCD ServiceAccount name changes

## Alternative Approach

If you prefer GitOps-managed RBAC, you can:
1. Move these files to `components/py3ocp7ai-svc/base/`
2. Apply manually the first time
3. Let ArgoCD manage them subsequently

However, keeping infrastructure RBAC separate from application RBAC is cleaner.

