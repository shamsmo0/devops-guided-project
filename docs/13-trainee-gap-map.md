# Trainee Gap Map

This repository is the trainee-facing version of the project.

It intentionally includes a small number of guided gaps so students can complete meaningful work instead of only reading a finished implementation.

Use the private full-reference version as the answer key.

## How To Use This File

Treat each gap as a milestone.

The intended pattern is:

1. read the matching document or lab
2. inspect the file listed here
3. fill the gap
4. rerun the matching validation script
5. compare the result with the expected runtime behavior

## Gap Order

### APP-01 Redis Cache Demo

Goal:

- make the GUI `Test Redis Cache` button work
- show a clear cache miss on the first request
- show a clear cache hit on the second request

Files:

- `app/src/server.js`
- `docs/04-app-gui.md`
- `docs/05-request-and-data-flow.md`

What to implement:

- restore the `GET /cache-demo` route
- read from Redis first
- if the key exists, return a cache-hit response
- if the key is missing, generate the payload, store it in Redis, and return it
- log whether the route was a cache hit or cache miss

What good looks like:

- first request returns `source: "app-generated"`
- second request returns `source: "redis-cache"`
- the GUI button stops returning `501`

Validation:

- `bash scripts/validate-local-stack.sh full`
- `bash scripts/validate-gui-requests.sh http://localhost:8080`

### OBS-01 Prometheus App Scrape Target

Goal:

- make Prometheus scrape the app metrics endpoint correctly

Files:

- `monitoring/prometheus/prometheus.yml`
- `docs/07-monitoring.md`

What to implement:

- replace the placeholder app target with the real internal app target and port

What good looks like:

- Prometheus target becomes healthy
- request metrics appear in Prometheus and Grafana

Validation:

- `bash scripts/validate-observability.sh`

### OBS-02 Promtail App Log Path

Goal:

- ship app logs into Loki so they appear in Grafana Explore

Files:

- `monitoring/promtail/promtail-config.yml`
- `docs/06-logging.md`
- `docs/07-monitoring.md`

What to implement:

- replace the placeholder app log file path with the real mounted path

What good looks like:

- app logs appear in Grafana Explore
- request IDs from GUI traffic can be searched in Grafana logs

Validation:

- `bash scripts/validate-observability.sh`

### OBS-03 Grafana Loki Datasource

Goal:

- make Grafana Explore connect to Loki correctly

Files:

- `monitoring/grafana/provisioning/datasources/datasources.yml`
- `docs/06-logging.md`
- `docs/07-monitoring.md`

What to implement:

- replace the placeholder Loki datasource URL with the real internal Loki service URL

What good looks like:

- Grafana Explore can query app and Nginx logs from Loki
- the observability validator stops failing on the Loki datasource placeholder

Validation:

- `bash scripts/validate-observability.sh`

### CICD-01 Publish The Correct App Image

Goal:

- make the publish workflow push the intended app image name

Files:

- `.github/workflows/publish-image.yml`
- `docs/08-registries.md`

What to implement:

- replace the TODO image name with the correct app image path
- keep the same image naming pattern used by deployment

What good looks like:

- `Publish Image` no longer stops at the trainee image-name guard
- image tags are clear and traceable

Validation:

- GitHub Actions `Publish Image`

### VM-01 Point Deployment At The Real Registry Image

Goal:

- make VM deployment pull the real published image

Files:

- `deploy/example.env`
- `deploy/deploy.sh`
- `docs/10-vm-deployment.md`

What to implement:

- replace the placeholder `APP_IMAGE` with the actual registry image path in `.env`
- keep the image path aligned with the publish workflow

What good looks like:

- `deploy/deploy.sh` stops failing on the guided VM image placeholder
- the VM can pull and run the published image

Validation:

- `bash scripts/validate-vm-deployment.sh http://YOUR_VM_PUBLIC_IP`
- `bash scripts/validate-runtime-contract.sh vm http://127.0.0.1`

## Recommended Team Split

- Student 1: APP-01
- Student 2: OBS-01, OBS-02, and OBS-03
- Student 3: CICD-01
- Student 4: VM-01

Then rotate and review each other’s work before the final full-stack validation.

## Final Check

When all guided gaps are complete, rerun:

- `bash scripts/validate-project.sh`
- `bash scripts/validate-doc-journey.sh`
- `bash scripts/validate-local-stack.sh foundation`
- `bash scripts/validate-local-stack.sh full`
- `bash scripts/validate-gui-requests.sh http://localhost:8080`
- `bash scripts/validate-observability.sh`
- `bash scripts/validate-vm-deployment.sh http://YOUR_VM_PUBLIC_IP`
