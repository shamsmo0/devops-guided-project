# VM Deployment

This project deploys to one Ubuntu VM with Docker Compose.

The VM deployment is intentionally image-based only:

1. GitHub Actions validates the change in a pull request
2. merge to `main` publishes the app image to ACR
3. the deploy workflow waits for `production` environment approval
4. the VM pulls the selected image tag
5. the VM never rebuilds the application

## Runtime Configuration Model

Keep non-secret VM settings in `.env`.

Keep secret values in `.env.secrets`, or let GitHub Actions create that file automatically from Azure Key Vault data during deployment.

Use these files:

- `.env` from `deploy/example.env`
- `.env.secrets` from `deploy/example.secrets.env` only when you are not using Azure Key Vault or GitHub Secrets fallback

In the trainee-facing version, `deploy/example.env` intentionally contains guided gap `VM-01`.
Students must replace the placeholder `APP_IMAGE` value before expecting VM deployment to succeed.

`deploy/deploy.sh` uses `.env` for the stable runtime settings and `.env.secrets` for sensitive values such as:

- `POSTGRES_PASSWORD`
- `GRAFANA_ADMIN_PASSWORD`
- `REGISTRY_LOGIN_SERVER`
- `REGISTRY_USERNAME`
- `REGISTRY_PASSWORD`

## Preferred Secret Source: Azure Key Vault

For the guided project, keep the VM connection values in GitHub Secrets:

- `VM_HOST`
- `VM_USER`
- `VM_SSH_KEY_B64` preferred
- `VM_SSH_KEY` optional fallback
- `VM_APP_DIR`

`VM_SSH_KEY_B64` should contain the base64-encoded PEM file content with line breaks removed.

Use Azure Key Vault for the runtime secrets by configuring these GitHub Secrets for Azure login:

- `AZURE_CLIENT_ID`
- `AZURE_TENANT_ID`
- `AZURE_SUBSCRIPTION_ID`
- `AZURE_KEYVAULT_NAME`

The workflow logs into Azure with OIDC, reads the required secrets from Azure Key Vault, then passes them into `deploy/deploy.sh`.

Expected Azure Key Vault secret names:

- `POSTGRES_PASSWORD`
- `GRAFANA_ADMIN_PASSWORD`
- `REGISTRY_LOGIN_SERVER`
- `REGISTRY_USERNAME`
- `REGISTRY_PASSWORD`

## GitHub Secrets Fallback

For immediate validation, GitHub Actions can still supply the runtime secrets directly without Azure Key Vault.

Store these GitHub Secrets:

- `POSTGRES_PASSWORD`
- `GRAFANA_ADMIN_PASSWORD`
- `REGISTRY_LOGIN_SERVER`
- `REGISTRY_USERNAME`
- `REGISTRY_PASSWORD`

The deploy workflow passes them to `deploy/deploy.sh`, which writes `.env.secrets` on the VM automatically during deployment.

The VM deployment workflow is `.github/workflows/deploy-production.yml`.

- automatic path: runs after `Publish Image` succeeds on `main`
- manual path: `workflow_dispatch` for rollback, recovery, or instructor demos

## Public and Private Services

Public by default:

- app through Nginx on port `80`

Private by default:

- Grafana
- Prometheus
- Loki
- Promtail
- PostgreSQL
- Redis

## Guided Validation

After the VM deployment completes, run:

```bash
bash scripts/validate-vm-deployment.sh http://YOUR_VM_PUBLIC_IP
```

That validation confirms:

- `/health`
- `/ready`
- `/version` deployment metadata

The `/version` response should match the published image metadata so students can trace the running VM back to a commit and image tag.

In the trainee-facing version, run this only after `VM-01` is complete.
Before that, the deployment path is expected to stop on the guided placeholder rather than pull an image successfully.

If you used the same Linux machine earlier for the local training stack, stop that stack before the VM deployment path so ports `80`, `3000`, and `9090` are free for the VM runtime layout:

```bash
docker compose down -v
```

If you are cloning this repository directly onto the VM, make sure the VM has access to the repository first. A private repository will require either GitHub authentication on the VM or a source copy step from a workstation.

If you need the source-copy path, create a clean archive from your workstation first so macOS metadata files do not break Linux services such as Grafana provisioning:

```bash
bash scripts/package-vm-source.sh
```

If you need to redeploy a previously known good image tag, use:

```bash
bash deploy/rollback.sh sha-<known-good-sha-tag>
```

## Next Step

After deployment is working, review [Troubleshooting](11-troubleshooting.md), then move to [LAB-08 Failure and Recovery](../labs/LAB-08-failure-and-recovery.md).

## SSH Tunnel for Grafana

Use:

```bash
ssh -L 3000:localhost:3000 USER@VM_PUBLIC_IP
```

Then open:

```text
http://localhost:3000
```

## Important Reminder

This is a controlled training VM.

It is not a hardened production platform.

Do not expose Grafana, Prometheus, Loki, or Promtail publicly unless you are doing that intentionally for a supervised class demo.
