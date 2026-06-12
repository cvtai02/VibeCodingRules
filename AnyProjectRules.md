# Project Rules

## 1. Project Structure

Every project should follow this root structure:

```text
project-root/
├── app/                    -> Main business source code. Exposes the application APIs.  
├── ui/                     -> Admin interface for interacting with the app API, logging in with the system secret, and configuring system settings.
├── handoffs/               -> Temporary coordination documents between backend and UI work. Backend handoffs describe updated APIs for UI. UI handoffs describe required backend API changes.
│   ├── backend-to-ui/
│   │   └── archive/
│   └── ui-to-backend/
│       └── archive/
├── index.md                -> Root index explaining each top-level folder and how to navigate the project
├── rules.md                -> Project Rules
├── AGENTS.md               -> Instructions for AI agents working in this project. These files must require the agent to read `index.md` and `rules.md` before making changes. 
└── CLAUDE.md               -> Same as AGENTS.md, but for Claude
```

`app/` and `ui/` are workspace roots; inside each, follow the framework's standard layout (e.g. `app/src/` for NestJS, `ui/app/` for Next.js). Framework best practice takes priority over the structure rules in this document.

## 2. Application Rules

### 2.1 Settings

Only bootstrap settings must be stored in `.env`.

Bootstrap settings include:

* SYSTEM_SECRET (app)
* ENCRYPTION_KEY (app)
* DATABASE_CONNECTION_STRING (app)
* API_BASE_URL (ui)

All other runtime settings must be stored in the database so they can be viewed and updated from the UI. Database-stored settings apply to `app/` only; the UI keeps nothing beyond its bootstrap settings.

Secret runtime settings stored in the database must be encrypted and decrypted with the encryption key.

Examples of settings include:

* External API configuration
* Storage configuration
* Feature flags
* Environment-specific settings

### 2.2 Admin UI and Authentication

Every project must include an admin UI.

The admin UI must support login with the system secret from `.env`.

Protected API endpoints must authenticate via the `Authorization: Bearer <token>` header. Never use cookie-based sessions, and never send tokens in query strings.

Bearer tokens are issued from the system secret login: an admin logs in with the system secret, and the app issues access tokens from that session.

### 2.3 CORS
CORS must allow all origins, methods, and headers.
Allow-all CORS is acceptable because authentication uses the bearer header, not cookies, so cross-origin requests from malicious pages carry no credentials.

### 2.4 Architecture

Do not add a repository layer or an abstract layer on top of the ORM.

The ORM context belongs in the application core. Use cases may depend directly on the ORM context.

The ORM itself is the database adapter: swapping databases (e.g. SQLite to PostgreSQL) is done through the ORM's datasource configuration, not through a custom abstraction.

Use dependency injection whenever possible.

Follow the naming conventions of the selected technology stack.

### 2.5 Modules

The app should be modularized properly. Each module should contain:

```text
module-name/
├── usecases/
├── api/              # or controllers/, depending on the stack convention
└── dtos/
```

Module rules:

* Each use case must be defined in its own file.
* Each API/controller must be defined in its own file.
* DTOs must be shared by both API/controllers and use cases.
* Each DTO must be defined in its own file.
* When a `usecases/`, `api/` or `controllers/`, or `dtos/` directory contains many files, group related files into descriptive subfolders.

### 2.6 Core / Shared Kernel

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

### 2.7 Entities and Aggregates

Entity classes must define and protect the constraints of that entity.

Use public and private properties/methods properly so invalid state cannot be created or persisted accidentally.

Aggregates must define and protect constraints involving relationships between multiple entities.

When the ORM returns plain objects instead of entity classes, validate invariants in use cases and shared value objects rather than introducing a mapping layer over the ORM.

At project setup, copy these entity and aggregate rules into the project's `rules.md` so agents preserve them during implementation.

### 2.8 Infrastructure

Infrastructure code is responsible for external services and adapters.

Examples include:

* Database configuration
* Storage services
* External service integrations
* File systems
* Message queues
* Email providers
* Cache providers

Infrastructure implementations must implement abstractions/contracts defined in the core or shared kernel.

All non-ORM infrastructure should be adapter-based so implementations can be replaced. The ORM is the database adapter (see 2.4) and does not need an additional adapter.

Examples:

* R2 adapter / Google Drive adapter
* Local file storage adapter / cloud storage adapter
* SMTP adapter / email API adapter

### 2.9 API Errors

APIs must return one consistent JSON error shape across all endpoints. Use the framework's default error format if it provides one (e.g. NestJS: `{ statusCode, message, error }`); otherwise define `{ code, message, details? }` once in the shared kernel.

Changing the error format is an API contract change and follows the handoff rules.

### 2.10 Database Migrations

All schema changes must go through ORM migrations, committed in the same change set as the code that needs them. Never edit the database schema manually.

### 2.11 Testing

A passing smoke test is the required gate for every change (see 3.2). Unit tests are encouraged for shared value objects, policies, and use cases with non-trivial logic.

## 3. Agent Workflows

Agents must report the skill names they used every time a skill is used.

Generated code must be separated into appropriate files and folders so it can be indexed and searched reliably.

### 3.1 Implement Infrastructure Workflow

Purpose: implement infrastructure adapters and external service integrations.

Rules:

* May access only core/shared-kernel contracts and infrastructure code.
* Must implement abstractions/contracts defined in the core or shared kernel.
* Must not leak provider-specific details into use cases or controllers.

### 3.2 Smoke Test Workflow

Purpose: verify that changed functionality works end-to-end.

After a successful smoke test, the agent must:

1. Create backend-to-ui handoff, unless the UI change ships in the same change set (see 4.1).
2. Commit the code. Agents are authorized to commit without asking after a passing smoke test.

The smoke test result should clearly state what was tested and whether it passed.

## 4. Handoff Rules

Handoff documents are temporary coordination documents between backend and UI work.

Each handoff must be marked with one of these statuses:

* `Pending`
* `Completed`

Completed handoffs must be moved to the `archive/` folder inside their direction folder:

```text
handoffs/backend-to-ui/archive/
handoffs/ui-to-backend/archive/
```

### 4.1 Backend-to-UI handoffs

Every API contract change must create a backend-to-UI handoff.

Exception: if the corresponding UI change ships in the same change set, the handoff may be skipped.

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

### 4.2 UI-to-Backend handoffs

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

### 4.3 Handoff document template

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
- [ ] Smoke test passes, if applicable.

## Notes

Add any extra coordination notes here.
```

## 5. Out of Scope

These concerns are intentionally not covered by this template. Address them per project only when a real need appears:

* Encryption key rotation and re-encryption of stored secrets
* Multi-user authentication and role-based access control
* Observability standards (structured logging, metrics, tracing)