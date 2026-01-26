# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This is a **Skills development project** for creating reusable Claude Code skills. Skills are modular packages that extend Claude's capabilities with specialized workflows, prompts, and reference materials.

## Skill Structure

Each skill follows this standard structure:

```
skill-name/
├── SKILL.md              # Required: Main skill documentation with YAML frontmatter
└── references/           # Optional: Supporting documentation and templates
    ├── prompts/          # Prompt templates
    ├── styles/           # Style definitions (for visual skills)
    └── layouts/          # Layout templates (for graph/diagram skills)
```

### SKILL.md Format

Every SKILL.md must have:
1. **YAML frontmatter** with `name` and `description` fields
2. **Markdown body** with workflow instructions, examples, and usage patterns

```yaml
---
name: skill-name
description: When to use this skill and what it does
---
```

## Current Skills

| Skill | Purpose |
|-------|---------|
| `cover-image/` | Generate article cover images with 8 style presets |
| `knowledge-graph/` | Generate knowledge graphs from documents (JSON triples, Mermaid, images) |

## Skill Development Workflow

1. Create skill folder with `SKILL.md`
2. Add reference files in `references/` subdirectory
3. Use skill-creator's validation: `python3 ~/.claude/skills/skill-creator/scripts/package_skill.py <skill-folder>`
4. Package outputs a `.skill` file (ZIP archive)

## Key Design Patterns

### Dual-Model Architecture
Skills often separate text generation (default model) from image generation (user-specified model like nano banana, DALL-E, Midjourney).

### Processing Modes
For batch operations, support both:
- **Parallel**: Concurrent subtasks when agent supports it
- **Sequential**: Fallback for rate-limited APIs or unsupported agents

### Schema-Constrained Extraction
For knowledge extraction tasks, predefine allowed entity types and relationship types to prevent hallucination.

## Environment

- Python: Use `python3` command (or `python` if it points to Python 3.x)
- If `python3` is not found, ask the user for their Python installation path
- Install Python packages: Use system's package manager or `pip3 install <package>`

## Conventions

- Skill names use `hyphen-case`
- Reference files are markdown (`.md`)
- Image prompts should specify model-specific adaptations
- Support both Chinese and English in user-facing content
- Output files organized in predictable directory structures

## Best Practices for Portable Skills

### Dynamic Path Resolution

**Problem:** Skills installed in different locations need to reference their own scripts and resources.

**Solution:** Implement dynamic path detection at skill initialization:

1. **Search common locations first** (fast path):
   - `~/.claude/skills/<skill-name>` (Claude Code default)
   - `./<skill-name>` (current directory)
   - `~/<skill-name>` (user home)

2. **Use find command as fallback**:
   ```bash
   find ~ -type f -name "<marker-file>" -path "*/<skill-name>/*" 2>/dev/null | head -1
   ```

3. **Ask user if not found**:
   Prompt user for the skill installation path

4. **Store path in session variable**:
   Use `SKILL_PATH` or similar variable for all subsequent script calls

5. **Example pattern**:
   ```bash
   # Find skill path once at initialization
   SKILL_PATH=$(test -f ~/.claude/skills/my-skill/script.py && echo "~/.claude/skills/my-skill" || find ~ -name "script.py" -path "*/my-skill/*" | sed 's|/script.py||')

   # Use in all subsequent calls
   python3 ${SKILL_PATH}/scripts/tool.py --args
   ```

### Python Environment Detection

- Never hardcode Python paths (e.g., `~/.uv/python3/bin/python`)
- Use `python3` as default, fall back to `python` if needed
- Detect and handle missing Python gracefully with installation suggestions
- Support custom Python paths from users
