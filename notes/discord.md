# OpenClaw Discord Channel

文档：https://docs.openclaw.ai/channels/discord

## 当前配置

配置文件：`~/.openclaw/openclaw.json`（`channels.discord` 字段）

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "<BOT_TOKEN>",
      "groupPolicy": "allowlist",
      "streaming": "partial",
      "guilds": {
        "<GUILD_ID>": {
          "requireMention": false
        }
      }
    }
  }
}
```

- `groupPolicy: "allowlist"` — 只有 guilds 字段中列出的服务器可以使用
- `streaming: "partial"` — 流式输出，边生成边编辑同一条消息
- `requireMention: false` — 在该服务器中无需 @bot 即可触发
- `dmPolicy` 未设置，默认 `"pairing"`（DM 需要配对码审批）

allowFrom 白名单：`~/.openclaw/credentials/discord-default-allowFrom.json`

---

## 核心配置项

### groupPolicy（服务器访问策略）

| 值 | 说明 |
|---|---|
| `"allowlist"` | 只允许 guilds 字段中列出的服务器 |
| `"open"` | 所有服务器都可以使用 |
| `"disabled"` | 关闭所有服务器频道 |

### dmPolicy（私信策略）

| 值 | 说明 |
|---|---|
| `"pairing"` | 默认，需要配对码审批（1小时有效） |
| `"allowlist"` | 只允许预设用户 |
| `"open"` | 完全开放（需配合 `allowFrom: "*"`） |
| `"disabled"` | 关闭私信 |

### streaming（流式输出模式）

| 值 | 说明 |
|---|---|
| `"off"` | 等全部生成完再发送 |
| `"partial"` | 边生成边编辑同一条消息（当前使用）|
| `"block"` | 分块发送，可配置断句方式 |
| `"progress"` | 等同于 `partial`（跨平台别名）|

### Guild 配置结构

```json
{
  "guilds": {
    "GUILD_ID": {
      "requireMention": false,
      "users": ["USER_ID"],
      "roles": ["ROLE_ID"],
      "channels": {
        "general": { "allow": true },
        "help": { "allow": true, "requireMention": true }
      }
    }
  }
}
```

---

## 常用调整

### 修改配置

```bash
# 直接编辑（推荐先停止容器再编辑，或修改后重启）
docker compose exec openclaw bash
nano ~/.openclaw/openclaw.json

# 或用 openclaw CLI 设置 token
openclaw config set channels.discord.token "YOUR_TOKEN"
```

### 配对私信用户（dmPolicy 为 pairing 时）

用户在私信中发送配对请求后，在容器内审批：
```bash
docker compose exec openclaw openclaw pair --approve
```

### 流式块配置（streaming: "block" 时可用）

```json
{
  "channels": {
    "discord": {
      "draftChunk": {
        "minChars": 200,
        "maxChars": 800,
        "breakPreference": "paragraph"
      }
    }
  }
}
```

### 超时配置（长任务）

```json
{
  "channels": {
    "discord": {
      "eventQueue": { "listenerTimeout": 120000 },
      "inboundWorker": { "runTimeoutMs": 1800000 }
    }
  }
}
```

---

## Bot 权限（创建时需要）

- View Channels
- Send Messages
- Read Message History
- Embed Links
- Attach Files
- Add Reactions（可选）

需要开启的 Intent：
- Message Content Intent
- Server Members Intent
