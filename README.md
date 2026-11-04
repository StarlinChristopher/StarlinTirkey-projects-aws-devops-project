StarlinTirkey-projects-aws-devops-project

It is an enterprise-grade cloud-native architecture blueprint designed for developing containerized microservices across multiple environments(dev,staging,prod). It decouples application runtime code from underlying cloud infrastructure using infrastructure as code(IaC), GitOps-friendly kubernetes manifests, and automated CI/CD pipelines.

Core Components &amp; Layer Breakdown

1. Infrastructure as Code (terraform/)
Uses reusable modules (`vpc/`, `eks/`, `rds/`) to declare cloud networking (subnets, NAT gateways, route tables), managed Amazon EKS clusters, and Amazon RDS databases.

Isolates environment states across `environments/dev/`, `staging/`, and `prod/` using distinct parameter files (`terraform.tfvars`) and remote S3 state backends with DynamoDB locking.

2. Backend Application (app/)
Houses the backend source code (such as a Java 17/Maven microservice) and application framework configurations.
Packages the service via a multi-stage `Dockerfile` to produce minimal, non-root, and secure container images stripped of build-time dependencies.

3. Kubernetes Declarative Manifests (k8s/)
Manages container workloads across `dev`, `staging`, and `prod` lifecycle environments using Kubernetes manifests (`deployment.yaml`, `service.yaml`, `ingress.yaml`, `configmap.yaml`, `secret.yaml`, and `hpa.yaml`).

Implements declarative base-and-overlay patching (via Kustomize) to prevent configuration drift across environments without duplicating YAML.

Exposes services externally through the AWS Application Load Balancer (ALB) Ingress Controller and routes traffic internally to ClusterIP endpoints.

4. Continuous Integration & Delivery (.github/workflows/ci-cd.yml)
Automates the delivery lifecycle: compiling code, running unit tests, executing static application security scans (SAST/SonarQube), building container images tagged with immutable Git commit SHAs, pushing to Amazon ECR, and orchestrating rolling zero-downtime updates onto Amazon EKS.

5. Operational Automation & Tooling (scripts/)
Provides lifecycle helper scripts (`deploy.sh`, `destroy.sh`) to automate local testing, cluster bootstrapping, and safe teardown of transient resources
