# One-Click Deploy: End-to-End Automation for Web3 Projects

> How to transform your Web3 deployment process from chaos to a coffee-break task

**Document ID**: 020
**Date**: 2025-01-01
**Tags**: `DevOps` `Automation` `Hardhat` `Firebase` `Web3`

---

## Overview

If you're a DeFi developer, you've probably experienced this scenario:

It's 2 AM, and you've finally fixed an urgent bug. Now you need to:
1. Deploy contracts to testnet for verification
2. Update frontend contract address configuration
3. Rebuild the frontend project
4. Deploy to hosting service
5. Notify the team in chat
6. Write a change log document

By the time you're done, the sun is rising.

This article discusses how to automate this process, making deployment a one-click task.

---

## The Problem We're Solving

A typical Web3 project release workflow includes these steps:

```
Code Commit → Contract Deploy → Config Update → Frontend Build → Environment Deploy → Doc Generation → Team Notification
```

Each step can fail:
- Gas estimation fails during contract deployment
- Frontend config forgets to update contract addresses
- Build misses environment variables
- Team notification forgotten after release
- Documentation doesn't match actual version

Our goal: **Chain these steps into a reliable automated workflow, reducing human error and improving deployment efficiency.**

---

## Technical Architecture

A typical DeFi project uses this tech stack:

| Layer | Technology |
|-------|------------|
| Smart Contracts | Solidity + Hardhat + hardhat-deploy |
| Frontend | React + Vite + Wagmi + Viem |
| Hosting | Firebase Hosting / Vercel |
| Documentation | Notion API |
| Team Notifications | Telegram Bot |

Overall architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Automated Release Pipeline                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │ Hardhat  │ → │ Post     │ → │ Vite     │ → │ Hosting  │  │
│  │ Deploy   │    │ Deploy   │    │ Build    │    │ Deploy   │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│       │                                               │         │
│       ↓                                               ↓         │
│  ┌──────────┐                                   ┌──────────┐   │
│  │ Contract │                                   │ Notion   │   │
│  │ Verify   │                                   │ + TG Bot │   │
│  └──────────┘                                   └──────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Core Implementation

### 1. Contract Deployment Automation

We use the `hardhat-deploy` plugin for contract management. The key is **idempotency** — running multiple times produces the same result.

```typescript
// deploy/002_deploy_token_staking.ts
const func: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments, getNamedAccounts } = hre;
  const { deploy } = deployments;
  const { deployer } = await getNamedAccounts();

  // Deploy with Transparent Proxy pattern
  await deploy('TokenStaking', {
    from: deployer,
    proxy: {
      proxyContract: 'TransparentUpgradeableProxy',
      execute: {
        init: {
          methodName: 'initialize',
          args: [stakingToken, rewardToken, dailyReward],
        },
      },
    },
    log: true,
    waitConfirmations: 5, // Recommended for Polygon
  });
};
```

**Key points:**
- Use `TransparentUpgradeableProxy` for upgradeable contracts
- Set sufficient `waitConfirmations` to ensure transaction confirmation
- Deploy scripts are idempotent — reruns won't redeploy

### 2. Automatic Config Sync

After contract deployment, automatically update frontend contract address config via `post-deploy` script:

```typescript
// deploy/999_post_deploy.ts
const func: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments } = hre;
  const chainId = await hre.getChainId();

  // Read deployed contract addresses
  const staking = await deployments.get('TokenStaking');
  const stakingToken = await deployments.get('StakingToken');
  const rewardToken = await deployments.get('RewardToken');

  // Update frontend config file
  const configPath = '../frontend/src/config/contracts.ts';
  updateContractConfig(configPath, chainId, {
    TokenStaking: staking.address,
    StakingToken: stakingToken.address,
    RewardToken: rewardToken.address,
  });
};

func.tags = ['post-deploy'];
func.runAtTheEnd = true;
```

This way, frontend config updates automatically after each deployment — no manual changes needed.

### 3. Frontend Build & Deploy

Use shell scripts to encapsulate deployment logic:

```bash
#!/bin/bash
# scripts/deploy.sh

TARGET=$1  # frontend | admin
ENV=$2     # preview | prod

# Build
pnpm build:$TARGET

# Deploy
if [ "$ENV" = "preview" ]; then
  firebase hosting:channel:deploy preview --only $TARGET
else
  firebase deploy --only hosting:$TARGET
fi
```

Usage:
```bash
# Deploy frontend to preview environment
bash ./scripts/deploy.sh frontend preview

# Deploy admin panel to production
bash ./scripts/deploy.sh admin prod
```

### 4. Documentation & Notification Automation

After deployment, automatically create Notion docs and send Telegram notifications:

```bash
# Create Notion page
curl -X POST 'https://api.notion.com/v1/pages' \
  -H "Authorization: Bearer $NOTION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -d '{
    "parent": { "page_id": "'$PARENT_PAGE_ID'" },
    "properties": {
      "title": { "title": [{ "text": { "content": "'$TITLE'" } }] }
    },
    "children": [...]
  }'

# Send Telegram notification
curl -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  -d '{
    "chat_id": "'$CHAT_ID'",
    "parse_mode": "Markdown",
    "text": "🚀 *'$VERSION' is live*\n\n📋 [View docs]('$NOTION_URL')"
  }'
```

---

## Complete Workflow Example

Full release workflow:

```bash
# Step 1: Commit code
git add -A && git commit -m "feat: TokenStaking v1.0.0"

# Step 2: Deploy contracts
cd contracts
npx hardhat deploy --tags TokenStaking --network polygon

# Step 3: Update config
npx hardhat deploy --tags post-deploy --network polygon

# Step 4: Build frontend
cd ../frontend
pnpm build:frontend && pnpm build:admin

# Step 5: Deploy to preview for verification
bash ./scripts/deploy.sh frontend preview
bash ./scripts/deploy.sh admin preview

# Step 6: Deploy to production
bash ./scripts/deploy.sh frontend prod
bash ./scripts/deploy.sh admin prod

# Step 7: Publish docs and notifications
# (Completed via API calls)
```

The entire process takes approximately 15 minutes from start to finish.

---

## Deliverables

After each release, you get:

| Deliverable | Description |
|-------------|-------------|
| Contract Addresses | Deployed contract addresses on target chain |
| Frontend App | Production URL |
| Admin Panel | Admin panel URL |
| Preview Environment | Temporary environment for verification |
| Notion Document | Document with complete release info |
| Telegram Notification | Message sent to team chat |
| Local Changelog | Markdown file saved in codebase |

---

## Lessons Learned

### What Worked Well

1. **Idempotent Design**: All scripts can run repeatedly without side effects
2. **Automatic Config Sync**: Reduced manual config update errors
3. **Preview Environment Verification**: Catches issues before production
4. **Auto-generated Documentation**: Ensures docs match code version

### Areas for Improvement

1. **CI/CD Integration**: Currently manual trigger, could integrate GitHub Actions
2. **Rollback Mechanism**: Need more robust rollback process
3. **Monitoring & Alerts**: Post-deployment monitoring needs improvement

---

## Conclusion

Automation isn't built overnight — it's a continuous optimization process. Recommendations:

1. **First, complete manual workflow**: Ensure each step is correct
2. **Automate incrementally**: Start with the most error-prone steps
3. **Keep it simple**: Don't over-engineer, just enough is fine
4. **Iterate continuously**: Optimize based on real-world issues

Hope this article helps. Feel free to reach out with questions.

---

## Resources

- [Hardhat Deploy Documentation](https://github.com/wighawag/hardhat-deploy)
- [Firebase Hosting Documentation](https://firebase.google.com/docs/hosting)
- [Notion API Documentation](https://developers.notion.com/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
