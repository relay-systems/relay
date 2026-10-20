# Architecture Overview

Status as of 2026-10-21. This document separates what exists today from what is planned; nothing is described as working unless it has been verified.

## System overview

Relay is a real-time messenger built as a polyglot microservices system. The architecture is designed for high availability, scalability and low latency communication. Each service lives in its own repository under [relay-systems](https://github.com/relay-systems) and is built and deployed independently.

## Status legend

| Status | Meaning |
|---|---|
| **Implemented** | Exists and runs |
| **Partially implemented** | Some parts exist, others are planned |
| **Planned** | Documented intent only; no implementation |

## Repositories and components

| Repository | Component | Stack | Status |
|---|---|---|---|
| [core-service](https://github.com/relay-systems/core-service) | Core Service | Java 21, Spring Boot | **Partially implemented** |
| [infrastructure](https://github.com/relay-systems/infrastructure) | PostgreSQL, ZITADEL, Traefik | Docker Compose | **Implemented** (local development only) |
| [api-gateway](https://github.com/relay-systems/api-gateway) | API Gateway | Java 21, Spring Cloud Gateway | **Planned** |
| [realtime-gateway](https://github.com/relay-systems/realtime-gateway) | Realtime Gateway | Go, WebSockets | **Planned** |
| [ai-service](https://github.com/relay-systems/ai-service) | AI Service | Python, FastAPI | **Planned** |
| [frontend](https://github.com/relay-systems/frontend) | Frontend | Angular, TypeScript | **Planned** |

The four **Planned** repositories contain only their project setup: a README, a `.gitignore` and CI workflows. They have no application code.

## Services

### Core Service: partially implemented

Owns application user profiles and core messaging logic.

Exists today:
- A Java 21 Spring Boot application with a health endpoint (`GET /api/v1/health`) and the Actuator `health` endpoint.
- A PostgreSQL connection through JDBC (database `relay`), configured with the `DB_URL`, `DB_USER` and `DB_PASSWORD` environment variables.
- Liquibase migrations that create the `users` and `user_identities` tables. A user is linked to an identity provider account by issuer and subject, and holds no credentials.
- A `local` profile with an opt-in tool that recovers from changed Liquibase changesets in local development only.

Planned, not implemented:
- User profile creation and lookup, and any other business logic.
- Validation of ZITADEL-issued tokens and mapping of identities to users. There is no security dependency in the service yet.
- Message storage.

### API Gateway: planned

The single entry point for clients, routing requests to the backend services. Planned stack: Java 21 and Spring Cloud Gateway.

### Realtime Gateway: planned

- WebSocket handling for real-time communication
- Message routing between services
- Redis for presence tracking and caching (Redis is planned and not deployed)

Planned stack: Go and WebSockets.

### AI Service: planned

- AI-powered features and natural language processing
- REST API for AI capabilities
- Integration with the core messaging system

Planned stack: Python and FastAPI.

### Frontend: planned

The web client. User actions are intended to trigger HTTP and WebSocket requests to the backend. Planned stack: Angular and TypeScript. Sign-in is planned through ZITADEL using OpenID Connect, Authorization Code flow with PKCE, as a public client that holds no secret.

## Infrastructure

| Component | Status | Details |
|---|---|---|
| PostgreSQL 16 | **Implemented** (local) | One server with the `relay` database for Core Service data and a separate `zitadel` database and role for ZITADEL. Published on `127.0.0.1:5432` only. |
| ZITADEL | **Implemented** (local) | Self-hosted identity provider (API and Login UI). It runs locally but is not yet integrated with any Relay service. |
| Traefik | **Implemented** (local) | Reverse proxy in front of ZITADEL, published on `127.0.0.1:8081` only. It is not an API gateway for Relay services. |
| Docker Compose | **Implemented** (local) | Local development setup in the `infrastructure` repository. Deployment to other environments is planned. |
| Kafka | **Planned** | Intended as the asynchronous message broker for event-driven communication. Not deployed and not used by any service. |
| Redis | **Planned** | Intended for presence tracking, caching and session management. Not deployed and not used by any service. |

## Communication paths

| Path | Status |
|---|---|
| Core Service to PostgreSQL (JDBC) | **Implemented**, verified |
| ZITADEL to PostgreSQL (its own `zitadel` database) | **Implemented**, verified |
| Client to ZITADEL API and Login UI through Traefik (HTTP, localhost only) | **Implemented**, verified |
| Frontend to API Gateway (HTTP) | **Planned** |
| Frontend to Realtime Gateway (WebSocket) | **Planned** |
| API Gateway to backend services | **Planned** |
| Realtime Gateway message routing between services | **Planned** |
| AI Service integration with the core messaging system | **Planned** |
| Frontend sign-in with ZITADEL (OIDC Authorization Code + PKCE) | **Planned** |
| Core Service validation of ZITADEL tokens | **Planned** |
| Event streaming through Kafka | **Planned** |
| Presence tracking and caching through Redis | **Planned** |

No other communication path between Relay services is implemented.

## Intended communication flow

The original design intent, none of which is implemented end to end:

1. User actions in the frontend trigger HTTP and WebSocket requests.
2. The Realtime Gateway handles real-time WebSocket connections.
3. The Core Service processes business logic and database operations.
4. The AI Service provides enhanced features.
5. Kafka (planned) facilitates asynchronous event streaming.
6. Redis (planned) provides fast access to frequently used data.

## Technology decisions

The polyglot approach allows each service to use the best language and tooling for its specific requirements:
- Java and Spring Boot for enterprise-grade backend services
- Go for high-performance real-time handling
- Python for rapid AI feature development
- Angular for a reactive frontend

Authentication is delegated to ZITADEL instead of being built into Core Service, so no Relay service stores passwords or credentials.

## Scaling

Each service is intended to scale independently based on load, with Kafka (planned) providing a unified messaging layer for inter-service communication. This is a design goal; it is not implemented.
