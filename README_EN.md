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
