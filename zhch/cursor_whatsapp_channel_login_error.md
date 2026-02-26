**User**

@nanobot/zhch/cursor_whatsapp.md
nanobot通过调整程序设置代理，可以login whatsapp

---

**Cursor**

查找 nanobot 桥接中实现代理的代码：

正在为 OpenClaw 添加与 nanobot 相同的 WhatsApp 代理支持，先查找 `channels login` 的完整调用链：

检查 `resolveWhatsAppAccount` 和配置结构：

实施代理支持：

正在实现代理支持：

传递 proxy 到 login-qr、inbound/monitor 和 login（含 restartLoginSocket）：

正在修复：`restartLoginSocket` 中 `account` 未定义，需将 `proxy` 加入 `ActiveLogin` 并在创建时设置。

运行 `pnpm install` 并验证构建：

正在运行格式化修复：

OpenClaw 已支持 WhatsApp 代理，用法与 nanobot 类似。

## 修改说明

1. **配置项**：在 `channels.whatsapp` 中增加 `proxy`，支持：
   - 顶层：`channels.whatsapp.proxy`
   - 账号级：`channels.whatsapp.accounts.<id>.proxy`（会覆盖顶层）

2. **代理类型**：
   - HTTP/HTTPS：如 `http://127.0.0.1:7890`（Clash 常用）
   - SOCKS5：如 `socks5://127.0.0.1:7891`

3. **使用方式**：在 `~/.openclaw/openclaw.json` 中配置：

```json
{
  "channels": {
    "whatsapp": {
      "proxy": "http://127.0.0.1:7890",
      "allowFrom": ["+你的手机号"]
    }
  }
}
```

4. **涉及模块**：登录扫码、gateway 监听和 `channels login` 均会通过 `proxy` 连接 web.whatsapp.com。

## 使用步骤

1. 启动 Clash 等代理，并确认 HTTP 代理端口（如 7890）
2. 在配置中加入 `proxy`
3. 执行：`openclaw channels login --channel whatsapp`

若仍出现 408，可尝试：

- 用 `http://127.0.0.1:7890` 替代 `socks5://127.0.0.1:7891`（和 nanobot 的体验一致）
- 确认 Clash 中代理节点可用，并支持访问 WhatsApp

---

**User**

[192.168.1.33:~/workspace/repository.git/DeepCode]$ openclaw channels login --channel whatsapp
Invalid config at /Users/zhch158/.openclaw/openclaw.json:\n- channels.whatsapp: Unrecognized key: "proxy"

🦞 OpenClaw 2026.2.22-2 (45febec) — The UNIX philosophy meets your DMs.

│
◇ Config ───────────────────────────────────────────────────╮
│ │
│ Config invalid; doctor will run with best-effort config. │
│ │
├────────────────────────────────────────────────────────────╯
│
◇ Doctor changes ──────────────────────────────╮
│ │
│ WhatsApp configured, enabled automatically. │
│ │
├───────────────────────────────────────────────╯
│
◇ Unknown config keys ─────────╮
│ │
│ - channels.whatsapp.proxy │
│ - channels.whatsapp.enabled │
│ │
├───────────────────────────────╯
│
◇ Doctor ──────────────────────────────────────────────╮
│ │
│ Run "openclaw doctor --fix" to apply these changes. │
│ Run "openclaw doctor --fix" to remove these keys. │
│ │
├───────────────────────────────────────────────────────╯
Config invalid
File: ~/.openclaw/openclaw.json
Problem:

- channels.whatsapp: Unrecognized key: "proxy"

Run: openclaw doctor --fix
[192.168.1.33:~/workspace/repository.git/DeepCode]$

---

**Cursor**

查找 config 校验中 “Unrecognized key” 的来源：

“Unrecognized key: proxy” 表示当前运行的 OpenClaw **还不认识** `proxy` 这个配置项。

## 原因

`channels.whatsapp.proxy` 是在本地 openclaw 仓库里刚加上的新配置，而你用的 OpenClaw 版本（2026.2.22-2）很可能是：

- 通过 `npm install -g openclaw` 装的旧版本，或者
- 从还未包含这些修改的源码构建的

所以当前使用的 schema 里还没有 `proxy`，会报 “Unrecognized key”。

## 做法

### 方案 1：用带 proxy 支持的新构建（推荐）

在 openclaw 仓库里构建并用本地构建的 CLI 登录：

```bash
cd /Users/zhch158/workspace/repository.git/openclaw
pnpm build
pnpm openclaw channels login --channel whatsapp
```

如果 192.168.1.33 上也有 openclaw 仓库，则在该机器上执行：

```bash
cd ~/workspace/repository.git/openclaw   # 或实际 openclaw 仓库路径
pnpm build
pnpm openclaw channels login --channel whatsapp
```

### 方案 2：等发布后再用 proxy

等包含 `proxy` 支持的版本正式发布后，再升级并配置：

```json
"channels": {
  "whatsapp": {
    "proxy": "http://127.0.0.1:7890",
    ...
  }
}
```

### 方案 3：临时去掉 proxy 配置，继续使用旧版

如果暂时不想用 proxy，可以先从 `~/.openclaw/openclaw.json` 中删除 `proxy` 字段，旧版就能正常加载配置。
