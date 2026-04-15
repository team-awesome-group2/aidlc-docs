# AI-DLC State Tracking

## Project Information

- **Project Type**: Brownfield
- **Start Date**: 2026-04-15T00:00:00Z
- **Current Stage**: INCEPTION - User Stories

## Workspace State

- **Existing Code**: Yes
- **Programming Languages**: Java 21, TypeScript, React/JSX
- **Build Systems**: Maven (Java services), Vite/npm (frontend)
- **Project Structure**: Microservices (6 repos)
- **Architecture Pattern**: DDD + Hexagonal Architecture
- **Reverse Engineering Needed**: Yes
- **Workspace Root**: /Users/wyz/project/aidlc-ws

## Repositories

| Repository | Type | Port | Tech Stack |
|---|---|---|---|
| awsome-shop-auth-service | Backend | 8001 | Java 21, Spring Boot 3.4.1, MyBatis-Plus, JWT |
| awsome-shop-product-service | Backend | 8002 | Java 21, Spring Boot 3.4.1, MyBatis-Plus |
| awsome-shop-points-service | Backend | 8003 | Java 21, Spring Boot 3.4.1, MyBatis-Plus |
| awsome-shop-order-service | Backend | 8004 | Java 21, Spring Boot 3.4.1, MyBatis-Plus |
| awsome-shop-gateway-service | Backend | 8080 | Java 21, Spring Boot 3.5.10, Spring Cloud Gateway |
| awsome-shop-frontend | Frontend | — | React 19, TypeScript, Vite 7, MUI 6, Zustand |

## Code Location Rules

- **Application Code**: Workspace root (NEVER in aidlc-docs/)
- **Documentation**: aidlc-docs/ only
- **Structure patterns**: See code-generation.md Critical Rules

## Extension Configuration

| Extension | Enabled | Decided At |
|---|---|---|
| Security Baseline | Yes | Requirements Analysis |
| Property-Based Testing | Yes (Full) | Requirements Analysis |

## Stage Progress

- [x] INCEPTION - Workspace Detection
- [x] INCEPTION - Reverse Engineering
- [x] INCEPTION - Requirements Analysis
- [x] INCEPTION - User Stories
- [ ] INCEPTION - Workflow Planning
- [ ] INCEPTION - Application Design
- [ ] INCEPTION - Units Generation
- [ ] CONSTRUCTION - (per-unit stages)
- [ ] CONSTRUCTION - Build and Test
