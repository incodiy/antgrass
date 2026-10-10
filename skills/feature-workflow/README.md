# Feature Workflow — FSM Orchestrator v1.1.1

Finite State Machine controller for enterprise feature execution. 3 operation modes: FULL (new features, 6 phases: PM → Architect → Developer → PM Verify → Tech Lead Audit → Execution), QUICK (bugs/refactors, 3 phases), and HOTFIX (emergency, 1 phase). Includes safety latch and 4-layer retry mechanism.

**Loaded:** Automatically by `sdlc-gate-development` (G3) during complex feature tasks, or manually when asked for structured feature development, bug fixing, or hotfix workflows.

**Produces:** Phase transition logs, persona-specific outputs from each phase (PM specs, architecture plans, implementation plans, audit reports), safety latch activation records, and execution-ready task breakdowns.

**v1.1.1 changes:** BLUEPRINT update step added to Workflow Completion (step 2), ensuring task status syncs after feature completion.

[Full instructions → SKILL.md](./SKILL.md)
