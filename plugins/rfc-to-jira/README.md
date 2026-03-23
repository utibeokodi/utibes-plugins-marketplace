# RFC to JIRA

A Claude Code plugin that converts RFC specification documents into structured JIRA tickets. Each ticket is detailed enough for Claude Code to pick up and implement end-to-end, with built-in validation loops.

## Features

- Reads standard 16-section RFC documents and companion PRDs
- Creates a ticket hierarchy: Epic (service) > Stories (phases) > Tasks (work items)
- Each task includes TDD workflow, implementation steps, manual validation steps, and acceptance criteria
- Cross-references CONSTITUTION, stack.md, and structure.md for coding standards
- Handles dependencies between phases and tasks

## Usage

```
/rfc-to-jira .specs/multi-tenancy-core/01-RFC-multi-tenancy-core.md
```

Or ask naturally: "create JIRA tickets from the billing service RFC", "break down the multi-tenancy spec into tickets"

## Ticket Structure

Each generated task includes:
1. **Context**: RFC path, module location, key invariants, dependencies
2. **TDD step**: Write failing tests first (mandatory)
3. **Implementation steps**: Exact files to create/modify, code patterns to follow
4. **Test validation**: Run tests, typecheck, lint
5. **Manual validation**: Start app locally, verify end-to-end
6. **Final checks**: Full test suite, build verification
7. **Acceptance criteria**: Checklist derived from RFC and CONSTITUTION

## Prerequisites

- Atlassian MCP server connected to Claude Code
- Jira project with Epic, Story, and Task issue types
- RFC following the standard 16-section template

## Installation

```
/install rfc-to-jira@claude-plugins-marketplace
```
