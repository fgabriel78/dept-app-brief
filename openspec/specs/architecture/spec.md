# Specification: Core Departmental Architecture

## Purpose
Establish the mandatory technical standards for departmental applications with fewer than 40 users, ensuring operational simplicity, end-to-end strong typing, and standardized deployment.

## Requirements

### Requirement: End-to-End Type Safety
The system MUST maintain strict TypeScript typing across the database, server, and client.

#### Scenario: User input validation
- GIVEN a Server Action or Route Handler receiving client data
- WHEN the action is invoked
- THEN the system MUST validate the payload against a Zod schema before interacting with Prisma
- AND return typed errors if validation fails.

### Requirement: Delegated Authentication (SSO)
The system MUST delegate authentication to the corporate identity provider (OIDC / Entra ID) via Auth.js.

#### Scenario: Access to protected routes and mutations
- GIVEN an unauthenticated user
- WHEN they attempt to access any application route or execute a Server Action
- THEN the system MUST reject the request or redirect to the SSO flow
- AND never store or manage passwords in the local database.

### Requirement: Modular Monolith Data Access
PostgreSQL access MUST be performed exclusively through Prisma Client from the server.

#### Scenario: Data reading and writing
- GIVEN a view that requires data from PostgreSQL
- WHEN the page is rendered
- THEN the data MUST be loaded via React Server Components (RSC)
- AND mutations MUST be executed via Server Actions with revalidation (`revalidatePath`).

### Requirement: Standardized Container Deployment
The application MUST be packageable as a self-contained, production-ready Docker image.

#### Scenario: Production image build
- GIVEN the repository source code
- WHEN `docker build` is executed
- THEN the build MUST use Next.js `standalone` mode
- AND produce a minimal image that requires no development dependencies at runtime.
