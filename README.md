# StarlinTirkey-projects-aws-devops-project

A traditional, layered .NET Framework architecture blueprint for a server-rendered web application deployed to AWS. This repository documents the intended design; it does not currently contain application, Terraform, deployment, or CI implementation.

## Architecture (legacy .NET only)

The application follows the conventional ASP.NET-on-IIS model and a layered solution structure. It does not use containers, Kubernetes, microservices, .NET (Core) / modern .NET, or cloud-native orchestration.

1. **Presentation — ASP.NET MVC / Web API**
   The web project handles HTTP requests, controllers, views, routing, authentication, and input validation. It runs under IIS on Windows Server and is the only layer exposed to the load balancer.

2. **Business — class library**
   A separate .NET Framework class library contains application services, business rules, and domain validation. Controllers call this layer rather than embedding business logic in views or data-access code.

3. **Data access — class library**
   A separate class library encapsulates repositories and database access using ADO.NET or Entity Framework 6, with SQL Server as the relational database. Database credentials and environment-specific settings are supplied at deployment time, not committed to source control.

4. **Solution and build**
   Use a Visual Studio solution (`.sln`) with projects targeting the selected .NET Framework version (default: .NET Framework 4.8). Manage dependencies with legacy NuGet `packages.config` where required. Build and test with MSBuild and the Visual Studio test tools on a Windows runner.

5. **AWS infrastructure (`terraform/`)**
   Terraform provisions separate dev, staging, and prod environments: an Application Load Balancer, Windows EC2/IIS application hosts, and Amazon RDS for SQL Server. Keep state in a remote S3 backend with locking, and keep environment-specific values out of the shared modules.

6. **Deployment and operations (`scripts/`)**
   CI produces a versioned Web Deploy package or equivalent deployment artifact. PowerShell scripts deploy the artifact to IIS, apply environment configuration, and support health checks and rollback. Use CloudWatch and standard Windows/IIS logging for operational visibility.

7. **CI/CD (`.github/workflows/`)**
   A Windows-based pipeline restores NuGet packages, builds the solution with MSBuild, runs unit tests and static/security checks, packages the web application, and deploys through controlled environment approvals. Keep secrets in the CI secret store and AWS role-based credentials rather than in repository files.

## Performance improvement opportunities

These are conventional optimization areas to validate with representative workload data; they are not measured bottlenecks in this repository.

| # | Opportunity | Potential impact | How to validate |
|---|---|---|---|
| 1 | Cache NuGet packages and MSBuild outputs in the Windows CI runner. | Faster repeated builds | Compare cold and warm workflow duration and cache-hit rate. |
| 2 | Profile ASP.NET request paths and remove redundant work from controllers and business services. | Lower application latency | Compare endpoint p95 latency and traces before and after. |
| 3 | Use bounded SQL connection pooling and explicit command timeouts. | Improved stability under concurrency | Monitor connection pool use, wait time, and request latency. |
| 4 | Tune SQL queries and add indexes based on actual execution plans. | Faster proven slow queries | Compare execution plans and query latency on representative data. |
| 5 | Paginate large reads and return only fields needed by the UI/API. | Lower database and application resource use | Measure response size, memory use, and endpoint latency. |
| 6 | Use output/data caching only for suitable read-heavy content, with explicit expiry and invalidation. | Reduced repeated database work | Track cache hit rate, database load, and data freshness. |
| 7 | Right-size IIS/EC2 and RDS capacity using observed CPU, memory, I/O, and connection metrics. | Better cost and capacity balance | Load-test and compare resource utilization, errors, and p95 latency. |
| 8 | Review IIS application-pool recycling, request limits, and worker-process settings against workload needs. | Improved reliability and throughput | Monitor recycle events, queue length, errors, and response times. |
| 9 | Minimize and version Web Deploy artifacts; avoid unnecessary work during deployment. | Shorter, more reliable releases | Measure artifact size, deployment duration, and health-check time. |
| 10 | Cache Terraform providers in CI and keep plans scoped to the environment being changed. | Faster infrastructure feedback | Compare clean and warm plan duration. |

**Recommended starting point:** Add the Windows CI build and NuGet caching, then establish baseline build durations. This is a proposed architecture; confirm runner availability, target framework, and project structure when implementing it.
