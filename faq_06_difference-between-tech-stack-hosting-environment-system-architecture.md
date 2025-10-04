# What Are The Differences Between Tech Stack, Hosting Environment, & System Architecture?

Here's a breakdown that separates the tech stack, hosting environment, and system architecture.

## Defining Tech Stack, Hosting Environment, & System Architecture

* **Tech stack** – The *ingredients*: programming languages, frameworks, runtimes, databases, queues, and tooling. It’s what the code is written in and what it directly depends on.
* **Hosting environment** – The *kitchen*: where and how the software runs (cloud/on-prem, regions, OS images, container/orchestration layer, networks, secrets, IaC). It’s the operational substrate.
* **System architecture** – The *recipe and plating*: how components are split, interact, scale, and fail (service boundaries, data flows, patterns like CQRS/event-driven, sync vs async, caching, consistency model).

| Dimension      | Tech stack                           | Hosting environment                  | System architecture               |
| -------------- | ------------------------------------ | ------------------------------------ | --------------------------------- |
| Core question  | “What do we build with?”             | “Where does it run?”                 | “How does it fit together?”       |
| Owned by       | Dev leads/platform eng               | SRE/infra/platform                   | Principal eng/architects          |
| Artifacts      | `package.json`, `pom.xml`, DB choice | Terraform, Helm, VPCs, AMIs, secrets | Diagrams, ADRs, sequence diagrams |
| Change cadence | Medium                               | Slower, riskier                      | Slowest, deliberate               |
| Risk type      | Library/API churn                    | Outage/misconfig                     | Coupling/latency/blast radius     |

### Example: B2B SaaS Web App

* **Tech stack:**

  * Frontend: TypeScript + React + Vite
  * Backend: Kotlin + Spring Boot
  * Data: PostgreSQL, Redis; Messaging: Kafka
  * Tooling: Gradle, OpenAPI, Testcontainers
* **Hosting environment:**

  * Cloud: AWS, us-east-1 + eu-west-1
  * Runtime: EKS (Kubernetes), container images from ECR
  * Networking: Private subnets, ALB ingress, WAF, mutual TLS between services
  * IaC/ops: Terraform + Helm; secrets in AWS Secrets Manager; observability via CloudWatch + Prometheus/Grafana
* **System architecture:**

  * Hexagonal microservices: `accounts`, `billing`, `reporting`
  * Sync REST for reads; async Kafka events for write propagation
  * Outbox pattern to ensure “exactly-once” event publication
  * Read-optimized projections for dashboards; Sagas for multi-service transactions

### Example: Internal Data Processing Pipeline

* **Tech stack:** Python (Pydantic, FastAPI), Spark, Delta Lake; DBT for transformations.
* **Hosting environment:** Azure Databricks + ADLS; private endpoints; Azure DevOps pipelines; Key Vault.
* **System architecture:** Event-driven ingestion from Event Hubs → bronze/silver/gold lake layers; CDC from OLTP via Debezium; downstream marts exposed via Synapse serverless.

### Example: Regulated On-Prem App Modernized

* **Tech stack:** C#/.NET 8, Blazor, Oracle DB, NServiceBus.
* **Hosting environment:** VMware clusters, RHEL workers, private PKI, HashiCorp Vault; GitLab runners inside DMZ.
* **System architecture:** Modular monolith with well-defined bounded contexts; integration via a message bus to isolate legacy ERP; blue/green deploys using parallel pools.

## Common Misunderstandings

* “We’re ‘serverless’, so our architecture is microservices.”
  Serverless (hosting choice) ≠ microservices (architectural style). You can run a modular monolith on Lambda or run microservices on VMs.
* “We use React and Postgres, so we’re scalable.”
  That’s stack. Scalability comes from architecture (stateless services, queues, partitioning) plus environment (autoscaling groups, capacity).
* “Moving to Kubernetes will fix our coupling.”
  K8s is environment. Tight coupling is an architectural concern; fix service boundaries first.

## How To Decide?

* **Choose the tech stack** for team fit, ecosystem maturity, and runtime constraints (e.g., JVM for Kafka ecosystem; .NET for Windows interop). Prefer boring tech for core flows.
* **Shape the architecture** around *domain boundaries, latency budgets, and failure modes*. Start with a modular monolith unless you have clear reasons to distribute.
* **Design the hosting environment** for *operability and compliance*: network boundaries, secrets, least privilege, backup/restore, infra-as-code, and observability from day one.

## Mental Model

Think of a three-layer cake:

```
[ System Architecture ]  ← shapes flows, SLAs, boundaries
[ Hosting Environment ]  ← provides runtime, security, ops guarantees
[ Tech Stack ]           ← implements the features
```

Change from the bottom up is cheaper; change from the top down is disruptive. Anchor long-term decisions at the architecture layer, lock down environment invariants for safety, and let the stack evolve carefully (library upgrades, runtime versions) without rewriting the world.

___