# OpenClaw 企业微信 (WeCom) AI 机器人插件

[简体中文](https://github.com/sunnoy/openclaw-plugin-wecom/blob/main/README_ZH.md) | [English](https://github.com/sunnoy/openclaw-plugin-wecom/blob/main/README.md)

`openclaw-plugin-wecom` 是一个专为 [OpenClaw](https://github.com/openclaw/openclaw) 框架开发的企业微信（WeCom）集成插件。它允许你将强大的 AI 能力无缝接入企业微信，并支持多项高级功能。

## ✨ 核心特性

- 🌊 **流式输出 (Streaming)**: 基于企业微信最新的 AI 机器人流式分片机制，实现流畅的打字机式回复体验。
- 🤖 **动态 Agent 管理**: 默认按"每个私聊用户 / 每个群聊"自动创建独立 Agent。每个 Agent 拥有独立的工作区与对话上下文，实现更强的数据隔离。
- 👥 **群聊深度集成**: 支持群聊消息解析，可通过 @提及（At-mention）精准触发机器人响应。
- 🛠️ **指令增强**: 内置常用指令支持（如 `/new` 开启新会话、`/status` 查看状态等），并提供指令白名单配置功能。
- 🔒 **安全与认证**: 完整支持企业微信消息加解密、URL 验证及发送者身份校验。
- ⚡ **高性能异步处理**: 采用异步消息处理架构，确保即使在长耗时 AI 推理过程中，企业微信网关也能保持高响应性。

## 🚀 快速开始

### 方式一：Docker 一键部署（推荐）

本仓库提供了完整的 Docker 部署方案，包含自动化的插件安装和配置。

```bash
# 1. 克隆仓库
git clone https://github.com/sunnoy/openclaw-plugin-wecom.git
cd openclaw-plugin-wecom/deploy

# 2. 复制环境变量配置
cp .env.example .env

# 3. 编辑 .env 文件，填写实际配置
vim .env

# 4. 运行部署脚本
./deploy.sh
```

部署脚本会自动执行：
- 创建数据目录和设置权限
- 生成配置文件
- 启动 Docker 容器
- 安装企业微信插件
- 配置并重启服务

#### 🌟 部署亮点

**自定义数据目录 & Agent Workspace 路径**

本部署方案的核心优势是将所有数据统一存储到自定义路径，有效利用数据盘：

```bash
# .env 配置示例
OPENCLAW_DATA_DIR=/data/openclaw    # 自定义数据目录
```

- **OpenClaw 状态目录**：`/data/openclaw/`
- **动态 Agent Workspace**：`/data/openclaw/.openclaw/` 
- **插件目录**：`/data/openclaw/extensions/`
- **Canvas 数据**：`/data/openclaw/canvas/`

这意味着：
- ✅ 所有 Agent 工作区数据存储在数据盘，避免占用系统盘空间
- ✅ 每个用户/群聊的独立 Agent 文件都在统一路径下管理
- ✅ 方便备份、迁移和扩容
- ✅ 适合企业级部署，数据盘可独立挂载和扩展

### 方式二：手动安装插件

在已有的 OpenClaw 环境中安装：

```bash
openclaw plugins install openclaw-plugin-wecom
```

或通过 npm：

```bash
npm install openclaw-plugin-wecom
```

然后在 OpenClaw 配置文件中添加：

```json
{
  "plugins": {
    "entries": {
      "openclaw-plugin-wecom": { "enabled": true }
    }
  },
  "channels": {
    "wxwork": {
      "enabled": true,
      "token": "你的 Token",
      "encodingAesKey": "你的 EncodingAESKey"
    }
  }
}
```

### 企业微信后台设置

1. 在企业微信管理后台创建一个"智能机器人"。
2. 将机器人的"接收消息配置"中的 URL 设置为你的服务地址（例如：`https://your-domain.com/webhooks/wxwork`）。
3. 填入对应的 Token 和 EncodingAESKey。

## 📂 项目结构

```
openclaw-plugin-wecom/
├── deploy/                      # 部署相关文件
│   ├── deploy.sh               # 一键部署脚本
│   ├── docker-compose.yml      # Docker Compose 配置
│   ├── .env.example            # 环境变量示例
│   ├── openclaw.json.base      # 基础配置模板
│   └── openclaw.json.template  # 完整配置模板
├── Dockerfile                   # OpenClaw 镜像构建文件
├── local.sh                     # 本地镜像构建脚本
├── index.js                     # 插件入口
├── webhook.js                   # 企业微信 HTTP 通信处理
├── dynamic-agent.js             # 动态 Agent 分配逻辑
├── stream-manager.js            # 流式回复管理
├── crypto.js                    # 企业微信加密算法
└── client.js                    # 客户端逻辑
```

## 🤖 动态 Agent 路由

OpenClaw 会通过解析 `SessionKey` 来决定本次消息由哪个 Agent 处理。本插件实现"按人/按群隔离"：

1. 企业微信消息到达后，插件生成确定性的 `agentId`：
   - 私聊：`wxwork-dm-<userId>`
   - 群聊：`wxwork-group-<chatId>`
2. OpenClaw 自动创建/复用对应的 Agent 工作区。

### 配置选项

配置在 `channels.wxwork` 下：

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `dynamicAgents.enabled` | boolean | `true` | 是否启用动态 Agent |
| `dm.createAgentOnFirstMessage` | boolean | `true` | 私聊使用动态 Agent |
| `groupChat.enabled` | boolean | `true` | 启用群聊处理 |
| `groupChat.requireMention` | boolean | `true` | 群聊必须 @ 提及才响应 |

如果需要所有消息进入默认 Agent：

```json
{
  "channels": {
    "wxwork": {
      "dynamicAgents": { "enabled": false }
    }
  }
}
```

## 🛠️ 指令白名单

为防止普通用户通过企业微信消息执行敏感的 Gateway 管理指令，本插件支持**指令白名单**机制。只有配置在白名单中的指令才会被执行，其他指令将被忽略。

> 💡 **提示**：此配置已包含在 `deploy/openclaw.json.template` 中，部署时会自动生效。

```json
{
  "channels": {
    "wxwork": {
      "commands": {
        "enabled": true,
        "allowlist": ["/new", "/status", "/help", "/compact"]
      }
    }
  }
}
```

| 指令 | 说明 | 安全级别 |
|------|------|----------|
| `/new` | 重置当前对话，开启全新会话 | ✅ 用户级 |
| `/compact` | 压缩当前会话上下文 | ✅ 用户级 |
| `/help` | 查看帮助信息 | ✅ 用户级 |
| `/status` | 查看当前 Agent 状态 | ✅ 用户级 |

> ⚠️ **安全提示**：不要将 `/gateway`、`/plugins` 等管理指令添加到白名单，避免普通用户获得 Gateway 实例的管理权限。

## 🏗️ 本地构建镜像

如果需要自定义构建 OpenClaw 镜像：

```bash
# 构建镜像（使用国内 npm 源）
./local.sh v2026.1.29

# 构建并推送镜像
./local.sh v2026.1.29 push
```

## 🤝 贡献规范

我们非常欢迎开发者参与贡献！如果你发现了 Bug 或有更好的功能建议，请提交 Issue 或 Pull Request。

## 📄 开源协议

本项目采用 [ISC License](./LICENSE) 协议。
