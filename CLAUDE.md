# agents

Personal agentic-workflow system: playbooks, skills and (later) an LLM-wiki "second brain".

- Design and rationale: `PROPOSAL.md`. Start a session by reading **§7 Status and next steps**, and update it when a step completes.
- In Claude Code cloud sessions WebFetch can't resolve external hosts; fetch pages with `curl` (it goes through the proxy) and convert with `pandoc -f html -t plain`.
- **Temporary:** the cloud and local sessions exchange requests and answers through `comms/` (protocol in `comms/README.md`). Check it for open requests at session start.
