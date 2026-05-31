# Any Project Rules

## Root Structure

Every project should contain:

```text
project/
├── app/                # Main business application
├── ui/                 # User interface application
├── api-clients/        # Reusable TypeScript API SDKs
├── api-mcp-server/     # Internal tooling & automation server
├── handoffs/           # Temporary coordination documents
├── agents/             # AI agent instructions
├── index.md            # Root folder index
└── rules.md            # Global project rules
```

## Root Folder Responsibilities

### app/
Main business application.

- Expose APIs
- Execute business use cases
- Manage application lifecycle
- Load runtime settings

### ui/
Frontend application.

- Consume APIs from app
- Display data
- Manage user interactions

### api-clients/
Reusable TypeScript SDKs.

- Interfaces
- Implementations
- DTOs / Types

Goals:

- Copyable to other projects
- Publishable as npm packages
- Acts as the contract between services

### api-mcp-server/
Internal tooling and automation server.

Use cases:

- Generate mock data
- Fix dirty data
- Data migration
- Smoke testing
- Maintenance scripts
- Operational tooling

### handoffs/

Temporary coordination documents.

Statuses:

- Pending
- Completed

Completed handoffs must be moved to:

```text
handoffs/archive/
```

## Application Rules

### Configuration

- No `.env` files.
- Store all settings in the database.
- Settings must be editable through the UI.
- Load settings on startup and after changes.

### Architecture

- Use Domain-Driven Design (DDD).
- No repository layer.
- ORM is the persistence core.
- Use Cases access ORM directly.

### Core Layer

Contains:

- Entities
- Aggregates
- Enums
- Constants
- Policies
- Contracts
- Abstractions

Entities encapsulate invariants and prevent invalid state.

Aggregates enforce constraints across multiple entities.

### Infrastructure Layer

Implements Core contracts.

Examples:

- SQLite / PostgreSQL
- Cloudflare R2
- Google Drive
- Local Storage

Requirements:

- Replaceable
- Runtime configurable
- No business logic

### Use Cases Layer

- Contains application business flows.
- Depends only on Core.
- Does not contain infrastructure implementations.

### API Layer

- Calls Use Cases only.
- Must not access Infrastructure directly.

### DTOs

Shared between API and Use Cases.

Rules:

- Single DTO definition
- No duplicate DTOs
- Consistent folder structure

## Documentation Requirements

Every major layer must contain:

- index.md
- rules.md

### index.md

Used for:

- Navigation
- Search
- Module discovery

### rules.md

Used for:

- Implementation rules
- Architectural constraints
- Coding conventions

## AI Agent Rules

### Implement API Skill

- Must use Use Cases as the boundary.
- Must not access Infrastructure directly.
- Must reuse existing DTOs.

### Implement Infrastructure Skill

- Accessible only to Core contracts and Infrastructure.
- Must not introduce business logic.

### Smoke Test Skill

May use api-mcp-server.

Successful smoke test requires:

1. Update api-clients.
2. Update index.md and rules.md where needed.
3. Commit code.

### Code Generation Rules

Generated code should:

- Be separated into small files.
- Be easy to search.
- Be easy to index.
- Avoid monolithic files.

When code is duplicated twice:

- Modularize it.
- Extract shared services.
- Extract helpers.
- Extract components.
- Extract base classes.
- Extract reusable use cases.

## Handoff Rules

Backend → UI handoff required for:

- API contract changes
- DTO changes
- Endpoint changes
- Authentication changes

UI → Backend handoff required for:

- New backend requirements
- Missing APIs
- New backend capabilities

## Architectural Boundaries

- Must not access Infrastructure directly from API.
- Must use Use Cases as the application boundary.
- Infrastructure implements Core contracts.
- DTOs are shared between API and Use Cases.
- Runtime settings come from the database.
- All major layers must contain index.md and rules.md.
- Completed handoffs must be archived.
