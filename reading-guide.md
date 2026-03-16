 # Claude Code Documentation — Reading Guide for Backend Engineers

> A curated summary of every section in the recommended reading order, written for a backend engineer at a large-scale distributed systems company.

---

## Phase 1 — Get the Mental Model Right

### 1. [Claude Code Overview](https://code.claude.com/docs/en/overview)

Claude Code is an agentic coding tool that reads your codebase, edits files, runs commands, and integrates with your development tools. It runs in your terminal, IDE (VS Code, JetBrains), a standalone desktop app, or on the web — all powered by the same underlying engine, so your `CLAUDE.md` files, settings, and MCP servers work everywhere.

**Core capabilities:**
- Automates tedious work: writing tests, fixing lint errors, resolving merge conflicts, updating dependencies
- Builds features and fixes bugs from plain-language descriptions, working across multiple files
- Interacts directly with git: stages changes, writes commit messages, opens PRs
- Connects to external tools via MCP (Jira, Slack, databases, Google Drive)
- Customizable via `CLAUDE.md` instructions, Skills (custom commands), and Hooks (shell automation)
- Supports multi-agent parallelism: spawn multiple Claude sessions working simultaneously

**Surfaces:** Terminal CLI, VS Code, JetBrains, Desktop App, Web (`claude.ai/code`), Slack, GitHub Actions, GitLab CI/CD.

---

### 2. [How Claude Code Works](https://code.claude.com/docs/en/how-claude-code-works)

The most important conceptual page. Understand this before building any habits.

**The agentic loop** runs in three phases: *gather context → take action → verify results*, cycling until the task is done. You can interrupt at any point to steer it.

**Built-in tool categories:**
| Category | What it does |
|---|---|
| File operations | Read, edit, create, rename files |
| Search | Find files by pattern, grep content with regex |
| Execution | Run shell commands, tests, git, start servers |
| Web | Search the web, fetch documentation |
| Code intelligence | Type errors, go-to-definition (requires plugin) |

**What Claude can access** when you run it in a directory: all files in that directory, your terminal (any command you can run), git state, your `CLAUDE.md`, auto-memory, and any MCP servers or Skills you've configured.

**Sessions** are persistent and local. Each new session starts with a fresh context window, but Claude can carry knowledge across sessions via auto-memory and `CLAUDE.md`. You can resume sessions with `claude --continue` or `claude --resume`. Fork a session with `--fork-session` to try a different approach without losing the original.

**The context window** is your most important resource to manage. As it fills, performance degrades. Tools like `/compact`, `/clear`, and subagents are the main levers for keeping it healthy.

**Checkpoints:** Every file edit is reversible. Double-tap `Esc` or run `/rewind` to restore code and/or conversation to any previous state.

**Permission modes** (cycle with `Shift+Tab`):
- `default` — asks before each file edit and shell command
- `acceptEdits` — auto-accepts file edits, still asks for commands
- `plan` — read-only; Claude creates a plan you approve before execution

---

### 3. [Quickstart](https://code.claude.com/docs/en/quickstart)

Hands-on walkthrough for your first session. Install, log in, navigate to a project, run `claude`.

**Essential CLI commands:**
| Command | What it does |
|---|---|
| `claude` | Start interactive session |
| `claude "task"` | Start with an initial prompt |
| `claude -p "query"` | One-shot query, then exit (useful for scripting) |
| `claude -c` | Continue most recent conversation |
| `claude -r` | Resume a specific past session |
| `claude commit` | Create a git commit |

Key insight: talk to Claude like a colleague. "Fix the login bug where users see a blank screen after entering wrong credentials" beats "fix the bug". Give context, mention constraints, point to files.

---

## Phase 2 — Core Daily Workflow

### 4. [Common Workflows](https://code.claude.com/docs/en/common-workflows)

Step-by-step recipes for everyday tasks.

**Understand a new codebase:** Ask broad questions first ("give me an overview"), then narrow ("explain how authentication is handled"). Claude reads files as needed — you don't have to manually add context.

**Fix bugs:** Share the error output and let Claude trace it. Give reproduction steps. For intermittent bugs, say so.

**Refactor:** Ask Claude to find deprecated usage, suggest the modern approach, then apply changes safely. Ask it to run tests after.

**Plan mode for safe analysis:** Use `--permission-mode plan` or `Shift+Tab` twice to put Claude in read-only exploration mode. It creates a plan, you review and edit it (`Ctrl+G` to open in editor), then approve before implementation.

**Tests:** Ask Claude to find untested functions, generate tests matching your existing style and frameworks, add edge cases, then run and fix them.

**Pull requests:** Ask Claude to summarize changes and create a PR. It uses the `gh` CLI if installed.

**Parallel sessions with git worktrees:** Use `claude --worktree feature-auth` to create an isolated worktree with its own branch. Run multiple Claude sessions in parallel without changes colliding.

**`@` references:** Type `@src/payments/` to include a file or directory path inline. Claude reads it before responding. Useful for directing attention without waiting for Claude to discover the file.

**Pipe in data:** `cat build-error.txt | claude -p 'explain the root cause'` — Claude follows Unix conventions and composes with other tools.

**Output formats:** Use `--output-format json` or `stream-json` when integrating Claude into scripts or CI pipelines.

---

### 5. [How Claude Remembers Your Project (Memory & CLAUDE.md)](https://code.claude.com/docs/en/memory)

**Critical for large codebases.** This is how you encode persistent context so you don't re-explain architecture every session.

**Two memory systems:**

| | CLAUDE.md files | Auto-memory |
|---|---|---|
| Who writes it | You | Claude |
| What it contains | Instructions and rules | Learnings and patterns |
| Scope | Project, user, or org | Per working tree |
| Loaded into | Every session | Every session (first 200 lines) |

**CLAUDE.md locations:**
- `./CLAUDE.md` or `./.claude/CLAUDE.md` — project-level, checked into git, shared with team
- `~/.claude/CLAUDE.md` — personal, applies to all your projects
- `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) — org-wide managed policy

**Writing effective instructions:**
- Target under 200 lines. Longer files reduce adherence.
- Be concrete: "Use 2-space indentation" beats "format code properly"
- Include build commands, test runner invocations, naming conventions, architectural decisions, service boundary rules
- Exclude things Claude can infer from code, standard language conventions, frequently-changing info

**Path-specific rules** (`.claude/rules/`): Scope instructions to specific file types using YAML frontmatter. A rule with `paths: ["src/api/**/*.ts"]` only loads when Claude reads matching files — reduces noise and saves context.

**Auto-memory:** Claude writes notes to `~/.claude/projects/<project>/memory/MEMORY.md` as it works — build commands, debugging insights, your preferences. The first 200 lines load every session. Run `/memory` to browse and edit these files.

**Imports:** CLAUDE.md files can import other files with `@path/to/file` syntax. Good for referencing READMEs, package.json, or team-specific style docs.

---

### 6. [Best Practices](https://code.claude.com/docs/en/best-practices)

The most useful "meta" page. Read this early.

**The core constraint:** Context window fills fast. Every file read, command output, and message consumes it. Performance degrades as it fills. Everything else is downstream of managing this.

**Give Claude something to verify against.** Include test cases, paste screenshots of expected UI, define expected output. Without verification criteria, Claude produces plausible-looking code that may not work.

**Explore first, then plan, then code.** Use Plan Mode for complex tasks. The four-phase workflow:
1. Enter Plan Mode → Claude reads relevant files
2. Ask Claude to create a detailed implementation plan
3. Review and edit the plan (`Ctrl+G`)
4. Switch to Normal Mode → implement

**Be specific upfront.** Reference files with `@`, mention constraints, point to example patterns in your codebase. Vague prompts work but cost more corrections.

**Write an effective CLAUDE.md.** Run `/init` to generate a starter file from your codebase. Keep it under 200 lines. Include things Claude can't figure out from reading code. Prune aggressively — if Claude already behaves correctly without a rule, delete it.

**What to include vs. exclude in CLAUDE.md:**
- ✅ Include: build/test commands Claude can't guess, code style rules that differ from defaults, repo etiquette, architectural decisions, common gotchas
- ❌ Exclude: anything Claude can infer from code, standard conventions, detailed API docs (link instead), info that changes frequently

**Course-correct early and often.** If Claude goes off track, press `Esc` to stop. Double-tap `Esc` or `/rewind` to restore state. After two failed corrections, `/clear` and rewrite the prompt with what you learned.

**Manage context aggressively.** Use `/clear` between unrelated tasks. Use `/compact` with instructions to control what's preserved. Use subagents for research so exploration doesn't bloat your main context.

**Common failure patterns to avoid:**
- The kitchen-sink session: unrelated tasks in one context
- Correcting over and over: two failed corrections → `/clear` and better prompt
- Over-specified CLAUDE.md: too long → Claude ignores half of it
- The infinite exploration: scope investigations narrowly or use subagents

**Fan-out pattern for bulk operations:** Generate a task list, loop through with `claude -p` for each file, use `--allowedTools` to scope permissions for unattended batch runs.

---

### 7. [Interactive Mode](https://code.claude.com/docs/en/interactive-mode)

Complete reference for keyboard shortcuts and features inside a session.

**Essential shortcuts:**
| Shortcut | Action |
|---|---|
| `Shift+Tab` | Cycle permission modes (normal → accept edits → plan) |
| `Esc` | Cancel current generation |
| `Esc + Esc` | Open rewind menu |
| `Ctrl+G` | Open current prompt or plan in your editor |
| `Ctrl+O` | Toggle verbose output (see thinking and tool use) |
| `Ctrl+B` | Background a running task |
| `Ctrl+T` | Toggle task list |
| `Ctrl+L` | Clear terminal screen (keeps conversation) |
| `Option+P` (mac) | Switch model without clearing prompt |

**Input modes:**
- `\` + `Enter` — multiline input (all terminals)
- `!` prefix — run bash directly and add output to context (e.g., `! git status`)
- `@` prefix — file path autocomplete
- `/` prefix — commands and skills

**`/btw` — side questions:** Ask a quick question without adding it to conversation history. Appears in a dismissible overlay. Useful for checking a detail mid-task. The answer never enters the context.

**Task list:** Claude creates a task list for complex multi-step work. Press `Ctrl+T` to toggle visibility.

**PR review status:** If you're on a branch with an open PR, Claude shows a clickable PR link in the footer with color-coded review state (green = approved, red = changes requested, etc.).

**Vim mode:** Enable with `/vim`. Supports full vim motions, text objects, and mode switching.

---

### 8. [CLI Reference](https://code.claude.com/docs/en/cli-reference)

Complete reference for all commands and flags. Bookmark this.

**Key flags for backend engineers:**
| Flag | What it does |
|---|---|
| `--worktree` / `-w` | Create isolated git worktree for this session |
| `--permission-mode plan` | Start in plan (read-only) mode |
| `--dangerously-skip-permissions` | Bypass all permission prompts (use in CI/sandboxed envs only) |
| `--allowedTools "Bash(git *)"` | Pre-approve specific tools |
| `--max-turns 10` | Limit agentic turns in non-interactive mode |
| `--max-budget-usd 5.00` | Budget cap for a run (print mode only) |
| `--output-format json` | Structured output for scripting |
| `--output-format stream-json` | Streaming JSON for real-time processing |
| `--append-system-prompt "..."` | Add rules to system prompt without replacing it |
| `--model claude-opus-4-6` | Specify model for session |
| `--verbose` | Show full turn-by-turn output |
| `-n "session-name"` | Name the session for easy resumption |
| `--remote "Fix login bug"` | Start a cloud session (web) |

**System prompt flags:**
- `--append-system-prompt` — adds to default (recommended for most cases)
- `--system-prompt` — replaces the entire default (only when you need full control)

**Piping:** `cat error.log | claude -p 'explain the root cause'` — Claude processes stdin as context.

---

## Phase 3 — Automation & Extensibility

### 9. [Automate Workflows with Hooks](https://code.claude.com/docs/en/hooks)

Hooks are user-defined automations that execute at specific lifecycle points. Unlike `CLAUDE.md` instructions (advisory), hooks are deterministic — they always fire.

**Hook types:**
| Type | What it is |
|---|---|
| `command` | Shell script that receives JSON via stdin, exits to signal decision |
| `http` | POST request to a local or remote endpoint |
| `prompt` | Single-turn LLM evaluation (for complex conditions) |
| `agent` | Spawn a subagent to verify conditions |

**Key lifecycle events:**
| Event | When |
|---|---|
| `PreToolUse` | Before a tool executes — can block |
| `PostToolUse` | After tool succeeds |
| `Stop` | When Claude finishes responding |
| `UserPromptSubmit` | When user submits a prompt |
| `SessionStart` | Session begins or resumes |
| `Notification` | Claude needs attention (idle, permission prompt, etc.) |

**Exit codes control behavior:**
- `0` = allow
- `2` = block (with stderr message fed back to Claude)
- Other = non-blocking error (log and continue)

**Hook configuration** lives in `settings.json`:
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{"type": "command", "command": "./scripts/run-lint.sh"}]
    }]
  }
}
```

**Practical uses:** Auto-lint after every file edit, block dangerous commands (`rm -rf`), require tests to pass before Claude stops, send Slack notifications when long tasks complete, validate that generated code doesn't touch certain service boundaries.

**Ask Claude to write hooks for you:** "Write a hook that runs eslint after every file edit" — it will write the config and script.

---

### 10. [Hooks Reference](https://code.claude.com/docs/en/hooks-reference)

Technical companion to the Hooks guide. Covers the complete schema for all hook events, the exact JSON input/output format, async hooks, HTTP hooks, prompt hooks, and agent hooks.

**JSON input** to hooks includes the full tool call context: event name, tool name, tool input parameters, session info. For `PreToolUse` on a Bash call, `tool_input.command` gives you the exact command string.

**Output format** lets you provide richer decisions:
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Write operations to migrations/ are blocked"
  }
}
```

**Advanced hook types:**
- **Async hooks**: fire and forget — useful for logging and notifications without blocking
- **HTTP hooks**: useful for central policy servers or audit logging infrastructure
- **Prompt hooks**: let an LLM evaluate whether to allow an action — powerful for nuanced policies

---

### 11. [Connect Claude Code to Tools via MCP](https://code.claude.com/docs/en/mcp)

MCP (Model Context Protocol) is an open standard for connecting Claude to external data sources and tools. This is how Claude queries your databases, posts to Slack, reads Jira tickets, pulls Figma designs, or calls any internal service.

**Adding a server:**
```bash
# Remote HTTP server
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# Local stdio server
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"
```

**Scopes:**
- `local` (default): only you, only this project (`~/.claude.json`)
- `project`: all team members via `.mcp.json` (check into git)
- `user`: you across all projects

**OAuth authentication:** Run `/mcp` inside a session to authenticate with servers that require OAuth 2.0.

**Practical examples:**
- `claude mcp add --transport http github https://api.githubcopilot.com/mcp/` — then ask Claude to review PRs, create issues, etc.
- PostgreSQL via stdio — ask Claude natural language queries against your DB
- Sentry — ask "what are the most common errors in the last 24 hours?"

**MCP resources:** Reference MCP data with `@server:resource/path` syntax, same as `@` for files.

**Tool Search:** When many MCP servers are configured, Claude Code defers tool loading and discovers tools on demand (triggered when tool descriptions exceed 10% of context). Keeps context lean.

**Managed MCP for orgs:** Deploy `managed-mcp.json` to give IT control over which servers all users can access. Supports allowlists/denylists by server name, command, or URL pattern.

---

### 12. [Extend Claude with Skills](https://code.claude.com/docs/en/skills)

Skills extend what Claude knows and can do. A Skill is a `SKILL.md` file with instructions — Claude invokes it automatically when relevant, or you trigger it with `/skill-name`.

**Where skills live:**
- `~/.claude/skills/<name>/SKILL.md` — personal, all projects
- `.claude/skills/<name>/SKILL.md` — this project only, shareable via git

**Frontmatter fields:**
| Field | Purpose |
|---|---|
| `name` | The `/command` name |
| `description` | Tells Claude when to invoke automatically |
| `disable-model-invocation: true` | Manual-only (Claude won't auto-invoke — use for side-effect workflows like `/deploy`) |
| `context: fork` | Run in isolated subagent context |
| `allowed-tools` | Restrict which tools Claude can use within this skill |

**Bundled skills** ship with Claude Code: `/batch` (parallel large-scale changes), `/simplify` (code quality review), `/debug` (session debugging), `/loop` (scheduled polling).

**`$ARGUMENTS` substitution:** `/fix-issue 1234` → `$ARGUMENTS` becomes `1234` in the skill content. Supports positional args with `$0`, `$1`, etc.

**Skills vs. CLAUDE.md:**
- CLAUDE.md loads every session, every request — use for "always do X" rules
- Skills load on demand — use for reference material or invocable workflows

**Skills vs. Subagents:**
- Skills: reusable instructions that run in your current context
- Subagents: isolated workers that run separately and return a summary

**Shell injection in skills:** Use `` !`command` `` syntax to pre-process data. The command runs before Claude sees the skill — output replaces the placeholder. Useful for pulling live PR diffs, log data, or API responses directly into the skill prompt.

---

### 13. [Extend Claude Code — Decision Guide](https://code.claude.com/docs/en/features-overview)

When you have multiple options, this page tells you which one to reach for. Read after you've seen all the extension types.

**Quick decision table:**
| Goal | Use |
|---|---|
| "Always do X" rules | CLAUDE.md |
| Reference docs Claude needs sometimes | Skill |
| Invocable workflow (`/deploy`) | Skill with `disable-model-invocation: true` |
| External tool/API access | MCP |
| Context isolation, parallel research | Subagent |
| Deterministic automation, no LLM involved | Hook |
| Bundle all of the above for distribution | Plugin |

**How features layer:** CLAUDE.md files are additive (all levels contribute simultaneously). Skills and subagents override by name (managed > user > project). Hooks merge (all registered hooks fire). MCP servers override by name (local > project > user).

**Context cost by feature:**
| Feature | Loads when | Cost |
|---|---|---|
| CLAUDE.md | Session start | Every request |
| Skill descriptions | Session start | Low — descriptions only |
| Full skill content | When invoked | On demand |
| MCP tool definitions | Session start | Every request (mitigated by Tool Search) |
| Subagents | When spawned | Isolated from main session |
| Hooks | On trigger | Zero, unless they return context |

---

## Phase 4 — Scale & Parallelism

### 14. [Orchestrate Teams of Claude Code Sessions (Sub-agents)](https://code.claude.com/docs/en/sub-agents)

Subagents are specialized AI assistants that run in their own context window with their own tools and permissions. Claude delegates tasks to them; they work independently and return summaries.

**Built-in subagents:**
- `Explore` — fast, read-only, Haiku model; used for codebase exploration
- `Plan` — read-only research used during plan mode
- `general-purpose` — full tools, inherits model; for complex multi-step tasks

**Creating custom subagents** (`.claude/agents/name.md`):
```yaml
---
name: security-reviewer
description: Reviews code for security vulnerabilities. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: opus
---
You are a senior security engineer...
```

**Key frontmatter fields:**
| Field | Purpose |
|---|---|
| `description` | Claude uses this to decide when to delegate |
| `tools` | Allowlist of tools (inherits all if omitted) |
| `model` | `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | `default`, `acceptEdits`, `bypassPermissions`, `plan` |
| `skills` | Skills to preload into subagent context at startup |
| `memory` | `user`/`project`/`local` — enables cross-session learning |
| `isolation: worktree` | Run in an isolated git worktree |

**Use subagents when:**
- The task produces verbose output you don't need in main context
- You want to enforce specific tool restrictions
- The work is self-contained (exploration, verification, parallel research)

**Common patterns:**
- Isolate high-volume ops: `"Use a subagent to run tests and report only failures"`
- Parallel research: `"Research the auth, database, and API modules in parallel using separate subagents"`
- Writer/Reviewer: Session A implements, Session B reviews in fresh context (unbiased, since it didn't write the code)

**Persistent memory:** Add `memory: user` to a subagent and it accumulates knowledge across sessions (codebase patterns, architecture notes, recurring issues). Stored in `~/.claude/agent-memory/<name>/`.

**Background vs. foreground:** Press `Ctrl+B` to background a running task. Background subagents auto-deny permission prompts they weren't pre-approved for; you can resume them in foreground to handle those.

**Subagents vs. agent teams:** Subagents work within one session; [agent teams](https://code.claude.com/docs/en/agent-teams) coordinate across fully independent sessions with peer-to-peer messaging — higher token cost but needed for sustained parallel work or competing hypotheses.

---

### 15. [Run Claude Code Programmatically — Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview)

The Agent SDK (Python + TypeScript) gives you the same agentic loop, built-in tools, and context management that power Claude Code, but in a programmable API. Use it to build internal tooling, automate workflows in CI/CD, or create products on top of Claude.

**Installation:**
```bash
pip install claude-agent-sdk        # Python
npm install @anthropic-ai/claude-agent-sdk  # TypeScript
```

**Core interface:**
```python
async for message in query(
    prompt="Find and fix the bug in auth.py",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"]),
):
    print(message)
```

**Capabilities:**
- All built-in tools (Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch)
- Hooks via callback functions (same events as interactive hooks)
- Custom subagents defined inline via `agents` option
- MCP server connections
- Session management: capture `session_id` from one run, `resume` in the next
- Structured output: `--json-schema` for validated JSON responses

**Key distinction from Claude Code CLI:** SDK is for production automation and custom applications. CLI is for interactive development. Workflows translate directly between them — same tools, same capabilities.

**Note:** The SDK lives on `platform.claude.com`, not `code.claude.com`. It's a separate product that builds on top of Claude Code's engine.

---

### 16. [GitHub Actions](https://code.claude.com/docs/en/github-actions) / [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd)

Brings Claude Code automation into your CI/CD pipeline. Trigger Claude with `@claude` in PR/issue comments, or run it automatically on events.

**Quick setup:** Run `/install-github-app` from Claude Code terminal. It guides you through installing the GitHub App and adding your `ANTHROPIC_API_KEY` secret.

**Basic workflow:**
```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    # Responds to @claude mentions in PR/issue comments
```

**Automation workflow (no trigger needed):**
```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "Review this PR for security issues"
    claude_args: "--max-turns 5 --model claude-sonnet-4-6"
```

**`@claude` in comments:** `@claude implement the feature described in this issue` — Claude creates a PR with the changes. `@claude fix the failing tests` — Claude pushes a fix.

**Enterprise cloud providers:** Supports AWS Bedrock (via OIDC) and Google Vertex AI (via Workload Identity Federation). Set `use_bedrock: "true"` or `use_vertex: "true"` and configure the relevant cloud authentication.

**Key configuration via `claude_args`:** Pass any Claude Code CLI flag — `--max-turns`, `--model`, `--mcp-config`, `--allowedTools`, etc.

---

### 17. [Automated Code Review](https://code.claude.com/docs/en/code-review)

Managed code review service that posts inline comments on PRs. A fleet of specialized agents analyze the diff against your full codebase in parallel, then a verification step filters false positives.

**Setup:** Admin enables it at `claude.ai/admin-settings/claude-code` for the organization. Choose per-repo trigger: on every push, once after PR creation, or manual only.

**Manual trigger:** Comment `@claude review` on any PR to start a review and opt that PR into push-triggered reviews going forward.

**Severity levels:**
- 🔴 Normal — bug that should be fixed before merging
- 🟡 Nit — minor, not blocking
- 🟣 Pre-existing — bug that exists but wasn't introduced by this PR

**Customization:** Add `REVIEW.md` to the repo root for review-specific rules (what to always flag, what to skip, style guidelines). Claude also reads your `CLAUDE.md` and flags newly-introduced violations.

**Pricing:** ~$15-25 per review average, billed separately from plan usage. Monitor spend via `claude.ai/analytics/code-review`.

---

## Phase 5 — Operational Concerns

### 18. [Configure Permissions](https://code.claude.com/docs/en/permissions)

Fine-grained control over what Claude can access and do. Critical for production codebases.

**Permission rule syntax:** `Tool` or `Tool(specifier)`. Rules are evaluated: deny → ask → allow. First match wins. Deny always beats allow.

**Rule examples:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Read(src/**)"
    ],
    "deny": [
      "Bash(git push *)",
      "Edit(//etc/**)",
      "Write(migrations/**)"
    ]
  }
}
```

**Path pattern prefixes for Read/Edit rules:**
| Prefix | Meaning |
|---|---|
| `//path` | Absolute path from filesystem root |
| `~/path` | Relative to home directory |
| `/path` | Relative to project root |
| `path` | Relative to current directory |

**Tool-specific rules:**
- `Bash(git * main)` — matches `git checkout main`, `git merge main`, etc.
- `WebFetch(domain:example.com)` — only fetches from example.com
- `mcp__sentry__*` — all tools from the Sentry MCP server
- `Agent(Explore)` — the Explore subagent specifically

**Permission modes:**
- `default` — prompts first use of each tool
- `acceptEdits` — auto-accepts file edits
- `plan` — read-only
- `dontAsk` — auto-denies unless pre-approved
- `bypassPermissions` — skips all checks (only in isolated/sandboxed envs)

**Managed settings for orgs:** Deploy rules via MDM/Group Policy that can't be overridden by users. `disableBypassPermissionsMode: "disable"` prevents developers from bypassing all checks. `allowManagedPermissionRulesOnly: true` locks down to org-approved rules only.

---

### 19. [Security](https://code.claude.com/docs/en/security)

**Architecture:** Read-only by default. Explicit permission required for file edits and shell commands. Write access is restricted to the working directory and subdirectories by default (can't modify parent dirs without explicit permission).

**Key protections:**
- Sandbox bash with filesystem and network isolation (see Sandboxing below)
- Prompt injection detection — suspicious patterns require manual approval even if previously allowlisted
- Command blocklist — `curl`, `wget`, and similar commands blocked by default (must be explicitly allowed)
- Input sanitization — prevents command injection
- Context-aware analysis — detects potentially harmful instructions
- Secure credential storage — API keys encrypted

**Prompt injection mitigations:** Isolated context windows for web fetch, trust verification for first-time codebases and new MCP servers, fail-closed matching (unrecognized commands default to requiring approval).

**Best practices:**
- Review proposed commands before approval
- Avoid piping untrusted content directly to Claude
- Use devcontainers or VMs for additional isolation when working with external content
- Use `ConfigChange` hooks to audit settings changes during sessions
- Audit permission settings regularly with `/permissions`

**For teams:** Use managed settings to enforce org-wide standards. Share approved permission configs via version control. Monitor via OpenTelemetry metrics.

---

### 20. [Manage Costs Effectively](https://code.claude.com/docs/en/costs)

**Typical costs:** ~$6/developer/day average with Sonnet, ~$100-200/developer/month.

**Track usage:** `/cost` shows token stats for the current session. `/stats` shows usage patterns for subscribers.

**Team management:** Set workspace spend limits in the Anthropic Console. For Bedrock/Vertex users, use LiteLLM (open source) for cost tracking by key.

**Rate limit recommendations by team size:**
| Team size | TPM per user |
|---|---|
| 1-5 users | 200k-300k |
| 5-20 users | 100k-150k |
| 20-50 users | 50k-75k |
| 50-100 users | 25k-35k |
| 100-500 users | 15k-20k |

**Cost reduction strategies:**
- `/clear` between unrelated tasks — stale context wastes tokens on every message
- Use Sonnet for most tasks, Opus only for complex architectural decisions
- Disable unused MCP servers (`/mcp`) — each adds tool definitions to every request
- Install code intelligence plugins for typed languages — reduces exploratory file reads
- Use subagents for verbose operations (test runs, log processing) — output stays isolated
- Move detailed instructions from CLAUDE.md to Skills — skills load on demand
- Lower extended thinking budget for simple tasks (`MAX_THINKING_TOKENS=8000`)
- Write specific prompts — "fix the validation in auth.ts" beats "improve the codebase"

---

### 21. [Monitoring (OpenTelemetry)](https://code.claude.com/docs/en/monitoring)

Claude Code supports OpenTelemetry for exporting metrics and traces to your observability stack.

**What you can monitor:** Token usage per session, tool call counts and latencies, permission decisions, hook execution, MCP server performance, subagent lifecycle.

**Configuration:** Set environment variables to point to your OTel collector:
```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://your-collector:4318
OTEL_SERVICE_NAME=claude-code
```

Fits naturally into existing observability infrastructure (Datadog, Grafana, Honeycomb, etc.). Useful for teams rolling out Claude Code at scale — track adoption, surface cost outliers, monitor usage patterns.

---

### 22. [Configure Permissions](https://code.claude.com/docs/en/permissions) *(revisit for sandboxing context)*

---

### 23. [Sandboxing](https://code.claude.com/docs/en/sandboxing)

OS-level filesystem and network isolation for bash commands. Complements permissions by enforcing restrictions at the kernel level — applies to all child processes, not just Claude's direct tool calls.

**How it works:**
- **Filesystem isolation:** Read/write access limited to configured directories. Enforced by macOS Seatbelt or Linux bubblewrap. Applies to `kubectl`, `terraform`, `npm`, and any other subprocess.
- **Network isolation:** All outbound traffic routed through a proxy. Only approved domains pass. New domain requests trigger permission prompts.

**Enable:** Run `/sandbox` in a session to choose a sandbox mode.

**Sandbox modes:**
- **Auto-allow:** Sandboxed bash commands run without asking permission (within defined boundaries). Still respects explicit deny rules.
- **Regular permissions:** Commands inside sandbox still require approval (more control, more friction).

**Configuration:**
```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": ["~/.kube", "//tmp/build"]
    }
  }
}
```

**Why it matters for large-scale systems:** Even if a prompt injection attack manipulates Claude's behavior, the sandbox prevents it from touching `~/.bashrc`, exfiltrating SSH keys, or calling attacker-controlled domains. Defense in depth.

**Security limitations to be aware of:**
- Network filtering doesn't inspect traffic content, only domains
- Allowing broad domains (e.g., `github.com`) could enable data exfiltration
- `allowUnixSockets` can inadvertently expose Docker socket or other privileged services
- Linux mode inside Docker requires `enableWeakerNestedSandbox` which considerably weakens isolation

**Open source runtime:** The sandbox runtime is available as `@anthropic-ai/sandbox-runtime` on npm for use in your own agent projects.

---

## Quick Reference

### The Most Important Pages (read these first)
1. How Claude Code Works — mental model
2. Memory & CLAUDE.md — persistent context
3. Best Practices — meta-strategy and common failure patterns
4. Hooks — deterministic automation

### Pages to Bookmark for Daily Use
- CLI Reference — every flag
- Interactive Mode — every shortcut
- Common Workflows — copy-paste recipes

### Pages for Team/Infrastructure Rollout
- Permissions — rule syntax and managed policies
- Security — threat model and mitigations
- Costs — rate limits and token budgets
- Monitoring — OpenTelemetry setup
- Sandboxing — OS-level isolation
