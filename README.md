# OpenCode + Ollama Guide

A practical guide to running [OpenCode](https://opencode.ai), the terminal AI coding agent, on top of [Ollama](https://ollama.com), using **local models** or **Ollama Cloud models**.

## What's inside

| File | Description |
|---|---|
| [`opencode-and-ollama.md`](./opencode-and-ollama.md) | Full guide: installation, model selection, context tuning, cloud setup, OpenCode config, troubleshooting, cheat sheet |

## Who this is for

- Developers who want an AI coding agent in the terminal without depending on a paid API
- People who want to keep code local (private) but can fall back to cloud models for heavy tasks
- Linux, macOS, and Windows (PowerShell / WSL) users

## Quick start

```bash
# 1. Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 2. Get a tool-capable coding model
ollama pull qwen3-coder:30b

# 3. Install OpenCode
curl -fsSL https://opencode.ai/install | bash

# 4. Point OpenCode at Ollama (create opencode.json in your project)
```

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": { "baseURL": "http://localhost:11434/v1" },
      "models": { "qwen3-coder:30b": { "name": "Qwen3 Coder 30B" } }
    }
  },
  "model": "ollama/qwen3-coder:30b"
}
```

```bash
# 5. Run it
opencode
```

Fastest path with a cloud model (no JSON editing):

```bash
ollama signin
ollama launch opencode --model glm-5.3:cloud
```

The list of cloud models changes often: see <https://ollama.com/search?c=cloud> (filter by **Tools**). For details and the direct API option, see the [full guide](./opencode-and-ollama.md#6-path-b-ollama-cloud-models).

## Local vs cloud at a glance

| | Local models | Cloud models |
|---|---|---|
| Privacy | Code stays on your machine | Prompts are sent to Ollama's servers |
| Cost | Free (your hardware) | Account required; usage limits/pricing apply |
| Model size | Limited by RAM/VRAM | Very large models available |
| Offline use | Yes | No |
| Setup | `ollama pull <model>` | `ollama signin` or API key |

## Requirements

- Ollama (latest version recommended)
- OpenCode
- Local models: sufficient RAM/VRAM for your chosen model
- Cloud models: a free [ollama.com](https://ollama.com) account

## Security notes

- Never commit API keys. Use environment variables (`{env:OLLAMA_API_KEY}` in `opencode.json`).
- Review agent changes with `git diff` before committing.
- Avoid exposing Ollama (`OLLAMA_HOST=0.0.0.0`) on untrusted networks.

## References

- Full guide: [OpenCode + Ollama Setup Guide](./opencode-and-ollama.md)
- OpenCode docs: https://opencode.ai/docs
- OpenCode providers: https://opencode.ai/docs/providers
- Ollama docs: https://docs.ollama.com
- Ollama model library: https://ollama.com/library
- Ollama Cloud: https://ollama.com/cloud
- Live cloud model list: https://ollama.com/search?c=cloud
- Ollama + OpenCode integration: https://docs.ollama.com/integrations/opencode

## Contributing

Issues and pull requests are welcome, especially for corrections as Ollama and OpenCode evolve (commands and config formats change between versions).

## Disclaimer

This guide is distributed as a public git repository and is provided "as is", without warranty of any kind. It is an unofficial community guide and is not affiliated with Ollama or OpenCode. Commands, config formats, and model availability change between versions — always consult the official documentation for the latest details.

## License

Released under the [MIT License](./LICENSE.txt).
