# Relay

Relay is a real-time messenger built as a polyglot microservices system. This repository is the central entry point: it holds the architecture documentation and links to the independent repositories that make up the project.

Relay is under early development. Most components are planned and not implemented yet; the table below shows the current status of each repository.

## Architecture

```mermaid
flowchart TB
  subgraph clients["Clients"]
    FE["<b>Frontend</b><br/>Angular · TypeScript<br/>○ Planned"]
  end

  subgraph gateways["Gateways"]
    RTGW["<b>Realtime Gateway</b><br/>Go · WebSockets<br/>○ Planned"]
    APIGW["<b>API Gateway</b><br/>Spring Cloud Gateway<br/>○ Planned"]
  end

  subgraph services["Services"]
    AI["<b>AI Service</b><br/>Python · FastAPI<br/>○ Planned"]
    CORE["<b>Core Service</b><br/>Java 21 · Spring Boot<br/>◐ Partial: users, identities"]
  end

  subgraph infra["Infrastructure"]
    REDIS[("<b>Redis</b><br/>Presence, transient state, cache<br/>○ Planned")]
    KAFKA(["<b>Kafka</b><br/>Async events<br/>○ Planned"])
    CAS[("<b>Apache Cassandra</b><br/>Message history<br/>○ Planned")]
    PG[("<b>PostgreSQL 16</b><br/>Users, chats, membership,<br/>relational metadata<br/>● Implemented (local)")]
    TR["<b>Traefik</b><br/>Local ZITADEL proxy<br/>● Implemented (local)"]
    IDP["<b>ZITADEL</b><br/>Identity provider<br/>● Implemented (local)"]
  end

  FE -.->|WebSocket| RTGW
  FE -.->|HTTP| APIGW
  FE -.->|"OIDC · Auth Code + PKCE"| TR
  APIGW -.-> AI
  APIGW -.-> CORE
  RTGW -.-> REDIS
  RTGW -.-> KAFKA
  CORE -.-> KAFKA
  CORE -.-> CAS
  CORE -->|JDBC| PG
  TR -->|HTTP| IDP
  IDP -->|"zitadel DB"| PG
  CORE ~~~ REDIS

  classDef done fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#0f172a;
  classDef partial fill:#e0f2fe,stroke:#0369a1,stroke-width:2px,stroke-dasharray:6 3,color:#0f172a;
  classDef planned fill:#f1f5f9,stroke:#64748b,stroke-width:1.5px,stroke-dasharray:4 4,color:#334155;
  class TR,IDP,PG done;
  class CORE partial;
  class FE,APIGW,RTGW,AI,CAS,REDIS,KAFKA planned;

  style clients fill:none,stroke:#94a3b8
  style gateways fill:none,stroke:#94a3b8
  style services fill:none,stroke:#94a3b8
  style infra fill:none,stroke:#94a3b8
```

**Legend**

| Marker | Meaning |
|---|---|
| ● solid blue border | Implemented (local development only) |
| ◐ dashed blue border | Partially implemented |
| ○ dashed gray border | Planned |
| Solid arrow | Verified, currently implemented integration |
| Dashed arrow | Proposed integration, not implemented |

PostgreSQL holds users and identity links today; chats, membership and other relational metadata are planned there. Message history is planned for Cassandra. Traefik is only the local reverse proxy in front of ZITADEL, not an API gateway. Details are in the [architecture overview](docs/architecture/overview.md).

## Repositories

| Repository | Role | Stack | Status |
|---|---|---|---|
| [core-service](https://github.com/relay-systems/core-service) | User profiles and core messaging logic | Java 21, Spring Boot, PostgreSQL, Liquibase | **Partially implemented** |
| [infrastructure](https://github.com/relay-systems/infrastructure) | Local PostgreSQL, ZITADEL and Traefik | Docker Compose | **Implemented** (local development only) |
| [api-gateway](https://github.com/relay-systems/api-gateway) | Single entry point for clients | Java 21, Spring Cloud Gateway | **Planned** (repository setup only) |
| [realtime-gateway](https://github.com/relay-systems/realtime-gateway) | WebSocket gateway for real-time communication | Go, WebSockets | **Planned** (repository setup only) |
| [ai-service](https://github.com/relay-systems/ai-service) | AI-powered features | Python, FastAPI | **Planned** (repository setup only) |
| [frontend](https://github.com/relay-systems/frontend) | Web client | Angular, TypeScript | **Planned** (repository setup only) |

Status legend: **Implemented** (exists and runs), **Partially implemented** (some parts exist), **Planned** (documented intent only). Details are in the [architecture overview](docs/architecture/overview.md).

## Getting started

The only runnable parts today are PostgreSQL and ZITADEL (from `infrastructure`) and Core Service:

1. Start PostgreSQL, and optionally ZITADEL, as described in the [infrastructure](https://github.com/relay-systems/infrastructure) repository.
2. Build and run Core Service as described in the [core-service](https://github.com/relay-systems/core-service) repository.

## Documentation

- [Architecture overview](docs/architecture/overview.md): components, status and communication paths

## Branches

Every Relay repository follows the same flow: `TASK_<kebab-case>` -> `test` -> `main`, both through pull requests, enforced by the `branch-policy` workflow. Feature branches are created from `test`.
