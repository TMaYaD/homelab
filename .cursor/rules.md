# Cursor Rules for GitOps Homelab

## Project Context
This is a GitOps repository for a Talos Kubernetes homelab using Argo CD. Follow these rules when making changes or suggestions.

## File Naming Conventions

### Sealed Secrets Naming
- Use format: `<secret-name>.sealed.yaml` (e.g., `secrets.sealed.yaml`, `github-creds.sealed.yaml`)
- NOT `sealed-<secret-name>.yaml` or `<secret-name>-sealed.yaml`

### Example Files Naming
- Use format: `<secret-name>.example.yaml` (e.g., `secrets.example.yaml`, `github-creds.example.yaml`)
- Always provide example templates for secrets

### Other Varient files Naming
- similarly use the format `<name>.<varient>.<ext>` where applicable.
- NOT `<name>-<varient>.<ext>` or `<varient>-<name>.<ext>`

### Application Structure
- Application folders: `<app-name>/` (kebab-case)
- Kubernetes resources: `<resource-type>.yaml` (e.g., `deployment.yaml`, `service.yaml`)
- Kustomization: Always `kustomization.yaml` in each app folder

### Resource Naming
- Use kebab-case for all resource names (e.g., `home-assistant`, `sealed-secrets`)
- Follow pattern: `<app-name>` for simple resources, `<app-name>-<component>` for complex ones

## GitOps and Kubernetes Best Practices

### YAML Formatting
- Always include trailing newlines in YAML files
- Use 2-space indentation consistently
- Quote strings that contain special characters or start with numbers
- Use `---` separator for multi-document YAML files
- Add comments for version pinning or complex configurations

### Repository Structure
- Each application gets its own folder with kustomization.yaml
- Use subfolders for external controllers (e.g., `sealed-secrets/`)
- Place .gitignore files in subfolders for specific exclusions
- Keep example templates as `.example.yaml` files alongside actual resources

### Sealed Secrets Management
- Never commit plain text secrets to repository
- Use naming format: `<secret-name>.sealed.yaml`
- Provide `.example.yaml` templates for all secret types
- Plain text secrets MUST be ignored in app folder .gitignore files

## Security Guidelines

### Secret Handling
- All secrets must use Sealed Secrets pattern
- Plain text secrets MUST be ignored in app folder .gitignore files
- Provide clear warnings about placeholder data in documentation
- Include private key backup instructions in README
- Never store encryption keys or passwords in repository

### Documentation Security
- Use `<private key>` and `<pass phrase>` as placeholders in docs
- Include security warnings for critical operations
- Explain consequences of not following security procedures
- Document disaster recovery procedures thoroughly

## Argo CD Application Patterns

### Application Structure
- Applications MUST use kustomize.
- Always prefer upstream/officially maintained kustomize templates and helm charts in that order.
- Only use manifests created from scratch when neither is available upstream.
- Prefer helmCharts functionality of kustomize to the helm charts support in ArgoCD.

### Kustomization Structure
- Group resources logically (deployments, services, secrets)
- Use external resources for controllers (sealed-secrets, etc.)
- Include version comments for external dependencies

## Application Development Rules

### New Application Checklist
1. Create application folder with kustomization.yaml
2. Add Argo CD application manifest in root
3. Update root kustomization.yaml to include new app
4. If secrets needed:
   - Create `<secret-name>.example.yaml` template with placeholder data
   - Generate `<secret-name>.sealed.yaml` from the example template.
   - Add .gitignore for plain text secrets
5. Test deployment and update documentation

### Configuration Management
- Use ConfigMaps for non-sensitive configuration
- Use Sealed Secrets for sensitive data
- Avoid hardcoding cluster-specific values
- Use descriptive comments for complex configurations

## Documentation Standards

### README Structure
- Start with project description and tech stack
- Include bootstrap section for cluster rebuild
- Provide step-by-step sealed secrets management
- List all applications with brief descriptions
- Include troubleshooting section

### Instruction Format
- Use numbered steps for sequential processes
- Include code blocks with proper syntax highlighting
- Provide both automated and manual alternatives
- Add security warnings for critical operations
- Include verification steps after changes

## Code Generation Guidelines

### When Creating YAML
- Follow existing patterns in the repository
- Include all required metadata fields
- Use appropriate namespaces (default for apps, kube-system for infrastructure)
- Add labels for Argo CD or other controllers as needed
- Use correct file naming conventions

### When Modifying Existing Code
- Maintain existing naming conventions
- Preserve comments and formatting
- Update related documentation
- Consider backward compatibility

## Error Prevention

### Common Mistakes to Avoid
- Don't use old naming format `sealed-*.yaml` - use `*.sealed.yaml`
- Don't change secret names without updating all references
- Don't use global gitignore patterns for app specific files
- Don't forget trailing newlines in YAML files

### Validation Checks
- Verify YAML syntax before committing
- Check that all referenced secrets exist
- Ensure gitignore patterns are in correct locations
- Validate Argo CD application references

## Commit Message Format
Use conventional commits with these types:
- `feat:` for new applications or major features
- `fix:` for bug fixes and corrections
- `docs:` for documentation changes
- `refactor:` for code reorganization
- `chore:` for maintenance tasks

Include scope when applicable (e.g., `feat(home-assistant):` or `fix(sealed-secrets):`).

## Testing and Verification

### Before Committing
- Verify all YAML files have trailing newlines
- Check that secret references are consistent
- Ensure documentation reflects actual implementation

### After Deployment
- Verify Argo CD applications sync successfully
- Check that sealed secrets decrypt properly
- Confirm applications start and function correctly
- Validate security controls are in place

## Maintenance Guidelines

### Regular Tasks
- Update sealed-secrets controller version periodically
- Rotate sealed secrets private key as needed
- Review and update documentation
- Clean up unused configurations
- Backup critical encryption keys

### When Adding New Technologies
- Follow established patterns for folder structure
- Create appropriate example files using correct naming
- Update main documentation
- Consider security implications
- Test integration with existing setup
