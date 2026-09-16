# Agent team

Mona's Project Pulse dashboard will be built by a specialist team orchestrated through the GitHub Copilot CLI in a Codespace. The Orchestrator coordinates the work, assigns non-overlapping file scopes, and verifies that the completed work integrates cleanly. All agents leave staging, commits, and pushes to the learner.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 4.7 (copilot) | Breaks requests into dependency-aware phases; delegates work to the specialists; coordinates parallel and sequential work; and checks the integrated result. It does not implement code itself. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the repository and relevant documentation, identifies requirements, risks, edge cases, and dependencies, then produces an actionable implementation plan with file ownership and validation expectations. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements scoped application logic and fixes with explicit, testable behavior. For Project Pulse, it can also create assigned runnable-app support such as `.vscode/launch.json`. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Owns UI/UX direction, accessibility, information architecture, interaction flow, and visual design. For Project Pulse, it ensures a polished, responsive dashboard with project cards, status and priority treatments, and predictable CSS hooks. | `.github/agents/designer.agent.md` |
