# RFC to JIRA

A Claude Code plugin that converts RFC specification documents into structured JIRA tickets. Each ticket is detailed enough for Claude Code to pick up and implement end-to-end, with built-in validation loops.

## Features

- Reads standard 16-section RFC documents plus any non-standard sections (Langfuse Baseline, Dunning Policy, Alert Rules, etc.)
- Maps companion PRD EARS requirements (`WHEN [trigger] THE SYSTEM SHALL [behaviour]`) to Given/When/Then acceptance criteria on each task
- Creates a ticket hierarchy: Epic (service) > Stories (phases) > Tasks (work items)
- Cross-references CONSTITUTION, stack.md, and structure.md for coding standards
- Handles cross-epic dependencies and bidirectional module relationships
- Reusable for any RFC following the standard template

## Ticket Types

| Template | For | Examples |
|----------|-----|---------|
| **Task** | Application code with TDD loop | Service methods, tRPC endpoints, Zod schemas |
| **BullMQ Job** | Repeatable/cron jobs in the worker | Usage sync, downgrade checks, queue-depth metrics |
| **Terraform** | Infrastructure as code | CloudWatch alarms, PagerDuty services, dashboards |
| **External Setup** | Third-party service configuration | Stripe products, SendGrid templates, AWS provisioning |

## What Each Task Includes

1. **Context**: RFC/PRD paths, module location, pattern-to-follow reference, Langfuse Baseline guardrails, key invariants
2. **Out of scope**: Explicit list of what this task does NOT do
3. **TDD step**: Write failing tests first with Given/When/Then cases (mandatory)
4. **Implementation steps**: Exact files to create/modify, code patterns to follow, Langfuse file modification warnings
5. **Test validation**: Run tests, typecheck, lint
6. **Manual validation**: Start app locally, verify end-to-end (with alternatives for internal-only modules)
7. **Acceptance criteria**: Functional (from PRD EARS) + cross-cutting checklist (security, observability, tenant isolation)

## Usage

```
/rfc-to-jira .specs/multi-tenancy-core/01-RFC-multi-tenancy-core.md
```

Or ask naturally: "create JIRA tickets from the billing service RFC", "break down the multi-tenancy spec into tickets"

## Prerequisites

- Atlassian MCP server connected to Claude Code
- Jira project with Epic, Story, and Task issue types
- RFC following the standard 16-section template (with optional non-standard sections)
- Companion PRD with EARS requirements in the same directory

## Installation

```
/plugin install rfc-to-jira@utibes-plugins-marketplace
```
