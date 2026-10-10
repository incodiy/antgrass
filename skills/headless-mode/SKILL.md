---
name: headless-mode
description: Non-interactive (headless) execution untuk CI/CD pipelines, webhook integration, GitHub Actions, dan automated scripting. Provider-agnostic.
version: 1.0.0
updated: 2026-06-21
---

# Headless Mode

> Headless mode = non-interactive execution tanpa user di-loop. Cocok buat CI/CD pipelines, automation scripts, dan GitHub Actions workflows.

---

## 1. Headless vs Interactive

| Aspek | Interactive | Headless |
|-------|-------------|----------|
| **Input** | Chat / prompt langsung | Script / API / webhook |
| **Output** | Real-time streaming | File / STDOUT / PR |
| **Approval** | Manual (user confirm) | Auto (policy-driven) |
| **Use case** | Daily dev, explorasi | CI/CD, batch processing |
| **Error handling** | Langsung tanya user | Log + exit code |

---

## 2. Headless Execution Patterns

### Pattern 1: Pipe-based

```bash
# Execute from stdin
echo "Refactor this function to use async/await" | agent --headless

# Execute with input file
agent --headless < task-prompt.md

# Pipe dari command lain
cat bugs.txt | agent --headless "Fix all listed bugs"

# Output ke file
agent --headless "Generate unit tests for src/utils.ts" > output.md
```

### Pattern 2: CLI Flags

```bash
# Force headless
agent --headless "Run tests and fix failures"

# Set output format
agent --headless --output json "Analyze codebase"

# Timeout
agent --headless --timeout 300 "Run migration script"

# Exit on first error
agent --headless --fail-fast "Run all linters"
```

---

## 3. Webhook / API Integration

### Webhook Receiver

```yaml
# app.js — webhook endpoint
app.post('/webhook/agent', async (req, res) => {
  const { prompt, model, sandbox } = req.body;

  const result = await runHeadless({
    prompt,
    model: model || 'gpt-4o-mini',
    sandbox: sandbox || 'seatbelt',
    timeout: 120,
  });

  res.json({
    status: 'completed',
    output: result,
    exit_code: 0,
  });
});
```

### Headless Execution Protocol

```
Request:
{
  "prompt": string,
  "model"?: string,
  "sandbox"?: "bubble-wrap" | "seatbelt",
  "approval_policy"?: "untrusted" | "request" | "never",
  "timeout"?: number,
  "session_id"?: string
}

Response:
{
  "id": string,
  "status": "running" | "completed" | "failed",
  "output": object,
  "usage": { "input_tokens": number, "output_tokens": number }
}
```

### Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | Proceed to next step |
| 1 | General error | Check logs, retry |
| 2 | Timeout | Increase timeout or split task |
| 3 | Sandbox violation | Review approval policy |
| 4 | Rate limited | Backoff and retry |

---

## 4. CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/agent-automation.yml
name: Agent Automation

on:
  issues:
    types: [labeled]
  pull_request:
    types: [opened]

jobs:
  agent-execute:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Execute Agent
        env:
          LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
        run: |
          agent --headless \
            --model gpt-4o-mini \
            --sandbox seatbelt \
            --approval-policy never \
            --timeout 300 \
            "${{ github.event.issue.title }}: ${{ github.event.issue.body }}"

      - name: Create PR
        uses: peter-evans/create-pull-request@v6
        with:
          commit-message: "🤖 Auto-fix: ${{ github.event.issue.title }}"
          branch: fix/auto-${{ github.event.issue.number }}
```

### GitLab CI

```yaml
# .gitlab-ci.yml
agent-job:
  script:
    - agent --headless --output diff "Fix lint errors"
  artifacts:
    paths:
      - output.diff
```

---

## 5. Logging & Monitoring

### Structured Logging

```json
{
  "timestamp": "2026-06-21T10:30:00Z",
  "level": "info",
  "session_id": "sess_abc123",
  "event": "headless_execution",
  "status": "completed",
  "exit_code": 0,
  "duration_ms": 45200,
  "tokens_used": 2450,
  "model": "gpt-4o-mini"
}
```

### Monitoring Checklist

```
□ Log semua execution (stdout + file)
□ Track exit codes — non-zero = alert
□ Set timeout — cegah infinite loop
□ Monitor token usage per pipeline run
□ Alert on repeated failures (3+ consecutive)
```

---

## 6. Integration dengan sdlc Pre-Flight

`sdlc` Pre-Flight item 6a:

```
□ [6a] Headless Mode Check
    → Tanya user: "Task ini jalan di headless mode?"
    → Jika ya → setup CI/CD pipeline
    → Pastikan approval_policy = never + sandbox = seatbelt
```

---

## 🚨 Self-Enforcement

**Sebelum setup headless mode di CI/CD, WAJIB cek:**

```
□ Task cocok untuk headless? (repetitive, automatable)
□ Approval policy = never? → WAJIB sandbox = seatbelt
□ API key disimpan di secrets (bukan hardcode)?
□ Ada timeout biar gak infinite loop?
□ Output format sesuai untuk pipeline? (diff / json / markdown)
□ Rollback plan ada kalau agent ngerusak sesuatu?
```
