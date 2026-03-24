# Manual Verify

Black-box manual QA verification via Playwright. Launches a headed browser, navigates the application UI, executes validation steps from JIRA tickets, captures video and screenshot evidence, and returns structured pass/fail results.

## Core Principle

**Black-box testing only.** All expected behavior is derived from JIRA tickets, PR descriptions, and Langfuse documentation. The skill never reads source code, component files, route definitions, or test files to inform what to verify. This ensures the skill tests what the application is supposed to do, not how it was implemented.

## Features

- **Black-box philosophy**: Derives all test context from JIRA tickets, PR descriptions, and Langfuse docs
- **Single or multi-PR**: Verify one PR branch, or merge multiple PR branches into a temporary branch for combined testing
- **Automatic branch checkout**: Fetches and checks out PR branches locally before starting the app
- **Playwright browser automation**: Headed Chromium browser with accessibility-tree navigation
- **Video recording**: Per-step `.webm` recordings for review and debugging
- **Screenshot evidence**: Before/after captures at every assertion point
- **Langfuse UI knowledge**: Fetches Langfuse documentation on-demand for standard UI navigation
- **Custom feature navigation**: Uses JIRA ticket descriptions to understand custom UI flows
- **Structured results**: Returns pass/fail with failure details actionable by swe-dev
- **Evidence artifacts**: Organized output in `validation/<ticket-key>/`

## Usage

```bash
# Single ticket / single PR
/manual-verify OBS-3
/manual-verify OBS-3 --pr 42

# Multiple PRs (merges all branches into a temporary verification branch)
/manual-verify --pr 42,43,44
/manual-verify OBS-3, OBS-4, OBS-5 --pr 42,43,44

# Called automatically by swe-dev after tests pass (Phase 4)
```

## How It Works

```
Step 1: Gather context (JIRA ticket, PR description, Langfuse docs)
Step 2: Set up git branch (checkout PR branch, or merge multiple PR branches into tmp branch)
Step 3: Build test plan from validation steps
Step 4: Start the dev environment
Step 5: Launch Playwright (headed, with video recording)
Step 6: Execute validation loop (navigate, act, assert, capture evidence)
Step 7: Compile evidence (screenshots, videos, report)
Step 8: Return structured results and clean up temporary branch
```

## Context Sources (Black-Box Only)

| Source | What It Provides | Used For |
|--------|-----------------|----------|
| JIRA ticket | Expected behavior, acceptance criteria, manual validation steps | What to test, how custom features work |
| PR description | Summary of what changed and which area is affected | Where to test (not how it was built) |
| Langfuse docs | Standard UI navigation patterns, page layouts, feature flows | How to navigate the base Langfuse product |

## What It Never Reads

- Source code or component files
- Route definitions or API handlers
- Test files or test fixtures
- PR diff contents (the actual code changes)

## Evidence Output

```
validation/<ticket-key>/
├── report.md              # Pass/fail summary with inline screenshot refs
├── screenshots/
│   ├── step-1-before.png
│   ├── step-1-after.png
│   └── step-2-failure.png
├── videos/
│   ├── step-1.webm
│   └── step-2.webm
└── logs/
    └── console-errors.log
```

## Companion Plugins

| Plugin | Role | Relationship |
|--------|------|-------------|
| **swe-dev** | Autonomous implementation from JIRA tickets | Invokes manual-verify in Phase 4 after tests pass |
| **rfc-to-jira** | Creates the tickets that define validation steps | Upstream: provides the test specifications |

## Prerequisites

- Playwright installed (`npx playwright --version`)
- Playwright browsers installed (`npx playwright install chromium`)
- `gh` CLI authenticated (to fetch PR descriptions)
- Dev environment runnable (e.g., `pnpm run dev:web`)
- Atlassian MCP server connected (to fetch JIRA ticket details)

## Installation

```
/install manual-verify@claude-plugins-marketplace
```
