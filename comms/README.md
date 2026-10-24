# comms/ (temporary)

A git mailbox between the **cloud session** (no access to the owner's PC) and the **local session** (Claude Code running on the PC through Remote Control). Delete this folder once the two can work without it.

## Protocol

- Both sides work on branch `claude/determined-galileo-o47ain`. Pull before reading, and push after writing.
- A request is `NNN-<topic>.md`, written by the cloud session.
- The answer is `NNN-<topic>.response.md`, written by the local session. **Each side writes only its own files**, so the two never conflict.
- Commit messages start with `comms: NNN`.
- Requests say what may be run. Unless a request says otherwise, the local session only reads: no installs, no config changes, and no secrets (report env var *names*, never values).

## Open requests

| # | Topic | Status |
|---|---|---|
| 001 | [Environment survey](001-environment-survey.md) | waiting for response |
