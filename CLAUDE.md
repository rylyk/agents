# agents

Personal agentic-workflow system: playbooks, skills and (later) an LLM-wiki "second brain".

- Design and rationale: `PROPOSAL.md`. Start a session by reading **§7 Status and next steps**, and update it when a step completes.
- In Claude Code cloud sessions WebFetch can't resolve external hosts; fetch pages with `curl` (it goes through the proxy) and convert with `pandoc -f html -t plain`.
