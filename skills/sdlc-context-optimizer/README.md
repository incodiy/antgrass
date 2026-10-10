# SDLC — Context Optimizer

Context budget management for LLM sessions. Handles token budget allocation, loading matrix optimization, auto-recovery from context overflow, archetype learning to improve future sessions, and session continuity optimization.

**Loaded:** Automatically by the SDLC orchestrator when token budget is tight, context needs trimming, or session recovery is needed. Can also be triggered manually.

**Produces:** Context budget reports, token allocation plans, loading matrix configurations, auto-recovery snapshots, archetype profiles learned from past sessions, and optimization recommendations.

**Changes:** Archetype field names unified to English (`common_mistakes` → `common_pitfalls`), Self-Check → Self-Enforcement naming.

[Full instructions → SKILL.md](./SKILL.md)
