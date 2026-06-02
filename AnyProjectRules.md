# Project Rules

## 1. Project Structure

Every project should follow this root structure:

```text
project-root/
├── app/
├── ui/
├── handoffs/
│   └── archive/
├── api-clients/
├── api-mcp-server/
├── index.md
├── rules.md
└── agents-instructions.md
```

### Root folders and files

| Path                     | Purpose                                                                                                                                                                |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `app/`                   | Main business source code. Exposes the application APIs.                                                                                                               |
| `ui/`                    | Admin interface for interacting with the app API, configuring app settings, and using access tokens during early login flows.                                          |
| `handoffs/`              | Temporary coordination documents between backend and UI work. Backend handoffs describe updated APIs for UI. UI handoffs describe required backend API changes.        |
| `handoffs/archive/`      | Completed handoff documents.                                                                                                                                           |
| `api-clients/`           | TypeScript API clients containing interfaces and implementations. These clients should be portable so they can be copied into other apps or published as npm packages. |
| `api-mcp-server/`        | MCP server for creating mock data, fixing dirty data, smoke testing, and other local automation tasks.                                                                 |
| `index.md`               | Root index explaining each top-level folder and how to navigate the project.                                                                                           |
| `rules.md`               | Root project rules.                                                                                                                                                    |
| `agents-instructions.md` | Instructions for AI agents working in this project.                                                                                                                    |

## 2. Application Rules

### 2.1 Settings

All runtime settings must be stored in a JSON configuration file so they can be viewed and updated from the UI.

Examples of settings include:

* Access tokens
* Database configuration
* External API configuration
* Storage configuration
* Feature flags
* Environment-specific settings

The settings JSON file must be included in `.gitignore`.

If settings are changed through the UI, the app may require a reset or restart for those settings to take effect.

### 2.2 Architecture

Do not add a repository layer or an abstract layer on top of the ORM.

The ORM context belongs in the application core. Use cases may depend directly on the ORM context.

Use dependency injection whenever possible.

Follow the naming conventions of the selected technology stack.

### 2.3 Modules

The app should be modularized properly. Each module should contain:

```text
module-name/
├── usecases/
├── api/              # or controllers/, depending on the stack convention
└── dtos/
```

Module rules:

* Use cases contain business actions and application logic.
* API/controllers expose use cases through HTTP or another API boundary.
* DTOs must be shared by both API/controllers and use cases.
* API/controllers and use cases should use the same DTO definitions.
* All DTO directories must follow the same naming and path pattern so agents and developers can search them reliably.

### 2.4 Core / Shared Kernel

The app must include a core or shared kernel area for definitions used across modules.

The shared kernel may contain:

* Enums
* Constants
* Policies
* Shared value objects
* Shared types
* Infrastructure abstractions/contracts
* Cross-module domain concepts

Shared kernel code should remain stable, intentional, and broadly reusable.

### 2.5 Entities and Aggregates

Entity classes must define and protect the constraints of that entity.

Use public and private properties/methods properly so invalid state cannot be created or persisted accidentally.

Aggregates must define and protect constraints involving relationships between multiple entities.

Persist this instruction in the project documentation so agents preserve these rules during implementation.

### 2.6 Infrastructure

Infrastructure code is responsible for external services and adapters.

Examples include:

* Database configuration
* Storage services
* External API clients
* File systems
* Message queues
* Email providers
* Cache providers

Infrastructure implementations must implement abstractions/contracts defined in the core or shared kernel.

All infrastructure should be adapter-based so implementations can be replaced.

Examples:

* SQLite adapter / PostgreSQL adapter
* R2 adapter / Google Drive adapter
* Local file storage adapter / cloud storage adapter

## 3. API Clients

The `api-clients/` package contains TypeScript API clients.

Rules:

* API DTOs must be synced from `app/`.
* Clients must use native `fetch`.
* Clients should include both interfaces and implementations.
* Clients should be portable enough to copy into another app.
* Clients should be publishable as npm packages when needed.

Recommended structure:

```text
api-clients/
├── src/
│   ├── dtos/
│   ├── interfaces/
│   ├── clients/
│   └── index.ts
├── package.json
├── tsconfig.json
├── index.md
└── rules.md
```

## 4. Documentation for AI Agents

Documentation must be created so AI agents can search, understand, and safely modify the project.

### 4.1 Required documentation in `app/`

The `app/` folder must include:

```text
app/
├── index.md
└── rules.md
```

* `index.md` explains the app architecture, important folders, APIs, modules, and use cases.
* `rules.md` explains implementation rules agents must follow when changing app code.

### 4.2 Required documentation per layer

Each major layer must include its own:

```text
layer-root/
├── index.md
└── rules.md
```

Rules:

* `index.md` is for searching and navigation.
* `rules.md` is for implementation guidance.
* Documentation should be updated whenever APIs, use cases, DTOs, infrastructure, or module boundaries change.

## 5. Agent Skills

Agents must report the skill names they used every time a skill is used.

Generated code must be separated into appropriate files and folders so it can be indexed and searched reliably.

### 5.1 Implement API Skill

Purpose: implement API/controller changes.

Rules:

* Must not access infrastructure directly.
* Must call use cases instead of implementing business logic inside controllers.
* Must use shared DTOs from the module DTO folder.
* Must update relevant `index.md` files when API routes or contracts change.
* Must create a backend-to-UI handoff for every API contract change.

### 5.2 Implement Infrastructure Skill

Purpose: implement infrastructure adapters and external service integrations.

Rules:

* May access only core/shared-kernel contracts and infrastructure code.
* Must implement abstractions/contracts defined in the core or shared kernel.
* Must not leak provider-specific details into use cases or controllers.
* Must document any new adapter in the relevant `index.md` and `rules.md` files.

### 5.3 Smoke Test Skill

Purpose: verify that changed functionality works end-to-end.

Use `api-mcp-server/` when needed.

After a successful smoke test, the agent must:

1. Update `api-clients/` if API contracts changed.
2. Update indexing documentation for each changed layer.
3. Commit the code.

The smoke test result should clearly state what was tested and whether it passed.

## 6. Handoff Rules

Handoff documents are temporary coordination documents between backend and UI work.

Each handoff must be marked with one of these statuses:

* `Pending`
* `Completed`

Completed handoffs must be moved to:

```text
handoffs/archive/
```

### 6.1 Backend-to-UI handoffs

Every API contract change must create a backend-to-UI handoff.

Use this when backend changes affect the UI, such as:

* New API endpoint
* Removed API endpoint
* Changed request DTO
* Changed response DTO
* Changed error format
* Changed authentication behavior
* Changed pagination, filtering, sorting, or validation behavior

Recommended filename pattern:

```text
handoffs/backend-to-ui/YYYY-MM-DD-short-description.md
```

### 6.2 UI-to-Backend handoffs

Every new UI requirement that needs backend support must create a UI-to-backend handoff.

Use this when the UI needs backend changes, such as:

* New endpoint request
* New field requirement
* New filtering or search behavior
* New admin action
* New validation requirement
* New data export/import behavior

Recommended filename pattern:

```text
handoffs/ui-to-backend/YYYY-MM-DD-short-description.md
```

### 6.3 Handoff document template

```md
# Handoff: <Short Description>

Status: Pending
Direction: Backend to UI | UI to Backend
Created: YYYY-MM-DD
Owner: <Name or Team>

## Summary

Describe the change or request.

## Context

Explain why this handoff exists.

## Contract / Requirement

Describe the API contract, DTO change, UI need, or backend support required.

## Files Changed or Expected

List related files, modules, routes, DTOs, or use cases.

## Acceptance Criteria

- [ ] Requirement is implemented.
- [ ] API client is updated, if needed.
- [ ] Relevant `index.md` files are updated.
- [ ] Relevant `rules.md` files are updated, if needed.
- [ ] Smoke test passes, if applicable.

## Notes

Add any extra coordination notes here.
```

## 7. Documentation Update Requirements

Update documentation whenever any of the following changes:

* Folder structure
* Module boundaries
* API contracts
* DTOs
* Use cases
* Infrastructure adapters
* Shared kernel concepts
* Agent implementation rules
* Handoff status

Documentation should remain searchable, consistent, and useful for both developers and AI agents.
