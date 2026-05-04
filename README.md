# 🚀 GitOps-Driven Microservices Platform on GKE

A production-grade Kubernetes platform built on **Google GKE**, using a fully declarative GitOps workflow. Every resource — from cloud infrastructure to application deployments — is version-controlled, automated, and repeatable.

## 🔁 GitOps Flow

This project uses a tree-repo GitOps pattern:
* [**Manifest Repo (Helm values) & ArgoCD Application file**](https://github.com/dev126712/microservice-charts-deployment)

* [**☁️ Infrastructure (Terraform)**](https://github.com/dev126712/micro-service-infra-management)
  
* [**Application code CI/CD Pipelines (App Repo)**](https://github.com/dev126712/microservices-app)
---

## 📐 Architecture Overview

![](https://github.com/dev126712/microservice-end-to-end/blob/9e866204ebdcf85dc37dcc8122f2d5ab583a455f/Untitled%20Diagram.drawio%20(5).png)

---

---

## 🛠️ Stack

| Layer | Tool |
|---|---|
| Cloud | Google Cloud Platform (GCP) |
| Kubernetes | Google Kubernetes Engine (GKE) |
| Infrastructure as Code | Terraform |
| GitOps | ArgoCD |
| CI/CD | GitHub Actions |
| Ingress | GKE Gateway API (HTTPRoute + ReferenceGrant) |
| Package Manager | Helm |
| Observability | VictoriaMetrics, Grafana, Fluent-bit, VMLogs |
| Security Scanning | Trivy Operator (runtime) + Trivy Action (CI) |
| Secrets Management | HashiCorp Vault |
| SAST | Semgrep |

---

## [**☁️ Infrastructure (Terraform)**](https://github.com/dev126712/micro-service-infra-management)

All cloud resources are provisioned with Terraform:

- **Custom VPC** — regional routing, no auto-subnets, CIDR `10.10.0.0/16`
- **GKE Cluster** — Workload Identity enabled, Gateway API `CHANNEL_STANDARD`
- **Node Pool** — `e2-medium`, autoscaling (min 1 → max 2), auto-repair & auto-upgrade
- **Cloud NAT + Cloud Router** — secure outbound internet access from private nodes
- **Firewall Rules** — IAP tunneling, GKE health check ranges, internal-only traffic
- **Helm Releases via Terraform** — ArgoCD, VictoriaMetrics, Vault, and Trivy Operator installed in-cluster declaratively

---



> No `kubectl apply` is ever run manually. A `git push` is the only deployment trigger.

---

## ⚙️ [**CI/CD Pipelines (GitHub Actions)**](https://github.com/dev126712/microservices-app)

Each microservice has an **independent, path-triggered workflow**. A change to `product-service/` only builds the product service.

Every pipeline runs 4 security gates before deploying:

1. **SCA Scan** — Trivy filesystem scan for vulnerable dependencies and misconfigurations
2. **SAST Scan** — Semgrep audit for code logic issues and leaked secrets
3. **Build & Push** — Docker image tagged with both `latest` and the immutable `git SHA`
4. **Post-build Image Scan** — Trivy fails the pipeline on any `CRITICAL` CVE

On success, `yq` patches `image.tag` in the manifest repo's Helm `values.yaml` with the git SHA. ArgoCD auto-syncs.

---

## 🏗️ Microservices

| Service | Namespace | Port | Backing Store |
|---|---|---|---|
| Frontend (Nginx) | `frontend-ns` | 8080 | — |
| Order API | `order-ns` | 5000 | PostgreSQL |
| Product API | `product-ns` | 3000 | PostgreSQL + Redis |
| Notification | `notification-ns` | 5001 | — |
| PostgreSQL 15 | `postgres-ns` | 5432 | PVC (1Gi) |
| Redis 7 | `redis-ns` | 6379 | PVC (1Gi, AOF) |

Each service has its own **Helm chart** containing: `Deployment`, `Service`, `ServiceAccount`, `HorizontalPodAutoscaler`, `HTTPRoute`, `ReferenceGrant`, `NetworkPolicy`, and `ConfigMap`.

---

## 🔐 Zero-Trust Networking

Every namespace starts with a **default-deny NetworkPolicy**. Traffic is only permitted where explicitly declared:

| Source | Destination | Port |
|---|---|---|
| `order-ns` | `postgres-ns` | 5432 |
| `product-ns` | `postgres-ns` | 5432 |
| `order-ns` | `product-ns` | 3000 |
| `order-ns` | `notification-ns` | 5001 |
| `product-ns` | `redis-ns` | 6379 |
| GKE Gateway | `frontend-ns` | 8080 |

Cross-namespace routing uses **Gateway API v1 `HTTPRoute` + `ReferenceGrant`** — more expressive and secure than classic Ingress.

---

## 📊 Observability

Deployed as an ArgoCD Application from the VictoriaMetrics Helm chart:

- **VMSingle** — metrics storage, 14-day retention
- **Grafana** — dashboards exposed at `/grafana` via HTTPRoute
- **Prometheus Node Exporter** — node-level resource metrics
- **VMLogs + Fluent-bit** — log collection and aggregation pipeline

---

## 🛡️ Security

- **Workload Identity** — pods authenticate to GCP APIs without static service account keys
- **HashiCorp Vault** — in-cluster secrets management (Helm, Terraform-managed)
- **Trivy Operator** — continuous vulnerability scanning of live workloads in the cluster
- **Trivy Action** — pre/post-build image scanning in every CI pipeline
- **Semgrep** — static code analysis and secret detection on every push

---

## 🚀 Getting Started

### Prerequisites
- `terraform` >= 1.5
- `kubectl`
- `helm` >= 3.x
- `gcloud` CLI authenticated to your GCP project

### 1. Provision Infrastructure
```bash
cd terraform/
terraform init
terraform plan
terraform apply
```

### 2. Connect to the Cluster
```bash
gcloud container clusters get-credentials my-gke-cluster \
  --region us-central1 \
  --project YOUR_PROJECT_ID
```

### 3. ArgoCD is already installed via Terraform. Sync your apps:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Login and sync ArgoCD Applications from argocd-apps/
```

### 4. CI/CD is triggered automatically on push to `main`
Each service pipeline is path-scoped — only the changed service rebuilds and deploys.

---

## 📌 Notes

- `deletion_protection = false` on the GKE cluster — intentional for demo/cost purposes
- Postgres credentials are currently in a ConfigMap — HashiCorp Vault integration is the planned next step
- Redis has no NetworkPolicy yet — a default-deny + allow-from-product-ns rule is pending

---

## 📄 License

MIT
