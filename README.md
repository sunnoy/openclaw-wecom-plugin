# OpenClaw WeCom (Enterprise WeChat) AI Bot Plugin

[简体中文](https://github.com/sunnoy/openclaw-plugin-wecom/blob/main/README_ZH.md) | [English](https://github.com/sunnoy/openclaw-plugin-wecom/blob/main/README.md)

`openclaw-plugin-wecom` is a WeCom (Enterprise WeChat) integration plugin developed for the [OpenClaw](https://github.com/openclaw/openclaw) framework. It enables seamless integration of powerful AI capabilities into WeCom with advanced features.

## ✨ Key Features

- 🌊 **Streaming Output**: Smooth typewriter-style responses using WeCom's latest AI bot streaming mechanism.
- 🤖 **Dynamic Agent Management**: Automatically creates independent Agents per user/group chat with isolated workspaces and conversation contexts.
- 👥 **Group Chat Integration**: Full support for group messages with @mention triggering.
- 🛠️ **Command Support**: Built-in commands (`/new`, `/status`, `/help`, `/compact`) with configurable whitelist.
- 🔒 **Security**: Complete support for WeCom message encryption/decryption and sender verification.
- ⚡ **Async Processing**: High-performance async architecture ensures gateway responsiveness during AI inference.

## 🚀 Quick Start

### Option 1: Docker Deployment (Recommended)

This repository provides a complete Docker deployment solution with automated plugin installation.

```bash
# 1. Clone the repository
git clone https://github.com/sunnoy/openclaw-plugin-wecom.git
cd openclaw-plugin-wecom/deploy

# 2. Copy environment configuration
cp .env.example .env

# 3. Edit .env file with your settings
vim .env

# 4. Run deployment script
./deploy.sh
```

The deployment script automatically:
- Creates data directories and sets permissions
- Generates configuration files
- Starts Docker containers
- Installs the WeCom plugin
- Configures and restarts services

#### 🌟 Deployment Highlights

**Custom Data Directory & Agent Workspace Paths**

The core advantage of this deployment is unified data storage in a custom path, effectively utilizing data disks:

```bash
# .env configuration example
OPENCLAW_DATA_DIR=/data/openclaw    # Custom data directory
```

- **OpenClaw State Directory**: `/data/openclaw/`
- **Dynamic Agent Workspace**: `/data/openclaw/.openclaw/` 
- **Plugin Directory**: `/data/openclaw/extensions/`
- **Canvas Data**: `/data/openclaw/canvas/`

Benefits:
- ✅ All Agent workspace data stored on data disk, avoiding system disk usage
- ✅ Independent Agent files for each user/group managed under unified path
- ✅ Easy backup, migration, and expansion
- ✅ Enterprise-ready deployment with independently mountable data disks

### Option 2: Manual Plugin Installation

Install in an existing OpenClaw environment:

```bash
openclaw plugins install openclaw-plugin-wecom
```

Or via npm:

```bash
npm install openclaw-plugin-wecom
```

Then add to your OpenClaw configuration:

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
      "token": "Your Token",
      "encodingAesKey": "Your EncodingAESKey"
    }
  }
}
```

### WeCom Backend Setup

1. Create an "Intelligent Bot" in WeCom Admin Console.
2. Set the "Receive Message" URL to your service address (e.g., `https://your-domain.com/webhooks/wxwork`).
3. Enter the corresponding Token and EncodingAESKey.

## 📂 Project Structure

```
openclaw-plugin-wecom/
├── deploy/                      # Deployment files
│   ├── deploy.sh               # One-click deployment script
│   ├── docker-compose.yml      # Docker Compose configuration
│   ├── .env.example            # Environment variables template
│   ├── openclaw.json.base      # Base configuration template
│   └── openclaw.json.template  # Full configuration template
├── Dockerfile                   # OpenClaw image build file
├── local.sh                     # Local image build script
├── index.js                     # Plugin entry point
├── webhook.js                   # WeCom HTTP communication
├── dynamic-agent.js             # Dynamic Agent routing
├── stream-manager.js            # Streaming response management
├── crypto.js                    # WeCom encryption
└── client.js                    # Client logic
```

## 🤖 Dynamic Agent Routing

The plugin implements per-user/per-group isolation:

1. On message arrival, generates a deterministic `agentId`:
   - DM: `wxwork-dm-<userId>`
   - Group: `wxwork-group-<chatId>`
2. OpenClaw automatically creates/reuses the corresponding Agent workspace.

### Configuration Options

Under `channels.wxwork`:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `dynamicAgents.enabled` | boolean | `true` | Enable dynamic Agents |
| `dm.createAgentOnFirstMessage` | boolean | `true` | Use dynamic Agent for DMs |
| `groupChat.enabled` | boolean | `true` | Enable group chat handling |
| `groupChat.requireMention` | boolean | `true` | Require @mention in groups |

To route all messages to the default Agent:

```json
{
  "channels": {
    "wxwork": {
      "dynamicAgents": { "enabled": false }
    }
  }
}
```

## 🛠️ Command Whitelist

To prevent regular users from executing sensitive Gateway management commands via WeCom messages, this plugin supports a **command whitelist** mechanism. Only commands in the whitelist will be executed; others are ignored.

> 💡 **Note**: This configuration is already included in `deploy/openclaw.json.template` and takes effect automatically upon deployment.

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

| Command | Description | Security Level |
|---------|-------------|----------------|
| `/new` | Reset conversation, start fresh | ✅ User-level |
| `/compact` | Compress conversation context | ✅ User-level |
| `/help` | Show help information | ✅ User-level |
| `/status` | Show Agent status | ✅ User-level |

> ⚠️ **Security Note**: Do not add `/gateway`, `/plugins`, or other management commands to the whitelist to prevent regular users from gaining Gateway instance admin privileges.

## 🏗️ Building Custom Images

To build a custom OpenClaw image:

```bash
# Build image (uses China npm mirror)
./local.sh v2026.1.29

# Build and push image
./local.sh v2026.1.29 push
```

## 🤝 Contributing

We welcome contributions! Please submit Issues or Pull Requests for bugs or feature suggestions.

## 📄 License

This project is licensed under the [ISC License](./LICENSE).
