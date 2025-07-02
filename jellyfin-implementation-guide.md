# Jellyfin Implementation Guide - Official Templates

## Better Alternatives to Custom Setup

Instead of the custom Kubernetes manifests I initially created, here are the **official and community-maintained options** that are better maintained and feature-rich:

## Option 1: TrueCharts Jellyfin (RECOMMENDED)

**Best choice** for production deployments with advanced features.

### Features:
- ✅ Official TrueCharts maintenance
- ✅ GPU transcoding support
- ✅ Multiple storage backends
- ✅ Security hardening
- ✅ Backup & restore capabilities
- ✅ Ingress & LoadBalancer support
- ✅ Comprehensive documentation

### Implementation:

#### Using Helm Directly:
```bash
# Add TrueCharts repository
helm repo add truecharts https://charts.truecharts.org
helm repo update

# Create namespace
kubectl create namespace jellyfin

# Install with custom values
helm install jellyfin truecharts/jellyfin \
  --namespace jellyfin \
  --set service.main.type=LoadBalancer \
  --set persistence.config.enabled=true \
  --set persistence.config.size=10Gi \
  --set persistence.config.storageClass=longhorn \
  --set persistence.media.enabled=true \
  --set persistence.media.size=500Gi \
  --set persistence.media.storageClass=longhorn
```

#### Using ArgoCD (Recommended for your GitOps setup):

**Create `jellyfin.yaml`:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: jellyfin
  namespace: argocd
spec:
  destination:
    name: in-cluster
    namespace: jellyfin
  project: default
  source:
    chart: jellyfin
    repoURL: https://charts.truecharts.org
    targetRevision: "*"
    helm:
      values: |
        # Service Configuration
        service:
          main:
            type: LoadBalancer
            annotations:
              metallb.universe.tf/address-pool: address-pool
              external-dns.alpha.kubernetes.io/hostname: jellyfin.local
            ports:
              http:
                port: 8096

        # Storage Configuration
        persistence:
          config:
            enabled: true
            size: 10Gi
            storageClass: longhorn
            accessMode: ReadWriteOnce
          cache:
            enabled: true
            size: 20Gi
            storageClass: longhorn
            accessMode: ReadWriteOnce
          media:
            enabled: true
            size: 500Gi
            storageClass: longhorn
            accessMode: ReadWriteMany

        # Resource Configuration
        resources:
          limits:
            cpu: 2000m
            memory: 4096Mi
          requests:
            cpu: 1000m
            memory: 2048Mi

        # GPU Support (uncomment if available)
        # devicePlugins:
        #   - name: nvidia.com/gpu
        #     count: 1

        # Security Context
        securityContext:
          runAsNonRoot: false
          runAsUser: 1000
          runAsGroup: 1000

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

**Add to your kustomization.yaml:**
```yaml
resources:
  - jellyfin.yaml
```

### Advanced Configuration:

**Create custom values file** `jellyfin-values.yaml`:
```yaml
image:
  repository: jellyfin/jellyfin
  tag: 10.9.11

# Enable hardware transcoding
jellyfin:
  env:
    JELLYFIN_PublishedServerUrl: "https://jellyfin.local"
  
# GPU acceleration (Intel QuickSync)
devicePlugins:
  intel.com/gpu: 1

# Multiple storage mounts
persistence:
  config:
    enabled: true
    size: 10Gi
    storageClass: longhorn
  cache:
    enabled: true
    size: 20Gi
    storageClass: longhorn
  media:
    enabled: true
    size: 500Gi
    storageClass: longhorn
  media-movies:
    enabled: true
    mountPath: /media/movies
    size: 200Gi
    storageClass: longhorn
  media-tv:
    enabled: true
    mountPath: /media/tv
    size: 300Gi
    storageClass: longhorn

# Ingress configuration
ingress:
  main:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: "nginx"
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
    hosts:
      - host: jellyfin.local
        paths:
          - path: /
            pathType: Prefix
    tls:
      - secretName: jellyfin-tls
        hosts:
          - jellyfin.local
```

## Option 2: Official Jellyfin Helm Chart

**Repository**: `https://github.com/jellyfin/jellyfin-helm`

### Implementation:
```bash
# Clone the official chart
git clone https://github.com/jellyfin/jellyfin-helm.git
cd jellyfin-helm

# Create custom values
cat > values-custom.yaml << EOF
image:
  repository: jellyfin/jellyfin
  tag: 10.9.11

service:
  type: LoadBalancer
  annotations:
    metallb.universe.tf/address-pool: address-pool
    external-dns.alpha.kubernetes.io/hostname: jellyfin.local

persistence:
  config:
    enabled: true
    size: 10Gi
    storageClass: longhorn
  media:
    enabled: true
    size: 500Gi
    storageClass: longhorn
    accessMode: ReadWriteMany

resources:
  limits:
    cpu: 2000m
    memory: 4096Mi
  requests:
    cpu: 1000m
    memory: 2048Mi
EOF

# Install
helm install jellyfin . -f values-custom.yaml --namespace jellyfin --create-namespace
```

## Option 3: Simple Kubernetes Manifests

**Repository**: `https://github.com/yegli/JellyfinK8s`

Good for **simple deployments** without Helm:

```bash
git clone https://github.com/yegli/JellyfinK8s.git
cd JellyfinK8s/jellyfin_server

# Edit the manifests to match your environment
# Update storage classes, image versions, service types

kubectl apply -f . --namespace jellyfin
```

## Comparison Table

| Feature | TrueCharts | Official Helm | Simple Manifests | Custom (Original) |
|---------|------------|---------------|------------------|-------------------|
| Maintenance | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐ |
| GPU Support | ✅ Built-in | ⚙️ Manual | ❌ | ⚙️ Manual |
| Security | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| Documentation | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐ |
| Features | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| Learning Curve | ⭐⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐ |

## Migration from Custom Setup

Since I removed the custom files I created, to implement Jellyfin:

1. **Choose your preferred option** (TrueCharts recommended)
2. **Create the ArgoCD application** using one of the templates above
3. **Add to kustomization.yaml**
4. **Commit and push** to trigger ArgoCD deployment
5. **Access via** `http://jellyfin.local:8096`

## Benefits of Using Official Templates

### TrueCharts Advantages:
- **Professional maintenance** with regular security updates
- **Hardware transcoding** ready out-of-the-box
- **Multiple storage backends** (NFS, SMB, etc.)
- **Backup integration** with popular backup tools
- **Security scanning** and hardening
- **GPU support** for Intel/NVIDIA/AMD
- **Advanced networking** with multiple ingress options

### Official Helm Chart Advantages:
- **Direct Jellyfin support** from the project
- **Simpler configuration** for basic setups
- **Standard Helm practices**
- **Regular updates** aligned with Jellyfin releases

## Recommended Implementation

For your infrastructure, I recommend the **TrueCharts approach with ArgoCD** because:

1. **Fits your GitOps pattern** perfectly
2. **Professional maintenance** and security
3. **Advanced features** you might need later
4. **Better integration** with your existing stack (Longhorn, MetalLB)
5. **Comprehensive documentation** and community support

## Next Steps

1. Choose your preferred template option
2. Create the appropriate configuration files
3. Add to your GitOps repository
4. Deploy and enjoy a professionally maintained Jellyfin setup!

The official templates provide much better long-term maintainability and features than custom manifests.