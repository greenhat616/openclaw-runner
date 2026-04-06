# OpenClaw Runner

[English](README_EN.md) | 中文

OpenClaw 一体化构建与部署容器镜像。预装完整的开发工具链，开箱即用。

## 预装工具

| 类别 | 工具 |
|------|------|
| JavaScript | Node.js 22+, npm, pnpm, Bun, Deno |
| Python | Python 3, pip, uv |
| Go | Go 1.23+ |
| Rust | rustc, cargo (via rustup) |
| C/C++ | GCC, Clang/LLVM 18, CMake, Ninja |
| Shell | Zsh (默认), Git, gh CLI |
| 网络诊断 | q (DNS), tcping, dig, nmap, mtr, traceroute, curl, socat |
| 系统工具 | htop, tmux, vim, nano, strace, lsof |

## 快速开始

### Docker Compose (推荐)

```bash
git clone https://github.com/greenhat616/openclaw-runner.git
cd openclaw-runner
```

创建 `.env` 文件：

```env
OPENCLAW_GATEWAY_TOKEN=your-token-here
ANTHROPIC_API_KEY=sk-ant-xxx
# OPENAI_API_KEY=sk-xxx
# OPENCLAW_PORT=18789
```

启动：

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

## 镜像源

| Registry | 地址 |
|----------|------|
| GitHub Packages | `ghcr.io/greenhat616/openclaw-runner` |
| 阿里云 ACR | `registry.cn-hangzhou.aliyuncs.com/hitokoto/openclaw-runner` |

## 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `OPENCLAW_GATEWAY_TOKEN` | Gateway 认证 Token | - |
| `ANTHROPIC_API_KEY` | Anthropic API 密钥 | - |
| `OPENAI_API_KEY` | OpenAI API 密钥 | - |
| `OPENCLAW_AUTO_UPDATE` | 启动时自动更新 OpenClaw | `false` |
| `OPENCLAW_MAX_RESTARTS` | 最大快速重启次数 | `5` |
| `OPENCLAW_STABLE_SECS` | 判定为稳定运行的最短时长（秒） | `30` |
| `OPENCLAW_RESTART_DELAY` | 重启间隔（秒） | `2` |

## 数据卷

| 卷 | 挂载点 | 说明 |
|----|--------|------|
| `openclaw-home` | `/home/openclaw` | 用户主目录（zsh 历史、工具缓存） |
| `openclaw-config` | `/home/openclaw/.openclaw` | OpenClaw 配置文件 |
| `openclaw-workspace` | `/home/openclaw/.openclaw/workspace` | 工作空间（Skills、Agents 等） |

## 入口脚本行为

容器启动时，`entrypoint.sh` 依次执行：

1. 修复 volume 目录权限
2. 首次运行时初始化主目录（拷贝 zsh 配置骨架）
3. 若 OpenClaw 未安装，则通过 `npm install -g openclaw@latest` 安装
4. 加载 `~/.openclaw/.env` 环境变量
5. 启动 OpenClaw Gateway（带自动重启保护）

## 模型配置

OpenClaw 内置支持多种 LLM Provider，同时支持自定义 OpenAI 兼容端点。

### 内置 Provider

设置对应的 API Key 环境变量即可使用：

| Provider | 环境变量 | 模型引用格式 |
|----------|---------|-------------|
| OpenAI | `OPENAI_API_KEY` | `openai/gpt-5.4` |
| Anthropic | `ANTHROPIC_API_KEY` | `anthropic/claude-opus-4-6` |
| Google Gemini | `GEMINI_API_KEY` | `google/gemini-3.1-pro-preview` |
| OpenRouter | `OPENROUTER_API_KEY` | `openrouter/auto` |
| Groq | `GROQ_API_KEY` | `groq/...` |
| Mistral | `MISTRAL_API_KEY` | `mistral/...` |

支持 Key 轮换：`OPENAI_API_KEYS`（逗号分隔）或 `OPENAI_API_KEY_1`、`OPENAI_API_KEY_2` 等。Anthropic / Gemini 同理。

### 自定义 OpenAI 兼容端点

编辑 `~/.openclaw/openclaw.json`（容器内路径 `/home/openclaw/.openclaw/openclaw.json`，已通过 volume 持久化）：

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

`api` 适配器类型：

| 值 | 适用场景 |
|----|---------|
| `openai-completions` | OpenAI 兼容 API（vLLM、LM Studio、LiteLLM、Ollama 等） |
| `anthropic-messages` | Anthropic Messages API 兼容端点 |
| `openai-responses` | OpenAI Responses API |
| `google-generative-ai` | Google Generative AI |

> **注意**：OpenClaw 不读取 `OPENAI_BASE_URL` / `OPENAI_API_BASE` 环境变量，自定义端点必须通过配置文件设置。

### 切换模型

```bash
# CLI
openclaw models set anthropic/claude-opus-4-6
openclaw models set openai/gpt-5.4
openclaw models list

# 会话内
/model anthropic/claude-opus-4-6
/model list
```

配置文件方式（支持自动 fallback）：

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

更多详情参见 [OpenClaw 模型文档](https://docs.openclaw.ai/concepts/models) 和 [Provider 配置文档](https://docs.openclaw.ai/concepts/model-providers)。

## 构建镜像

```bash
docker build -t openclaw-runner .
```

自定义构建参数：

```bash
docker build \
  --build-arg NODE_MAJOR=22 \
  --build-arg GO_VERSION=1.23.4 \
  --build-arg LLVM_VERSION=18 \
  -t openclaw-runner .
```

## License

MIT
