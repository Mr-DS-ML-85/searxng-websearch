<div align="center">

# 🔍 searxng-websearch

**A zero-API-key web search skill for AI agents — powered by your own [SearXNG](https://docs.searxng.org/) instance.**

[![Python 3.8+](https://img.shields.io/badge/python-3.8%2B-blue?style=flat-square&logo=python)](https://python.org)
[![SearXNG](https://img.shields.io/badge/SearXNG-self--hosted-orange?style=flat-square)](https://docs.searxng.org/)
[![OpenClaude](https://img.shields.io/badge/OpenClaude-skill-blueviolet?style=flat-square)](https://github.com/openclaude)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

[Features](#features) · [Install](#install) · [Usage](#usage) · [OpenClaude](#openclaude-skill) · [Config](#configuration)

</div>

---

## What is this?

Three Python scripts that give **any AI agent or CLI workflow** the ability to search the web, fetch pages, and do multi-angle deep research — **without paying for a search API.**

It talks directly to a SearXNG JSON endpoint you self-host. No Google API key. No Bing subscription. No rate-limit anxiety.

```
wsearch   "attention is all you need"  --format agent
wfetch    https://arxiv.org/abs/1706.03762
wresearch "transformer KV cache optimisation"  --fetch-top-pages 5
```

---

## Features

| | |
|---|---|
| 🔒 **100% private** | Searches go through your own SearXNG — no third-party API keys |
| 🤖 **Agent-ready output** | `--format agent` emits compact, token-efficient context for LLMs |
| 🧠 **Deep research mode** | Auto-generates 5 sub-queries, deduplicates, ranks, and fetches top pages |
| 📊 **Smart source ranking** | Prioritises GitHub, arXiv, official docs, and dated sources automatically |
| 🌍 **Multi-engine coverage** | SearXNG aggregates Google, Bing, DuckDuckGo, Brave, and 70+ others |
| ⚙️ **Fully configurable** | `.env` file + per-call CLI overrides for every parameter |
| 🔌 **OpenClaude skill** | Drop-in skill for [OpenClaude](https://github.com/openclaude) with `SKILL.md` |

---

## Install

### 1 · Self-host SearXNG (one-time)

```bash
docker run -d --name searxng -p 8080:8080 searxng/searxng
```

Enable the JSON format (required):

```bash
# searxng/settings.yml
search:
  formats: [html, json]
```

### 2 · Clone this skill

```bash
git clone https://github.com/Mr-DS-ML-85/searxng-websearch
cd searxng-websearch
pip install requests beautifulsoup4 lxml python-dotenv
```

### 3 · Configure

```bash
cp _env .env
```

```ini
# .env
SEARXNG_URL=http://localhost:8080   # your SearXNG instance
SEARXNG_LANGUAGE=en
SEARXNG_SAFE_SEARCH=0
SEARXNG_TIMEOUT=20
```

### 4 · Install shell shims (optional but recommended)

```bash
bash install.sh
# → installs wsearch / wfetch / wresearch to ~/.local/bin
```

---

## Usage

### `wsearch` — single query

```bash
wsearch "rust async runtime internals" --format markdown
wsearch "latest llama models 2025"     --format agent --max-results 8
wsearch "stable diffusion 3"           --category it  --time-range month
```

| Flag | Default | Options |
|---|---|---|
| `--format` | `markdown` | `markdown` · `text` · `json` · `agent` |
| `--category` | `general` | `general` · `news` · `science` · `it` · `images` · … |
| `--max-results` | `5` | any int |
| `--time-range` | — | `day` · `week` · `month` · `year` |
| `--language` | env default | ISO 639-1 code |
| `--safe-search` | `0` | `0` · `1` · `2` |
| `--searxng-url` | env default | any URL |

---

### `wfetch` — fetch & extract a page

```bash
wfetch https://arxiv.org/abs/2307.09288 --max-chars 6000
```

Strips scripts, styles, and boilerplate. Returns clean readable text.

---

### `wresearch` — deep research

```bash
wresearch "paged KV cache eviction strategies" --fetch-top-pages 3 --output markdown
```

Internally runs **5 sub-queries**, deduplicates all results, scores each source, fetches the top N pages, and emits a structured markdown report.

**Source scoring:**

| Signal | Score |
|---|---|
| `github.com` / `gitlab.com` | +5 |
| `arxiv.org` / `openreview.net` | +5 |
| `docs.*` / `readthedocs.*` | +4 |
| `research` / `paper` in domain | +3 |
| Has a publication date | +1 |
| Has a snippet | +1 |

---

## OpenClaude Skill

This repo is structured as an [OpenClaude](https://github.com/openclaude) skill.

```
searxng-websearch/
├── SKILL.md          ← skill manifest + agent instructions
├── scripts/
│   ├── websearch.py
│   ├── fetch_page.py
│   └── deep_research.py
├── install.sh
└── _env              ← copy to .env and fill in
```

### Install as OpenClaude skill

```bash
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  ~/.openclaude/skills/searxng-websearch

bash ~/.openclaude/skills/searxng-websearch/install.sh
```

The model will invoke skills via absolute path (never `cd`) per the `SKILL.md` instructions — this avoids the shell CWD reset bug in OpenClaude's bash runner.

> **Note for other agents:** Always call scripts by full path.  
> `python3 ~/.openclaude/skills/searxng-websearch/scripts/wsearch.py "query"` ✅  
> `cd ~/.../skills && python3 wsearch.py "query"` ❌

---

## Configuration

All variables can be set in `.env` or as real environment variables. CLI flags override both.

| Variable | Default | Description |
|---|---|---|
| `SEARXNG_URL` | `http://localhost:8080` | Base URL of your SearXNG instance |
| `SEARXNG_LANGUAGE` | `en` | Default search language |
| `SEARXNG_SAFE_SEARCH` | `0` | `0` off · `1` moderate · `2` strict |
| `SEARXNG_TIMEOUT` | `20` | HTTP timeout in seconds |

---

## Pipe-friendly examples

```bash
# Save a research report
wresearch "LoRA adapter hot-swapping inference" > report.md

# Feed search context directly into another LLM call
CONTEXT=$(wsearch "vLLM continuous batching" --format agent)

# Chain fetch into a summariser
wfetch https://vllm.readthedocs.io/en/latest/ --max-chars 4000 | llm "summarise this"
```

---

## Requirements

- Python 3.8+
- `requests`, `beautifulsoup4`, `lxml`, `python-dotenv`
- A running [SearXNG](https://docs.searxng.org/) instance with JSON format enabled

---

## License

MIT — do whatever you want, just don't remove the license header.

---

## Agent Compatibility

This skill follows the **[Agent Skills open standard](https://agentskills.io)** — the same `SKILL.md` format works natively across every major AI coding agent. One install, eight tools.

---

### 🟣 Claude Code

Personal skills live in `~/.claude/skills/` (available across all projects); project skills go in `.claude/skills/` inside the repo.

```bash
# Personal install (all projects)
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  ~/.claude/skills/searxng-websearch

# Project-scoped install
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  .claude/skills/searxng-websearch
```

List all discovered skills with `/skills` inside a session. Invoke explicitly with `/searxng-websearch` or let Claude trigger it automatically when you ask to search or research anything.

> **CWD note:** When you or Claude invoke a skill, the rendered `SKILL.md` content enters the conversation as a single message and stays there for the rest of the session. Always call scripts by full path — Claude Code's bash runner may reset the working directory between tool calls:
> ```bash
> # ✅ correct
> python3 ${CLAUDE_SKILL_DIR}/scripts/websearch.py "query" --format agent
> ```

---

### 🔵 OpenClaude

OpenClaude reads the same `~/.claude/skills/` directory and parses skill files in the same markdown format — no conversion needed.

```bash
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  ~/.claude/skills/searxng-websearch
bash ~/.claude/skills/searxng-websearch/install.sh
```

OpenClaude supports multiple providers — switch between Claude, GPT, Gemini, DeepSeek, or any local Ollama model by setting env vars:

```bash
# Use Qwen locally
export CLAUDE_CODE_USE_OPENAI=1
export OPENAI_BASE_URL=http://localhost:11434/v1
export OPENAI_MODEL=qwen2.5-coder:7b
openclaude
```

> **CWD bug:** Each `Bash()` call spawns a fresh shell — `cd skill-dir && python3 script.py` resets on the next call. Always use the absolute path shims installed by `install.sh`: `wsearch`, `wfetch`, `wresearch`.

---

### 🟠 Gemini CLI

Gemini CLI discovers skills from `~/.gemini/skills/` (user scope) or `<workspace>/.gemini/skills/` (project scope). The `.agents/skills/` alias is also supported and provides an interoperable path compatible with other AI tools.

```bash
# Via git install (recommended)
gemini skills install https://github.com/Mr-DS-ML-85/searxng-websearch.git

# Or manually
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  ~/.gemini/skills/searxng-websearch

# Verify
gemini           # start session
/skills          # confirm skill appears
```

Skills under `~/.gemini/skills/` (user scope) are not affected by workspace trust settings. When Gemini identifies a task matching the skill's description, it calls the `activate_skill` tool and prompts for your consent before execution.

---

### 🟤 Hermes Agent (Nous Research)

Hermes Agent is compatible with the `agentskills.io` open standard. Skills are stored in `~/.hermes/skills/` and are portable, shareable, and community-contributed.

```bash
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  ~/.hermes/skills/searxng-websearch
```

After complex tasks (5+ tool calls), Hermes automatically captures the workflow and creates reusable skill documents. This skill will trigger implicitly when you ask Hermes to search the web or research a topic.

---

### ⚫ NemoClaw (NVIDIA)

NemoClaw is NVIDIA's entry into the open-source agent space, designed to run on NVIDIA GPU infrastructure using NIM (NVIDIA Inference Microservices). It runs on top of the OpenClaw/OpenClaude skill format.

```bash
# NemoClaw uses the same ~/.openclaude/skills/ path as OpenClaude
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  ~/.openclaude/skills/searxng-websearch

bash ~/.openclaude/skills/searxng-websearch/install.sh
```

> NemoClaw is in early preview (March 2026). Check [NVIDIA/NemoClaw](https://github.com/NVIDIA/NemoClaw) for the latest install instructions.

---

### 🟢 OpenAI Codex CLI

Codex loads skills from `~/.agents/skills/` automatically when the task matches. Skills support launched for Codex in December 2025.

```bash
# Global install via ~/.agents/skills (cross-tool standard path)
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  ~/.agents/skills/searxng-websearch

# Or Codex-native path
git clone https://github.com/Mr-DS-ML-85/searxng-websearch \
  ~/.codex/skills/searxng-websearch
```

Skills can be invoked explicitly, and Codex can also choose them implicitly when the task matches the skill description. Mention the skill in your prompt with `$searxng-websearch` or let it trigger automatically.

To disable a skill without deleting it, set `enabled = false` in `~/.codex/config.toml`:

```toml
# ~/.codex/config.toml
[[skills.config]]
path = "~/.codex/skills/searxng-websearch/SKILL.md"
enabled = false
```

---

### 🔴 Claude Desktop

Claude Desktop uses the **MCP + Skills** stack. Add a skill via an MCP Skill Hub server, then restart Claude Desktop — it will automatically load and use the appropriate skills from your MCP server.

**Option A — Upload via UI (simplest):**

```bash
# Zip the skill folder
zip -r searxng-websearch.zip \
  ~/.claude/skills/searxng-websearch/SKILL.md \
  ~/.claude/skills/searxng-websearch/scripts/
```

Then in Claude Desktop → **Skills → Upload Skill** → drag in the `.zip`. The skill appears in your list and activates automatically.

**Option B — MCP Skill Hub (hot-reload):**

```jsonc
// claude_desktop_config.json  (usually ~/Library/Application Support/Claude/)
{
  "mcpServers": {
    "skill-hub": {
      "command": "uvx",
      "args": ["mcp-skill-hub"],
      "env": {
        "MCP_SKILLS_DIR": "/home/Mr-DS-ML-85/.claude/skills"
      }
    }
  }
}
```

Restart Claude Desktop. Drop a new skill folder into the skills directory and MCP Skill Hub instantly discovers it — no restart, no reconfiguration.

> **Plan requirement:** Claude Desktop Skills require a paid plan (Pro, Max, Team, or Enterprise). Free-tier accounts don't support Skills. For Team or Enterprise, an administrator must enable Skills before individual users can access them.

---

### Quick-reference: Install paths

| Tool | Skill path | Notes |
|---|---|---|
| **Claude Code** | `~/.claude/skills/searxng-websearch/` | Personal; or `.claude/skills/` in project |
| **OpenClaude** | `~/.claude/skills/searxng-websearch/` | Same format as Claude Code |
| **Gemini CLI** | `~/.gemini/skills/searxng-websearch/` | Also: `~/.agents/skills/` alias |
| **Hermes Agent** | `~/.hermes/skills/searxng-websearch/` | Auto-creates skills after complex tasks |
| **NemoClaw** | `~/.openclaude/skills/searxng-websearch/` | Early preview; NVIDIA NIM backend |
| **Codex CLI** | `~/.codex/skills/searxng-websearch/` | Also: `~/.agents/skills/` (cross-tool) |
| **Claude Desktop** | Upload `.zip` via UI or MCP Skill Hub | Paid plan required |

**Universal install — works for Claude Code, OpenClaude, Codex, Gemini, and Hermes in one shot:**

```bash
REPO=https://github.com/Mr-DS-ML-85/searxng-websearch
for dir in ~/.claude/skills ~/.agents/skills ~/.codex/skills ~/.gemini/skills ~/.hermes/skills; do
  mkdir -p "$dir"
  git clone "$REPO" "$dir/searxng-websearch" 2>/dev/null || \
    git -C "$dir/searxng-websearch" pull
done
bash ~/.claude/skills/searxng-websearch/install.sh
echo "✓ installed across all agents"
```
