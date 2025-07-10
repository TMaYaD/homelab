# Homelab GitOps Repository

This repository contains the GitOps configuration for a Talos Kubernetes cluster using Argo CD and Sealed Secrets for secure secret management.

## A) Bootstrap - Cluster Setup from Scratch

### Prerequisites
- Talos Linux cluster running
- `kubectl` configured for cluster access
- Domain and DNS configured

### 1. Install Sealed Secrets and Restore Private Key  (if available)
If you have a backed up private key from password manager:

```bash
# Get the private key from password manager and restore it
echo "<private key from password manager>" > temp-private-key.yaml
kubectl apply -f temp-private-key.yaml
rm temp-private-key.yaml
```

**⚠️ Security Note:** If you don't restore the private key before deploying sealed secrets, a new key will be auto-generated and any existing sealed secrets in the repository will become undecryptable.

```bash
kubectl apply -k sealed-secrets/
```

### 2. Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -k argocd/
```

### 3. Deploy Root Application

```bash
kubectl apply -f root.yaml
```

This will automatically deploy:
- Sealed Secrets controller
- All other applications in the repository

### 4. Access Argo CD

```bash
# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Access via the configured LoadBalancer at https://argocd.local with username `admin` and the password from above.

Alternatively, use port forwarding for local access:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## B) Sealed Secrets Management

### B.1) Setup on New Laptop

Install kubeseal CLI:

```bash
# macOS
brew install kubeseal

# Linux
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.5/kubeseal-0.24.5-linux-amd64.tar.gz
tar -xvzf kubeseal-0.24.5-linux-amd64.tar.gz
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

Get the public key from the cluster:

```bash
kubeseal --fetch-cert > public-key.pem
```

**Note:** Once you have the public key file, you can use kubeseal offline without cluster access for encryption.

### B.2) Creating Secrets from Templates

1. **Copy example template:**
   ```bash
   cp path/to/app/secrets.example.yaml path/to/app/secrets.yaml
   # or
   cp path/to/app/github-creds.example.yaml path/to/app/github-creds.yaml
   ```

2. **Edit with real values:**
   ```bash
   vim path/to/app/secrets.yaml  # Add real secret values
   ```

3. **Encrypt and create sealed secret:**

   ```bash
   # Copy and edit example template with real values
   cp home-assistant/secrets.example.yaml home-assistant/secrets.yaml
   # Edit secrets.yaml with your actual secret values

   # Encrypt the secret file
   kubeseal --cert=public-key.pem -f home-assistant/secrets.yaml -w home-assistant/secrets.sealed.yaml
   ```

   **⚠️ Important:** The sealed secret files in this repository contain placeholder encrypted data. They must be replaced with actual encrypted secrets before applications will function correctly.

4. **Commit sealed secret:**
   ```bash
   git add sealed-secrets.yaml
   git commit -m "Add encrypted secrets for app"
   git push
   ```

5. **Clean up plain text:**
   ```bash
   rm secrets.yaml  # Never commit this
   rm public-key.pem
   ```

### B.3) Backup Private Key

**Critical:** Always backup the private key after first deployment:

```bash
# Generate backup
kubectl get secret -n kube-system -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > sealed-secrets-master-key-backup.yaml

# Store the entire backup file content in your password manager
# Label: "Sealed Secrets Private Key - Homelab Cluster"
```

**Important Notes:**
- Without this key, you cannot decrypt any sealed secrets if you rebuild the cluster
- Store the backup securely in your password manager
- Never commit the private key to Git

## Applications

The following applications are deployed:

- Argo CD - GitOps controller
- Sealed Secrets - Secret encryption
- Home Assistant - Home automation
- Jellyfin - Media server
- Longhorn - Storage
- MetalLB - Load balancer
- Pi-hole - DNS filtering
- Caddy - Reverse proxy
- External DNS - DNS management
- Transmission - Downloads
- Samba - File sharing
- MQTT/Mosquitto - IoT messaging

For application-specific configuration, see individual directories.
