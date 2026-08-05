# Claude Code Skills

This directory contains engineering skills that provide Claude with domain knowledge and best practices for engineering projects.

## Skills by Category

### Reproducibility & Debugging
| Skill | Description |
|-------|-------------|
| [reproducibility](./reproducibility/SKILL.md) | Seed management, environment pinning, deterministic operations, provenance tracking |
| [systematic-debugging](./systematic-debugging/SKILL.md) | Four-phase debugging for engineering code, NaN diagnosis, OOM fixes, shape debugging |

### Diagramming & Visualization
| Skill | Description |
|-------|-------------|
| [excalidraw-skill](./excalidraw-skill/SKILL.md) | Sketch, edit, and refine diagrams on a live Excalidraw canvas; export/import `.excalidraw`, PNG/SVG, and Mermaid |

`excalidraw-skill` gives Claude a live drawing canvas for architecture diagrams, pipeline sketches, and flowcharts. It is the bundled skill from the [mcp_excalidraw](https://github.com/yctimlin/mcp_excalidraw) server, which is wired into `.mcp.json` as the `excalidraw` MCP server (26 drawing tools backed by a canvas at `http://127.0.0.1:3000`). The skill drives the same canvas via MCP tools, its CLI (`npx -y mcp-excalidraw-server <command>`), or REST, following a plan → draw → screenshot-verify → refine loop. Open `http://127.0.0.1:3000` in a browser for screenshots, image export, and Mermaid conversion.

## Skill Combinations for Common Tasks

### Debugging an Issue
1. **systematic-debugging** — Root cause analysis
2. **reproducibility** — Consistent reproduction

### Sketching a Diagram
1. **excalidraw-skill** — Plan layout, draw on the canvas, screenshot-verify, refine

## How Skills Work

Skills are automatically suggested when Claude recognizes relevant context in your prompt. Each skill provides:

- **When to Use** — Trigger conditions
- **Core Patterns** — Best practices with code examples
- **Anti-Patterns** — What to avoid
- **Integration** — How skills connect

## Adding New Skills

1. Create directory: `.claude/skills/skill-name/`
2. Add `SKILL.md` (case-sensitive) with YAML frontmatter:
   ```yaml
   ---
   name: skill-name
   description: What it does and when to use it. Include trigger keywords.
   ---
   ```
3. Include standard sections: When to Use, Core Patterns, Anti-Patterns, Integration
4. Add to this README
5. Add triggers to `.claude/hooks/skill-rules.json`

**Important:** The `description` field is critical — Claude uses semantic matching on it to decide when to apply the skill. Include keywords users would naturally mention.
