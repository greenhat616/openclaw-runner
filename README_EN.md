# OpenClaw Runner

English | [中文](README.md)

All-in-one build and deployment container image for OpenClaw. Ships with a complete development toolchain, ready to use out of the box.

## Pre-installed Tools

| Category | Tools |
|----------|-------|
| JavaScript | Node.js 22+, npm, pnpm, Bun, Deno |
| Python | Python 3, pip, uv |
| Go | Go 1.23+ |
| Rust | rustc, cargo (via rustup) |
| C/C++ | GCC, Clang/LLVM 18, CMake, Ninja |
| Shell | Zsh (default), Git, gh CLI |
| Networking | q (DNS), tcping, dig, nmap, mtr, traceroute, curl, socat |
| System | htop, tmux, vim, nano, strace, lsof |

## Quick Start

### Docker Compose (Recommended)

```bash
git clone https://github.com/greenhat616/openclaw-runner.git
cd openclaw-runner
```

Create a `.env` file:

```env
OPENCLAW_GATEWAY_TOKEN=your-token-here
ANTHROPIC_API_KEY=sk-ant-xxx
# OPENAI_API_KEY=sk-xxx
# OPENCLAW_PORT=18789
```

Start the container:

```bash
docker compose up -d
```

### Docker Run

```bash
docker run -d \
  --name openclaw \
  --restart unless-stopped \
  -p 18789:18789 \
  -e ANTHROPIC_API_KEY=sk-ant-xxx \
  -v openclaw-home:/home/openclaw \
  ghcr.io/greenhat616/openclaw-runner:latest
```

## Image Registries

| Registry | Address |
|----------|---------|
| GitHub Packages | `ghcr.io/greenhat616/openclaw-runner` |
| Aliyun ACR | `registry.cn-hangzhou.aliyuncs.com/hitokoto/openclaw-runner` |

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENCLAW_GATEWAY_TOKEN` | Gateway authentication token | - |
| `ANTHROPIC_API_KEY` | Anthropic API key | - |
| `OPENAI_API_KEY` | OpenAI API key | - |
| `OPENCLAW_AUTO_UPDATE` | Auto-update OpenClaw on startup | `false` |
| `OPENCLAW_MAX_RESTARTS` | Max rapid restart attempts | `5` |
| `OPENCLAW_STABLE_SECS` | Minimum uptime (seconds) to consider a run stable | `30` |
| `OPENCLAW_RESTART_DELAY` | Delay between restarts (seconds) | `2` |

## Volumes

| Volume | Mount Point | Description |
|--------|-------------|-------------|
| `openclaw-home` | `/home/openclaw` | User home directory (zsh history, tool caches) |
| `openclaw-config` | `/home/openclaw/.openclaw` | OpenClaw configuration |
| `openclaw-workspace` | `/home/openclaw/.openclaw/workspace` | Workspace (Skills, Agents, etc.) |

## Entrypoint Behavior

On container startup, `entrypoint.sh` performs the following steps:

1. Fixes volume directory ownership
2. Initializes the home directory on first run (copies zsh config skeleton)
3. Installs OpenClaw via `npm install -g openclaw@latest` if not present
4. Loads environment variables from `~/.openclaw/.env`
5. Starts the OpenClaw Gateway with automatic restart protection

## Model Configuration

OpenClaw has built-in support for multiple LLM providers and allows custom OpenAI-compatible endpoints.

### Built-in Providers

Just set the corresponding API key environment variable:

| Provider | Environment Variable | Model Reference |
|----------|---------------------|-----------------|
| OpenAI | `OPENAI_API_KEY` | `openai/gpt-5.4` |
| Anthropic | `ANTHROPIC_API_KEY` | `anthropic/claude-opus-4-6` |
| Google Gemini | `GEMINI_API_KEY` | `google/gemini-3.1-pro-preview` |
| OpenRouter | `OPENROUTER_API_KEY` | `openrouter/auto` |
| Groq | `GROQ_API_KEY` | `groq/...` |
| Mistral | `MISTRAL_API_KEY` | `mistral/...` |

Key rotation is supported via `OPENAI_API_KEYS` (comma-separated) or `OPENAI_API_KEY_1`, `OPENAI_API_KEY_2`, etc. Same pattern applies to Anthropic / Gemini.

### Custom OpenAI-Compatible Endpoints

Edit `~/.openclaw/openclaw.json` (container path `/home/openclaw/.openclaw/openclaw.json`, persisted via volume):

```json5
{
  "models": {
    "mode": "merge",
    "providers": {
      "my-proxy": {
        "baseUrl": "http://localhost:4000/v1",
        "apiKey": "${MY_CUSTOM_API_KEY}",
        "api": "openai-completions",
        "models": [
          { "id": "my-model", "name": "My Model", "contextWindow": 128000, "maxTokens": 8192 }
        ]
      }
    }
  }
}
```

Supported `api` adapter types:

| Value | Use Case |
|-------|----------|
| `openai-completions` | OpenAI-compatible APIs (vLLM, LM Studio, LiteLLM, Ollama, etc.) |
| `anthropic-messages` | Anthropic Messages API-compatible endpoints |
| `openai-responses` | OpenAI Responses API |
| `google-generative-ai` | Google Generative AI |

> **Note**: OpenClaw does **not** read `OPENAI_BASE_URL` / `OPENAI_API_BASE` environment variables. Custom endpoints must be configured via the config file.

### Switching Models

```bash
# CLI
openclaw models set anthropic/claude-opus-4-6
openclaw models set openai/gpt-5.4
openclaw models list

# In-session
/model anthropic/claude-opus-4-6
/model list
```

Config file (with automatic fallback):

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["openai/gpt-5.4"]
      }
    }
  }
}
```

See [OpenClaw Models Docs](https://docs.openclaw.ai/concepts/models) and [Provider Configuration](https://docs.openclaw.ai/concepts/model-providers) for more details.

## Building the Image

```bash
docker build -t openclaw-runner .
```

With custom build arguments:

```bash
docker build \
  --build-arg NODE_MAJOR=22 \
  --build-arg GO_VERSION=1.23.4 \
  --build-arg LLVM_VERSION=18 \
  -t openclaw-runner .
```

## License

MIT
