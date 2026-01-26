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
| `classical-poetry/` | Create classical Chinese poetry (诗/词/曲/对联) with meter and rhyme validation |
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

### 1. Path Convention (SKILL_ROOT Pattern)

**Problem:** Skills with scripts/resources need to work regardless of installation location.

**❌ Bad Practice:**
```bash
# Hardcoded path - only works on one machine
python3 ~/.uv/python3/bin/python ~/.claude/skills/my-skill/scripts/tool.py

# Relative path without context - fails in different working directories
python3 ../my-skill/scripts/tool.py
```

**✅ Good Practice: SKILL_ROOT Convention**

#### Step 1: Define SKILL_ROOT in SKILL.md

Add this section at the beginning of your SKILL.md (after frontmatter, before workflow):

```markdown
## 路径约定 / Path Convention

**SKILL_ROOT 定义：**
- **Claude Code 用户**：`SKILL_ROOT="$HOME/.claude/skills/<skill-name>"`
- **其他环境用户**：首次使用时询问 skill 安装路径，记录为 `SKILL_ROOT`，整个会话复用

所有脚本调用使用格式：`python3 $SKILL_ROOT/scripts/xxx.py`

---

**SKILL_ROOT definition:**
- **Claude Code users**: `SKILL_ROOT="$HOME/.claude/skills/<skill-name>"`
- **Other environments**: Ask user for skill path on first use, store as `SKILL_ROOT` for session

All script calls use format: `python3 $SKILL_ROOT/scripts/xxx.py`
```

#### Step 2: Use SKILL_ROOT in All Script Calls

**Example from classical-poetry skill:**

```bash
# Reference builder
python3 $SKILL_ROOT/scripts/reference_builder.py \
  --keyword "中秋" --pages 2 --out "/tmp/refs.json"

# Poetry checker
python3 $SKILL_ROOT/scripts/poetry_checker.py \
  --mode shi --text "明月几时有" --yun-shu 1
```

#### Step 3: Handle First-Time Path Detection

**For Claude Code users** (automatic):
```bash
# In Claude Code environment, CLAUDECODE=1 is set
SKILL_ROOT="$HOME/.claude/skills/<skill-name>"
```

**For other environments** (ask once):

When first script call is about to execute, if `SKILL_ROOT` is not set:

```markdown
（中文提示）
请告诉我 <skill-name> skill 的安装路径
（例如：/Users/username/skills/<skill-name> 或 ~/my-skills/<skill-name>）

（English prompt）
Please provide the installation path for <skill-name> skill
(e.g., /Users/username/skills/<skill-name> or ~/my-skills/<skill-name>)
```

Store the provided path as `SKILL_ROOT` and use it for the entire session.

#### Benefits

- ✅ **Simple**: No complex search logic, just a convention
- ✅ **Explicit**: Clear what the variable represents
- ✅ **Portable**: Works on any platform and installation location
- ✅ **User-friendly**: Claude Code users have zero configuration, others provide path once
- ✅ **Session-cached**: No repeated asking during the same conversation

#### Anti-Patterns to Avoid

❌ **Don't do 3-tier path search** (too complex):
```bash
# Too complicated - avoid this
test -f ~/.claude/skills/my-skill/script.py || \
test -f ./my-skill/script.py || \
find ~ -name "script.py" ...
```

❌ **Don't ask "where is the skill"** when user is already using it:
```
# Bad UX - user just invoked the skill!
"Please locate the my-skill installation..."
```

❌ **Don't change working directory**:
```bash
# Avoid - affects user's environment
cd $SKILL_ROOT && python3 scripts/tool.py
```

### 2. Python Environment Detection

**❌ Bad Practice:**
```bash
# Hardcoded to specific machine
~/.uv/python3/bin/python script.py
/usr/local/bin/python3 script.py
```

**✅ Good Practice:**

#### Use Standard Commands

```bash
# Default - works on most systems
python3 $SKILL_ROOT/scripts/tool.py
```

#### Handle Missing Python Gracefully

When first script call fails with "command not found":

**Chinese prompt:**
```
检测到您的系统中未找到 Python 3。

本技能需要 Python 3 来运行工具脚本。

选项 1：如果您已安装 Python 3，请告诉我完整路径（例如：/usr/local/bin/python3）
选项 2：如果尚未安装，建议通过以下方式安装：
- macOS: brew install python3
- Ubuntu/Debian: sudo apt install python3
- Windows: 从 python.org 下载安装
```

**English prompt:**
```
Python 3 not found on your system.

This skill requires Python 3 to run tool scripts.

Option 1: If Python 3 is installed, please provide the full path (e.g., /usr/local/bin/python3)
Option 2: If not installed, install via:
- macOS: brew install python3
- Ubuntu/Debian: sudo apt install python3
- Windows: Download from python.org
```

#### Fallback Order

Try in sequence:
1. `python3` (preferred)
2. `python` (if it points to Python 3.x, check with `python --version`)
3. User-provided custom path

Store the working command and reuse for the session.

### 3. Complete Example Template

Here's a complete template for a portable skill:

```markdown
---
name: my-skill
description: What this skill does and when to use it
---

# My Skill Workflow

Description of the skill...

## Path Convention

**SKILL_ROOT definition:**
- **Claude Code users**: `SKILL_ROOT="$HOME/.claude/skills/my-skill"`
- **Other environments**: Ask user for skill path on first use, store as `SKILL_ROOT` for session

All script calls use format: `python3 $SKILL_ROOT/scripts/xxx.py`

## Stage 1: First Step

...workflow steps...

### Using the Tool

```bash
python3 $SKILL_ROOT/scripts/my_tool.py --arg1 value1 --arg2 value2
```

## Technical Reference

### Scripts
- `scripts/my_tool.py` - Main processing script
- `scripts/helper.py` - Helper utilities

### Environment Requirements
- Python 3.7+
- See "Path Convention" section for SKILL_ROOT setup
```

### 4. Session State Management

**Key Principle:** Once set, SKILL_ROOT and Python command should persist for the entire session.

**Implementation:**
- Set `SKILL_ROOT` once at first use
- Set Python command once at first use
- Reuse both for all subsequent calls
- No repeated detection or asking

**Example session flow:**
1. User invokes skill
2. First script call needed
3. If Claude Code: Auto-set `SKILL_ROOT="$HOME/.claude/skills/my-skill"`
4. If other: Ask user once, store response
5. Try `python3`, if fails ask user or suggest installation
6. Execute script: `python3 $SKILL_ROOT/scripts/tool.py`
7. All subsequent calls reuse same `SKILL_ROOT` and `python3`
