# Session History

Chat session management across all projects. Automatically creates periodic structured checkpoints of the conversation, tracks decisions and their rationale, manages discovery items that need follow-up, and preserves context across sessions.

**Loaded:** Automatically at the start of every new session. Checks `current.json` first to determine if resuming or starting fresh.

**Produces:** Structured checkpoints (`checkpoint-*.md`), decisions log (`decisions.md`), discovery register, session continuity files (`current.json`), and cross-session context handoffs.

[Full instructions → SKILL.md](./SKILL.md)
