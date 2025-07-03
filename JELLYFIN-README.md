# Jellyfin Media Server Setup

This document describes the Jellyfin media server setup following your project's **Helm chart + Kustomize pattern** exactly like Longhorn.

## 🏗️ Architecture

Following your project's **Longhorn pattern**:
- **� Helm Charts**: Using Kustomize's `helmCharts` feature
- **� TrueCharts**: Professional Jellyfin Helm chart
- **📁 Directory Structure**: `jellyfin/` with kustomization + namespace
- **🚀 ArgoCD**: Managed via `jellyfin.yaml` Application
- **🎯 Dedicated Namespace**: `jellyfin` namespace

## 📂 File Structure

```
jellyfin/
├── kustomization.yaml # Helm chart deployment + values
├── namespace.yaml     # Dedicated namespace definition
└── .gitignore        # Ignore Helm chart artifacts

jellyfin.yaml         # ArgoCD Application (root level)
```

**Exactly like Longhorn:**
```
longhorn/
├── kustomization.yaml # ✅ Same pattern
├── namespace.yaml     # ✅ Same pattern
└── .gitignore        # ✅ Same pattern

longhorn.yaml         # ✅ Same pattern
```

## 🔧 Components

### 1. **Helm Chart Deployment** (`jellyfin/kustomization.yaml`)
Following **Longhorn's exact pattern**:
- **Chart**: TrueCharts Jellyfin (`charts.truecharts.org`)
- **Version**: 23.0.10 (latest stable)
- **Namespace**: `jellyfin` (dedicated like `longhorn-system`)
- **Values**: Inline configuration for your infrastructure

### 2. **Namespace** (`jellyfin/namespace.yaml`)
Dedicated namespace following Longhorn pattern:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: jellyfin
  labels:
    kubernetes.io/metadata.name: jellyfin
```

### 3. **ArgoCD Application** (`jellyfin.yaml`)
Identical pattern to Longhorn:
- **Source**: Git path `jellyfin/`
- **Destination**: `jellyfin` namespace
- **Sync Options**: Same as Longhorn

## 🎛️ Configuration

### Helm Values (in `kustomization.yaml`)
```yaml
valuesInline:
  service:
    main:
      type: LoadBalancer  # MetalLB integration
      annotations:
        metallb.universe.tf/address-pool: address-pool
        external-dns.alpha.kubernetes.io/hostname: jellyfin.local

  persistence:
    config: 10Gi    # Configuration & databases
    cache: 20Gi     # Transcoding cache  
    media: 500Gi    # Media library

  resources:
    limits: 2 CPU, 4GB RAM
    requests: 1 CPU, 2GB RAM
```

## 🌐 Network Access

Jellyfin will be accessible at:
- **Primary**: `http://jellyfin.local:8096`
- **Via MetalLB**: Integrated with your LoadBalancer setup
- **External DNS**: Automatic DNS record creation

## 🚀 Deployment

1. **Verify Files**: All files are ready in your repository
2. **Commit Changes**:
   ```bash
   git add .
   git commit -m "Add Jellyfin using Helm chart pattern (like Longhorn)"
   git push
   ```
3. **ArgoCD Sync**: ArgoCD will automatically:
   - Download the TrueCharts Helm chart
   - Apply your custom values
   - Deploy to `jellyfin` namespace
4. **Monitor**: 
   ```bash
   kubectl get pods -n jellyfin
   kubectl get pvc -n jellyfin
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
| config | `/config` | 10Gi | Configuration, databases, metadata |
| cache | `/cache` | 20Gi | Transcoding cache and temp files |
| media | `/media` | 500Gi | Your media collection |

## 🔧 Maintenance

### View Logs
```bash
kubectl logs -n jellyfin -l app.kubernetes.io/name=jellyfin -f
```

### Check Storage
```bash
kubectl get pvc -n jellyfin
kubectl describe pvc -n jellyfin
```

### Update Configuration
Edit `jellyfin/kustomization.yaml` `valuesInline` section and commit to Git.

### Update Chart Version
Edit `version:` in `jellyfin/kustomization.yaml` and commit.

## 🔄 Integration with Existing Stack

This setup integrates perfectly with your existing infrastructure using the **same pattern as Longhorn**:

- **✅ Longhorn**: Provides all persistent storage
- **✅ MetalLB**: LoadBalancer service type
- **✅ External-DNS**: Automatic DNS management  
- **✅ ArgoCD**: GitOps deployment and management
- **✅ Kustomize + Helm**: Same deployment pattern

## 📝 Following Longhorn Pattern

This implementation **exactly matches** your Longhorn pattern:

| Pattern | Longhorn | Jellyfin |
|---------|----------|----------|
| ArgoCD app | `longhorn.yaml` | ✅ `jellyfin.yaml` |
| Directory | `longhorn/` | ✅ `jellyfin/` |
| Kustomization | `helmCharts` + `valuesInline` | ✅ Same |
| Namespace | `longhorn-system` | ✅ `jellyfin` |
| Chart source | `charts.longhorn.io` | ✅ `charts.truecharts.org` |
| Resources | `namespace.yaml` | ✅ Same |

## 🎯 Benefits of Helm Chart Approach

### Over Raw Manifests:
- **🔧 Professional maintenance** - TrueCharts team updates
- **🛡️ Security hardening** - Built-in security best practices  
- **⚡ Advanced features** - GPU transcoding, monitoring, backup
- **📚 Comprehensive options** - Hundreds of configuration options
- **🔄 Easy updates** - Just change version number

### Following Your Pattern:
- **✅ Consistent** - Same as Longhorn deployment method
- **✅ GitOps** - Managed through Git like other services
- **✅ Kustomize** - Uses your existing tooling
- **✅ Namespace isolation** - Dedicated namespace like Longhorn

This ensures the **same operational experience** as your Longhorn deployment! 🎉