# OpenClaw 安装指南 🦞

> 一款支持多平台（WhatsApp、Telegram、Discord、iMessage 等）的自托管 AI Agent 网关

**更新时间：** 2026-03-25（新增局域网访问配置）

---

## 目录

- [简介](#简介)
- [系统要求](#系统要求)
- [安装方式](#安装方式)
  - [方式一：安装脚本（推荐）](#方式一安装脚本推荐)
- [安装后配置](#安装后配置)
  - [局域网访问配置](#局域网访问配置)
- [验证安装](#验证安装)
- [常见问题](#常见问题)

---

## 简介

OpenClaw 是一个**自托管网关**，可以将你的聊天应用（微信海外版、Telegram、Discord 等）连接到 AI 助手。你在自己的电脑或服务器上运行一个 Gateway 进程，它就成为了消息应用和 AI 助手之间的桥梁。

**特点：**
- ✅ 自托管，数据完全由你控制
- ✅ 支持多平台同时连接
- ✅ 内置多 Agent 路由
- ✅ 支持图片、音频、文件收发
- ✅ Web 控制面板

---

## 系统要求

| 项目 | 要求 |
|------|------|
| Node.js | **24**（推荐）或 22 LTS（22.16+） |
| 操作系统 | macOS、Linux、Windows（推荐 WSL2） |
| 其他 | npm 或 pnpm |

> ⚠️ Windows 用户强烈推荐使用 WSL2

### Windows 用户必看：安装前检查脚本执行策略

打开 PowerShell，输入以下命令检查：

```powershell
Get-ExecutionPolicy
```

- 如果显示 **RemoteSigned** → 可以直接安装
- 如果显示 **Restricted**（完全禁止运行脚本）→ 需要先修改策略：
  ```powershell
  Set-ExecutionPolicy RemoteSigned
  ```
  然后再次运行 `Get-ExecutionPolicy` 确认已改为 RemoteSigned

---

## 安装方式

### 方式一：安装脚本（推荐）

安装脚本会自动检测 Node、安装 CLI 并引导你完成初始化配置。

#### macOS / Linux / WSL2

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

#### Windows (PowerShell)

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

安装完成后会自动进入引导向导。

**跳过引导，直接安装：**

```bash
# macOS / Linux
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard

# Windows
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
```

---

## 安装后配置

### 启动网关

```bash
# 启动网关服务
openclaw gateway --port 18789

# 或者作为后台服务运行
openclaw onboard --install-daemon
```

### 连接聊天渠道

```bash
# 登录 WhatsApp
openclaw channels login

# 查看状态
openclaw status
```

### 打开控制面板

启动后，在浏览器中打开：

- **本地：** http://127.0.0.1:18789/
- **远程访问：** 需要配置 SSH 或 Tailscale

---

### 局域网访问配置

让局域网内其他设备也能访问 Web UI。

#### 方法一：使用配置向导（推荐）

1. 运行配置向导：
   ```bash
   openclaw onboard
   ```
2. 选择 `Enable LAN Access` 或类似选项
3. 向导会自动配置 `bind: lan` 并重启 Gateway

4. **手动添加 controlUi 配置**（向导无法直接配置）：
   ```bash
   nano ~/.openclaw/openclaw.json
   ```
   在 gateway 配置中添加：
   ```json
   "controlUi": {
     "dangerouslyAllowHostHeaderOriginFallback": true,
     "allowInsecureAuth": true,
     "dangerouslyDisableDeviceAuth": true
   }
   ```
5. 重启 Gateway：
   ```bash
   openclaw gateway restart
   ```

#### 方法二：手动配置

1. 备份配置文件：
   ```bash
   cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup
   ```

2. 编辑配置：
   ```bash
   nano ~/.openclaw/openclaw.json
   ```

3. 修改 gateway 配置：
   ```json
   {
     "gateway": {
       "port": 18789,
       "mode": "local",
       "bind": "lan",
       "auth": {
         "mode": "token",
         "token": "your_token_here"
       },
       "controlUi": {
         "dangerouslyAllowHostHeaderOriginFallback": true,
         "allowInsecureAuth": true,
         "dangerouslyDisableDeviceAuth": true
       }
     }
   }
   ```

4. 重启 Gateway：
   ```bash
   openclaw gateway restart
   ```

#### 获取局域网 IP

```bash
ip addr show | grep -E "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"
```

#### 访问 Web UI

局域网内其他设备访问：`http://<你的局域网IP>:18789/`

例如：`http://192.168.0.241:18789/`

#### 获取 Token

```bash
grep '"token"' ~/.openclaw/openclaw.json
```

> ⚠️ **安全提醒：** 启用局域网访问后，请在受信任的网络中使用，定期检查 Gateway 日志，轮换 Token 和 API Key。

---

## 验证安装

运行以下命令检查安装状态：

```bash
# 检查配置问题
openclaw doctor

# 查看网关状态
openclaw status

# 打开浏览器控制台
openclaw dashboard
```

---

## 常见问题

### 1. 找不到 `openclaw` 命令？

检查 PATH 是否包含 npm 全局 bin 目录：

```bash
# 查看 npm 全局路径
npm prefix -g

# 确保添加到 PATH（编辑 ~/.zshrc 或 ~/.bashrc）
export PATH="$(npm prefix -g)/bin:$PATH"

# 然后刷新
source ~/.zshrc
```

### 2. Node 版本不对？

推荐使用 Node 24：

```bash
# 使用 nvm 管理 Node 版本
nvm install 24
nvm use 24
```

### 3. Windows 下安装失败？

强烈推荐使用 WSL2（Windows Subsystem for Linux）：

```powershell
wsl --install
```

然后在 WSL2 终端中运行安装命令。

### 4. 更新 OpenClaw

```bash
npm install -g openclaw@latest
```

### 5. 卸载 OpenClaw

```bash
# 查看卸载说明
openclaw uninstall
```

---

## 相关链接

- 📖 官方文档：https://docs.openclaw.ai
- 💬 Discord 社区：https://discord.com/invite/clawd
- 🐙 GitHub：https://github.com/openclaw/openclaw
- 🛒 ClawHub（技能市场）：https://clawhub.com

---

*祝安装顺利！有问题随时提问 🐱*
