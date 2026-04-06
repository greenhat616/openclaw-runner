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
