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

## Performance improvement opportunities

These are architecture-level opportunities inferred from the blueprint, not measured bottlenecks. The repository currently contains no application, Terraform, Kubernetes, Docker, or workflow implementation to benchmark or tune; validate each against representative load before applying it.

| # | Opportunity | Potential impact | Effort | How to validate |
|---|---|---|---|---|
| 1 | Cache Maven dependencies and Docker BuildKit layers in CI; keep cache keys tied to dependency lock/config files. | High for repeated build feedback time | Low once the CI workflow exists | Compare cold and warm workflow duration and cache-hit rate. |
| 2 | Set realistic CPU and memory requests for each container, then tune the existing-described HPA targets against load-test data. | High for latency and pod stability | Medium | Track p95 latency, throttling, restarts, and replica count during a load test. |
| 3 | Configure a bounded database connection pool with connection and acquisition timeouts. | High under concurrent traffic | Low–medium | Measure pool wait time, active connections, and request latency at peak load. |
| 4 | Add indexes for frequently filtered or joined RDS columns, guided by actual query plans. | High for proven slow queries | Low–medium | Compare `EXPLAIN` plans and query latency before and after on representative data. |
| 5 | Paginate large API/database reads and select only required columns. | High for large result sets | Low–medium | Compare response size, memory use, and p95 endpoint latency. |
| 6 | Cache repeatedly requested, read-heavy data with bounded TTLs and explicit invalidation. | Medium–high when reads are repeated | Medium | Measure cache hit rate and origin database load; verify freshness behavior. |
| 7 | Right-size RDS instance/storage and enable only workload-appropriate I/O and connection settings. | High if the database is saturated | Medium | Use database CPU, I/O latency, queue depth, and connection metrics. |
| 8 | Tune JVM heap and garbage-collection settings to the container memory limit rather than host memory. | Medium–high for Java service latency | Low–medium | Compare GC pause time, allocation rate, memory headroom, and p95 latency. |
| 9 | Reduce Kubernetes image-pull and rollout time with smaller images, immutable tags, and appropriate node image caching. | Medium for startup and deploy time | Low–medium | Measure image size, pull duration, and pod readiness time. |
| 10 | Use Terraform provider/plugin caching in CI and avoid unnecessary module or data-source work in plans. | Medium for infrastructure feedback time | Low | Compare plan duration and provider download time on clean and warm runs. |

**Recommended first implementation:** CI dependency and layer caching (#1) is a high-impact, low-effort candidate for shortening repeated build feedback without changing runtime behavior. The workflow and Maven project are not present in this repository yet, so the actual cache configuration should be added alongside those build files and validated with cold-versus-warm runs; this blueprint records the priority rather than claiming an unverified runtime improvement.
