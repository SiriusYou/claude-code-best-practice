# Skills Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2020%2C%202026-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-skills-implementation.md)

Claude Code skills — frontmatter fields, skill types taxonomy, writing tips, and distribution strategies.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (10)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Display name and `/slash-command` identifier. Defaults to the directory name if omitted |
| `description` | string | Recommended | What the skill does. Shown in autocomplete and used by Claude for auto-discovery |
| `argument-hint` | string | No | Hint shown during autocomplete (e.g., `[issue-number]`, `[filename]`) |
| `disable-model-invocation` | boolean | No | Set `true` to prevent Claude from automatically invoking this skill |
| `user-invocable` | boolean | No | Set `false` to hide from the `/` menu — skill becomes background knowledge only, intended for agent preloading |
| `allowed-tools` | string | No | Tools allowed without permission prompts when this skill is active |
| `model` | string | No | Model to use when this skill runs (e.g., `haiku`, `sonnet`, `opus`) |
| `context` | string | No | Set to `fork` to run the skill in an isolated subagent context |
| `agent` | string | No | Subagent type when `context: fork` is set (default: `general-purpose`) |
| `hooks` | object | No | Lifecycle hooks scoped to this skill |

---

## Skill Types Taxonomy (9)

> Based on patterns observed across hundreds of skills in active use at Anthropic.
> The best skills fit cleanly into one category; confusing ones straddle several.

### 1. Library & API Reference

Explain how to correctly use a library, CLI, or SDK — especially internal ones or common libraries Claude sometimes gets wrong.

| Example | What it does |
|---------|-------------|
| `billing-lib` | Internal billing library: edge cases, footguns, usage patterns |
| `internal-platform-cli` | Every subcommand with examples on when to use them |
| `frontend-design` | Make Claude better at your design system |

**Skill structure:** SKILL.md + `references/` folder with code snippets and a gotchas list.

### 2. Product Verification

Describe how to test or verify code is working. Often paired with Playwright, tmux, or other automation tools.

| Example | What it does |
|---------|-------------|
| `signup-flow-driver` | Headless browser: signup → email verify → onboarding, with assertions at each step |
| `checkout-verifier` | Drives checkout UI with Stripe test cards, verifies invoice state |
| `tmux-cli-driver` | Interactive CLI testing where the thing being verified needs a TTY |

**Tip:** Have Claude record a video of its output, or enforce programmatic assertions on state at each step. Include scripts in the skill folder for maximum robustness.

### 3. Data Fetching & Analysis

Connect to data and monitoring stacks. Include libraries, credentials helpers, dashboard IDs, and instructions for common workflows.

| Example | What it does |
|---------|-------------|
| `funnel-query` | Which events to join for signup → activation → paid, plus canonical user_id table |
| `cohort-compare` | Compare retention/conversion between cohorts, flag statistically significant deltas |
| `grafana` | Datasource UIDs, cluster names, problem → dashboard lookup table |

### 4. Business Process & Team Automation

Automate repetitive workflows into one command. Saving previous results in log files helps the model stay consistent.

| Example | What it does |
|---------|-------------|
| `standup-post` | Aggregates ticket tracker + GitHub activity + Slack → formatted standup |
| `create-jira-ticket` | Enforces schema (valid enums, required fields) + post-creation workflow |
| `weekly-recap` | Merged PRs + closed tickets + deploys → formatted recap |

### 5. Code Scaffolding & Templates

Generate framework boilerplate. Especially useful when scaffolding has natural language requirements that can't be purely covered by code.

| Example | What it does |
|---------|-------------|
| `new-workflow` | Scaffolds a new service/workflow/handler with your annotations |
| `new-migration` | Migration file template + common gotchas |
| `create-app` | New internal app with auth, logging, and deploy config pre-wired |

### 6. Code Quality & Review

Enforce code quality. Can include deterministic scripts for maximum robustness. Run automatically via hooks or in CI.

| Example | What it does |
|---------|-------------|
| `adversarial-review` | Spawns a fresh-eyes subagent to critique, iterates until findings degrade to nitpicks |
| `code-style` | Enforces styles Claude doesn't do well by default |
| `testing-practices` | Instructions on how and what to test |

### 7. CI/CD & Deployment

Fetch, push, and deploy code. May reference other skills to collect data.

| Example | What it does |
|---------|-------------|
| `babysit-pr` | Monitors PR → retries flaky CI → resolves merge conflicts → enables auto-merge |
| `deploy-service` | Build → smoke test → gradual rollout → auto-rollback on regression |
| `cherry-pick-prod` | Isolated worktree → cherry-pick → conflict resolution → PR with template |

### 8. Runbooks

Take a symptom (Slack thread, alert, error signature), walk through multi-tool investigation, produce a structured report.

| Example | What it does |
|---------|-------------|
| `service-debugging` | Maps symptoms → tools → query patterns for high-traffic services |
| `oncall-runner` | Fetches alert → checks usual suspects → formats findings |
| `log-correlator` | Given a request ID, pulls matching logs from every system that touched it |

### 9. Infrastructure Operations

Routine maintenance and operational procedures — some involving destructive actions that benefit from guardrails.

| Example | What it does |
|---------|-------------|
| `orphan-cleanup` | Finds orphaned pods/volumes → Slack notification → soak period → user confirms → cleanup |
| `dependency-management` | Your org's dependency approval workflow |
| `cost-investigation` | "Why did our bill spike" with specific buckets and query patterns |

---

## Tips for Writing Skills

### Don't State the Obvious

Claude already knows a lot about coding. Focus on information that pushes Claude **out of its normal way of thinking** — your team's specific patterns, not generic best practices.

### Build a Gotchas Section

The highest-signal content in any skill. Build it up from common failure points Claude encounters. Update it over time as new edge cases emerge.

```markdown
## Gotchas
- NEVER use `db.raw()` for user input — always use parameterized queries
- The `billing.charge()` method is idempotent but `billing.refund()` is NOT
- Feature flags are cached for 5 min — don't test flag changes in a tight loop
```

### Use the File System & Progressive Disclosure

A skill is a **folder**, not just a markdown file. Think of the entire file system as context engineering:

```
.claude/skills/billing-lib/
├── SKILL.md                    # Overview + gotchas
├── references/
│   ├── api.md                  # Detailed function signatures
│   └── examples.md             # Usage examples
├── assets/
│   └── report-template.md      # Output template to copy
└── scripts/
    └── validate-invoice.sh     # Verification script
```

Tell Claude what files are in your skill in SKILL.md, and it will read them at appropriate times.

### Avoid Railroading Claude

Give Claude the information it needs, but give it flexibility to adapt. Overly specific instructions in reusable skills lead to brittle behavior across different contexts.

### Think Through the Setup

Some skills need user-specific config. Store setup info in a `config.json` in the skill directory:

```markdown
## Setup
If `config.json` does not exist in this skill directory, ask the user:
1. Which Slack channel to post to
2. Their team name for the standup header
Save responses to `config.json` for future runs.
```

Use the `AskUserQuestion` tool for structured, multiple-choice setup questions.

### The Description Field Is for the Model

Claude scans every skill's `description` at session start to decide which skill to trigger. Write it as a **trigger condition**, not a summary:

```yaml
# Bad
description: "A skill for managing database migrations"

# Good
description: "Use when creating, running, or rolling back database migrations.
  TRIGGER when user mentions migrations, schema changes, or database versioning."
```

### Memory & Storing Data

Skills can maintain state by storing data within them — append-only logs, JSON files, or even SQLite databases:

```
# A standup skill that remembers previous posts
.claude/skills/standup-post/
├── SKILL.md
└── data/
    └── standups.log     # Append-only history
```

Use `${CLAUDE_PLUGIN_DATA}` for a stable per-plugin data folder that survives upgrades.

### Store Scripts & Generate Code

Give Claude composable scripts and helper functions. Claude spends its turns on **composition** rather than reconstructing boilerplate:

```python
# scripts/fetch_events.py — helper for data analysis
def get_events(event_type, start_date, end_date):
    """Fetch events from the event source."""
    ...

def get_user_sessions(user_id, date):
    """Get all sessions for a user on a given date."""
    ...
```

Claude generates scripts on the fly to compose these functions for complex analysis.

### On-Demand Hooks

Skills can register hooks that activate only when the skill is called and last for the session:

```yaml
---
name: careful
description: "Use when working with production systems"
hooks:
  PreToolUse:
    - type: command
      command: "python3 .claude/skills/careful/guard.py"
      matcher: "Bash"
---
```

Examples:
- `/careful` — blocks `rm -rf`, `DROP TABLE`, force-push via PreToolUse matcher
- `/freeze` — blocks Edit/Write outside a specific directory

---

## Distribution

### In-Repo Skills

Check skills into `.claude/skills/` for small teams with few repos. Every checked-in skill adds context at session start.

### Plugin Marketplace

For larger organizations, use an [internal plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) to let teams install only what they need.

**Organic curation process:**
1. Author uploads skill to a sandbox folder in GitHub
2. Share in Slack / internal forums for feedback
3. Once skill has traction, PR to move into marketplace
4. Curate before release — bad or redundant skills are easy to create

### Composing Skills

Reference other skills by name in your SKILL.md — Claude will invoke them if installed. Native dependency management is not yet built into marketplaces.

### Measuring Usage

Use a PreToolUse hook to log skill invocations ([example code](https://gist.github.com/ThariqS/24defad423d701746e23dc19aace4de5)). Track which skills are popular or under-triggering.

---

## ![Official](../!/tags/official.svg) **(5)**

| # | Skill | Description |
|---|-------|-------------|
| 1 | `simplify` | Review changed code for reuse, quality, and efficiency — refactors to eliminate duplication |
| 2 | `batch` | Run commands across multiple files in bulk |
| 3 | `debug` | Debug failing commands or code issues |
| 4 | `loop` | Run a prompt or slash command on a recurring interval (up to 3 days) |
| 5 | `claude-api` | Build apps with the Claude API or Anthropic SDK — triggers on `anthropic` / `@anthropic-ai/sdk` imports |

See also: [Official Skills Repository](https://github.com/anthropics/skills/tree/main/skills) for community-maintained installable skills.

---

## Sources

- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Lessons from Building Claude Code: How We Use Skills](https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills) — Anthropic blog
- [Skill Creator](https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills) — tool for creating skills
- [Agent Skills Course](https://anthropic.skilljar.com/introduction-to-agent-skills) — Skilljar
- [Skills Discovery in Monorepos](../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
