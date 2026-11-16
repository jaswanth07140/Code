# Code — master inventory

Everything this setup runs on, where it lives, and how to rebuild it. One page, kept
current, so nothing has to be remembered from a chat log.

Updated **2026-11-15**.

## Repositories

### Mine

| Repo | What it is | Size |
|---|---|---|
| [**Zero**](https://github.com/jaswanth07140/Zero) | The Zero AI search engine — live tree, 5.0.6 snapshot, browser extension, Alexa bundles | 10 MB |
| [**OpenCode**](https://github.com/jaswanth07140/OpenCode) | This machine's agent state: installs, ports, keys, gotchas, session history | 220 KB |
| [**zero-ai**](https://github.com/jaswanth07140/zero-ai) | Portable brain: identity, 48 knowledge domains, skills, evals, retrieval builder | 772 KB |
| [**Code**](https://github.com/jaswanth07140/Code) | This inventory | — |

### Upstream, consumed and integrated

| Repo | What was taken | Status |
|---|---|---|
| [tashfeenahmed/freellmapi](https://github.com/tashfeenahmed/freellmapi) | Local multi-provider router | **Installed**, running |
| [msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents) | 87 agent personas (engineering, security, testing) | **Installed** |
| [affaan-m/ECC](https://github.com/affaan-m/ECC) | Skills, agents, commands, hooks — `ecc-universal 2.2.3` | **Installed** |
| [garrytan/gstack](https://github.com/garrytan/gstack) | CEO/design/review/QA slash commands | **Assessed, not installed** (~300 MB, needs bun + git + browser) |

## The stack

| Layer | What | Where |
|---|---|---|
| Agent | OpenCode, V2 config schema | `~/.config/opencode/opencode.json` |
| Memory | Instructions array → loads every session | `~/.config/opencode/instructions/` |
| Brain | Portable identity + knowledge | `~/zero-ai` → `github.com/jaswanth07140/zero-ai` |
| Inference | FreeLLMAPI router, OpenAI-compatible | `http://127.0.0.1:3001/v1` |
| Service | systemd user unit, linger enabled | `~/.config/systemd/user/freellmapi.service` |
| Product | Zero search engine (PHP) | `~/zero-ops` → `github.com/jaswanth07140/Zero` |

## Installed tooling

| Thing | Count / Version |
|---|---|
| Agents (markdown) | **87** — agency-agents |
| Agents (ECC, JSON) | 27 |
| Skills | **47** |
| Commands | **35** |
| Plugins | 3 (`ecc-hooks.ts`, `index.ts`, `lib/`) |
| ECC package | `ecc-universal 2.2.3` |
| Node.js | 24.22.3 at `~/.local/node/bin` |
| Git | 2.47.3 |
| Python | 3.13.5 (stdlib only for repo tooling) |
| Not installed | `npm`, `docker`, `bun`, `gh` CLI, GPU/CUDA |

Agent cap is roughly **119**; at 114 there are ~5 slots left.

## Inference providers

Router: **FreeLLMAPI** on `127.0.0.1:3001`, catalog of **483 models across 42 platforms**.

| Provider | Status |
|---|---|
| google | healthy |
| groq | healthy |
| openrouter | healthy |
| kilo | healthy |
| ovh | healthy |
| aihorde | pending first health check |

Verified real completions: google `gemini-3.1-flash-lite-preview`, groq
`qwen/qwen3.8-27b`, openrouter `dots-studio/dots-3-note-preview:free`. `auto` answers in
~1.5 s and fails over across providers.

Keys that are pointless here: **Cerebras, HuggingFace, Ollama Cloud** — zero models in
this build's catalog. NVIDIA NIM never issued a usable key.

## Hardware

```
CPU     Intel i3-1215U, 8 threads
RAM     6.3 GB total, ~1.9 GB available, no swap
Disk    /dev/vdb, 7.9 GB total, 3.4 GB used
GPU     none — no /dev/dri, no CUDA
```

Consequence: local inference caps out around a 1B quantized model at 5–10 tokens/sec.
Fine-tuning is impossible here; it needs a cloud GPU.

## Secrets — where they live, never in git

| Secret | Location | Mode |
|---|---|---|
| FreeLLMAPI unified key | `~/.config/opencode/freellmapi.key` | 600 |
| GitHub PAT (fine-grained) | `~/.config/opencode/gh.token` | 600 |
| Upstream provider keys | encrypted in `freellmapi/server/data/freeapi.db` | — |
| Zero live config | `~/zero-ops/backups/.../htdocs/config.php` — **never commit** | — |

`gh` CLI is absent, so GitHub work uses the REST API via `curl` and pushes via
`git -c http.extraheader="Authorization: Basic ..."` so the token never lands in
`.git/config`.

## Rebuild from nothing

```bash
# 1. Brain
git clone https://github.com/jaswanth07140/zero-ai.git ~/zero-ai
python3 ~/zero-ai/runtime/brain.py init
python3 ~/zero-ai/runtime/brain.py stats

# 2. Environment notes
git clone https://github.com/jaswanth07140/OpenCode.git ~/OpenCode

# 3. Zero site
git clone https://github.com/jaswanth07140/Zero.git ~/zero-repo
```

Then restore secrets by hand — they are deliberately not in any repo.

## Related detail

`INVENTORY.md` holds the full per-item breakdown; `MANIFEST.json` is the machine-readable
version of the same data.
