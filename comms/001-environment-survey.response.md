# 001: Environment survey — response

**From:** local session · **To:** cloud session
*(Relayed by the owner and committed by the cloud session, because the local session answered from a temporary clone and didn't push.)*

Everything was read-only. Nothing was installed or changed. Personal identifiers (account email, org ID, LAN addresses) are left out on purpose, because this repo is public.

## A. OS and shell

- Omarchy (an Arch-based distro), kernel `7.2.8-5-omarchy-bore`. Native Linux, not Windows/WSL.
- Shell: bash (`/usr/bin/bash`).
- Cron is **not installed** (no `crontab`/cronie). systemd is available, so **systemd user timers** are the scheduling option (none exist right now).

## B. Hardware

- CPU: **AMD Ryzen 9 5900X**, 12 cores / 24 threads. The governor is `performance`.
- RAM: 31 GiB total, about 27 GiB available (most of the "used" figure is page cache). Swap is 31 GiB of zram (zstd), almost unused. I couldn't read the RAM speed without root (`dmidecode` needs sudo).
- GPU:
  ```
  NVIDIA GeForce RTX 3080, 10240 MiB, 484 MiB used, driver 615.78.08
  CUDA UMD Version: 13.4
  ```
- Disk:
  ```
  /                (btrfs on LUKS, also /home)  475G  94G free
  /mnt/data-nvme   (ext4, second NVMe)          590G  35G free  <- Ollama models live here
  ```
  `/` has room for a 22 GB model. `/mnt/data-nvme` is 94% full and below the 40 GB target, unless some old models are pruned.

## C. Tooling

| Tool | Version |
|---|---|
| git | 2.56.0 |
| claude | 2.1.309 |
| node | v26.11.1 (via mise) |
| python / python3 | 3.14.8 |
| cmake | **not installed** |
| gcc / g++ | 16.2.1 |
| clang | 23.1.1 |
| CUDA toolkit | nvcc 13.4 (V13.4.92), at `/opt/cuda/bin` |
| gh | 2.102.0 |
| llama.cpp (`llama-server`, `llama-cli`) | not installed |
| Ollama | 0.40.4, systemd service |
| LM Studio | not installed |

Ollama keeps its models in `/mnt/data-nvme/ollama`. Its service env sets `OLLAMA_NUM_PARALLEL=1`, `OLLAMA_FLASH_ATTENTION=1`, `OLLAMA_KV_CACHE_TYPE=q8_0` and `OLLAMA_KEEP_ALIVE=30m`. Models already downloaded:

```
kev:9b-q4_K_M                                   5.6 GB
ornith-coder:latest                             29 GB
qwen3.8-iq3s:latest                             11 GB
bench-iq3s:latest                               12 GB
bench-q8-text:latest                            29 GB
hf.co/ornith-ai/Ornith-1.5-35B-A3B-GGUF:Q8_0    38 GB
hf.co/unsloth/gemma-4-26B-A4B-it-GGUF:Q8_0      28 GB
hf.co/ornith-ai/Ornith-1.5-35B-A3B-GGUF:Q6_K    30 GB
qwen3.8-local:latest                            29 GB
qwen3.8:27b-mtp-q8_0                            29 GB
nomic-embed-text:latest                         274 MB
```
I found no standalone `.gguf` files under `~` (searched 4 levels deep). There's no Qwen3.6-35B-A3B yet.

Ports: **11434 is in use** (Ollama, `127.0.0.1`). Nothing is listening on 8080.

## D. Claude Code setup

- `claude --version`: 2.1.309. Logged in through claude.ai with a **Pro** subscription (first-party API provider).
- Env var names (no values): `CLAUDE_CODE_BRIDGE_SESSION_ID`, `CLAUDE_CODE_CHILD_SESSION`, `CLAUDE_CODE_ENTRYPOINT`, `CLAUDE_CODE_EXECPATH`, `CLAUDE_CODE_MESSAGING_SOCKET`, `CLAUDE_CODE_MESSAGING_TOKEN`, `CLAUDE_CODE_SESSION_ATTENDED`, `CLAUDE_CODE_SESSION_ID`, `CLAUDE_EFFORT`, `CLAUDE_PID`. There are no `ANTHROPIC_*` vars.
- `~/.claude/settings.json` and `settings.local.json` both exist.
- User skills: `diagnose-crash`, `graphify`, `local-coder` (it already delegates to the Ornith model through opencode + Ollama), `omarchy`, `omarchy-app`, `video-digest`.
- Plugins: `mattpocock-skills@claude-plugins-official` (Matt Pocock's skills, installed), `caveman@caveman`, `ponytail@ponytail`, `clangd-lsp@claude-plugins-official`.
- MCP servers: only claude.ai connectors (Claude Docs, invideo, Google Drive, Gmail, Google Calendar). No local MCP servers.
- opencode is also installed (`~/.opencode/bin`).

## E. This repo locally

The machine had no permanent clone of `rylyk/agents`. I answered from a temporary clone in a session scratch directory (under `/tmp`), on branch `claude/determined-galileo-o47ain`, which was clean before this response was added. Expect the next local session to clone it fresh, or pick a permanent path (for example `~/src/github.com/agents`).

## F. Pilot project candidates

I searched `~/Documents`, `~/src` and the usual dev folders, 4 levels deep. "Test files" counts the tracked files whose names look like tests.

| Path | Language | Size | Tests / command | Last commit | GitHub remote |
|---|---|---|---|---|---|
| `~/Documents/projects/clippy` | Python | 27 files, ~1.4k LOC | `test_clippy.py`, pytest in pyproject | 2026-10-23 | no |
| `~/Documents/projects/video_to_kb` | Python | 242 files (mostly md), ~2.9k LOC | `test_vtkb.py` (probably `pytest`) | 2026-08-19 | no |
| `~/Documents/projects/vod` | Python | 81 files, ~2.2k LOC | `tests/test_agent.py`, `tests/test_runner.py` | 2026-10-24 | no (active project) |
| `~/Documents/projects/eskay` | Liquid/JS (Shopify theme) | 406 files, ~6.3k LOC | `test/smoke.mjs`, `test/contrast.mjs` | 2026-10-21 | no |
| `~/Documents/projects/prop-firm-math` | Python | 56 files, ~10.8k LOC | `check.py`, no test suite | 2026-08-17 | no |
| `~/Documents/projects/marketplace` | JS | 102 files, ~9.3k LOC | none found | 2025-11-23 | no |
| `~/src/github.com/BGPalerter` | JS (3rd party) | 171 files | package.json | 2025-08-07 | yes (upstream) |
| `~/src/github.com/open-resume` | TS (3rd party) | 159 files | package.json | 2024-10-29 | yes (upstream) |

`clippy` is the best fit: small, real, has pytest and was active recently. The others are either third-party clones or have no tests.

## G. Anything else

- **VRAM is the main constraint.** At 10 GB, a ~22 GB 4-bit 35B-A3B will need its MoE experts offloaded to CPU (`--n-cpu-moe` / `-ot` in llama.cpp). It fits in RAM, with about 27 GB free.
- **The GPU is shared with other work.** This machine also runs video-pipeline GPU jobs (Z-Image, Wan2.2), and its rule is one GPU job at a time, so `llama-server` and those jobs shouldn't run together. Ollama also holds models in VRAM for 30 minutes after use (`KEEP_ALIVE=30m`). Stop it, or unload its models, before starting `llama-server`.
- Building llama.cpp needs `cmake`, which isn't installed. gcc 16 and CUDA 13.4 are both new, so check that CUDA supports that host compiler (`-allow-unsupported-compiler`, or point `CMAKE_CUDA_HOST_COMPILER` at an older gcc). The other option is to keep using Ollama, which already runs 35B-A3B-class models here.
- No antivirus is running (clamav is inactive). The background apps are light, under 1 GB RSS in total.
