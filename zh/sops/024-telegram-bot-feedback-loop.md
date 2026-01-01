# SOP: Telegram Bot 问题反馈与修复闭环

> 通过 Bot 机器人建立正反馈闭环机制，实现用户反馈到修复上线的全流程自动化

**文档编号**: 024
**日期**: 2025-01-02
**标签**: `Telegram Bot` `反馈闭环` `自动化` `DevOps`

---

## 核心价值

通过 Bot 机器人建立**正反馈闭环机制**：

```
用户反馈 → Bot 拉取 → AI 总结 → 立即修复 → 自动部署 → Bot 通知 → 用户确认
    ↑                                                              ↓
    └──────────────────── 闭环 ────────────────────────────────────┘
```

**关键点**：
- **即时响应**：Bot 拉取群消息，无需人工监控
- **问题聚合**：AI 自动总结多条消息中的问题
- **快速闭环**：修复后 Bot 主动通知，用户立即验证
- **正向循环**：用户感受到反馈被重视，更愿意持续反馈

## 传统 vs 闭环

| 传统流程 | 闭环流程 |
|---------|---------|
| 用户反馈后不知道有没有人看 | Bot 拉取 = 开发者已收到 |
| 修复后用户不知道 | Bot 通知 = 用户立即知道 |
| 反馈石沉大海 | 形成正反馈，鼓励持续反馈 |
| 开发者被动响应 | 开发者主动拉取、主动通知 |

## 流程架构

```
Telegram 群消息 → Bot API 读取 → AI 分析问题 → 定位代码 → 修复 → 部署 → Bot 通知
```

### 实际案例

#### 背景

用户在 Telegram 群里反馈了管理后台的问题：
- "这个管理后台是不是调整不了"
- "刚才操作把配置给删除了"
- "现在加不上去了"
- "1.0 可以，设置 1.2 倍不行"

#### 执行过程

**第一步：读取群消息**

```bash
# 调用 Telegram Bot API 获取最新消息
curl -s "https://api.telegram.org/bot${BOT_TOKEN}/getUpdates?offset=-20"
```

从消息中提取关键信息：
- 问题现象：设置 1.2 倍失败
- 错误提示：合约错误，无法预估 Gas
- 附带截图：钱包报错界面

**第二步：AI 分析问题**

通过截图分析（Gemini 多模态）识别到：
- MetaMask 钱包显示 "execution reverted" 错误
- 交易无法预估 Gas，说明合约调用参数有问题

结合代码分析：
```solidity
// 合约定义
uint256 public constant MULTIPLIER_PRECISION = 1000;

// 前端定义 - 错误！
const CONFIG = {
  MULTIPLIER_PRECISION: 10000,  // 应该是 1000
}
```

**问题定位**：
- 用户输入 1.2 倍
- 前端计算：`1.2 * 10000 = 12000`
- 合约验证：`multiplier > 10000` 触发 `InvalidMultiplier()` 错误
- 实际应该：`1.2 * 1000 = 1200`

**第三步：发现第二个问题**

再次读取群消息，发现还有一个问题：
- "代币充不进去"

分析代码发现页面缺少 ERC20 授权流程：
- depositReward 需要先 approve 代币给合约
- 页面缺少授权按钮

**第四步：修复代码**

修复 1 - 精度常量：
```typescript
const CONFIG = {
  MULTIPLIER_PRECISION: 1000,  // 与合约保持一致
}
```

修复 2 - 添加授权流程：
```typescript
const { needsApproval, approve, isApproving } = useTokenApproval({
  tokenAddress: rewardToken,
  spenderAddress: stakingAddress,
  requiredAmount: depositAmountWei,
})

// 添加授权按钮
{needsApproval && (
  <button onClick={approve}>授权</button>
)}
```

**第五步：部署更新**

```bash
# 构建并部署
pnpm build
firebase deploy --only hosting:your-app
```

**第六步：Bot 通知用户**

```bash
# 发送修复通知到群
curl -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
  -d "chat_id=${CHAT_ID}" \
  -d "text=管理后台已更新：
1. 修复倍数设置问题
2. 添加代币充值授权流程
请刷新页面重试"
```

## 关键配置

### Telegram Bot 配置

```yaml
bot_token: "your_bot_token"
chat_id: "your_chat_id"  # 群组 ID 通常为负数
```

### API 调用示例

**读取消息**：
```bash
GET https://api.telegram.org/bot{token}/getUpdates?offset=-20
```

**发送消息**：
```bash
POST https://api.telegram.org/bot{token}/sendMessage
Content-Type: application/x-www-form-urlencoded

chat_id={chat_id}&text={message}
```

**下载文件**（如截图）：
```bash
# 1. 获取文件路径
GET https://api.telegram.org/bot{token}/getFile?file_id={file_id}

# 2. 下载文件
GET https://api.telegram.org/file/bot{token}/{file_path}
```

## 闭环的威力

### 为什么"通知"很重要？

传统开发中，修复 Bug 后开发者觉得"完成了"，但用户不知道：
- 用户不确定问题是否被看到
- 用户不知道什么时候修好
- 用户需要自己反复检查

**Bot 通知建立了闭环**：
- 用户收到通知 → 知道问题被重视
- 用户立即验证 → 确认修复有效
- 用户更愿意反馈 → 形成正向循环

### 实际效果

本次案例中：
1. 用户反馈问题
2. 30 分钟内修复并部署
3. Bot 发送通知到群
4. 用户立即验证，确认问题解决

**这就是正反馈机制的价值**。

## 最佳实践

### 1. 消息解析技巧

- 关注带截图的消息（通常包含关键错误信息）
- 提取用户描述的操作步骤
- 注意"可以/不行"等对比描述

### 2. 问题定位策略

- 从错误信息反向追踪代码路径
- 对比合约和前端的常量定义
- 检查是否缺少必要的流程步骤（如授权）

### 3. 通知模板

```
[项目名] 更新通知

修复内容：
1. [问题描述1]
2. [问题描述2]

请刷新页面后重试，如有问题请继续反馈。
```

## 自动化脚本

可以将整个流程封装为脚本：

```bash
#!/bin/bash
# scripts/tg-feedback-handler.sh

BOT_TOKEN="your_bot_token"
CHAT_ID="your_chat_id"

# 读取最新消息
fetch_messages() {
  curl -s "https://api.telegram.org/bot${BOT_TOKEN}/getUpdates?offset=-20"
}

# 发送通知
send_notification() {
  local message=$1
  curl -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
    -d "chat_id=${CHAT_ID}" \
    -d "text=${message}"
}

# 主流程
main() {
  echo "=== 获取群消息 ==="
  fetch_messages | jq '.result[-5:] | .[].message.text'

  echo "=== 分析问题后执行修复 ==="
  # ... 修复代码 ...

  echo "=== 部署更新 ==="
  pnpm build
  firebase deploy --only hosting:your-app

  echo "=== 发送通知 ==="
  send_notification "已更新，请刷新页面重试"
}

main
```

## 总结

### 核心理念

**Bot 不只是工具，是建立信任的桥梁**。

传统模式下，用户反馈像是"扔进黑洞"，不知道有没有人看、什么时候修。Bot 闭环机制改变了这一切：

```
用户反馈 ──→ Bot 拉取（我收到了）
    ↓
问题修复 ──→ Bot 通知（已经修好了）
    ↓
用户验证 ──→ 正反馈（下次还会反馈）
```

### 三个关键动作

1. **Bot 拉取** = 告诉用户"我在听"
2. **快速修复** = 证明反馈有价值
3. **Bot 通知** = 闭环，建立信任

### 适用场景

- 小团队快速迭代
- 用户反馈驱动的产品
- 需要建立用户信任的早期项目
- 任何需要"让用户感到被重视"的场景

## 参考资源

- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Firebase Hosting](https://firebase.google.com/docs/hosting)
