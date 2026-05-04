### End-to-end Microservice Project from security scanning through CI/CD(GitHub Action) to monitoring on Grafana.

[** Application Code Repository**](https://github.com/dev126712/microservices-app): All 5 microservice CI Workflows has SCA SAST scan
5 microservice plus Redis(caching) and Postgress Database.
#### Project Overview:
- Frontend HTML/Nginx
- Order Service ( Python/Flask )
- Product Service ( Node.js/Express )
- Notification Service ( Golang )
- Storage ( Postgress )
- Caching ( Redis )
- CI Pipeline
- SCA Scanning ( Trivy )
- SAST ( Semgrep )
- Docker
- Docker Compose

### Micorservice Application Deployment CD
- Infra-management Deployment: https://github.com/dev126712/micro-service-infra-management
  - Google Cloud Platform
  - GKE cluster
  - Kubernetes
  - Terraform
  - Security pipeline
  - Trivy ( Cluster DAST )
  - HashircorpVault ( Secrets )
  - VictoriaMetrics ( Monitoring )
    
- Application Charts Deployment: https://github.com/dev126712/microservice-charts-deployment
  - Helm Chart
  - Argocd Application
