# OpenCode + Ollama: Setup Guide (Local and Cloud Models)

A step-by-step guide to running [OpenCode](https://opencode.ai) (a terminal AI coding agent) with [Ollama](https://ollama.com) as the model backend, using either **local models** on your own hardware or **Ollama Cloud models**.

> Commands, config formats, and the list of cloud models change between versions. If something doesn't match, check `ollama --help`, `opencode --help`, and the official docs.

## Table of Contents

1. [How it fits together](#1-how-it-fits-together)
2. [Prerequisites](#2-prerequisites)
3. [Install Ollama](#3-install-ollama)
4. [Install OpenCode](#4-install-opencode)
5. [Path A: Local models](#5-path-a-local-models)
6. [Path B: Ollama Cloud models](#6-path-b-ollama-cloud-models)
7. [Configure OpenCode](#7-configure-opencode)
8. [Fastest setup: `ollama launch`](#8-fastest-setup-ollama-launch)
9. [Using OpenCode](#9-using-opencode)
10. [Tuning and performance](#10-tuning-and-performance)
11. [Windows + WSL notes](#11-windows--wsl-notes)
12. [Troubleshooting](#12-troubleshooting)
13. [Cheat sheet](#13-cheat-sheet)

---

## 1. How it fits together

```
OpenCode (TUI agent)  --->  Ollama (OpenAI-compatible API)  --->  Model
                            http://localhost:11434/v1            - local (your GPU/CPU)
                                                                 - cloud (:cloud models, via Ollama)
```

- **Ollama** runs models and exposes an API on port `11434`.
- **OpenCode** talks to that API through an OpenAI-compatible provider entry in its config.
- **Local models**: private, free, limited by your hardware.
- **Cloud models**: large models (hundreds of billions of parameters) run on Ollama's servers, and you use them through the same interface.

## 2. Prerequisites

- A terminal (Linux, macOS, or Windows PowerShell / WSL)
- For local models: enough RAM/VRAM (see [section 5](#5-path-a-local-models))
- For cloud models: a free account at [ollama.com](https://ollama.com)
- Node.js is only needed if you install OpenCode via npm

## 3. Install Ollama

**Linux**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**macOS**

```bash
brew install ollama
# or download the app from https://ollama.com/download
```

**Windows (PowerShell)**

```powershell
winget install Ollama.Ollama
# or download the installer from https://ollama.com/download
```

Verify:

```bash
ollama --version
```

Make sure the server is running:

```bash
# Linux (systemd install): usually already running
systemctl status ollama

# Otherwise start it manually
ollama serve
```

Check the API:

```bash
curl http://localhost:11434
# Expected: "Ollama is running"
```

## 4. Install OpenCode

Pick one method:

```bash
# Install script (Linux/macOS/WSL)
curl -fsSL https://opencode.ai/install | bash

# npm
npm install -g opencode-ai

# Homebrew (macOS/Linux)
brew install anomalyco/tap/opencode

# Windows
scoop install opencode
choco install opencode
```

Verify:

```bash
opencode --version
```

## 5. Path A: Local models

### 5.1 Choose a model

OpenCode is an agent, so it needs a model with **tool calling** support. Browse tool-capable models at <https://ollama.com/search?c=tools>.

Commonly used coding-capable choices (sizes are approximate; check each model page):

| Model | Pull command | Rough memory need |
|---|---|---|
| Qwen3-Coder 30B | `ollama pull qwen3-coder:30b` | ~20 GB |
| gpt-oss 20B | `ollama pull gpt-oss:20b` | ~14 GB |
| Devstral | `ollama pull devstral` | ~15 GB |
| Qwen3 8B (small machines) | `ollama pull qwen3:8b` | ~6 GB |

Small models work for simple tasks but are noticeably weaker as agents. Expect better results from the largest model your hardware can run comfortably.

### 5.2 Pull and test

```bash
ollama pull qwen3-coder:30b
ollama run qwen3-coder:30b "Write a hello world in Python"
```

Useful management commands:

```bash
ollama list          # installed models
ollama ps            # running models, memory use, GPU/CPU split
ollama show qwen3-coder:30b
ollama rm <model>    # delete a model
ollama stop <model>  # unload from memory
```

### 5.3 Increase the context window (important)

Coding agents send large prompts (system prompt + tools + files). A small context window causes truncated or broken behavior. Ollama's OpenCode integration docs recommend **at least 64k tokens** (64000) for local models. Cloud models manage their own context, so this section is for local models only. Choose one method:

**Method 1: environment variable (applies to all models)**

```bash
# one-off
OLLAMA_CONTEXT_LENGTH=65536 ollama serve
```

Linux with systemd:

```bash
sudo systemctl edit ollama.service
```

Add:

```ini
[Service]
Environment="OLLAMA_CONTEXT_LENGTH=65536"
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

macOS (app): `launchctl setenv OLLAMA_CONTEXT_LENGTH 65536`, then restart Ollama.
Windows: set a user environment variable `OLLAMA_CONTEXT_LENGTH=65536`, then restart Ollama.

**Method 2: custom model via Modelfile (per model)**

```bash
cat > Modelfile <<'EOF'
FROM qwen3-coder:30b
PARAMETER num_ctx 65536
EOF

ollama create qwen3-coder-64k -f Modelfile
```

Use `qwen3-coder-64k` as the model name in OpenCode.

Confirm with `ollama ps` (the `CONTEXT` column shows the active value). A larger context uses more memory; if the model spills to CPU, use a smaller model or lower this value.

## 6. Path B: Ollama Cloud models

Cloud models run on Ollama's infrastructure and are useful when your hardware can't run large models. Availability, limits, and pricing tiers change, so check <https://ollama.com/cloud> and your account page.

### Find current cloud models

The list of cloud models changes often. Always check the live list: <https://ollama.com/search?c=cloud>

- Add the **Tools** filter: OpenCode needs tool calling.
- Open the model page for the exact tag. Some use `:cloud` (for example `glm-5.3:cloud`), others use a size plus suffix (for example `gpt-oss:120b-cloud`).
- Each model page shows ready-made commands, including the `ollama launch opencode --model ...` line.

Examples at the time of writing (may be outdated when you read this):

| Model | Tag | Notes |
|---|---|---|
| GLM-5.3 | `glm-5.3:cloud` | Flagship model from Z.ai, tuned for coding and long agentic tasks |
| GLM-5.3 Flash | `glm-5.3-flash:cloud` | Faster and lighter, multimodal |
| DeepSeek V4 Flash | `deepseek-v4-flash:cloud` | Efficient reasoning, 1M context |
| Kimi K3 | `kimi-k3:cloud` | Multimodal agentic model |
| MiniMax M3 | `minimax-m3:cloud` | Coding and agentic, 1M context |
| Qwen 3.5 | `qwen3.5:cloud` | Multimodal, tools, thinking |
| GPT-OSS | `gpt-oss:120b-cloud` | OpenAI open-weight model |

### Do I need an API key in `opencode.json`?

Usually no. Pick the route that fits:

| Route | Credentials | Key in `opencode.json`? |
|---|---|---|
| `ollama launch opencode --model <cloud-model>` | `ollama signin` once | No |
| Local Ollama + cloud model in `opencode.json` ([7.2](#72-cloud-models-through-local-ollama)) | `ollama signin` once | No |
| OpenCode built-in **Ollama Cloud** provider ([7.6](#76-alternative-opencodes-built-in-ollama-cloud-provider)) | Add the key via `/connect` (OpenCode stores it itself) | No |
| Custom direct-API provider ([7.3](#73-direct-cloud-api-provider)) | `OLLAMA_API_KEY` env var | No, only a `{env:OLLAMA_API_KEY}` reference |

Never paste a raw key into a file that goes into git.

### Option 1: via the local Ollama (simplest)

Sign in once:

```bash
ollama signin
```

Then use any cloud model. Requests are offloaded automatically while you keep using the local API at `localhost:11434`:

```bash
ollama run glm-5.3:cloud
```

To use it in OpenCode, run `ollama launch opencode --model glm-5.3:cloud` ([section 8](#8-fastest-setup-ollama-launch)) or add the model to `opencode.json` ([section 7.2](#72-cloud-models-through-local-ollama)).

### Option 2: direct API (no local Ollama needed)

1. Create an API key at <https://ollama.com/settings/keys>.
2. Export it:

   ```bash
   export OLLAMA_API_KEY="your_api_key_here"
   ```

   Add to `~/.bashrc` / `~/.zshrc` to persist. On Windows PowerShell: `setx OLLAMA_API_KEY "your_api_key_here"`.

3. Test it:

   ```bash
   curl https://ollama.com/api/tags -H "Authorization: Bearer $OLLAMA_API_KEY"
   ```

4. Use the cloud provider config from [section 7.3](#73-direct-cloud-api-provider). Model IDs here generally have **no** cloud suffix (for example `gpt-oss:120b`). The `/api/tags` call above shows the exact IDs available to your account.

> Never commit your API key to git. Reference it through an environment variable, as shown below.

## 7. Configure OpenCode

OpenCode reads JSON config from:

- **Global:** `~/.config/opencode/opencode.json`
- **Per project:** `opencode.json` in the project root (overrides global)

### 7.1 Local Ollama provider

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen3-coder:30b": {
          "name": "Qwen3 Coder 30B"
        },
        "gpt-oss:20b": {
          "name": "GPT-OSS 20B"
        }
      }
    }
  },
  "model": "ollama/qwen3-coder:30b"
}
```

Rules:

- The keys under `models` must match the exact names from `ollama list`.
- `"model"` uses the format `provider-id/model-name`.
- If you built a custom model (for example `qwen3-coder-64k`), add it under `models` too.

### 7.2 Cloud models through local Ollama

Same as above, just add the cloud model names (after `ollama signin`):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen3-coder:30b": { "name": "Qwen3 Coder 30B (local)" },
        "glm-5.3:cloud": { "name": "GLM 5.3 (cloud)" }
      }
    }
  },
  "model": "ollama/glm-5.3:cloud"
}
```

### 7.3 Direct cloud API provider

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama-cloud": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama Cloud",
      "options": {
        "baseURL": "https://ollama.com/v1",
        "apiKey": "{env:OLLAMA_API_KEY}"
      },
      "models": {
        "gpt-oss:120b": { "name": "GPT-OSS 120B" }
      }
    }
  },
  "model": "ollama-cloud/gpt-oss:120b"
}
```

`{env:OLLAMA_API_KEY}` reads the key from your environment so it never lands in the file.

### 7.4 Optional: declare context limits

Newer OpenCode versions accept a `limit` block so OpenCode knows the model's window (this does **not** change what Ollama actually allocates; set that in [5.3](#53-increase-the-context-window-important)):

```json
"models": {
  "qwen3-coder:30b": {
    "name": "Qwen3 Coder 30B",
    "limit": { "context": 65536, "output": 8192 }
  }
}
```

### 7.5 Validate

```bash
opencode models          # lists models OpenCode can see
```

### 7.6 Alternative: OpenCode's built-in Ollama Cloud provider

For direct cloud use you can skip the JSON provider block:

1. Create an API key at <https://ollama.com/settings/keys>.
2. Run `opencode`, type `/connect`, then search for **Ollama Cloud** and paste your API key.
3. Run `/models` and pick a cloud model.

If a brand-new model doesn't appear in the picker, OpenCode's model cache may not know it yet (this has happened with new model families). Launching via `ollama launch opencode --model <model>` or adding the model manually in `opencode.json` works around it.

## 8. Fastest setup: `ollama launch`

Ollama (v0.15 and newer) can configure and start OpenCode for you, with no JSON editing:

```bash
# guided: pick a model interactively, then start OpenCode
ollama launch opencode

# choose the model directly
ollama launch opencode --model glm-5.3:cloud        # cloud (run `ollama signin` first)
ollama launch opencode --model qwen3-coder          # local (pull it first)

# write the config only, don't start OpenCode
ollama launch opencode --config
```

Check `ollama launch --help` for options in your version. The manual config in [section 7](#7-configure-opencode) always works and gives you more control (multiple models, custom names, context limits).

## 9. Using OpenCode

```bash
cd /path/to/your/project
opencode
```

First-run steps:

1. Run `/init` inside OpenCode to generate an `AGENTS.md` describing your project.
2. Run `/models` to pick your Ollama model.
3. Type a task, for example: `Explain the structure of this repo`.

Handy controls:

| Action | How |
|---|---|
| Switch model | `/models` |
| Toggle Plan / Build agent | `Tab` |
| Reference a file | `@filename` in your prompt |
| Run a shell command | start a line with `!` |
| Undo / redo last changes | `/undo`, `/redo` |
| New session | `/new` |
| Help | `/help` |

Non-interactive use:

```bash
opencode run "Add type hints to utils.py"
```

**Plan mode** is safer for exploring: it analyzes without editing files. Switch to **Build** when you're ready to apply changes. Use git so you can review and revert:

```bash
git status
git diff
```

## 10. Tuning and performance

- Run `ollama ps`. If it shows a CPU/GPU split, the model is too large for your VRAM; use a smaller model, a quantized variant, or a shorter context.
- Keep a model loaded longer: `OLLAMA_KEEP_ALIVE=30m`.
- Larger context = more memory. Ollama recommends at least 64k for OpenCode; go lower only if memory forces you to.
- Flash attention is enabled by default on supported GPUs; no flag needed.
- For big tasks on modest hardware, use a cloud model. For sensitive code, prefer local models.

## 11. Windows + WSL notes

If Ollama runs on **Windows** and OpenCode inside **WSL**:

- With WSL mirrored networking, `http://localhost:11434/v1` often works directly.
- Otherwise, expose Ollama to WSL by setting `OLLAMA_HOST=0.0.0.0` on Windows (restart Ollama), allow it through the firewall, and use the Windows host IP as `baseURL`:

  ```bash
  # inside WSL: find the Windows host IP
  ip route show | grep -i default | awk '{ print $3 }'
  ```

  ```json
  "baseURL": "http://<windows-host-ip>:11434/v1"
  ```

Alternatively, install both Ollama and OpenCode inside WSL and use `localhost`.

> Setting `OLLAMA_HOST=0.0.0.0` exposes the API to your network. Restrict it with a firewall and avoid this on untrusted networks.

## 12. Troubleshooting

| Problem | Likely cause / fix |
|---|---|
| `connection refused` on port 11434 | Ollama isn't running: `ollama serve` or `systemctl start ollama` |
| Model not listed in OpenCode | Name in `opencode.json` doesn't exactly match `ollama list`; restart OpenCode |
| Model ignores tools / just prints JSON | Model lacks tool-calling support; pick one from the tools category |
| Agent forgets earlier context or acts erratically | Context too small; raise `num_ctx` / `OLLAMA_CONTEXT_LENGTH` (see 5.3) |
| Very slow responses | Model spilling to CPU (`ollama ps`); use a smaller model or shorter context |
| Cloud model errors / auth failures | Run `ollama signin` again, or verify `OLLAMA_API_KEY` is exported in the same shell |
| `401 Unauthorized` on direct API | Missing or wrong key; check the `Authorization: Bearer` header / env var |
| Config changes not applied | Check JSON syntax (no trailing commas); project `opencode.json` overrides global |
| Linux GPU not used | Check drivers (`nvidia-smi` / ROCm) and `journalctl -u ollama` |

Useful diagnostics:

```bash
ollama --version
opencode --version
curl http://localhost:11434/api/tags      # models the API sees
journalctl -u ollama -f                   # Linux server logs
```

## 13. Cheat sheet

```bash
# Ollama
ollama pull <model>              # download
ollama run <model>               # chat
ollama list | ollama ps          # installed | running
ollama signin                    # enable cloud models
ollama create <name> -f Modelfile
ollama launch opencode --model glm-5.3:cloud   # start with a specific model

# OpenCode
opencode                         # start TUI in current project
opencode run "<prompt>"          # one-shot
opencode models                  # list models
```

**Minimal working local setup in 4 commands:**

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen3-coder:30b
curl -fsSL https://opencode.ai/install | bash
# then create opencode.json from section 7.1 and run: opencode
```

## References

- OpenCode docs: <https://opencode.ai/docs>
- OpenCode providers: <https://opencode.ai/docs/providers>
- Ollama docs: <https://docs.ollama.com>
- Ollama model library: <https://ollama.com/library>
- Live list of cloud models: <https://ollama.com/search?c=cloud>
- Ollama OpenCode integration: <https://docs.ollama.com/integrations/opencode>
