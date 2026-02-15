# 6 Agent 群聊完整方案

> 研发 / 设计 / 产品 / 财务 / 管理者 / 运营

## 文档依据

- [providers](https://docs.openclaw.ai/providers) - 模型列表
- [multi-agent](https://docs.openclaw.ai/concepts/multi-agent) - Agent配置
- [session-tool](https://docs.openclaw.ai/concepts/session-tool) - Agent间通信
- [telegram](https://docs.openclaw.ai/channels/telegram) - 频道配置

## 6 个 Agent 规划

| Agent ID | 名称 | 推荐模型 | 触发词 | 功能 |
|----------|------|---------|--------|------|
| main | 管理者 | MiniMax-M2.5 | @管理者 | 协调/分发 |
| coder | 研发 | Claude Code | @研发 | 编程开发 |
| designer | 设计 | MiniMax-V1.6 | @设计 | UI/UX设计 |
| product | 产品 | Sonnet 4.5 | @产品 | 产品规划 |
| finance | 财务 | MiniMax-M2.5 | @财务 | 财务管理 |
| ops | 运营 | MiniMax-M2.1 | @运营 | 自媒体运营 |

## 模型推荐理由

- **管理者 - MiniMax-M2.5**：204,800 上下文，适合协调多 Agent
- **研发 - Claude Code**：编程优化模型，代码能力最强
- **设计 - MiniMax-V1.6**：支持视觉理解设计稿
- **产品 - Sonnet 4.5**：分析规划能力平衡
- **财务 - MiniMax-M2.5**：大上下文处理财务数据
- **运营 - MiniMax-M2.1**：内容生成成本最低

## 交互模式

- **分散式**：@研发 → 研发 Agent 回复
- **集中式**：@管理者 → sessions_send → 目标 Agent

## 目录结构

```
~/.openclaw/workspace/           ← 公共
~/.openclaw/workspace-main/    ← 管理者
~/.openclaw/workspace-coder/   ← 研发
~/.openclaw/workspace-designer/ ← 设计
~/.openclaw/workspace-product/ ← 产品
~/.openclaw/workspace-finance/ ← 财务
~/.openclaw/workspace-ops/     ← 运营
```

## 配置 JSON

```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "accounts": {
        "main": { "botToken": "【填入】" },
        "coder": { "botToken": "【填入】" },
        "designer": { "botToken": "【填入】" },
        "product": { "botToken": "【填入】" },
        "finance": { "botToken": "【填入】" },
        "ops": { "botToken": "【填入】" }
      },
      "groups": { "【群ID】": { "requireMention": false } }
    }
  },
  "agents": {
    "list": [
      { "id": "main", "default": true, "workspace": "~/.openclaw/workspace-main", "model": { "primary": "minimax/MiniMax-M2.5" } },
      { "id": "coder", "workspace": "~/.openclaw/workspace-coder", "model": { "primary": "anthropic/claude-code" } },
      { "id": "designer", "workspace": "~/.openclaw/workspace-designer", "model": { "primary": "minimax/MiniMax-V1.6" } },
      { "id": "product", "workspace": "~/.openclaw/workspace-product", "model": { "primary": "anthropic/claude-sonnet-4-5" } },
      { "id": "finance", "workspace": "~/.openclaw/workspace-finance", "model": { "primary": "minimax/MiniMax-M2.5" } },
      { "id": "ops", "workspace": "~/.openclaw/workspace-ops", "model": { "primary": "minimax/MiniMax-M2.1" } }
    ]
  },
  "bindings": [
    { "agentId": "main", "match": { "channel": "telegram", "accountId": "main" } },
    { "agentId": "coder", "match": { "channel": "telegram", "accountId": "coder" } },
    { "agentId": "designer", "match": { "channel": "telegram", "accountId": "designer" } },
    { "agentId": "product", "match": { "channel": "telegram", "accountId": "product" } },
    { "agentId": "finance", "match": { "channel": "telegram", "accountId": "finance" } },
    { "agentId": "ops", "match": { "channel": "telegram", "accountId": "ops" } }
  ]
}
```

## 🔑 模型 API Key 配置

| 模型 | 是否需要 API Key | 如何填写 |
|------|-----------------|----------|
| MiniMax-M2.5 | ✅ 不需要 | 已配置 OAuth，填 mini-max-oauth 即可 |
| MiniMax-M2.1 | ✅ 不需要 | 已配置 OAuth，填 mini-max-oauth 即可 |
| MiniMax-V1.6 | ✅ 不需要 | 已配置 OAuth，填 mini-max-oauth 即可 |
| Claude Code | ⚠️ 需要 | 需要你提供 Anthropic API Key |
| Sonnet 4.5 | ⚠️ 需要 | 需要你提供 Anthropic API Key |

## 你需要提供（直接发给我）

```
1. Telegram 机器人 Token（6个）：
main: 【从 @BotFather 获取】
coder: 【从 @BotFather 获取】
designer: 【从 @BotFather 获取】
product: 【从 @BotFather 获取】
finance: 【从 @BotFather 获取】
ops: 【从 @BotFather 获取】

2. Telegram 群 ID：
【转发消息给 @userinfobot 获取】

3. Anthropic API Key（可选）：
【如需 Claude Code 或 Sonnet，从 https://console.anthropic.com 获取】
```

## 实施任务

1. 整理 workspace
2. 创建 6 个 workspace
3. 创建配置文件
4. 配置 openclaw.json
5. 重启 Gateway
6. 验证

## Git 仓库

projects/6-agent-plan/
