---
name: rfc-to-jira
description: This skill should be used when converting RFC specification documents into JIRA tickets. Triggers on requests like "create JIRA tickets from this RFC", "break down this spec into tickets", "/rfc-to-jira [path-to-rfc]", or when users want to decompose an RFC into implementable work items that Claude can pick up and implement end-to-end.
---

# RFC to JIRA Ticket Generator

Convert RFC specification documents into structured, implementation-ready JIRA tickets. Each ticket contains enough context and detail for Claude Code to implement it end-to-end, including a built-in validation loop (write tests first, implement, run tests, validate locally).

## Overview

This skill reads an RFC document following the standard 16-section RFC template, analyzes its structure (Purpose, Responsibilities, Public Interface, Data Model, Key Data Flows, Implementation Phases, Testing, etc.), and produces a hierarchy of JIRA tickets:

- **1 Epic** per RFC (the full service/module)
- **Stories** per Implementation Phase defined in the RFC
- **Tasks** (subtasks) per discrete unit of work within each phase

Each task includes everything Claude needs to implement it without additional context-gathering.

## When to Use

Invoke this skill when:
- User requests: "create JIRA tickets from this RFC"
- User says: "break down [service-name] spec into tickets"
- User provides: "/rfc-to-jira [path-to-rfc]"
- User wants to: "turn this RFC into implementable work items"
- User needs: JIRA tickets that Claude can pick up and implement autonomously

## Prerequisites

Before running this skill, confirm:
1. The user has a Jira project set up (get the `cloudId` and `projectKey`)
2. The RFC file exists and follows the standard RFC template structure
3. The user has the Atlassian MCP server connected

## Workflow

### Step 1: Gather JIRA Project Info

Ask the user for:
- **Jira Cloud ID or site URL** (e.g., `your-site.atlassian.net`)
- **Project key** (e.g., `OBS`, `PLAT`)
- **Path to the RFC file** (e.g., `.specs/multi-tenancy-core/01-RFC-multi-tenancy-core.md`)

If the user has already provided any of these in their message, skip asking for those.

If the user hasn't provided a Jira Cloud ID, try to discover it:

```
mcp__claude_ai_Atlassian__getAccessibleAtlassianResources()
```

### Step 2: Read and Analyze the RFC

Read the RFC file and extract all 16 sections. Also read the companion files that provide implementation context:

```
Read(file_path="<path-to-rfc>")
```

Also read these supporting documents for cross-cutting context:
- The project's `CLAUDE.md` (root of repo) for conventions, dev commands, and guidelines
- `.specs/CONSTITUTION.md` for inviolable rules and forbidden patterns
- `.specs/stack.md` for technology stack and engineering standards
- `.specs/structure.md` for repository layout and file conventions
- `.specs/architecture-overview.md` for system architecture and module interactions
- The companion PRD file (same directory as the RFC, named `*-PRD-*.md`) for requirements

Parse the RFC into these key sections for ticket generation:

| RFC Section | Used For |
|-------------|----------|
| **Purpose** | Epic description |
| **Responsibilities** | Scope boundaries for the epic |
| **Public Interface** | Story/task acceptance criteria (what functions to implement) |
| **tRPC / API Endpoints** | Dedicated tasks for each endpoint |
| **Data Model** | Phase 0 tasks (migrations, schema) |
| **Key Data Flows** | Implementation guidance in task descriptions |
| **Error Handling** | Task acceptance criteria (error cases to handle) |
| **Environment Variables** | Setup/config tasks + external prerequisite tasks |
| **Key Invariants** | Non-negotiable rules included in every task description |
| **Dependencies** | Task ordering and blocked-by relationships |
| **Implementation Phases** | Story breakdown (1 story per phase) |
| **Service-Specific Testing** | Test requirements per task |
| **Deferred to Post-MVP** | Explicitly excluded from tickets (mention in epic as out-of-scope) |

### Step 3: Generate the Ticket Hierarchy

Create tickets in this order (parent must exist before children):

#### 3a. Create the Epic

```
mcp__claude_ai_Atlassian__createJiraIssue(
  cloudId: "<cloud-id>",
  projectKey: "<project-key>",
  issueTypeName: "Epic",
  summary: "[Service Name]: [Purpose summary]",
  description: <see Epic Template below>,
  contentFormat: "markdown"
)
```

#### 3b. Create Stories (one per Implementation Phase)

For each phase in the RFC's "Implementation Phases" table, create a Story:

```
mcp__claude_ai_Atlassian__createJiraIssue(
  cloudId: "<cloud-id>",
  projectKey: "<project-key>",
  issueTypeName: "Story",
  summary: "[Service Name] Phase [N]: [Phase Description]",
  description: <see Story Template below>,
  parent: "<epic-key>",
  contentFormat: "markdown"
)
```

#### 3c. Create Tasks (subtasks under each Story)

Break each phase into discrete, implementable tasks. Each task should be completable in a single Claude session.

```
mcp__claude_ai_Atlassian__createJiraIssue(
  cloudId: "<cloud-id>",
  projectKey: "<project-key>",
  issueTypeName: "Task",
  summary: "[Action verb]: [specific deliverable]",
  description: <see Task Template below>,
  parent: "<story-key>",
  contentFormat: "markdown"
)
```

### Step 4: Report Results

After all tickets are created, output a summary table:

```markdown
## Tickets Created

| Key | Type | Summary | Parent |
|-----|------|---------|--------|
| OBS-1 | Epic | Multi-Tenancy Core: Tenant isolation and lifecycle management | — |
| OBS-2 | Story | Multi-Tenancy Core Phase 0: Schema and migrations | OBS-1 |
| OBS-3 | Task | Add SaaS columns to organizations table via Prisma migration | OBS-2 |
| ... | ... | ... | ... |
```

---

## Ticket Templates

### Epic Template

```markdown
## Purpose

[Copy from RFC § Purpose]

## Responsibilities

[Copy from RFC § Responsibilities, both "what Langfuse already does" and "what this module adds"]

## Public Interface

[Copy from RFC § Public Interface — list all exported service methods with signatures]

## Scope

**In scope (MVP):**
- [Derived from Implementation Phases]

**Out of scope (deferred):**
- [Copy from RFC § Deferred to Post-MVP]

## Key Invariants

[Copy from RFC § Key Invariants — these are non-negotiable rules that apply to ALL tasks in this epic]

## Dependencies

[Copy from RFC § Dependencies — upstream and downstream modules]

## Architecture Reference

- Module location: [from RFC header, e.g., `packages/shared/src/saas/multi-tenancy/`]
- tRPC router: [from RFC header, e.g., `web/src/server/api/routers/saas/multi-tenancy.ts`]
- Reference docs: `CLAUDE.md`, `.specs/CONSTITUTION.md`, `.specs/stack.md`, `.specs/structure.md`

## Scale Constraints

[Copy from RFC § Scale Constraints]

## External Prerequisites

[Copy from RFC § Environment Variables > "External prerequisites" subsection. These are setup steps that must be completed OUTSIDE the codebase before implementation can begin.]

Examples of external prerequisites:
- Stripe: Create products, prices, and webhook endpoints in the Stripe Dashboard
- SendGrid: Configure sender identity, create API key, set up email templates
- AWS: Provision RDS, ElastiCache, S3 buckets, ECS clusters via Terraform
- PagerDuty: Create services, escalation policies, and integration keys
- ClickHouse: Apply schema migrations before ingestion begins
- DNS: Configure domain records for the application

**IMPORTANT:** These prerequisites generate dedicated "External Setup" tasks in Phase 0 (see Task Decomposition Guidelines). Each task includes step-by-step instructions for configuring the external service, what credentials/IDs to capture, and where to store them (environment variables, AWS Secrets Manager).
```

### Story Template

```markdown
## Phase [N]: [Phase Description]

**Depends on:** [From RFC Implementation Phases "Depends On" column]

## Objective

[1-2 sentences describing what this phase delivers, derived from the RFC phase description and the relevant sections of the RFC]

## Deliverables

- [ ] [List each concrete deliverable: files created, functions implemented, migrations applied]

## Key Context from RFC

[Include relevant excerpts from Data Model, Key Data Flows, Error Handling sections that apply to this phase]

## Acceptance Criteria

- [ ] All deliverables implemented
- [ ] Unit tests written and passing
- [ ] Integration tests written and passing (especially cross-tenant isolation tests if applicable)
- [ ] Code passes lint (`pnpm run format` + `pnpm tc`)
- [ ] Application runs locally without errors (`pnpm run dev:web`)
- [ ] Manual validation completed (see validation steps in tasks)
```

### Task Template (Most Important)

Each task MUST be detailed enough for Claude to implement it autonomously. Use this structure:

````markdown
## Objective

[1-2 sentences: what this task delivers and why it matters]

## Context

**RFC:** [path to the RFC file]
**Module:** [e.g., `packages/shared/src/saas/multi-tenancy/`]
**Reference docs to read before starting:**
- `CLAUDE.md` (root) — project conventions, dev commands, technology stack
- `.specs/CONSTITUTION.md` — inviolable rules, forbidden patterns
- `.specs/stack.md` — engineering standards, error handling patterns, testing strategy
- `.specs/structure.md` — where files go, naming conventions
- [RFC file path] — full service specification

**Key Invariants (from RFC — non-negotiable):**
- [List the specific invariants from the RFC that apply to this task]
- [e.g., "ALL PostgreSQL queries MUST include WHERE org_id in the where clause"]
- [e.g., "Never accept org_id from user input — derive from JWT or API key"]

**Dependencies:**
- Blocked by: [ticket key(s) or "none"]
- Blocks: [ticket key(s) or "none"]

## Implementation Steps

### 1. Create feature branch

```bash
git checkout -b feat/[ticket-key]-[short-description] main
```

### 2. Write failing tests first (TDD — mandatory per CONSTITUTION)

**Test file location:** [exact path, e.g., `web/src/__tests__/multi-tenancy.test.ts`]

Write tests that cover:
- [Specific test case 1 — happy path]
- [Specific test case 2 — error case from RFC § Error Handling]
- [Specific test case 3 — tenant isolation / security]
- [Any service-specific tests from RFC § Service-Specific Testing]

```bash
# Run tests to confirm they fail
pnpm test --testPathPatterns="[test-file-pattern]"
```

### 3. Implement the solution

**Files to create/modify:**
- [Exact file path 1] — [what to add/change]
- [Exact file path 2] — [what to add/change]

**Implementation details:**
[Specific instructions derived from the RFC sections: Data Model for schema, Public Interface for function signatures, Key Data Flows for logic, Error Handling for error cases]

**Code patterns to follow:**
- [e.g., "Use Prisma typed client methods, not $queryRaw"]
- [e.g., "Import Zod from 'zod/v4' for input validation"]
- [e.g., "Follow error format from stack.md: { error, error_description, statusCode }"]
- [e.g., "Add JSDoc/TSDoc comments to all public functions"]

### 4. Run tests and fix until green

```bash
# Run the specific tests
pnpm test --testPathPatterns="[test-file-pattern]"

# Run full typecheck
pnpm tc

# Run linter/formatter
pnpm run format
```

If tests fail, analyze the error output, fix the implementation, and re-run. Repeat until all tests pass.

### 5. Manual validation

Start the application locally and verify the implementation works end-to-end:

```bash
# Start infrastructure (if not already running)
pnpm run infra:dev:up

# Start the web application
pnpm run dev:web
```

**Manual validation steps:**
- [ ] [Step 1: e.g., "Log in at localhost:3000 with demo@langfuse.com / password"]
- [ ] [Step 2: e.g., "Navigate to the project and verify the new setting appears"]
- [ ] [Step 3: e.g., "Use curl/httpie to hit the endpoint and verify the response"]
- [ ] [Step 4: e.g., "Check the database to confirm the data was written correctly"]

```bash
# Example validation commands
# Check database state
psql $DATABASE_URL -c "SELECT * FROM organizations WHERE id = '...' LIMIT 5;"

# Hit an API endpoint
curl -s http://localhost:3000/api/... | jq .
```

### 6. Final checks before PR

```bash
# Full typecheck
pnpm tc

# Format code
pnpm run format

# Run ALL tests (not just the ones you wrote)
pnpm test

# Verify the build succeeds
pnpm build:check
```

## Acceptance Criteria

- [ ] Tests written BEFORE implementation (TDD)
- [ ] All tests passing
- [ ] Typecheck passes (`pnpm tc`)
- [ ] Lint/format passes (`pnpm run format`)
- [ ] Manual validation steps completed successfully
- [ ] No `any` types used
- [ ] No imports from `web/src/ee/` or `worker/src/ee/`
- [ ] All queries include tenant-scoping (org_id or project_id in WHERE clause)
- [ ] No secrets hardcoded
- [ ] JSDoc/TSDoc on all public functions
- [ ] [Any task-specific acceptance criteria from the RFC]

## Definition of Done

This task is complete when:
1. All acceptance criteria are checked off
2. A PR is created with the changes
3. PR follows the template in `.github/PULL_REQUEST_TEMPLATE.md`
````

### External Setup Task Template

Some RFCs depend on external services that must be configured before code implementation can begin. These are non-code tasks (no TDD, no PR) but are critical blockers. The RFC's "Environment Variables" section lists these under "External prerequisites", and the "Dependencies" section may reference "Wraps external service". Use this template for those tasks:

````markdown
## Objective

Configure [External Service] for [module name] integration.

## Context

**RFC:** [path to the RFC file]
**Module that wraps this service:** [e.g., `packages/shared/src/saas/billing/stripe.ts`]
**Why this must happen first:** [e.g., "The billing service needs Stripe product IDs and webhook signing secrets before any code can reference them"]

## Configuration Steps

### 1. [Service] Account Setup

- [ ] [Step 1: e.g., "Log in to Stripe Dashboard at dashboard.stripe.com"]
- [ ] [Step 2: e.g., "Switch to Test Mode for development"]

### 2. Create Required Resources

- [ ] [Resource 1: e.g., "Create Product: 'AI Observability Platform'"]
- [ ] [Resource 2: e.g., "Create Price: Free tier ($0/month, metered usage)"]
- [ ] [Resource 3: e.g., "Create Price: Starter tier ($49/month, metered usage)"]
- [ ] [Resource 4: e.g., "Create Price: Pro tier ($199/month, metered usage)"]
- [ ] [Resource 5: e.g., "Create Webhook endpoint: https://your-domain/api/webhooks/stripe"]
  - Events to subscribe: [list specific events, e.g., "customer.subscription.created, customer.subscription.updated, customer.subscription.deleted, invoice.payment_succeeded, invoice.payment_failed"]

### 3. Capture Credentials and IDs

Record the following values (these become environment variables):

| Value | Where to Find | Env Variable |
|-------|--------------|--------------|
| [e.g., API Secret Key] | [e.g., Stripe Dashboard > Developers > API Keys] | `STRIPE_SECRET_KEY` |
| [e.g., Webhook Signing Secret] | [e.g., Stripe Dashboard > Developers > Webhooks > Signing secret] | `STRIPE_WEBHOOK_SECRET` |
| [e.g., Free Price ID] | [e.g., Stripe Dashboard > Products > Free > Price ID] | `STRIPE_PRICE_FREE` |

### 4. Store Credentials

- [ ] Add values to `.env` for local development (copy from `.env.dev.example`)
- [ ] For production: store in AWS Secrets Manager
- [ ] NEVER commit credentials to the repository

### 5. Verify Setup

- [ ] [Verification step: e.g., "Run a test API call to confirm the key works"]
- [ ] [Verification step: e.g., "Send a test webhook event from Stripe Dashboard and confirm it arrives"]

## Acceptance Criteria

- [ ] All required resources created in [Service]
- [ ] All credentials captured and stored in `.env` (local) and AWS Secrets Manager (production)
- [ ] Test API call or webhook confirms connectivity
- [ ] No credentials committed to git

## Common External Services and What to Configure

For reference, here are typical setup requirements for services used in this project:

| Service | What to Configure | Key Outputs |
|---------|-------------------|-------------|
| **Stripe** | Products, Prices (per plan tier), Webhook endpoint, Customer portal settings | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, Price IDs per plan |
| **Twilio SendGrid** | Sender identity verification, API key, email templates (welcome, quota warning, invoice), domain authentication (DNS records for DKIM/SPF) | `SENDGRID_API_KEY`, `SENDGRID_FROM_EMAIL`, template IDs |
| **AWS RDS** | PostgreSQL instance (Multi-AZ), security groups, parameter groups | `DATABASE_URL` connection string |
| **AWS ElastiCache** | Redis cluster, security groups, encryption in transit | `REDIS_URL` connection string |
| **AWS S3** | Buckets for backups and exports, lifecycle policies, encryption | `S3_BUCKET_NAME`, `S3_REGION` |
| **AWS ECS** | Cluster, task definitions, service configuration | Cluster ARN, service names |
| **PagerDuty** | Service, escalation policy, integration key (Events API v2) | `PAGERDUTY_ROUTING_KEY` |
| **Sentry** | Project, DSN, release tracking setup | `SENTRY_DSN` |
| **ClickHouse** | Version verification (>=25.1.5.5 for CVE patches), disable library_bridge, user/password | `CLICKHOUSE_URL`, `CLICKHOUSE_USER`, `CLICKHOUSE_PASSWORD` |
| **Cloudflare/Route 53** | DNS records, SSL certificates | Domain configuration |
````

---

## Task Decomposition Guidelines

When breaking phases into tasks, follow these rules:

### What makes a good task

- **One concern per task**: Don't mix schema migrations with business logic
- **Completable in one session**: A task should take 1-4 hours for Claude, not days
- **Independently testable**: Each task should have its own test cases
- **Clear start and end**: The implementer knows exactly when the task is done

### Standard task patterns per RFC section

| RFC Content | Task Type | Example |
|-------------|-----------|---------|
| Data Model (PostgreSQL) | Schema migration | "Add SaaS columns to organizations table via Prisma migration" |
| Data Model (ClickHouse) | Schema migration | "Add org_id column and bloom filter indexes to ClickHouse tables" |
| Data Model (Redis) | Config/setup | "Define Redis key patterns for usage counters" |
| Public Interface (each method) | Implementation | "Implement createOrg() service method with tests" |
| tRPC Endpoints (each endpoint) | Implementation | "Add saas.billing.getSubscription tRPC procedure" |
| API Routes (webhooks) | Implementation | "Implement Stripe webhook handler at /api/webhooks/stripe" |
| Error Handling (each error type) | Included in the function task | Folded into the service method task, not a separate task |
| Environment Variables | Config/setup | "Add environment variable validation for [service] config" |
| Environment Variables > External prerequisites | External setup | "Configure Stripe products, prices, and webhook endpoint" |
| Key Data Flows | Implementation guidance | Referenced in task descriptions, not a separate task |
| Service-Specific Testing | Testing | "Add cross-tenant isolation integration tests" |

### Task ordering rules

1. **External setup tasks come first** in Phase 0 (configure Stripe, SendGrid, AWS, PagerDuty, etc. before any code depends on them)
2. **Schema/migration tasks follow external setup** within a phase
3. **Environment variable/config tasks** go in Phase 0 (after external setup, since env vars often contain credentials from external services)
4. **Service method tasks follow schema tasks** (they depend on the schema)
5. **tRPC/API route tasks come after service methods** (they call service methods)
6. **Integration test tasks come last** (they test the full flow)

### Sizing guidance

- **Too small**: "Add an import statement" (merge into the task that needs it)
- **Just right**: "Implement createOrg() service method with unit tests" (clear scope, testable)
- **Too large**: "Implement the entire billing service" (break into per-method tasks)

---

## Example: Multi-Tenancy Core RFC Decomposition

Given the Multi-Tenancy Core RFC with 3 implementation phases:

### Epic
- **Summary**: Multi-Tenancy Core: Tenant isolation and lifecycle management
- **Type**: Epic

### Phase 0 Story + Tasks
- **Story**: Multi-Tenancy Core Phase 0: Schema, migrations, and external setup
  - **Task 1**: Verify ClickHouse version meets security requirements (>=25.1.5.5) and disable library_bridge
  - **Task 2**: Add SaaS columns (slug, status, settings) to organizations table via Prisma migration
  - **Task 3**: Add org_id column and bloom filter indexes to ClickHouse tables (traces, observations, scores)
  - **Task 4**: Add environment variable validation for ClickHouse connection config

### Phase 1 Story + Tasks
- **Story**: Multi-Tenancy Core Phase 1: Core service methods
  - **Task 1**: Implement createOrg() service method with unit tests
  - **Task 2**: Implement getOrg() and getOrgSettings() service methods with unit tests
  - **Task 3**: Implement updateOrgSettings() with Zod validation and unit tests
  - **Task 4**: Implement deleteOrg() soft-delete with unit tests
  - **Task 5**: Add deleted-org filtering to NextAuth session callback

### Phase 2 Story + Tasks
- **Story**: Multi-Tenancy Core Phase 2: Integration and end-to-end validation
  - **Task 1**: Implement tenant context injection into request scope
  - **Task 2**: Add cross-tenant isolation integration tests (PostgreSQL)
  - **Task 3**: Add cross-tenant isolation integration tests (ClickHouse)
  - **Task 4**: End-to-end validation of all exported service methods

---

## Error Handling

If the RFC is missing required sections:
- Warn the user about which sections are missing
- Generate tickets for the sections that ARE present
- Add a "Resolve open questions" task if the RFC has unresolved open questions

If JIRA API calls fail:
- Report the error clearly
- Continue creating remaining tickets
- Provide a summary of what was created vs. what failed

If the RFC references dependencies on other services that don't have RFCs yet:
- Note the dependency in the task description
- Add a blocker note: "Blocked by: [Service Name] RFC must be finalized"
