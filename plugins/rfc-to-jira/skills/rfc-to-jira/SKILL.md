---
name: rfc-to-jira
description: This skill should be used when converting RFC specification documents into JIRA tickets. Triggers on requests like "create JIRA tickets from this RFC", "break down this spec into tickets", "/rfc-to-jira [path-to-rfc]", or when users want to decompose an RFC into implementable work items that Claude can pick up and implement end-to-end.
---

# RFC to JIRA Ticket Generator

Convert RFC specification documents into structured, implementation-ready JIRA tickets. Each ticket contains enough context and detail for Claude Code to implement it end-to-end, including a built-in validation loop (write tests first, implement, run tests, validate locally).

## Overview

This skill reads an RFC document following the standard 16-section RFC template (plus any non-standard sections), analyzes its structure, reads the companion PRD for EARS requirements, and produces a hierarchy of JIRA tickets:

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

### Step 2: Read and Analyze the RFC + PRD

Read the RFC file and extract ALL sections (both standard and non-standard). Also read the companion files that provide implementation context:

```
Read(file_path="<path-to-rfc>")
```

Also read these supporting documents for cross-cutting context:
- The project's `CLAUDE.md` (root of repo) for conventions, dev commands, and guidelines
- `.specs/CONSTITUTION.md` for inviolable rules and forbidden patterns
- `.specs/stack.md` for technology stack and engineering standards
- `.specs/structure.md` for repository layout and file conventions
- `.specs/architecture-overview.md` for system architecture and module interactions
- The companion PRD file (same directory as the RFC, named `*-PRD-*.md`) for EARS requirements

#### 2a. Parse Standard RFC Sections

| RFC Section | Used For |
|-------------|----------|
| **Purpose** | Epic description |
| **Responsibilities** | Scope boundaries for the epic |
| **Public Interface** | Story/task acceptance criteria (what functions to implement) |
| **tRPC / API Endpoints** | Dedicated tasks for each endpoint |
| **Data Model** | Phase 0 tasks (migrations, schema, inline decision tables preserved verbatim) |
| **Key Data Flows** | Implementation guidance in task descriptions; complex flows (e.g., each webhook event type) become separate tasks |
| **Error Handling** | Task acceptance criteria (error cases to handle) |
| **Environment Variables** | Setup/config tasks + external prerequisite tasks |
| **Key Invariants** | Non-negotiable rules included in every task description (Lua scripts, code snippets preserved verbatim) |
| **Dependencies** | Task ordering, blocked-by relationships, cross-epic sequencing |
| **Implementation Phases** | Story breakdown (1 story per phase; further decompose under-specified phases) |
| **Service-Specific Testing** | Test requirements per task |
| **Deferred to Post-MVP** | Explicitly excluded from tickets (mentioned in epic as out-of-scope) |

#### 2b. Parse Non-Standard RFC Sections

RFCs frequently include sections beyond the 16-section template. These MUST be parsed and mapped to tickets. Common non-standard sections:

| Non-Standard Section | How to Handle |
|----------------------|---------------|
| **Langfuse Baseline** | Extract as "DO NOT implement" guardrails. Include in Epic and every task to prevent duplicating existing functionality |
| **PostgreSQL Tenant Isolation Strategy** | Include as implementation guidance in every task that writes Prisma queries |
| **ClickHouse Tenant Isolation** | Generate tasks for row-policy setup, CVE version checks, CI pipeline checks |
| **Dunning Policy** | Generate external setup task (Stripe Smart Retries config) + dedicated implementation task for dunning state machine |
| **Late Payment / `past_due` Behavior** | Generate dedicated task with timeline-based test cases |
| **Alert Rules (Critical/Warning/Info)** | Generate one task per alert tier with specific alarm definitions |
| **MVP Implementation Approach** | Use as an alternative/supplement to Implementation Phases for story breakdown |
| **Runbook Stubs** | Generate a documentation task for creating runbook stubs |

#### 2c. Map PRD EARS Requirements to Acceptance Criteria

The companion PRD contains requirements in EARS notation: `WHEN [trigger] THE SYSTEM SHALL [behaviour]`. These are directly usable as acceptance criteria.

For each PRD requirement (REQ-N):
1. Identify which task implements that requirement
2. Convert the EARS criterion to a Given/When/Then acceptance criterion on that task
3. Pay special attention to:
   - **Security constraints** (e.g., "welcome email MUST NOT include API keys")
   - **Performance SLAs** (e.g., "enqueue and return within 50ms")
   - **Negative path behavior** (e.g., "return 401 if context cannot be derived")
   - **PRD Constraints section** (often stricter than RFC, e.g., "`$queryRaw` forbidden" vs. RFC's "SHOULD use Prisma")

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

For each phase in the RFC's "Implementation Phases" table, create a Story. If a phase is under-decomposed (contains 4+ distinct deliverables), split it into multiple stories:

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

#### 3d. Link Cross-Epic Dependencies

After all tickets are created, add blocking links between epics when one module depends on another being completed first. Use `mcp__claude_ai_Atlassian__createIssueLink` with link type "Blocks".

Common cross-epic dependencies:
- Multi-Tenancy Core blocks ALL other epics (it provides org_id infrastructure)
- Billing blocks Entitlement & Quota (pushes plan limits)
- Entitlement & Quota and Billing have a bidirectional dependency (document the fallback path in both epics)
- Notification is consumed by Billing and Entitlement (but can be stubbed)

### Step 4: Report Results

After all tickets are created, output a summary table:

```markdown
## Tickets Created

| Key | Type | Summary | Parent | Blocked By |
|-----|------|---------|--------|------------|
| OBS-1 | Epic | Multi-Tenancy Core: Tenant isolation and lifecycle management | — | — |
| OBS-2 | Story | Multi-Tenancy Core Phase 0: Schema and migrations | OBS-1 | — |
| OBS-3 | Task | Add SaaS columns to organizations table via Prisma migration | OBS-2 | — |
| ... | ... | ... | ... | ... |

## Cross-Epic Dependencies

| Source Epic | Blocks | Reason |
|-------------|--------|--------|
| OBS-1 (Multi-Tenancy) | OBS-10 (Billing), OBS-20 (Entitlement) | Provides org_id infrastructure |
| ... | ... | ... |
```

---

## Ticket Templates

### Epic Template

```markdown
## Purpose

[Copy from RFC § Purpose]

## Responsibilities

[Copy from RFC § Responsibilities, both "what Langfuse already does" and "what this module adds"]

## Langfuse Baseline (DO NOT Implement)

[If the RFC has a "Langfuse Baseline" section, copy it here. These capabilities already exist in Langfuse upstream and MUST NOT be re-built. Every task in this epic must respect these boundaries.]

- [e.g., "Organization model and RBAC already exist — do not create new org tables"]
- [e.g., "protectedOrganizationProcedure already injects orgId into tRPC context"]
- [e.g., "ClickHouse queries already filter by project_id"]

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

**Cross-epic sequencing:**
- [e.g., "This epic MUST be completed before Billing, Entitlement, Provisioning, Notification epics can begin"]
- [e.g., "Bidirectional dependency with Entitlement — document the fallback cache-miss path"]

## Glossary

[Define every domain term used in this epic so the implementer does not have to guess:]

| Term | Definition |
|------|-----------|
| [e.g., org_id] | [The organization identifier, derived from JWT or API key lookup, never from user input] |
| [e.g., tenant] | [An organization with an active subscription; maps to the `organizations` table] |
| [e.g., plan limits] | [Trace count thresholds per billing period, pushed by Billing via `updatePlanLimits()`] |

## Architecture Reference

- Module location: [from RFC header, e.g., `packages/shared/src/saas/multi-tenancy/`]
- tRPC router: [from RFC header, or "None — this module has no tRPC endpoints" if applicable]
- Reference docs: `CLAUDE.md`, `.specs/CONSTITUTION.md`, `.specs/stack.md`, `.specs/structure.md`

## External Prerequisites

[Copy from RFC § Environment Variables > "External prerequisites" subsection. These are setup steps that must be completed OUTSIDE the codebase before implementation can begin.]

Examples of external prerequisites:
- Stripe: Create products, prices, and webhook endpoints in the Stripe Dashboard
- SendGrid: Configure sender identity, create API key, set up email templates, domain authentication (DKIM/SPF DNS records)
- AWS: Provision RDS, ElastiCache, S3 buckets, ECS clusters via Terraform
- PagerDuty: Create services, escalation policies, and integration keys
- Sentry: Create project, capture DSN, configure release tracking in CI/CD
- ClickHouse: Apply schema migrations before ingestion begins
- DNS: Configure domain records for the application

**IMPORTANT:** These prerequisites generate dedicated "External Setup" tasks in Phase 0 (see Task Decomposition Guidelines). Each task includes step-by-step instructions for configuring the external service, what credentials/IDs to capture, and where to store them (environment variables, AWS Secrets Manager).

## Scale Constraints

[Copy from RFC § Scale Constraints]
```

### Story Template

```markdown
## Phase [N]: [Phase Description]

**Depends on:** [From RFC Implementation Phases "Depends On" column]

## User Story

As a [persona derived from the RFC context], I want [phase objective] so that [business value from RFC Purpose].

## Objective

[1-2 sentences describing what this phase delivers, derived from the RFC phase description and the relevant sections of the RFC]

## Deliverables

- [ ] [List each concrete deliverable: files created, functions implemented, migrations applied]

## Key Context from RFC

[Include relevant excerpts from Data Model, Key Data Flows, Error Handling sections that apply to this phase]

## Langfuse Baseline Guardrails

[If the RFC has a "Langfuse Baseline" section, list what this phase must NOT re-implement]

## PRD Requirements Covered

[List which PRD REQ-N items are delivered by this phase]

## Acceptance Criteria

- [ ] All deliverables implemented
- [ ] Unit tests written and passing (omit for external-setup-only phases)
- [ ] Integration tests written and passing (especially cross-tenant isolation tests if applicable)
- [ ] Code passes lint (`pnpm run format` + `pnpm tc`) (omit for external-setup-only phases)
- [ ] Application runs locally without errors (`pnpm run dev:web`) (omit for external-setup-only phases)
- [ ] Manual validation completed (see validation steps in tasks)
```

### Task Template (Most Important)

Each task MUST be detailed enough for Claude to implement it autonomously. Use this structure:

````markdown
## Objective

[1-2 sentences: what this task delivers and why it matters]

## User Story

As a [persona], I want [action] so that [benefit].

## Context

**RFC:** [path to the RFC file]
**PRD:** [path to the companion PRD file]
**Module:** [e.g., `packages/shared/src/saas/multi-tenancy/`]
**Reference docs to read before starting:**
- `CLAUDE.md` (root) — project conventions, dev commands, technology stack
- `.specs/CONSTITUTION.md` — inviolable rules, forbidden patterns
- `.specs/stack.md` — engineering standards, error handling patterns, testing strategy
- `.specs/structure.md` — where files go, naming conventions
- [RFC file path] — full service specification

**Pattern to follow:** [Link to an existing file in the codebase that demonstrates the pattern this task should follow, e.g., "Follow the pattern in `web/src/features/public-api/server/RateLimitService.ts`" or "Follow the tRPC router pattern in `web/src/server/api/routers/traces.ts`"]

**Langfuse Baseline (DO NOT re-implement):**
- [List specific Langfuse functionality this task must use, not rebuild]
- [e.g., "Use Langfuse's existing `protectedOrganizationProcedure` — do not create a new auth middleware"]

**Key Invariants (from RFC — non-negotiable):**
- [List the specific invariants from the RFC that apply to this task]
- [e.g., "ALL PostgreSQL queries MUST include WHERE org_id in the where clause"]
- [e.g., "Never accept org_id from user input — derive from JWT or API key"]
- [Preserve verbatim any code snippets, Lua scripts, or SQL from the RFC invariants]

**Out of scope (do NOT implement in this task):**
- [Explicit list of adjacent work handled by other tickets]
- [e.g., "tRPC endpoint for this method is in task OBS-15, not here"]

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
**Test runner:** [Jest for web/ tests, Vitest for worker/ tests]

Write tests that cover:
- [Specific test case 1 — happy path, in Given/When/Then format]
- [Specific test case 2 — error case from RFC § Error Handling, in Given/When/Then format]
- [Specific test case 3 — tenant isolation / security, in Given/When/Then format]
- [Any service-specific tests from RFC § Service-Specific Testing]
- [PRD EARS criteria mapped to this task, converted to Given/When/Then]

**Example test cases (Given/When/Then):**
```
Given an authenticated user with orgId "org-1"
When they call createOrg("My Org", "user-1")
Then a new organization is created with status "active" and the user is set as OWNER

Given an authenticated user with orgId "org-1"
When they call getOrg("org-2") (a different org)
Then an OrgNotFoundError is returned (tenant isolation enforced)
```

```bash
# Run tests to confirm they fail
pnpm test --testPathPatterns="[test-file-pattern]"
```

### 3. Implement the solution

**Files to create/modify:**
- [Exact file path 1] — [what to add/change]
- [Exact file path 2] — [what to add/change]

**CAUTION — Existing Langfuse file modification:**
[If this task modifies one of the ~6 permitted existing Langfuse files, flag it explicitly:]
- This task modifies `[file path]`, which is an existing Langfuse file
- Record this modification in the rebase checklist for future upstream merges
- Make the smallest possible change to minimize rebase conflicts

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

[For internal-only modules with no HTTP/tRPC endpoints, replace browser steps with programmatic validation:]
- [ ] [e.g., "Run a test script that calls the service method directly"]
- [ ] [e.g., "Check CloudWatch metrics via `aws cloudwatch get-metric-data`"]
- [ ] [e.g., "Verify Redis state via `redis-cli GET plan:{orgId}`"]

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

**Functional (from RFC + PRD):**
- [ ] [Given/When/Then criterion 1 from PRD EARS mapping]
- [ ] [Given/When/Then criterion 2 from PRD EARS mapping]
- [ ] [Given/When/Then criterion 3 — at least one negative/error path]
- [ ] [Any task-specific acceptance criteria from the RFC]

**Cross-cutting checklist (every code task):**
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
- [ ] Error responses follow stack.md format: `{ error, error_description, statusCode }`
- [ ] Structured logging used (no `console.log`)

## Definition of Done

This task is complete when:
1. All acceptance criteria are checked off
2. A PR is created with the changes
3. PR follows the template in `.github/PULL_REQUEST_TEMPLATE.md`
````

### BullMQ Job Task Template

Several RFCs define BullMQ repeatable/cron jobs (usage sync, downgrade checks, queue-depth metrics). These are distinct from service methods and API routes. Use this template:

````markdown
## Objective

Implement the `[job name]` BullMQ repeatable job that [what it does].

## Context

**RFC:** [path to the RFC file]
**Worker location:** `worker/src/saas/[job-file].ts`
**Queue registration:** `packages/shared/src/server/queues.ts` (existing Langfuse file — rebase-sensitive)
**Worker entrypoint:** `worker/src/app.ts` (existing Langfuse file — rebase-sensitive)
**Test runner:** Vitest (worker tests use Vitest, not Jest)

**Schedule:** [e.g., "Every hour", "Daily at 00:00 UTC", "Every 1 minute"]
**BullMQ repeat config:** `{ pattern: '[cron expression]' }` or `{ every: [milliseconds] }`

## Implementation Steps

### 1. Create feature branch

```bash
git checkout -b feat/[ticket-key]-[short-description] main
```

### 2. Define the queue

Add the queue definition to `packages/shared/src/server/queues.ts`:
- Queue name: `[queue-name]`
- This modifies an existing Langfuse file — record in rebase checklist

### 3. Write failing tests first

**Test file:** `worker/src/__tests__/[job-name].test.ts`
**Test runner:** Vitest

```bash
# Run worker tests
pnpm run test --filter=worker -- [job-name] -t "[test description]"
```

Write tests that cover:
- [Happy path: job processes successfully]
- [Error path: external service unavailable — job retries]
- [Isolation: job processes only the target tenant's data]
- [Idempotency: running the job twice produces the same result]
- [PRD EARS criteria for this job, if any]

### 4. Implement the job processor

**Files to create/modify:**
- `worker/src/saas/[job-file].ts` — job processor function
- `packages/shared/src/server/queues.ts` — queue definition (EXISTING FILE)
- `worker/src/app.ts` — register processor (EXISTING FILE)

**Job processor pattern:**
```typescript
// worker/src/saas/[job-file].ts
import { Job } from "bullmq";

export async function process[JobName](job: Job) {
  const { orgId } = job.data;
  if (!orgId) throw new Error("Missing orgId in job payload");
  // ... implementation
}
```

**Registration in worker entrypoint:**
```typescript
// In worker/src/app.ts, add:
import { process[JobName] } from "./saas/[job-file]";
// Register with the queue worker
```

### 5. Run tests and validate

```bash
pnpm run test --filter=worker -- [job-name]
pnpm tc
pnpm run format
```

### 6. Manual validation

```bash
pnpm run dev:worker
# Verify job executes on schedule by checking worker logs
# Verify side effects (Redis state, DB records, external API calls)
```

## Acceptance Criteria

- [ ] Queue defined in `packages/shared/src/server/queues.ts`
- [ ] Processor registered in `worker/src/app.ts`
- [ ] Job includes orgId in payload and rejects jobs without it
- [ ] Repeatable schedule configured correctly
- [ ] Tests passing (Vitest)
- [ ] Partial failure in one tenant does not block other tenants
- [ ] [PRD criteria in Given/When/Then format]
````

### Terraform / Infrastructure Task Template

Some RFCs (especially Monitoring & Alerting) require infrastructure provisioned via Terraform, not application code. These tasks have no TDD loop, no `pnpm test`, and are validated via `terraform plan` and AWS CLI.

````markdown
## Objective

Provision [infrastructure resource] via Terraform for [module name].

## Context

**RFC:** [path to the RFC file]
**Terraform directory:** [e.g., `infra/terraform/modules/monitoring/`]
**Key invariant:** "All AWS and PagerDuty infrastructure MUST be provisioned via Terraform — never via the AWS Console"

## Implementation Steps

### 1. Create feature branch

```bash
git checkout -b feat/[ticket-key]-[short-description] main
```

### 2. Write Terraform configuration

**Files to create/modify:**
- [e.g., `infra/terraform/modules/monitoring/cloudwatch-alarms.tf`]
- [e.g., `infra/terraform/modules/monitoring/pagerduty.tf`]
- [e.g., `infra/terraform/modules/monitoring/variables.tf`]

**Resources to define:**
- [e.g., "aws_cloudwatch_metric_alarm for each Critical-tier alert"]
- [e.g., "pagerduty_service with escalation policy"]

**Alert thresholds MUST be configurable via Terraform variables, not hardcoded.**

### 3. Validate

```bash
cd infra/terraform
terraform fmt
terraform validate
terraform plan -out=plan.out
```

Review the plan output to confirm:
- [ ] Correct resources will be created
- [ ] No unintended modifications to existing resources
- [ ] Variables are parameterized (no hardcoded thresholds)

### 4. Apply (with approval)

```bash
# Only after human review of the plan
terraform apply plan.out
```

### 5. Verify

```bash
# Verify CloudWatch alarms
aws cloudwatch describe-alarms --alarm-names "[alarm-name]"

# Verify PagerDuty service
# Check PagerDuty Dashboard or use PagerDuty API

# Verify CloudWatch dashboard
aws cloudwatch get-dashboard --dashboard-name "[dashboard-name]"
```

## Acceptance Criteria

- [ ] Terraform config passes `terraform validate` and `terraform fmt`
- [ ] `terraform plan` shows only intended changes
- [ ] All thresholds are Terraform variables (not hardcoded)
- [ ] Resources provisioned and verified via AWS CLI / provider dashboard
- [ ] No manual console changes (everything in Terraform state)
````

### External Setup Task Template

Some RFCs depend on external services that must be configured before code implementation can begin. These are non-code tasks (no TDD, no PR) but are critical blockers. The RFC's "Environment Variables" section lists these under "External prerequisites", and the "Dependencies" section may reference "Wraps external service". Use this template:

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

| Value | Where to Find | Env Variable | Test vs. Live |
|-------|--------------|--------------|---------------|
| [e.g., API Secret Key] | [e.g., Stripe Dashboard > Developers > API Keys] | `STRIPE_SECRET_KEY` | Separate keys per environment |
| [e.g., Webhook Signing Secret] | [e.g., Stripe Dashboard > Developers > Webhooks > Signing secret] | `STRIPE_WEBHOOK_SECRET` | Separate per environment |
| [e.g., Free Price ID] | [e.g., Stripe Dashboard > Products > Free > Price ID] | `STRIPE_PRICE_FREE` | Same in test and live |

### 4. Store Credentials

- [ ] Add values to `.env` for local development (copy from `.env.dev.example`)
- [ ] For production: store in AWS Secrets Manager
- [ ] NEVER commit credentials to the repository
- [ ] Verify test and production credentials are strictly separated

### 5. Verify Setup

- [ ] [Verification step: e.g., "Run a test API call to confirm the key works"]
- [ ] [Verification step: e.g., "Send a test webhook event from Stripe Dashboard and confirm it arrives"]

## Acceptance Criteria

- [ ] All required resources created in [Service]
- [ ] All credentials captured and stored in `.env` (local) and AWS Secrets Manager (production)
- [ ] Test and production credentials are separate
- [ ] Test API call or webhook confirms connectivity
- [ ] No credentials committed to git

## Common External Services and What to Configure

For reference, here are typical setup requirements for services used in this project:

| Service | What to Configure | Key Outputs |
|---------|-------------------|-------------|
| **Stripe** | Products, Prices (per plan tier), Webhook endpoint, Customer portal settings, Smart Retries / dunning config | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, Price IDs per plan |
| **Twilio SendGrid** | Sender identity verification, API key, email templates (welcome, quota warning, invoice), domain authentication (DKIM/SPF DNS records), DMARC policy (`p=none` initially), disable click tracking on billing/security templates | `SENDGRID_API_KEY`, `SENDGRID_FROM_EMAIL`, template IDs |
| **AWS RDS** | PostgreSQL instance (Multi-AZ), security groups, parameter groups, IAM roles | `DATABASE_URL` connection string |
| **AWS ElastiCache** | Redis cluster, security groups, encryption in transit | `REDIS_URL` connection string |
| **AWS S3** | Buckets for backups and exports, lifecycle policies, encryption | `S3_BUCKET_NAME`, `S3_REGION` |
| **AWS ECS** | Cluster, task definitions, service configuration, IAM task role with `cloudwatch:PutMetricData` | Cluster ARN, service names |
| **PagerDuty** | Service, escalation policy, integration key (Events API v2) | `PAGERDUTY_ROUTING_KEY` |
| **Sentry** | Project, DSN, release tracking setup, CI/CD `sentry-cli releases new` integration | `SENTRY_DSN` |
| **ClickHouse** | Version verification (>=25.1.5.5 for CVE patches), disable library_bridge, user/password, CI pipeline version check | `CLICKHOUSE_URL`, `CLICKHOUSE_USER`, `CLICKHOUSE_PASSWORD` |
| **Cloudflare/Route 53** | DNS records, SSL certificates | Domain configuration |
````

---

## Task Decomposition Guidelines

When breaking phases into tasks, follow these rules:

### What makes a good task (INVEST criteria)

- **Independent**: Can be implemented and merged without waiting for other in-progress tasks (except explicit blockers)
- **Negotiable**: Describes the outcome, not every implementation detail
- **Valuable**: Delivers a testable, demonstrable increment
- **Estimable**: Scope is clear enough to estimate (1-4 hours for Claude)
- **Small**: One concern per task; if it needs 7+ acceptance criteria, split it
- **Testable**: Has explicit Given/When/Then acceptance criteria including at least one negative path

### Standard task patterns per RFC section

| RFC Content | Task Type | Example |
|-------------|-----------|---------|
| Data Model (PostgreSQL) | Schema migration | "Add SaaS columns to organizations table via Prisma migration" |
| Data Model (ClickHouse) | Schema migration | "Add org_id column and bloom filter indexes to ClickHouse tables" |
| Data Model (Redis) | Config/setup | "Define Redis key patterns for usage counters" |
| Public Interface (each method) | Implementation | "Implement createOrg() service method with tests" |
| tRPC Endpoints (each endpoint) | Implementation | "Add saas.billing.getSubscription tRPC procedure" |
| API Routes (webhooks, per event type) | Implementation | "Handle invoice.payment_failed webhook event with dunning logic" |
| NextAuth event hooks | Implementation | "Add events.createUser callback for tenant provisioning trigger" |
| BullMQ repeatable/cron jobs | BullMQ job | "Implement hourly usage-sync BullMQ repeatable job" |
| BullMQ queue registration | Langfuse file mod | "Register SaasEmailQueue in queue definitions and worker entrypoint" |
| Error Handling (each error type) | Included in the function task | Folded into the service method task, not a separate task |
| Environment Variables | Config/setup | "Add environment variable validation for [service] config" |
| Environment Variables > External prerequisites | External setup | "Configure Stripe products, prices, and webhook endpoint" |
| Key Data Flows (complex multi-step) | Implementation | "Implement Redis-mediated API key reveal flow with GETDEL" |
| Terraform infrastructure (alarms, dashboards) | Terraform | "Provision CloudWatch alarms for Critical-tier alerts via Terraform" |
| Alert Rules (per tier) | Terraform | "Define Warning-tier CloudWatch alarms with configurable thresholds" |
| Runbook stubs | Documentation | "Create runbook stubs for all Critical and Warning alert categories" |
| CI/CD pipeline changes | Config | "Add Sentry release tracking to GitHub Actions deploy workflow" |
| Non-standard sections (Dunning, Late Payment) | Implementation | "Implement dunning state machine for past_due subscriptions" |

### Task ordering rules

1. **External setup tasks come first** in Phase 0 (configure Stripe, SendGrid, AWS, PagerDuty, etc. before any code depends on them)
2. **Schema/migration tasks follow external setup** within a phase
3. **Environment variable/config tasks** go in Phase 0 (after external setup, since env vars often contain credentials from external services)
4. **BullMQ queue registration** before job processor tasks (queue must exist before processor)
5. **Service method tasks follow schema tasks** (they depend on the schema)
6. **tRPC/API route tasks come after service methods** (they call service methods)
7. **Webhook handlers: one task per event type** (not one monolithic handler task)
8. **Integration test tasks come last** (they test the full flow)
9. **Terraform/infrastructure tasks** can run in parallel with code tasks if independent

### Phase decomposition rules

If an RFC phase is under-decomposed (contains 4+ distinct deliverables of different types), split it into multiple stories:
- **Example**: Notification Phase 1 has 5 send methods + BullMQ worker. Split into: Story A (BullMQ worker + queue registration), Story B (send methods 1-3), Story C (send methods 4-5 + integration tests)

If a phase contains only external setup tasks and env var config (no code), omit TDD acceptance criteria from the Story.

### Handling modules with no endpoints

Some modules (Multi-Tenancy Core, Entitlement & Quota, Monitoring & Alerting) have no tRPC or REST endpoints. For these:
- Skip the "tRPC Endpoints" task pattern
- Manual validation uses programmatic checks (Redis CLI, psql, AWS CLI) instead of browser/curl
- Note in the Epic that this module is consumed by other modules via exported service functions, not via HTTP

### Handling modifications to existing Langfuse files

When a task requires modifying one of the ~6 permitted existing Langfuse files, flag it explicitly:
- Mark the task title with a prefix: "[Langfuse mod]"
- Include in the task description: "This task modifies an existing Langfuse file — record in rebase checklist"
- List the specific file and the minimal change required
- The affected files across all RFCs: `auth.ts` (NextAuth config), `web/src/server/api/root.ts` (tRPC router), `packages/shared/src/server/queues.ts` (queue definitions), `worker/src/app.ts` (worker entrypoint), `packages/shared/prisma/schema.prisma` (Prisma schema)

### Sizing guidance

- **Too small**: "Add an import statement" (merge into the task that needs it)
- **Just right**: "Implement createOrg() service method with unit tests" (clear scope, testable)
- **Too large**: "Implement the entire billing service" (break into per-method tasks)
- **Too large**: "Implement Stripe webhook handler" (break into per-event-type tasks)
- **Just right**: "Handle invoice.payment_failed webhook event with dunning notification" (one event, one flow)

---

## Example: Multi-Tenancy Core RFC Decomposition

Given the Multi-Tenancy Core RFC with 3 implementation phases:

### Epic
- **Summary**: Multi-Tenancy Core: Tenant isolation and lifecycle management
- **Type**: Epic
- **Includes**: Glossary, Langfuse Baseline guardrails, cross-epic dependency note ("blocks all other epics")

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
  - **Task 5**: [Langfuse mod] Add deleted-org filtering to NextAuth session callback

### Phase 2 Story + Tasks
- **Story**: Multi-Tenancy Core Phase 2: Integration and end-to-end validation
  - **Task 1**: Implement tenant context injection into request scope
  - **Task 2**: Implement background job tenant context validation (BullMQ orgId enforcement)
  - **Task 3**: Add cross-tenant isolation integration tests (PostgreSQL)
  - **Task 4**: Add cross-tenant isolation integration tests (ClickHouse)
  - **Task 5**: End-to-end validation of all exported service methods

---

## Example: Billing Service RFC Decomposition (Webhook Per-Event Pattern)

### Phase 2 Story: Webhook Event Handlers (broken into per-event tasks)
- **Story**: Billing Phase 2: Stripe webhook event processing
  - **Task 1**: [Langfuse mod] Implement webhook route at `/api/webhooks/stripe` with signature verification and `bodyParser: false`
  - **Task 2**: Create `processed_webhook_events` idempotency table via Prisma migration
  - **Task 3**: Handle `invoice.payment_succeeded` webhook event
  - **Task 4**: Handle `invoice.payment_failed` webhook event with dunning notification
  - **Task 5**: Handle `customer.subscription.updated` webhook event (plan changes)
  - **Task 6**: Handle `customer.subscription.deleted` webhook event (cancellation)
  - **Task 7**: Implement `subscription-downgrade-check` daily BullMQ cron job

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

If the RFC has non-standard sections not listed above:
- Read them carefully for implementable content
- Generate tasks for any deliverables found
- Include the section content as context in relevant task descriptions

---

## Self-Improvement Protocol

This skill tracks corrections and improvements over time. When the user corrects Claude's behavior while this skill is active, the skill writes those observations to an improvement plan file in the marketplace repository. These plans are later reviewed by the skill author to implement permanent changes to the skill.

### During Execution: Monitor for Corrections

While executing this skill's workflow, watch for signals that the user is correcting your behavior:

**Correction signals to watch for:**
- Direct corrections: "no", "don't do that", "stop", "wrong", "that's not right", "I said..."
- Redirection: "instead, do X", "I meant Y", "use Z approach"
- Repeated instructions: the user restating something they already said (indicates you missed it)
- Frustration indicators: "again", "I already told you", "as I said before"
- Preference expressions: "I prefer", "always do X", "never do Y"
- Approval of non-obvious approaches: "yes exactly", "perfect", "that's the right way" (for approaches that were judgment calls, not obvious from the instructions)

**What to capture for each correction:**
1. **What you did wrong** (or what non-obvious approach worked well)
2. **What the user wanted instead** (or confirmed as correct)
3. **Which workflow step it relates to**
4. **Why the correction matters** (impact on output quality)

### Writing Improvements

After the skill's workflow completes (or at natural breakpoints if the session is long), write or update the improvement plan file at:

```
/Applications/workspace/gen-ai-projects/utibes-plugins-marketplace/improvement-plans/rfc-to-jira.md
```

Use this format:

```markdown
# Improvement Plan: rfc-to-jira

Last updated: {date}
Total lessons: {count}

## Lessons Learned

### Lesson {N}: {short title}
- **Date:** {date}
- **Workflow Step:** {step name/number}
- **What happened:** {what you did that was corrected, or what approach was confirmed}
- **What to do instead:** {the correct behavior, or the confirmed approach to keep using}
- **Why:** {why this matters for output quality}

## Patterns to Watch For

{Summarize recurring themes from the lessons above. For example, if the user has corrected ticket granularity multiple times, note: "User prefers smaller, more focused tickets over large multi-file tasks."}

## Proposed Skill Changes

{If a lesson is clear and repeated enough to warrant a permanent change to the SKILL.md instructions, document the proposed change here. Include which section to modify and the suggested new wording.}

| # | Section | Current Behavior | Proposed Change | Based on Lessons |
|---|---------|-----------------|-----------------|------------------|
| 1 | {section} | {what the skill currently says} | {what it should say} | Lessons {N, M} |
```

### Rules

1. **Never modify SKILL.md directly.** All improvements go to the improvement plan file as proposals for the skill author.
2. **Be specific.** "User didn't like the tickets" is useless. "User wants each ticket to include the exact file paths to create/modify, not just descriptions" is actionable.
3. **Capture successes too.** If you made a judgment call and the user confirmed it was right, record that as a positive lesson so future sessions maintain that behavior.
4. **Deduplicate.** If a new correction matches an existing lesson, update the existing lesson's count or add context rather than creating a duplicate.
5. **Keep it concise.** Target under 50 lessons; if it grows beyond that, consolidate related lessons into patterns.
