# claude-code-engineer-skills

Claude Code configuration template for **engineers** working on Language AI systems. Provides a complete `.claude/` setup with engineering-focused skills, hooks, agents, and commands — grounded in industry standards from the PyTorch docs and cookiecutter-data-science.

## Quick Start

1. Copy `.claude/`, `CLAUDE.md`, and `.mcp.json` into your project
2. Customize `CLAUDE.md` with your project's stack, directories, and conventions
3. Run `chmod +x .claude/hooks/skill-eval.sh`
4. Start Claude Code — skills are suggested automatically as you work

## Directory Structure

```
your-project-name/
├── CLAUDE.md                                  # Project conventions (loaded every session)
├── .mcp.json                                  # MCP server config (GitHub, Notion)
├── .claude/
│   ├── settings.json                          # Hooks, env vars, permissions
│   ├── settings.md                            # Human-readable hook documentation
│   ├── .gitignore                             # Ignores local settings + tasks
│   ├── agents/
│   │   ├── engineering-reviewer.md            # Code reviewer (Opus) — correctness, reproducibility
│   │   └── git-workflow.md                    # Git workflow advisory (Sonnet) — drafts commits, branches, MRs/PRs
│   ├── commands/
│   │   ├── onboard.md                         # /onboard — deep codebase exploration
│   │   ├── mr-review.md                       # /mr-review — review MR/PR against engineering standards
│   │   └── mr-summary.md                      # /mr-summary — generate MR/PR body from branch diff
│   ├── hooks/
│   │   ├── skill-eval.sh                      # Bash wrapper for skill evaluation engine
│   │   ├── skill-eval.js                      # Node.js skill matching engine
│   │   ├── skill-rules.json                   # Trigger rules for the skill categories
│   │   └── skill-rules.schema.json            # JSON Schema for skill-rules validation
│   └── skills/
│       ├── README.md                          # Skills overview + how to add new ones
│       ├── excalidraw-skill                   # Live diagramming canvas (bundled)
│       └── impeccable                         # Frontend/UI design system (bundled)
```

## How Skills Work

Skills are domain-knowledge modules that Claude loads on demand. They contain code patterns, anti-patterns, and conventions specific to engineering workflows.

### Automatic Activation

When you type a prompt, the skill evaluation engine (`.claude/hooks/skill-eval.js`) runs automatically and matches your prompt against trigger rules in `skill-rules.json`. It scores matches using:

- **Keywords** (2 pts) — e.g., "diagram", "sketch", "flowchart"
- **Keyword patterns** (3 pts) — regex matches like `\bexcalidraw\b`
- **Intent patterns** (4 pts) — e.g., "draw.*architecture" or "visualize.*pipeline"
- **Path patterns** (4 pts) — file paths in your prompt matching globs like `**/*.excalidraw`
- **Content patterns** (3 pts) — code snippets containing markers like ` ```mermaid `

When a skill scores above the threshold (3 pts), Claude is prompted to evaluate and activate it before proceeding. The engine only surfaces skills that actually exist on disk, so trigger rules can never point at a missing skill.

### Manual Activation

You can also invoke skills directly by name in your prompt:

```
Use the excalidraw-skill to sketch the service architecture
```

Or reference the skill file:

```
Follow the patterns in .claude/skills/excalidraw-skill/SKILL.md
```

### What Each Skill Provides

Every skill follows a consistent structure — **When to Use**, **Core Patterns**, **Anti-Patterns**, and **Integration**. See [`.claude/skills/README.md`](.claude/skills/README.md) for full descriptions and recommended skill combinations.

| Skill | What It Provides |
|-------|------------------|
| `excalidraw-skill` | Live Excalidraw canvas for architecture diagrams and flowcharts; export/import `.excalidraw`, PNG/SVG, and Mermaid (bundled third-party skill) |
| `impeccable` | Frontend/UI design guidance — visual hierarchy, accessibility, theming, and reusable design systems (bundled third-party skill) |

## Hooks

The template includes 7 automated hooks (see `.claude/settings.md` for full documentation):

| Hook | Type | What It Does |
|------|------|-------------|
| Skill evaluation | UserPromptSubmit | Suggests relevant skills based on your prompt |
| Branch protection | PreToolUse | Blocks edits on the `main` branch, suggests creating a feature branch |
| Enforce uv | PreToolUse | Blocks `pip install`, redirects to `uv add` |
| Auto-format | PostToolUse | Runs `ruff format` + import sorting on `.py` files |
| Auto-test | PostToolUse | Runs `pytest` when test files change |
| Lint check | PostToolUse | Runs `ruff check` on edited `.py` files |
| Data/results warning | PostToolUse | Warns when editing files in `data/` or `results/` |

## Permissions

`.claude/settings.json` ships with a **deny-first git posture**: Claude inspects the repo and *drafts* git operations, but never mutates history or publishes changes on your behalf. Blocked via `permissions.deny`:

| Denied | Why |
|--------|-----|
| `git add`, `git commit`, `git push` | Staging/committing/pushing are human-run — Claude drafts the commands and messages instead (see the `git-workflow` agent) |
| `glab mr create`/`merge`, `gh pr create`/`merge` | Opening or merging MRs/PRs is human-run |
| `git worktree`, `EnterWorktree` | Avoids background worktree side effects |
| `cd` | Keeps Bash commands statically analyzable — use absolute paths |
| reading `*.env` | Prevents secret exposure |

`permissions.ask` additionally prompts before `rm` and `mv`, and `defaultMode` is `plan` (Claude proposes a plan for approval before acting). A short `permissions.allow` list pre-approves read-only inspection commands (`git status`, `git diff`, `git log`, `ls`, `wc`, `jq`). To loosen any of these, edit the `permissions` block in `.claude/settings.json`.

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| `engineering-reviewer` | Opus | Proactive code reviewer. Checks numerical correctness (tensor shapes, loss order, gradient flow), reproducibility (seeds, configs, determinism), data integrity (leakage, consistent preprocessing), training loop correctness, and Python quality. |
| `git-workflow` | Sonnet | Read-only git workflow advisor (GitLab-first, GitHub fallback). Drafts branch names (`{initials}/{desc}`, `exp/{name}`), conventional commits, and MR/PR descriptions — outputs commands for you to run rather than committing or opening MRs/PRs itself. |

## Commands

| Command | What It Does |
|---------|-------------|
| `/onboard` | Deep-dives into the codebase and records findings in `.claude/tasks/` for future sessions |
| `/mr-review` | Reviews a merge request (GitLab) or pull request (GitHub) against the `engineering-reviewer` checklist |
| `/mr-summary` | Generates an MR/PR body from `git log main..HEAD` |

## MCP Servers

Configured in `.mcp.json` (all optional — remove what you don't use):

| Server | Purpose | Required Env Vars |
|--------|---------|-------------------|
| GitHub | PR and issue integration | `GITHUB_TOKEN` |
| Notion | Documentation access | `NOTION_API_KEY` |

## Setup: API Keys

MCP servers read credentials from **shell environment variables** (not from a `.env` file). You need to export the required variables before launching Claude Code.

### Option 1: Shell Profile (simplest)

Add to your `~/.zshrc` or `~/.bashrc`:

```bash
export GITHUB_TOKEN="ghp_your_token_here"
export NOTION_API_KEY="ntn_your_key_here"
```

Then reload: `source ~/.zshrc`

### Option 2: settings.local.json (Claude Code only)

Create `.claude/settings.local.json` (already gitignored by this template):

```json
{
  "env": {
    "GITHUB_TOKEN": "ghp_your_token_here",
    "NOTION_API_KEY": "ntn_your_key_here"
  }
}
```

This sets the variables only within Claude Code sessions, not your general shell.

### Where to Get Tokens

| Token | Where to Create |
|-------|----------------|
| `GITHUB_TOKEN` | [GitHub → Settings → Developer settings → Personal access tokens](https://github.com/settings/tokens) — create a token with `repo` scope |
| `NOTION_API_KEY` | [Notion Integrations](https://www.notion.so/my-integrations) — create an internal integration, then share target pages with it |

### Removing an MCP Server

If you don't use a server (e.g., Notion), delete its entry from `.mcp.json`. Claude Code will skip servers that aren't configured.

## Customization

### Adapting CLAUDE.md

Update the Quick Facts section with your actual stack, commands, and directory layout. The template assumes Python/PyTorch/uv, but the patterns work with JAX, TensorFlow, or any framework.

The `data/` directory follows the [cookiecutter-data-science](https://cookiecutter-data-science.drivendata.org/) convention: `raw/` (immutable originals), `interim/` (intermediate transforms), `processed/` (training-ready), `external/` (third-party).

### Adding a New Skill

1. Create `.claude/skills/your-skill/SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: your-skill
   description: What it does and when to use it. Include trigger keywords.
   ---
   ```
2. Include sections: When to Use, Core Patterns, Anti-Patterns, Integration
3. Add trigger rules to `.claude/hooks/skill-rules.json`
4. Update `.claude/skills/README.md`

See the [Agent Skills Specification](https://agentskills.io/specification) for the full SKILL.md format.

### Modifying Hooks

Edit `.claude/settings.json` to add, remove, or adjust hooks. See `.claude/settings.md` for documentation on all hooks, response formats, and exit codes.

### Disabling the enforce-uv Hook

If your project uses `pip` or `conda` instead of `uv`, remove the enforce-uv `PreToolUse` entry from `.claude/settings.json`.

## References

This template incorporates patterns from:

- [pydevtools.com](https://pydevtools.com/handbook/tutorial/set-up-a-python-project-for-claude-code/) — enforce-uv hook pattern
- [Agent Skills Specification](https://agentskills.io/specification) — SKILL.md format standard
