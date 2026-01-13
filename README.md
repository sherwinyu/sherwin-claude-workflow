# Claude Development Workflow

A reusable, agent-based development workflow for Claude Code. Manages tickets, plans, implementation, and QA through Linear integration and browser automation.

## Features

- **Linear Integration**: All state lives in Linear tickets - no local file duplication
- **Multi-Agent Architecture**: Specialized agents for context, development, and QA
- **Fast-Track Mode**: Well-specced tickets skip exploration and planning phases
- **Human-in-the-Loop**: Plans require approval before implementation
- **Browser Automation**: E2E testing via Claude in Chrome MCP

## Installation

### 1. Copy Workflow Files

Copy the workflow directory into your project:

```bash
# Option A: Copy entire directory
cp -r ~/projects/sherwin-claude-workflow/workflow your-project/path/to/workflow

# Option B: Clone and symlink (keeps updates)
ln -s ~/projects/sherwin-claude-workflow/workflow your-project/path/to/workflow
```

Recommended locations:
- `packages/client/src/modules/your-feature/workflow/`
- `src/workflow/`
- `.claude/workflow/`

### 2. Configure for Your Project

Edit `workflow/CLAUDE.md` and update the Configuration section (or have Claude do it for you when running the skill)

```markdown
| Placeholder     | Value                              |
| --------------- | ---------------------------------- |
| `PROJECT_NAME`  | Your Project Name                  |
| `PROJECT_ID`    | your-linear-project-id             |
| `TEAM_NAME`     | Your Team                          |
| `TEAM_ID`       | TEAM                               |
| `TICKET_PREFIX` | TEAM                               |
| `WORKSPACE`     | your-workspace                     |
| `WORKFLOW_PATH` | path/to/your/workflow              |
| `LINT_COMMAND`  | yarn lint:fix                      |
| `SCOPE`         | your-feature                       |
```

### 3. Update Agent References

In each agent file under `workflow/agents/`, update:
- File paths to match your project structure
- Module names and directories
- Any project-specific patterns

### 4. Install the Claude Code Skill (Optional)

Add the skill to your project's `.claude/skills/` directory:

```bash
mkdir -p your-project/.claude/skills
cp ~/projects/sherwin-claude-workflow/skill.yaml your-project/.claude/skills/workflow.yaml
```

Then users can invoke with:
```
/workflow:load
```

### 5. Reference from CLAUDE.md

Add to your project's main `CLAUDE.md`:

```markdown
## Development Workflow

For feature development with Linear integration, see @path/to/workflow/CLAUDE.md

Quick start:
- "Pick up ticket TEAM-123 and run the workflow"
- "List open tickets for [Project Name]"
```

## Usage

### Quick Start Commands

```
"Pick up ticket TEAM-XXXX and run the workflow"
"List open [Project Name] tickets and help me triage"
"Create a new ticket for [issue description]"
"Let's discuss the requirements for TEAM-XXXX"
```

### Workflow Phases

1. **Ticket Management** - List, read, create tickets in Linear
2. **Assessment** - Determine if ticket is well-specced (fast-track) or needs exploration
3. **Context Agent** - Explore codebase for vague tickets (posts findings to Linear)
4. **Dev Plan** - Generate implementation plan (posts to Linear, awaits approval)
5. **QA Plan** - Generate test plan (posts to Linear)
6. **Dev Agent** - Implement the feature/fix
7. **QA Agent** - Browser automation verification
8. **Results** - Update Linear, ready for human review

### Workflow States

| State      | Meaning                    | Who Sets It    |
| ---------- | -------------------------- | -------------- |
| Backlog    | Ready for pickup           | Human or Claude|
| Developing | Implementation in progress | Claude         |
| In Review  | Awaiting human review      | Claude         |
| Done       | Verified complete          | **HUMAN ONLY** |

## Architecture

```
ORCHESTRATOR (CLAUDE.md)
    |
    +-- 1. Ticket Management (triage, clarify, create)
    +-- 2. ASSESS TICKET -> well-specced? fast-track or full pipeline
    +-- 3. CONTEXT AGENT -> explore codebase (only if needed)
    +-- 4. DEV PLAN -> post to Linear -> human approval (only if needed)
    +-- 5. QA PLAN -> post to Linear -> human approval (unless "no qa")
    +-- 6. DEV AGENT -> implements feature/fix
    +-- 7. QA AGENT -> browser automation verification (unless "no qa")
    +-- 8. Update Linear + knowledge base
```

### Key Principles

1. **Single Source of Truth**: All state lives in Linear
2. **Delegation**: Orchestrator routes to specialized agents via `Task` tool
3. **Human Approval**: Plans require explicit approval before implementation
4. **Escalation**: Agents stop and report blockers immediately

## File Structure

```
workflow/
+-- CLAUDE.md               # Main orchestrator
+-- agents/
|   +-- context.md          # Codebase exploration agent
|   +-- dev.md              # Implementation agent
|   +-- qa.md               # Browser automation agent
+-- knowledge/
    +-- patterns.md         # Accumulated learnings
    +-- edge-cases.md       # Known edge cases
```

## Requirements

- **Claude Code** with MCP support
- **Linear MCP Server** (`mcp__linear__*` or `mcp__linear-server__*` tools)
- **Claude in Chrome MCP** (for QA agent browser automation)

## Customization

### Adding New Agents

Create a new file in `agents/` following the pattern:

```markdown
# Agent Name

Purpose description.

## REQUIRED Return Format
[What the agent must return to orchestrator]

## Input
[What the agent receives]

## Workflow
[Step-by-step process]

## Escalation Protocol
[When and how to stop and report blockers]
```

### Updating Knowledge Base

After each workflow run, update:
- `knowledge/patterns.md` - Add new patterns discovered
- `knowledge/edge-cases.md` - Document edge cases encountered

## License

MIT
