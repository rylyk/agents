# 002: Phase 0, local builder bring-up on Ollama

**From:** cloud session · **To:** local session on the owner's PC
**Why:** Phase 0 in `PROPOSAL.md` §5. Request 001 showed that Ollama 0.40 is already installed and runs 35B-A3B-class models here, and Ollama supports Claude Code natively. So Phase 0 starts on Ollama rather than building llama.cpp, which would need cmake plus a CUDA 13.4 / gcc 16 workaround. llama.cpp only comes in if Ollama misses the speed target.

**Rules for this request:** the owner is present in this session. **Ask the owner before every step marked CHANGE**, and do nothing destructive (deleting models, editing service config) without an explicit yes. Everything else is read-only. Report env var and settings *names*, never values. This repo is public, so keep personal details out of the response.

**Answer** in `comms/002-phase0-bringup.response.md`, using the headings below. Set 002 to `answered` in `comms/README.md`, commit with `comms: 002 phase 0 response`, and **push** to `claude/determined-galileo-o47ain`. Last time the answer wasn't pushed.

## 1. Permanent clone (CHANGE)
Clone `rylyk/agents` to `~/src/github.com/agents`, which matches the existing `~/src/github.com/*` layout, and check out `claude/determined-galileo-o47ain`. Use that clone for everything below. Check that `bin/station` is executable.

## 2. Read-only questions
a. **GPU lock.** How does this machine enforce "one GPU job at a time" for the video pipeline (Z-Image, Wan2.2)? A lock file (give the path and whether it uses `flock`), a systemd unit, a convention in a skill? `bin/station` takes a lock for local-model sessions, and it must be the *same* lock.
b. **Claude settings.** In `~/.claude/settings.json` and `settings.local.json`, list the key names under `env`, say whether `model` is set, and say whether any `ANTHROPIC_*` key appears. Settings override shell variables, so any of these could undo what `bin/station` sets.
c. **The existing local coder.** How does the `local-coder` skill call opencode and Ollama? From `ollama show --modelfile ornith-coder`, list the `PARAMETER` lines (`num_ctx` and so on) and what it's built `FROM`, as a model name only, not a path. Ask the owner: how well has Ornith done as a local coder so far?
d. **Models on the Pro plan.** Run each of these once and report success or the error. It costs a few tokens.
   `claude -p --model opus "Reply with OK"`, then the same with `sonnet` and with `haiku`.

## 3. Get the model (CHANGE, ask first)
a. **Disk.** `/mnt/data-nvme` has about 35 GB free, and `qwen3.6:35b-a3b` at 4-bit is about 22–24 GB. Report the actual download size before pulling, if you can. If less than 10 GB would be left afterwards, ask the owner which models to remove, or whether to move the Ollama store. `bench-*` and duplicate quantizations are the obvious candidates, but the owner decides.
b. **GPU.** Check `nvidia-smi` for running GPU jobs. Don't start while a video job is running.
c. **Pull** `qwen3.6:35b-a3b`. If that tag doesn't exist, find the right Qwen3.6-35B-A3B Q4 tag in the Ollama library, or use an Unsloth GGUF via `hf.co/unsloth/...:Q4_K_M`. Report the tag you used.
d. **Create the agent variant.** It needs a 64k context, and Claude Code can't set context length through the API:
   ```
   printf 'FROM <tag from 3c>\nPARAMETER num_ctx 65536\n' > /tmp/Modelfile.qwen3.6-agent
   ollama create qwen3.6-agent -f /tmp/Modelfile.qwen3.6-agent
   ```
   Don't change the Ollama systemd service config.

## 4. Speed measurements
a. **Short prompt:** `ollama run qwen3.6-agent --verbose "Write a Python function that reverses a linked list."` Report the `prompt eval rate` and `eval rate`, then `ollama ps` (size, CPU/GPU split) and VRAM use from `nvidia-smi`.
b. **~30k-token prompt** (adapt as needed; use Python if `jq` is missing):
   ```
   yes "The quick brown fox jumps over the lazy dog near the riverbank today." | head -n 2500 > /tmp/long.txt
   jq -n --rawfile p /tmp/long.txt '{model:"qwen3.6-agent", prompt:($p+"\n\nSay how many sentences are above, roughly."), stream:false}' \
     | curl -s http://127.0.0.1:11434/api/generate -d @- \
     | jq '{prompt_eval_count, prompt_tps:(.prompt_eval_count/(.prompt_eval_duration/1e9)), eval_tps:(.eval_count/(.eval_duration/1e9))}'
   ```
   Report all three numbers. **The target is at least 25 tok/s `eval_tps` at about 30k context.**

## 5. Toy TDD task through `bin/station` (CHANGE: creates a scratch folder)
a. Create a scratch git repo, for example `~/scratch/phase0-toy`. Add an empty `slug.py`, a minimal `pyproject.toml` with pytest configured, and `TASK.md` containing:
   ```
   Implement slugify(text: str, max_len: int | None = None) -> str in slug.py.
   Acceptance criteria:
   - [ ] lowercases the text
   - [ ] folds accented letters to ASCII ("Crème Brûlée" -> "creme-brulee")
   - [ ] every run of non-alphanumeric characters becomes a single hyphen
   - [ ] no leading or trailing hyphens; empty or all-symbol input returns ""
   - [ ] with max_len, truncates at a hyphen boundary and never ends with a hyphen
   Work test-first: write failing tests in test_slug.py, then make them pass. Run `python -m pytest -q` to verify.
   Do not edit TASK.md. Stop when all tests pass, and print a one-paragraph summary.
   ```
   Commit that as the initial state.
b. From inside the toy repo, run the builder station headless:
   ```
   time ~/src/github.com/agents/bin/station build -p "$(cat TASK.md)" \
     --output-format stream-json --verbose \
     --allowedTools "Read,Edit,Write,Glob,Grep,Bash(python -m pytest:*),Bash(pytest:*),Bash(git status:*),Bash(git diff:*)" \
     > run1.jsonl
   ```
   Then reset to the initial commit and run it a second time (`run2.jsonl`).
c. For each run, report:
   - whether `python -m pytest -q` is green afterwards, and whether any criterion is untested
   - wall-clock time and number of turns
   - any tool-call errors or malformed tool calls (look for `is_error` or parse errors in the jsonl)
   - whether the model edited `TASK.md` or did anything outside the task
   - whether the `tdd` skill was picked up
   - whether any request went to Anthropic rather than Ollama. Check the Ollama log (`journalctl -u ollama --since "-15 min"`) for the requests, and if the owner can, glance at their Claude usage before and after.
   Summarize; don't commit the jsonl files.
d. If a run fails because of the harness (for example an error in `bin/station` or an env var that doesn't take effect), describe exactly what happened. Don't patch `bin/station` yourself: report it, and the cloud session will fix it.

## 6. Optional: Ornith bake-off (only if 4 and 5 are done and the owner agrees)
The owner already uses Ornith-1.5-35B-A3B, which reports 79% SWE-bench Verified against Qwen3.6's 73.4% (both self-reported). Create `ornith-agent` from the existing `hf.co/ornith-ai/Ornith-1.5-35B-A3B-GGUF:Q6_K` with `num_ctx 65536`. Note that at about 30 GB it may spill into zram; if so, report it. Run 4b and 5b–c against it without editing the repo's config:
```
sed 's/^STATION_BUILD=.*/STATION_BUILD="ollama:ornith-agent"/' ~/src/github.com/agents/config/models.env > /tmp/models.ornith.env
STATION_CONFIG=/tmp/models.ornith.env ~/src/github.com/agents/bin/station build -p "$(cat TASK.md)" ...
```

## 7. Leave it clean
Make sure no model is left loaded (`ollama ps`) and nothing is holding the GPU lock. Keep `qwen3.6-agent` and `ornith-agent`.
