# 001: Environment survey for Phase 0

**From:** cloud session · **To:** local session on the owner's PC
**Why:** Phase 0 in `PROPOSAL.md` §5 sets up a local builder (llama.cpp running Qwen3.6-35B-A3B) behind Claude Code. Before writing the config file, the station wrapper and the `llama-server` launch script, the cloud session needs the facts below about this machine.

**Rules for this request:** read-only. Don't install, update or configure anything, and don't print secrets. If a command doesn't exist on this OS, say so and use the closest equivalent. If something fails, record the error and move on.

## How to answer

1. `git pull origin claude/determined-galileo-o47ain`
2. Write your answers to `comms/001-environment-survey.response.md`, using the headings below. Paste short raw command output in code blocks, trimmed to the relevant lines.
3. Set request 001 to `answered` in the table in `comms/README.md`.
4. Commit with `comms: 001 environment survey response`, then `git push origin claude/determined-galileo-o47ain`.

## Questions

### A. OS and shell
- OS name and version. On Windows, also: are you running natively or inside WSL2? If WSL2, which distro, and does `nvidia-smi` work inside it?
- Which shell this session runs in (PowerShell, cmd, bash, zsh).
- Is cron (Linux, WSL) or Task Scheduler (Windows) available to this user?

### B. Hardware
- CPU model and core/thread count. (The owner said "Ryzen 7 5900X"; that model is a Ryzen 9, so please confirm which it is.)
- Total and currently free RAM, plus RAM speed if it's easy to find.
- `nvidia-smi` output: GPU name, VRAM total/used, driver version, and the CUDA version it reports.
- Free disk space on the drive where models would go. The 4-bit model is about 20–23 GB, so at least 40 GB free is preferable.

### C. Tooling already installed (version, or "not installed")
- `git`, `claude` (Claude Code), `node`, `python`/`python3`, `cmake`, a C++ compiler (MSVC, gcc or clang), the CUDA toolkit (`nvcc --version`), `gh`
- Any local LLM runtimes: llama.cpp (`llama-server`, `llama-cli`), Ollama, LM Studio. If any exist, list the models already downloaded and where they are stored.
- Is anything already listening on ports 8080 or 11434?

### D. Claude Code setup
- `claude --version`, and whether it's logged in with the subscription (don't print tokens).
- The *names* of any environment variables starting with `ANTHROPIC_` or `CLAUDE_`. No values.
- Are any of these present: user-level settings (`~/.claude/settings.json`), user skills or plugins (especially Matt Pocock's skills), or MCP servers? List names only.

### E. This repo locally
- The absolute path of the local clone of `rylyk/agents`, the current branch, and whether the working tree is clean.

### F. Pilot project candidates (optional; the owner makes the final call)
List up to 10 git repos on this machine that might suit the Phase 1 pilot: real, small, and with tests. Look in the usual dev folders only, at most 3 levels deep. For each one give: path, main language, rough size (files or lines of code), the test command if there is one (from `package.json` scripts, `Makefile`, `pyproject.toml` and so on), date of the last commit, and whether it has a GitHub remote. Don't read or report file contents beyond that.

### G. Anything else
Anything you noticed that would affect running a ~22 GB model plus Claude Code on this machine, such as heavy background apps, a small page file or swap, or antivirus scanning model folders.
