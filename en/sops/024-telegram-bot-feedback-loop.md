# SOP: Telegram Bot Feedback Loop for Bug Fixes

> Build a positive feedback loop using Bot automation for user feedback collection, bug fixing, and deployment notification

**Document ID**: 024
**Date**: 2025-01-02
**Tags**: `Telegram Bot` `Feedback Loop` `Automation` `DevOps`

---

## Core Value

Establish a **positive feedback loop** through Bot automation:

```
User Feedback → Bot Pull → AI Summary → Quick Fix → Auto Deploy → Bot Notify → User Verify
      ↑                                                                    ↓
      └─────────────────────── Closed Loop ────────────────────────────────┘
```

**Key Points**:
- **Instant Response**: Bot pulls group messages, no manual monitoring needed
- **Issue Aggregation**: AI automatically summarizes issues from multiple messages
- **Fast Closure**: Bot proactively notifies users after fixes, enabling immediate verification
- **Positive Cycle**: Users feel valued, encouraging continued feedback

## Traditional vs Closed Loop

| Traditional Flow | Closed Loop Flow |
|-----------------|------------------|
| Users don't know if feedback was seen | Bot pull = Developer received it |
| Users don't know when it's fixed | Bot notification = Users know immediately |
| Feedback falls into a black hole | Forms positive feedback, encourages more |
| Developers react passively | Developers proactively pull and notify |

## Architecture

```
Telegram Group Messages → Bot API Read → AI Analyze → Locate Code → Fix → Deploy → Bot Notify
```

### Real Case Study

#### Background

Users reported issues in the Telegram group:
- "Is the admin panel not working?"
- "Just accidentally deleted the configuration"
- "Can't add it back now"
- "1.0 works, but setting 1.2x doesn't work"

#### Execution Process

**Step 1: Read Group Messages**

```bash
# Call Telegram Bot API to get latest messages
curl -s "https://api.telegram.org/bot${BOT_TOKEN}/getUpdates?offset=-20"
```

Extract key information:
- Issue: Setting 1.2x multiplier fails
- Error: Contract error, unable to estimate Gas
- Attachment: Wallet error screenshot

**Step 2: AI Analysis**

Through screenshot analysis (Gemini multimodal):
- MetaMask shows "execution reverted" error
- Unable to estimate Gas indicates contract call parameter issues

Combined with code analysis:
```solidity
// Contract definition
uint256 public constant MULTIPLIER_PRECISION = 1000;

// Frontend definition - Wrong!
const CONFIG = {
  MULTIPLIER_PRECISION: 10000,  // Should be 1000
}
```

**Issue Located**:
- User inputs 1.2x
- Frontend calculates: `1.2 * 10000 = 12000`
- Contract validates: `multiplier > 10000` triggers `InvalidMultiplier()` error
- Should be: `1.2 * 1000 = 1200`

**Step 3: Discover Second Issue**

Reading more messages reveals another problem:
- "Can't deposit tokens"

Code analysis shows missing ERC20 approval flow:
- depositReward requires approve before transfer
- Page lacks approval button

**Step 4: Fix Code**

Fix 1 - Precision constant:
```typescript
const CONFIG = {
  MULTIPLIER_PRECISION: 1000,  // Match contract
}
```

Fix 2 - Add approval flow:
```typescript
const { needsApproval, approve, isApproving } = useTokenApproval({
  tokenAddress: rewardToken,
  spenderAddress: stakingAddress,
  requiredAmount: depositAmountWei,
})

// Add approval button
{needsApproval && (
  <button onClick={approve}>Approve</button>
)}
```

**Step 5: Deploy Update**

```bash
# Build and deploy
pnpm build
firebase deploy --only hosting:your-app
```

**Step 6: Bot Notification**

```bash
# Send fix notification to group
curl -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
  -d "chat_id=${CHAT_ID}" \
  -d "text=Admin panel updated:
1. Fixed multiplier setting issue
2. Added token deposit approval flow
Please refresh and retry"
```

## Key Configuration

### Telegram Bot Setup

```yaml
bot_token: "your_bot_token"
chat_id: "your_chat_id"  # Group IDs are usually negative
```

### API Examples

**Read Messages**:
```bash
GET https://api.telegram.org/bot{token}/getUpdates?offset=-20
```

**Send Message**:
```bash
POST https://api.telegram.org/bot{token}/sendMessage
Content-Type: application/x-www-form-urlencoded

chat_id={chat_id}&text={message}
```

**Download File** (e.g., screenshot):
```bash
# 1. Get file path
GET https://api.telegram.org/bot{token}/getFile?file_id={file_id}

# 2. Download file
GET https://api.telegram.org/file/bot{token}/{file_path}
```

## The Power of Closed Loop

### Why "Notification" Matters?

In traditional development, after fixing a bug, developers think "done", but users don't know:
- Users unsure if the issue was seen
- Users don't know when it's fixed
- Users need to check repeatedly

**Bot notification creates closure**:
- User receives notification → Knows issue is valued
- User verifies immediately → Confirms fix works
- User more willing to provide feedback → Forms positive cycle

### Actual Results

In this case:
1. User reports issue
2. Fixed and deployed within 30 minutes
3. Bot sends notification to group
4. User verifies immediately, confirms resolution

**This is the value of positive feedback mechanism**.

## Best Practices

### 1. Message Parsing Tips

- Focus on messages with screenshots (usually contain key error info)
- Extract user-described operation steps
- Note comparative descriptions like "works/doesn't work"

### 2. Issue Location Strategy

- Trace code path backwards from error message
- Compare contract and frontend constant definitions
- Check for missing required steps (like approval)

### 3. Notification Template

```
[Project] Update Notification

Fixed:
1. [Issue description 1]
2. [Issue description 2]

Please refresh and retry. Continue to report any issues.
```

## Automation Script

Encapsulate the entire flow in a script:

```bash
#!/bin/bash
# scripts/tg-feedback-handler.sh

BOT_TOKEN="your_bot_token"
CHAT_ID="your_chat_id"

# Fetch latest messages
fetch_messages() {
  curl -s "https://api.telegram.org/bot${BOT_TOKEN}/getUpdates?offset=-20"
}

# Send notification
send_notification() {
  local message=$1
  curl -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
    -d "chat_id=${CHAT_ID}" \
    -d "text=${message}"
}

# Main flow
main() {
  echo "=== Fetching group messages ==="
  fetch_messages | jq '.result[-5:] | .[].message.text'

  echo "=== Analyzing and fixing issues ==="
  # ... fix code ...

  echo "=== Deploying updates ==="
  pnpm build
  firebase deploy --only hosting:your-app

  echo "=== Sending notification ==="
  send_notification "Updated, please refresh and retry"
}

main
```

## Summary

### Core Philosophy

**Bot is not just a tool, it's a bridge for building trust**.

In traditional mode, user feedback feels like "throwing into a black hole" - no idea if anyone sees it or when it gets fixed. The Bot closed-loop mechanism changes this:

```
User Feedback ──→ Bot Pull (I received it)
       ↓
Bug Fixed ──→ Bot Notify (It's fixed now)
       ↓
User Verifies ──→ Positive Feedback (Will report again)
```

### Three Key Actions

1. **Bot Pull** = Telling users "I'm listening"
2. **Quick Fix** = Proving feedback has value
3. **Bot Notify** = Closing the loop, building trust

### Applicable Scenarios

- Small teams with rapid iteration
- User feedback-driven products
- Early-stage projects needing to build user trust
- Any scenario requiring "making users feel valued"

## References

- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Firebase Hosting](https://firebase.google.com/docs/hosting)
