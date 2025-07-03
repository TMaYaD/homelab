# Jellyfin Media Server Setup

This document describes the Jellyfin media server setup following your project's **exact pattern** of using raw Kubernetes manifests with Kustomize and ArgoCD.

## 🏗️ Architecture

Following your project's consistent pattern:
- **📁 Directory Structure**: `jellyfin/` with individual manifest files
- **📦 Kustomize**: Organizes resources in `jellyfin/kustomization.yaml`
- **🚀 ArgoCD**: Managed via `jellyfin.yaml` Application
- **🔧 Raw Manifests**: No Helm charts, just Kubernetes YAML

## 📂 File Structure

```
jellyfin/
├── deployment.yaml    # Jellyfin deployment with volume mounts
├── service.yaml       # LoadBalancer with MetalLB + external-dns
├── storage.yaml       # PVCs for config, cache, and media
└── kustomization.yaml # Resource organization

jellyfin.yaml          # ArgoCD Application (root level)
```

## 🔧 Components

### 1. **Deployment** (`jellyfin/deployment.yaml`)
- **Image**: `jellyfin/jellyfin:10.9.11`
- **Resources**: 1-2 CPU cores, 2-4GB RAM
- **Strategy**: `Recreate` (database constraints)
- **Health Checks**: HTTP probes on `/health`
- **Environment**: Published server URL configuration

### 2. **Service** (`jellyfin/service.yaml`)
- **Type**: LoadBalancer (MetalLB integration)
- **Annotations**: 
  - `metallb.universe.tf/address-pool: address-pool`
  - `external-dns.alpha.kubernetes.io/hostname: jellyfin.local`
- **Ports**: 80, 443, 8096

### 3. **Storage** (`jellyfin/storage.yaml`)
Three persistent volumes using Longhorn:
- **Config**: 10Gi (ReadWriteOnce) - Settings and databases
- **Cache**: 20Gi (ReadWriteOnce) - Transcoding cache
- **Media**: 500Gi (ReadWriteMany) - Media files

### 4. **Kustomization** (`jellyfin/kustomization.yaml`)
Simple resource list following project pattern:
```yaml
resources:
  - deployment.yaml
  - service.yaml
  - storage.yaml
```

### 5. **ArgoCD Application** (`jellyfin.yaml`)
Follows identical pattern as other services:
- **Source**: Git path `jellyfin/`
- **Destination**: `default` namespace
- **Sync Options**: Same as other services

## 🌐 Network Access

Jellyfin will be accessible at:
- **Primary**: `http://jellyfin.local:8096`
- **HTTP**: `http://jellyfin.local` (port 80)
- **HTTPS**: `https://jellyfin.local` (port 443)
- **Direct**: `http://jellyfin.local:8096`

## 🚀 Deployment

1. **Verify Files**: All files are ready in your repository
2. **Commit Changes**:
   ```bash
   git add .
   git commit -m "Add Jellyfin media server"
   git push
   ```
3. **ArgoCD Sync**: ArgoCD will automatically detect and deploy
4. **Monitor**: 
   ```bash
   kubectl get pods -l app.kubernetes.io/name=jellyfin
   kubectl get pvc | grep jellyfin
   ```

## 🎛️ Initial Configuration

1. **Access**: Open `http://jellyfin.local:8096`
2. **Setup Wizard**:
   - Create admin account
   - Set preferred language
   - Configure remote access
3. **Add Libraries**: Point to `/media` directory in container

## 📊 Storage Layout

| Volume | Path | Size | Purpose |
|--------|------|------|---------|
| jellyfin-config | `/config` | 10Gi | Configuration, databases, metadata |
| jellyfin-cache | `/cache` | 20Gi | Transcoding cache and temp files |
| jellyfin-media | `/media` | 500Gi | Your media collection |

## 🔧 Maintenance

### View Logs
```bash
kubectl logs -l app.kubernetes.io/name=jellyfin -f
```

### Check Storage
```bash
kubectl get pvc | grep jellyfin
kubectl describe pvc jellyfin-media
```

### Scale Resources
Edit `jellyfin/deployment.yaml` and commit to Git.

### Updates
Update image tag in `jellyfin/deployment.yaml` and commit.

## 🔒 Security Considerations

- Jellyfin runs as non-root user
- Network access controlled by LoadBalancer
- Storage isolated to specific PVCs
- Health checks ensure availability

## 🔄 Integration with Existing Stack

This setup integrates perfectly with your existing infrastructure:
- **✅ Longhorn**: Provides all persistent storage
- **✅ MetalLB**: LoadBalancer service type
- **✅ External-DNS**: Automatic DNS management  
- **✅ ArgoCD**: GitOps deployment and management
- **✅ Consistent Pattern**: Follows exact same structure as other services

## 📝 Following Project Patterns

This implementation **exactly matches** your project's patterns:

| Pattern | Example Service | Jellyfin Implementation |
|---------|----------------|------------------------|
| Directory structure | `transmission/` | `jellyfin/` |
| ArgoCD app | `transmission.yaml` | `jellyfin.yaml` |
| Kustomization | `transmission/kustomization.yaml` | `jellyfin/kustomization.yaml` |
| Service type | LoadBalancer + MetalLB | ✅ Same |
| Storage | Longhorn PVCs | ✅ Same |
| Namespace | `default` | ✅ Same |

This ensures consistency with your existing infrastructure and maintenance practices! 🎉