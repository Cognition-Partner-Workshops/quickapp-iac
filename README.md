# App-Specific Infrastructure as Code

Helm charts and Kubernetes manifests for the decomposed .NET + Angular application. This repo contains **only** app-specific resources — shared platform infrastructure (EKS cluster, networking, ingress, monitoring) lives in [`platform-engineering-shared-services`](https://github.com/Cognition-Partner-Workshops/platform-engineering-shared-services).

## Architecture

```
EKS Cluster (managed by platform-engineering-shared-services)
├── ingress-nginx (shared)
├── cert-manager (shared)
├── monitoring (shared)
│
└── app namespace: decomposition-app
    ├── api-gateway          ← this repo
    ├── identity-service     ← this repo
    ├── customer-service     ← this repo
    ├── order-service        ← this repo
    ├── product-service      ← this repo
    ├── notification-service ← this repo
    ├── shell-mfe            ← this repo
    ├── identity-mfe         ← this repo
    ├── customer-mfe         ← this repo
    ├── order-mfe            ← this repo
    ├── product-mfe          ← this repo
    ├── postgresql            ← this repo (per-service DBs)
    └── rabbitmq              ← this repo (message broker)
```

## Helm Charts

| Chart | Description |
|-------|-------------|
| `api-gateway` | YARP-based API gateway — routes to backend microservices |
| `identity-service` | Auth and user management microservice |
| `customer-service` | Customer CRUD microservice |
| `order-service` | Order management microservice |
| `product-service` | Product catalog microservice |
| `notification-service` | Email/in-app notification microservice |
| `shell-mfe` | Angular shell host (Module Federation) |
| `identity-mfe` | Angular identity remote MFE |
| `customer-mfe` | Angular customer remote MFE |
| `order-mfe` | Angular order remote MFE |
| `product-mfe` | Angular product remote MFE |

## Environments

| Environment | Namespace | Description |
|-------------|-----------|-------------|
| `dev` | `decomposition-dev` | Development — lower resource limits, debug logging |
| `staging` | `decomposition-staging` | Pre-production validation |
| `prod` | `decomposition-prod` | Production deployment |

## Usage

```bash
# Deploy to dev
helm upgrade --install api-gateway charts/api-gateway \
  -f environments/dev/values.yaml \
  -n decomposition-dev --create-namespace

# Deploy all services to dev
for chart in charts/*/; do
  name=$(basename "$chart")
  helm upgrade --install "$name" "$chart" \
    -f "environments/dev/values.yaml" \
    -n decomposition-dev --create-namespace
done
```

## Prerequisites

- EKS cluster provisioned by [`platform-engineering-shared-services`](https://github.com/Cognition-Partner-Workshops/platform-engineering-shared-services)
- `kubectl` configured for the target cluster
- Helm 3.x installed
- Container images pushed to ECR (build from the microservices and microfrontends repos)

## Related Repositories

| Repo | Purpose |
|------|---------|
| [`app_dotnet_angular_containerized_decomposition_monolith`](https://github.com/Cognition-Partner-Workshops/app_dotnet_angular_containerized_decomposition_monolith) | Before-state monolith |
| [`app_dotnet_angular_containerized_decomposition_microservices`](https://github.com/Cognition-Partner-Workshops/app_dotnet_angular_containerized_decomposition_microservices) | .NET microservices source |
| [`app_dotnet_angular_containerized_decomposition_microfrontends`](https://github.com/Cognition-Partner-Workshops/app_dotnet_angular_containerized_decomposition_microfrontends) | Angular micro-frontends source |
| [`platform-engineering-shared-services`](https://github.com/Cognition-Partner-Workshops/platform-engineering-shared-services) | Shared EKS + platform infra |
