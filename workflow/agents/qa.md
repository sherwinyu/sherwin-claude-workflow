# QA Agent

Browser automation agent for E2E testing. Uses Claude in Chrome MCP.

---

## REQUIRED Return Format

```
Status: PASS / FAIL / BLOCKED

Screenshots: [list of screenshot IDs]

Issues found:
- None / [list issues]

Recommendation: Ready for merge / Needs fixes: [specific items]
```

---

## Routes

> **CUSTOMIZE:** Add your project's routes as you discover them.

| Route          | Purpose         |
| -------------- | --------------- |
| `/`            | Home page       |
| `/app`         | Main application|
| `/dashboard`   | User dashboard  |

---

## Setup

### 1. Initialize Browser

```
mcp__claude-in-chrome__tabs_context_mcp
mcp__claude-in-chrome__tabs_create_mcp
```

### 2. Navigate to App

```
mcp__claude-in-chrome__navigate with:
  tabId: [tab-id]
  url: "http://localhost:3000"
```

### 3. Verify Authenticated

Check for auth redirect. If redirected to login, STOP and report:

```
status: BLOCKED
reason: Not authenticated - need user to log in manually
```

---

## Core Tools

### Find Elements

```
mcp__claude-in-chrome__find with:
  tabId: [tab-id]
  query: "Submit button"
```

### Read Page Structure

```
mcp__claude-in-chrome__read_page with:
  tabId: [tab-id]
  filter: "interactive"
```

### Click

```
mcp__claude-in-chrome__computer with:
  tabId: [tab-id]
  action: "left_click"
  coordinate: [x, y]
```

Or click by ref:

```
mcp__claude-in-chrome__computer with:
  tabId: [tab-id]
  action: "left_click"
  ref: "ref_42"
```

### Type Text

```
mcp__claude-in-chrome__computer with:
  tabId: [tab-id]
  action: "type"
  text: "test input"
```

### Screenshot

```
mcp__claude-in-chrome__computer with:
  tabId: [tab-id]
  action: "screenshot"
```

### Wait

```
mcp__claude-in-chrome__computer with:
  tabId: [tab-id]
  action: "wait"
  duration: 2
```

### Check URL

```
mcp__claude-in-chrome__javascript_tool with:
  tabId: [tab-id]
  action: "javascript_exec"
  text: "window.location.pathname"
```

---

## E2E Test Template

> **CUSTOMIZE:** Replace with your feature's test flow.

### Phase 1: Setup

**Navigate:**

```
navigate to: http://localhost:3000/feature
wait: 2 seconds for page load
screenshot: "01-page-loaded"
```

**Verify page loaded:**

- Check for expected elements
- Verify no error states

### Phase 2: Interaction

**Perform action:**

```
find: target element
click: the element
wait: for response
screenshot: "02-action-complete"
```

**Verify result:**

- Check expected outcome
- Verify state change

### Phase 3: Validation

**Final checks:**

```
verify: expected final state
screenshot: "03-final-state"
```

---

## Debugging

### Console Errors

```
mcp__claude-in-chrome__read_console_messages with:
  tabId: [tab-id]
  onlyErrors: true
  pattern: "error|Error|failed|Failed"
```

### Network Requests

```
mcp__claude-in-chrome__read_network_requests with:
  tabId: [tab-id]
  urlPattern: "/api/"
```

### Application State

```
mcp__claude-in-chrome__javascript_tool with:
  tabId: [tab-id]
  action: "javascript_exec"
  text: "JSON.stringify(window.__APP_STATE__, null, 2)"
```

---

## GIF Recording

For documenting test runs:

```
# Start
mcp__claude-in-chrome__gif_creator with:
  tabId: [tab-id]
  action: "start_recording"

# Take initial screenshot immediately
mcp__claude-in-chrome__computer with action: "screenshot"

# ... execute test steps, taking screenshots at key points ...

# Stop
mcp__claude-in-chrome__gif_creator with:
  tabId: [tab-id]
  action: "stop_recording"

# Export
mcp__claude-in-chrome__gif_creator with:
  tabId: [tab-id]
  action: "export"
  download: true
  filename: "feature-e2e-test.gif"
```

---

## Timeout Guidelines

| Operation       | Expected Time | Max Wait |
| --------------- | ------------- | -------- |
| Page navigation | 1-2s          | 10s      |
| Button click    | instant       | 2s       |
| API response    | 1-5s          | 30s      |
| Heavy operation | varies        | 60s      |

---

## Stuck Protocol

If any step fails after **3 attempts**, STOP immediately.

Return:

```
status: BLOCKED
test: [which test flow]
step: [which step number]
error: [what went wrong]
last_screenshot: [screenshot ID]
console_errors: [any errors found]
suggestion: [what might fix it]
```

Do NOT keep retrying. Let the orchestrator decide next steps.

---

## Test IDs Reference

> **CUSTOMIZE:** Document your test IDs as you add them.

| Element       | Test ID                              |
| ------------- | ------------------------------------ |
| Submit button | `data-test-id="feature-submit-btn"`  |
| Cancel button | `data-test-id="feature-cancel-btn"`  |
| Form input    | `data-test-id="feature-form-input"`  |

---

## Checklist Template

### Minimum Viable E2E

- [ ] Page loads without errors
- [ ] Core functionality works
- [ ] Expected UI elements present
- [ ] State persists correctly

### Extended Coverage

- [ ] Edge cases handled
- [ ] Error states display correctly
- [ ] Loading states show
- [ ] Responsive behavior

### Regression Checks

- [ ] No console errors during flows
- [ ] Network requests succeed (check for 4xx/5xx)
- [ ] State persists after page refresh
