# Project Rules & Architecture Standards

## Project Structure

Every project must contain the following top-level folders and files:

```text
/
├── app/
├── ui/
├── api-clients/
├── api-mcp-server/
├── handoffs/
│   └── archive/
├── index.md
├── rules.md
└── agents/
```

### Folder Responsibilities

| Folder           | Purpose                                                                                                                                       |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `app`            | Main business application. Exposes APIs and contains all business logic.                                                                      |
| `ui`             | Administrative interface used to interact with App APIs, manage settings, and authenticate using access tokens.                               |
| `api-clients`    | Reusable TypeScript API client package containing DTOs, interfaces, and implementations. Can be copied to other projects or published to npm. |
| `api-mcp-server` | Utilities for mock data generation, dirty data cleanup, smoke testing, diagnostics, and automation tasks.                                     |
| `handoffs`       | Temporary coordination documents between backend and UI teams.                                                                                |
| `index.md`       | Project navigation and architecture index.                                                                                                    |
| `rules.md`       | Project-wide implementation standards and architecture rules.                                                                                 |
| `agents`         | AI-agent instructions, skills, workflows, and operating guidelines.                                                                           |

---

# Application Architecture Rules

## Configuration Management

All runtime settings must be stored in a JSON configuration file.

Examples:

* Access tokens
* Database settings
* External API credentials
* Storage provider settings
* Feature flags

Requirements:

* Configuration must be editable from the UI.
* Configuration must be reloadable without code changes.
* Application reset functionality may recreate this file if needed.
* Configuration files **must be excluded from source control** via `.gitignore`.

---

## Dependency Management

* Use Dependency Injection whenever practical.
* Dependencies should be resolved through contracts/interfaces defined in Core.
* Avoid direct infrastructure dependencies inside Use Cases.

---

## Architecture Layers

```text
app/
├── Core/
├── Infrastructure/
├── UseCases/
├── Api/
└── Dtos/
```

### Core Layer

The Core layer contains:

* Entities
* Aggregates
* Value Objects
* Enums
* Constants
* Policies
* Contracts / Interfaces
* Domain Services

#### Entity Rules

Entities must enforce their own invariants.

Use:

* Private setters
* Encapsulated state changes
* Domain methods

Avoid:

* Public mutable properties
* Anemic domain models

#### Aggregate Rules

Aggregates are responsible for enforcing constraints spanning multiple entities.

Examples:

* Ownership rules
* Membership limits
* Referential consistency
* Business transaction boundaries

---

### Infrastructure Layer

Infrastructure implements contracts defined by Core.

Examples:

* Database providers
* Storage providers
* External APIs
* Authentication providers
* Messaging systems

Requirements:

* Infrastructure implementations must be swappable.
* Provider selection should be configurable through application settings.

Examples:

* SQLite ↔ PostgreSQL
* R2 ↔ Google Drive
* Local Storage ↔ S3

---

### ORM Rules

Repository patterns are prohibited.

Do not create:

```text
Repository
GenericRepository
UnitOfWork
DataAccessLayer
```

Instead:

* ORM context belongs directly in Core.
* Use Cases may depend directly on ORM context abstractions.
* Avoid unnecessary abstraction layers above the ORM.

---

### Use Cases Layer

The Use Cases layer contains application workflows and business operations.

Responsibilities:

* Execute business logic.
* Coordinate entities and aggregates.
* Access infrastructure through contracts.
* Return DTOs.

---

### API Layer

Responsibilities:

* HTTP endpoints
* Request validation
* Authentication
* DTO mapping

The API layer must not contain business logic.

---

### DTO Rules

DTOs are shared between:

* API Layer
* Use Cases Layer

Requirements:

* Single source of truth.
* Consistent folder structure.
* Predictable naming conventions.
* Easy discovery through search.

Example:

```text
Dtos/
├── Users/
├── Orders/
└── Settings/
```

---

# API Client Rules

## Synchronization

API DTOs must be synchronized from the App project.

The App project is the source of truth.

---

## Implementation

Requirements:

* TypeScript only.
* Use native `fetch`.
* No external HTTP client dependencies unless explicitly approved.

Generated package should contain:

```text
api-clients/
├── dtos/
├── interfaces/
├── implementations/
└── index.ts
```

---

# Documentation Standards

Documentation exists primarily to support AI agents and developers.

---

## Project-Level Documentation

Required:

```text
/
├── index.md
└── rules.md
```

### index.md

Purpose:

* Project navigation
* Architecture overview
* Folder discovery
* Search entry point

### rules.md

Purpose:

* Implementation rules
* Coding standards
* Architectural constraints

---

## Layer-Level Documentation

Every layer root must contain:

```text
Layer/
├── index.md
└── rules.md
```

### Layer index.md

Purpose:

* Searchable navigation
* File discovery
* API discovery
* Use Case discovery

### Layer rules.md

Purpose:

* Layer-specific implementation guidance
* Architecture constraints
* Coding standards

---

# Agent Skills

Whenever an agent uses a skill, it must explicitly report the skill name used.

---

## Skill: Implement API

Allowed Access:

* Core
* Use Cases
* DTOs
* API

Forbidden Access:

* Infrastructure implementation details

---

## Skill: Implement Infrastructure

Allowed Access:

* Core
* Infrastructure

Forbidden Access:

* API
* UI
* Use Cases

---

## Skill: Smoke Test

May use:

* API MCP Server
* Test data
* Mock services

A successful smoke test must:

### 1. Update API Clients

Regenerate or synchronize API clients if contracts changed.

### 2. Update Documentation

Update:

* Layer `index.md`
* Project `index.md`

for any architectural changes.

### 3. Commit Changes

Create a commit after successful validation.

---

## Skill: Code Generation

Generated code must be organized for discoverability.

Requirements:

* Clear folder ownership
* Predictable file locations
* Search-friendly structure
* Minimal file coupling

---

# Handoff Process

Handoffs are temporary coordination documents.

---

## Handoff Status

Every handoff must have one of the following statuses:

* Pending
* Completed

Example:

```md
Status: Pending
```

or

```md
Status: Completed
```

---

## Archive Rules

Completed handoffs must be moved to:

```text
handoffs/archive/
```

---

## Backend → UI Handoff

A backend-to-ui handoff must be created whenever:

* API contracts change
* DTOs change
* Authentication changes
* API behavior changes

---

## UI → Backend Handoff

A ui-to-backend handoff must be created whenever:

* New backend functionality is required
* New endpoints are needed
* Existing APIs require changes
* Additional data must be exposed

---

# Architecture Principles

1. Configuration is user-controlled and UI-editable.
2. Core owns business rules.
3. Entities protect their own invariants.
4. Aggregates protect cross-entity invariants.
5. Infrastructure is replaceable.
6. Repository patterns are prohibited.
7. DTOs are shared across API and Use Cases.
8. API Clients are generated from App contracts.
9. Documentation is optimized for AI-agent navigation.
10. Handoffs provide traceable coordination between teams.
11. Generated code must remain discoverable and searchable.
