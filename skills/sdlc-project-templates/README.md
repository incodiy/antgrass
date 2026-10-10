# SDLC — Project Templates

Project-type registry, gap detection engine, compliance engine, and default scaffolding templates. Defines project archetypes (API, full-stack, library, CLI tool, etc.) with their required gates, artifacts, and workflows.

**Loaded:** Automatically by `sdlc-gate-prd` during G0 to detect gaps and scaffold project structure per type. Not loaded independently — triggered by the SDLC orchestrator.

**Produces:** Gap analysis reports, project skeleton with recommended directory structure, compliance checklists per project type, and scaffolding templates for new projects.

**Changes:** E-Commerce & Microservices scaffolds standardized to `.opencode/artifacts/` structure, Self-Check → Self-Enforcement naming.

[Full instructions → SKILL.md](./SKILL.md)
