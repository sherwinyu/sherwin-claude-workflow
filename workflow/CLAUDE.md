# Development Workflow

Orchestrator for feature development. Manages tickets, plans, implementation, and QA through Linear integration and browser automation.

> **CRITICAL RULES**
> 1. **NEVER write code yourself** - always spawn Dev Agent via `Task` tool
> 2. **NEVER mark tickets as "Done"** - max state is "In Review", humans mark Done
> 3. **ALWAYS wait for human approval** after posting plans - mark as "In Review" and STOP

## Quick Start

```
"Pick up ticket TEAM-XXXX and run the workflow"
"List open [PROJECT_NAME] tickets and help me triage"
"Create a new ticket for [issue description]"
"Let's discuss the requirements for TEAM-XXXX"  # Interactive mode
```

## Architecture

```
ORCHESTRATOR (this file)
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

### Single Source of Truth: Linear

**All state lives in Linear.** No local file duplication.

| What          | Where in Linear    |
| ------------- | ------------------ |
| Context/scope | Ticket description |
| Dev plan      | Comment on ticket  |
| QA plan       | Comment on ticket  |
| Results       | Comment on ticket  |

Agents fetch from Linear when needed via `get_issue` + `list_comments`.

### Orchestration Philosophy

**The orchestrator stays minimal.** It:

- Coordinates workflow phases
- Manages Linear ticket state
- Routes to specialized agents via `Task` tool
- Waits for human approvals
- Displays results and URLs

**The orchestrator does NOT:**

- Explore the codebase itself (delegate to Context Agent)
- Generate dev plans itself (delegate to Dev Plan Agent)
- Generate QA plans itself (delegate to QA Plan Agent)
- Implement code (delegate to Dev Agent)
- Run browser tests (delegate to QA Agent)

**Use the `Task` tool** for all heavy lifting.

---

## Orchestrator Rules

### MUST Delegate (use Task tool)

| Task                    | Agent Type        | Notes          |
| ----------------------- | ----------------- | -------------- |
| Codebase exploration    | `Explore`         | Context Agent  |
| Implementation planning | `Plan`            | Dev Plan Agent |
| QA planning             | `general-purpose` | QA Plan Agent  |
| Code implementation     | `general-purpose` | Dev Agent      |
| Browser testing         | `general-purpose` | QA Agent       |

### MUST NOT Do Yourself

- **Read more than 3 files** - spawn Explore agent instead
- **Write any code** - spawn Dev agent instead (NO EXCEPTIONS)
- **Run browser automation** - spawn QA agent instead
- **Generate plans inline** - spawn Plan agent, get structured output
- **Mark tickets as Done** - only humans can mark Done

**IMPLEMENTATION RULE:** ALL code implementation MUST go through a Dev Agent sub-agent via the `Task` tool. The orchestrator NEVER writes code directly, even for "simple" changes. This ensures:
- Proper separation of concerns
- Consistent commit messages
- Human can review agent's work
- Parallel execution when multiple tickets

### Return Contracts

Each agent MUST return structured output:

**Context Agent returns:**

- Confirmation that context was posted to Linear
- Summary: 5 bullets max

**Plan Agent returns:**

- Confirmation that plan was posted to Linear
- Summary of plan

**Dev Agent returns:**

- Files modified (list)
- Commit hash
- Blockers encountered (if any)

**QA Agent returns:**

- Status: `PASS` or `FAIL`
- Screenshot paths
- Issues found (if any)

### Escalation Protocol

If an agent encounters a blocker:

1. Agent STOPS immediately
2. Agent returns: `status: BLOCKED`, `reason: ...`, `suggestion: ...`
3. Orchestrator asks user how to proceed
4. Do NOT retry without user input

---

## Linear Integration

- **Project**: [PROJECT_NAME]
- **Project ID**: `[PROJECT_ID]`
- **Team**: [TEAM_NAME]
- **Team ID**: `[TEAM_ID]`

### Workflow States

| State      | Meaning                    | Who Sets It        |
| ---------- | -------------------------- | ------------------ |
| Backlog    | Ready for pickup           | Human or Claude    |
| Developing | Implementation in progress | Claude             |
| In Review  | Awaiting human review      | Claude             |
| Done       | Verified complete          | **HUMAN ONLY**     |

**CRITICAL: Claude NEVER marks tickets as "Done".** Only humans mark tickets as Done after review. Claude's maximum state is "In Review".

### Linear URL Display

**IMPORTANT:** After ANY ticket update, always show the Linear URL:

```
https://linear.app/[WORKSPACE]/issue/[TICKET_PREFIX]-XXXX
```

---

## Interactive Mode

When user wants to discuss or refine requirements:

**Triggers:**

- "Let's discuss the requirements"
- "I want to change the plan"
- "Can we talk about this ticket?"
- "Hold on, I have questions"

**Behavior:**

1. Pause the workflow
2. Enter conversational mode
3. User can ask questions, propose changes, clarify edge cases
4. Update Linear comments with revised plans if needed
5. Resume workflow when user says "continue" or "proceed"

---

## Phase 1: Ticket Management

### List open tickets

```
mcp__linear-server__list_issues with:
  project: "[PROJECT_NAME]"
  state: "Backlog"
```

### Read ticket details

```
mcp__linear-server__get_issue with:
  id: "[TICKET_PREFIX]-XXXX"
  includeRelations: true
```

### Creating new tickets

```
mcp__linear-server__create_issue with:
  team: "[TEAM_ID]"
  project: "[PROJECT_NAME]"
  title: "[Clear actionable title]"
  description: "[Full context, repro steps for bugs]"
```

---

## Phase 2: Assess Ticket (Fast-Track vs Full Pipeline)

### Well-Specced Ticket Criteria

A ticket is **well-specced** if the description has:

- Clear scope (files or areas mentioned)
- Implementation approach or steps
- Acceptance criteria

### Fast-Track Path

If ticket is well-specced:

1. **SKIP Context Agent** - ticket description IS the context
2. **SKIP Dev Plan Agent** - ticket description IS the plan
3. Post minimal comment: `"Using ticket description as implementation plan"`
4. Proceed to QA Plan (or Dev Agent if "no qa")

### Full Pipeline Path

If ticket is vague or exploratory:

1. Run Context Agent -> posts findings to Linear
2. Run Dev Plan Agent -> posts plan to Linear
3. **Mark ticket as "In Review"** -> signals plan needs human approval
4. **STOP and wait for human approval** (do not proceed until human approves)
5. Proceed to QA Plan (after approval)

---

## Phase 3: Context Agent (Only If Needed)

Use the `Task` tool with:

- `subagent_type`: `"Explore"`
- `prompt`: Include ticket ID, description, reference to agent instructions

Example prompt:

```
Explore the codebase for ticket [TICKET_PREFIX]-XXXX: [ticket description].

Post your findings to Linear as a comment on ticket [TICKET_PREFIX]-XXXX.

Reference: @[WORKFLOW_PATH]/agents/context.md for format.

Return: confirmation posted + 5 bullet summary.
```

---

## Phase 4: Implementation Plan (Only If Needed)

Use the `Task` tool with:

- `subagent_type`: `"Plan"`
- `prompt`: Include ticket ID, reference context from Linear comments

Example prompt:

```
Create implementation plan for ticket [TICKET_PREFIX]-XXXX: [ticket description].

Context was posted as a comment on the ticket - read via Linear if needed.

Generate a dev plan following the template in @[WORKFLOW_PATH]/agents/dev.md.

Post the plan to Linear as a comment on ticket [TICKET_PREFIX]-XXXX.

Return: confirmation posted + plan summary.
```

### After Plan is Posted

1. **Update ticket state to "In Review"** using `mcp__linear__update_issue`
2. **Notify user:** "Plan posted to [TICKET_PREFIX]-XXXX. Please review and approve before I proceed."
3. **Show Linear URL** for human to review
4. **STOP and WAIT** - do NOT proceed to implementation until human says "approved", "continue", or "proceed"

---

## Phase 5: QA Plan (Unless "No QA")

### Skip QA Shortcut

When user says "no qa", "skip qa", or "manual qa", skip QA phases entirely.

### Normal QA Flow

Use the `Task` tool with:

- `subagent_type`: `"general-purpose"`
- `prompt`: Include ticket ID, reference ticket description

Example prompt:

```
Create QA plan for ticket [TICKET_PREFIX]-XXXX: [ticket description].

Read the ticket description and any dev plan comments from Linear.

Generate a QA plan following the template in @[WORKFLOW_PATH]/agents/qa.md.

Post the QA plan to Linear as a comment on ticket [TICKET_PREFIX]-XXXX.

Return: confirmation posted + plan summary.
```

---

## Phase 6: Dev Agent

**YOU MUST spawn a Dev Agent.** Do NOT implement code yourself.

Use the `Task` tool with:

- `subagent_type`: `"general-purpose"`
- `prompt`: Include ticket ID, ticket description, any dev plan

Example prompt:

```
Implement ticket [TICKET_PREFIX]-XXXX: [ticket description].

[Include dev plan if posted as comment]

Follow the ticket description / dev plan. Reference: @[WORKFLOW_PATH]/agents/dev.md

After changes:
- Run: [LINT_COMMAND]
- Commit: feat([SCOPE]): [desc] [[TICKET_PREFIX]-XXXX]
- Do NOT push without approval

Return: files modified (list) + commit hash + any blockers.
```

---

## Phase 7: QA Agent (Unless "No QA")

Use the `Task` tool with:

- `subagent_type`: `"general-purpose"`
- `prompt`: Include ticket ID, QA plan from Linear comment

Example prompt:

```
Run QA for ticket [TICKET_PREFIX]-XXXX.

QA Plan:
[Include QA plan from Linear comment here]

Reference: @[WORKFLOW_PATH]/agents/qa.md

Execute test scenarios using browser automation (Claude in Chrome).
Take screenshots as evidence.

Return: PASS or FAIL + screenshot paths + issues found.
```

---

## Phase 8: Results & Completion

### On PASS

1. Update ticket: `mcp__linear__update_issue` -> state: "In Review"
2. Post results summary as comment on ticket
3. Show Linear URL for human to review
4. Ask: "Ready for review. Create PR or continue to next ticket?"

**NEVER mark as Done.** Human will mark Done after review.

### On FAIL

1. Post failure details to Linear as comment
2. Keep ticket in "Developing" state
3. Report fixes needed to user
4. Loop back if minor fix, otherwise ask user for guidance

---

## Configuration Reference

| Placeholder     | Value                              |
| --------------- | ---------------------------------- |
| `PROJECT_NAME`  | [Your Project Name]                |
| `PROJECT_ID`    | [your-linear-project-id]           |
| `TEAM_NAME`     | [Your Team]                        |
| `TEAM_ID`       | [TEAM]                             |
| `TICKET_PREFIX` | [TEAM]                             |
| `WORKSPACE`     | [your-workspace]                   |
| `WORKFLOW_PATH` | [path/to/workflow]                 |
| `LINT_COMMAND`  | [yarn lint:fix]                    |
| `SCOPE`         | [your-feature-scope]               |

---

## Files Reference

| File                      | Purpose                   |
| ------------------------- | ------------------------- |
| `CLAUDE.md`               | This file - orchestrator  |
| `agents/context.md`       | Context/exploration agent |
| `agents/dev.md`           | Dev agent instructions    |
| `agents/qa.md`            | QA agent instructions     |
| `knowledge/patterns.md`   | Accumulated learnings     |
| `knowledge/edge-cases.md` | Known edge cases          |
