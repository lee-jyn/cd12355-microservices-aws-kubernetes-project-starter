# Coworking Space Service Extension

A Flask analytics API (`analytics/`) that reports daily check-ins and per-user visits, backed by PostgreSQL and deployed to Amazon EKS.

## Build and release flow

- Every push to `main` triggers an AWS CodeBuild project (via a GitHub webhook) that runs `buildspec.yaml`, builds the image from `analytics/Dockerfile`, and pushes it to the Amazon ECR repository `coworking`.
- Images use semantic versioning: `1.0.<CODEBUILD_BUILD_NUMBER>`, so the patch number increases automatically with each build; bump `IMAGE_VERSION_PREFIX` in `buildspec.yaml` for a new major or minor release.
- Kubernetes manifests live in `deployments/`: the ConfigMap holds plaintext settings, the Secret holds `DB_PASSWORD`, and PostgreSQL runs as its own Deployment, Service and PersistentVolumeClaim.

## Releasing a new version

1. Push the code change and wait for the CodeBuild run to finish; the new tag appears in ECR.
2. Set that tag in the `image` field of `deployments/coworking.yaml` and run `kubectl apply -f deployments/coworking.yaml`.
3. Check the rollout with `kubectl rollout status deployment/coworking`; Kubernetes replaces the pod only after the new one passes its readiness probe, and `kubectl rollout undo deployment/coworking` reverts it.

## Operations

- The pod exposes `/health_check` (liveness) and `/readiness_check` (database connectivity) on port 5153, and it logs its daily-usage report every 30 seconds.
- Logs are shipped to CloudWatch Container Insights, in the log group `/aws/containerinsights/my-cluster/application`.
- Database credentials are sample values; replace them with real secrets before any production use.
