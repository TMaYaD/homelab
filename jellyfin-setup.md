# Jellyfin Media Server Setup

This document describes the Jellyfin media server setup for your Kubernetes homelab infrastructure.

## Overview

Jellyfin is a free and open-source media server that allows you to organize, manage, and stream your digital media files to any device. This setup integrates Jellyfin into your existing Kubernetes infrastructure using ArgoCD for GitOps deployment.

## Architecture

The Jellyfin deployment consists of:

- **ArgoCD Application**: `jellyfin.yaml` - Manages the GitOps deployment
- **Kubernetes Deployment**: Runs the Jellyfin server container
- **LoadBalancer Service**: Exposes Jellyfin via MetalLB with external DNS
- **Persistent Storage**: Three PVCs for config, cache, and media storage

## Storage Configuration

Three persistent volumes are created:

1. **jellyfin-config** (10Gi): Stores Jellyfin configuration, databases, and metadata
2. **jellyfin-cache** (20Gi): Stores transcoding cache and temporary files  
3. **jellyfin-media** (500Gi): Stores your media files (movies, TV shows, music, etc.)

## Network Access

Jellyfin will be accessible at:
- **Primary URL**: `http://jellyfin.local:8096`
- **HTTP**: `http://jellyfin.local` (port 80)
- **HTTPS**: `https://jellyfin.local` (port 443) 

*Note: HTTPS requires additional SSL/TLS configuration*

## Initial Setup

1. **Deploy the Application**:
   ```bash
   git add .
   git commit -m "Add Jellyfin media server"
   git push
   ```

2. **Wait for Deployment**: ArgoCD will automatically deploy Jellyfin. Monitor the deployment:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=jellyfin
   kubectl get pvc | grep jellyfin
   ```

3. **Access Web Interface**: Open `http://jellyfin.local:8096` in your browser

4. **Initial Configuration**:
   - Create admin user account
   - Set preferred language and metadata settings
   - Configure remote access (if needed)
   - Add media libraries pointing to `/media` directory

## Adding Media

To add media files to Jellyfin:

1. **Copy files to the media volume**: You'll need to copy your media files to the jellyfin-media PVC
2. **Organize by type**: Create subdirectories like:
   - `/media/movies/`
   - `/media/tv-shows/`
   - `/media/music/`
3. **Add libraries**: In Jellyfin web interface, add libraries pointing to these directories

## Resource Allocation

Current resource limits:
- **CPU**: 1-2 cores (1000m-2000m)
- **Memory**: 2-4GB (2048Mi-4096Mi)

These can be adjusted based on:
- Number of concurrent users
- Transcoding requirements
- Available cluster resources

## Hardware Acceleration (Optional)

For better transcoding performance, you can enable GPU acceleration by uncommenting the relevant sections in `deployment.yaml`:

```yaml
securityContext:
  privileged: true
volumeMounts:
- mountPath: /dev/dri
  name: device-dri
```

And add the volume:
```yaml
- name: device-dri
  hostPath:
    path: /dev/dri
```

*Note: This requires compatible GPU hardware and drivers on your nodes*

## Maintenance

### Viewing Logs
```bash
kubectl logs -l app.kubernetes.io/name=jellyfin -f
```

### Scaling (if needed)
The deployment uses `Recreate` strategy and should remain at 1 replica due to database constraints.

### Backup
Important files to backup:
- Jellyfin configuration: `/config` directory in the container
- Media files: `/media` directory

### Updates
Jellyfin updates are managed through the container image tag in `deployment.yaml`. Update the image version and commit to trigger a deployment.

## Troubleshooting

### Common Issues

1. **Pod not starting**: Check resource availability and PVC status
2. **Can't access web interface**: Verify service and ingress configuration
3. **Media not detected**: Check file permissions and directory structure
4. **Transcoding issues**: Check available CPU/GPU resources

### Useful Commands
```bash
# Check pod status
kubectl get pods -l app.kubernetes.io/name=jellyfin

# View pod logs
kubectl logs <jellyfin-pod-name>

# Check service and endpoints
kubectl get svc jellyfin
kubectl get endpoints jellyfin

# Check storage
kubectl get pvc | grep jellyfin
```

## Security Considerations

- Jellyfin is exposed via LoadBalancer - ensure your network security is properly configured
- Consider enabling HTTPS for production use
- Regular backups of configuration and media
- Keep Jellyfin updated to latest stable version

## Integration with Existing Services

This Jellyfin setup integrates with your existing infrastructure:
- **Longhorn**: Provides persistent storage
- **MetalLB**: Provides LoadBalancer service
- **External-DNS**: Manages DNS records
- **ArgoCD**: Handles GitOps deployment

The setup follows the same patterns as your other services for consistency and maintainability.