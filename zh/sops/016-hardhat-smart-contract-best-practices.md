# Hardhat 智能合约开发最佳实践

> 从零开始搭建生产级 Hardhat 项目，包含多链部署、自动化测试、ABI 导出及技能模板化的完整工作流。

**文档编号**: 016
**日期**: 2025-12-30
**标签**: `Hardhat` `Solidity` `Smart Contract` `Multi-chain` `hardhat-deploy` `Skill`

---

## 概述

本文档记录了在 Web3 开发中使用 Hardhat 框架的最佳实践，涵盖：

1. **项目结构规范** - 清晰的目录组织
2. **依赖版本锁定** - 避免兼容性问题
3. **多链部署配置** - 一套代码多链部署
4. **自动化工具链** - Linter、Formatter、Gas Reporter
5. **技能模板化** - 将最佳实践封装为可复用技能

本文档基于实际生产项目的开发经验总结。

---

## 第一部分：项目结构

### 1.1 推荐目录结构

```
contracts/
├── contracts/
│   ├── core/           # 核心业务合约
│   ├── interfaces/     # 接口定义 (IXxx.sol)
│   ├── libraries/      # 可复用库
│   └── mocks/          # 测试用 Mock 合约
├── test/
│   ├── unit/           # 单元测试
│   └── integration/    # 集成测试
├── scripts/
│   ├── deploy/         # hardhat-deploy 部署脚本
│   │   ├── config/     # 多链配置
│   │   ├── 001_deploy_xxx.ts
│   │   ├── 002_deploy_yyy.ts
│   │   └── 999_update_frontend_config.ts
│   └── interact/       # 交互脚本
├── deployments/        # 部署记录（自动生成）
├── artifacts/          # 编译产物（自动生成）
├── typechain-types/    # TypeScript 类型（自动生成）
├── hardhat.config.ts
├── package.json
├── tsconfig.json
├── .env.example
├── .solhint.json
└── .prettierrc
```

### 1.2 目录职责

| 目录 | 职责 | 示例文件 |
|------|------|----------|
| `contracts/core/` | 核心业务逻辑 | `TokenStaking.sol` |
| `contracts/interfaces/` | 接口定义 | `ITokenStaking.sol` |
| `contracts/libraries/` | 可复用工具库 | `SafeMath.sol` |
| `contracts/mocks/` | 测试用 Mock | `MockERC20.sol` |
| `scripts/deploy/` | 部署脚本（编号前缀） | `002_deploy_staking.ts` |
| `scripts/deploy/config/` | 多链配置 | `deploy-config.ts` |

---

## 第二部分：关键依赖版本

### 2.1 核心依赖

```json
{
  "devDependencies": {
    "hardhat": "^2.22.0",
    "@nomicfoundation/hardhat-toolbox": "^5.0.0",
    "@nomicfoundation/hardhat-verify": "^2.1.1",
    "@openzeppelin/hardhat-upgrades": "^3.9.1",
    "hardhat-deploy": "^1.0.4",
    "hardhat-abi-exporter": "^2.11.0",
    "hardhat-contract-sizer": "^2.10.0",
    "hardhat-gas-reporter": "^2.3.0",
    "ethers": "^6.15.0",
    "typescript": "^5.5.0"
  },
  "dependencies": {
    "@openzeppelin/contracts": "^5.4.0",
    "@openzeppelin/contracts-upgradeable": "^5.4.0"
  }
}
```

### 2.2 版本兼容性警告

| 依赖 | 要求 | 原因 |
|------|------|------|
| `hardhat` | 使用 2.x | Hardhat 3 的 toolbox 生态尚未完全兼容 |
| `hardhat-deploy` | **必须 ^1.0.4** | 旧版 0.14.x 与 ethers v6 不兼容 |
| `ethers` | ^6.x | 现代项目标准 |

**常见错误**：使用 `hardhat-deploy@0.14.x` 会报错：
```
Error: invalid object key - customData
```

**解决方案**：升级到 `hardhat-deploy@^1.0.4`

---

## 第三部分：Hardhat 配置

### 3.1 完整配置模板

```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "@nomicfoundation/hardhat-verify";
import "@openzeppelin/hardhat-upgrades";
import "hardhat-deploy";
import "hardhat-gas-reporter";
import "hardhat-contract-sizer";
import "hardhat-abi-exporter";
import * as dotenv from "dotenv";

dotenv.config();

const PRIVATE_KEY = process.env.PRIVATE_KEY || "";

const config: HardhatUserConfig = {
  // 多版本编译器
  solidity: {
    compilers: [
      {
        version: "0.8.28",
        settings: {
          optimizer: { enabled: true, runs: 200 },
          viaIR: true,
          evmVersion: "cancun",
        },
      },
      {
        version: "0.8.20",
        settings: {
          optimizer: { enabled: true, runs: 200 },
          viaIR: true,
        },
      },
    ],
  },

  // 多链网络配置
  networks: {
    hardhat: {
      chainId: 31337,
      accounts: { count: 20 },
      loggingEnabled: true,
      throwOnTransactionFailures: true,
      throwOnCallFailures: true,
    },
    localhost: {
      url: "http://127.0.0.1:8545",
      chainId: 31337,
    },
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL || "",
      accounts: PRIVATE_KEY ? [PRIVATE_KEY] : [],
      chainId: 11155111,
    },
    mainnet: {
      url: process.env.MAINNET_RPC_URL || "",
      accounts: PRIVATE_KEY ? [PRIVATE_KEY] : [],
      chainId: 1,
    },
    bscTestnet: {
      url: process.env.BSC_TESTNET_RPC_URL || "https://data-seed-prebsc-1-s1.binance.org:8545",
      accounts: PRIVATE_KEY ? [PRIVATE_KEY] : [],
      chainId: 97,
      gasPrice: 5000000000,
    },
    bscMainnet: {
      url: process.env.BSC_MAINNET_RPC_URL || "https://bsc-dataseed.binance.org",
      accounts: PRIVATE_KEY ? [PRIVATE_KEY] : [],
      chainId: 56,
      gasPrice: 50000000,
    },
    polygon: {
      url: process.env.POLYGON_RPC_URL || "https://polygon-rpc.com",
      accounts: PRIVATE_KEY ? [PRIVATE_KEY] : [],
      chainId: 137,
      gasPrice: 70000000000,
    },
    polygonAmoy: {
      url: process.env.POLYGON_AMOY_RPC_URL || "https://rpc-amoy.polygon.technology",
      accounts: PRIVATE_KEY ? [PRIVATE_KEY] : [],
      chainId: 80002,
      gasPrice: 30000000000,
    },
    base: {
      url: process.env.BASE_RPC_URL || "https://mainnet.base.org",
      accounts: PRIVATE_KEY ? [PRIVATE_KEY] : [],
      chainId: 8453,
    },
  },

  // Etherscan 验证配置
  etherscan: {
    apiKey: {
      mainnet: process.env.ETHERSCAN_API_KEY || "",
      sepolia: process.env.ETHERSCAN_API_KEY || "",
      bsc: process.env.BSCSCAN_API_KEY || "",
      bscTestnet: process.env.BSCSCAN_API_KEY || "",
      polygon: process.env.POLYGONSCAN_API_KEY || "",
      polygonAmoy: process.env.POLYGONSCAN_API_KEY || "",
      base: process.env.BASESCAN_API_KEY || "",
    },
    customChains: [
      {
        network: "polygonAmoy",
        chainId: 80002,
        urls: {
          apiURL: "https://api-amoy.polygonscan.com/api",
          browserURL: "https://amoy.polygonscan.com",
        },
      },
    ],
  },

  // Gas Reporter
  gasReporter: {
    enabled: process.env.REPORT_GAS === "true",
    currency: "USD",
    coinmarketcap: process.env.COINMARKETCAP_API_KEY || "",
    offline: true,
  },

  // 合约大小检查
  contractSizer: {
    alphaSort: true,
    runOnCompile: true,
    strict: true,
  },

  // ABI 导出（编译时自动导出到前端）
  abiExporter: {
    path: "../frontend/src/abi",
    runOnCompile: true,
    clear: true,
    flat: true,
    format: "json",
    only: ["^contracts/core/"],
  },

  // hardhat-deploy 配置
  namedAccounts: {
    deployer: { default: 0 },
  },

  // 路径配置
  paths: {
    sources: "./contracts",
    tests: "./test",
    cache: "./cache",
    artifacts: "./artifacts",
    deploy: "./scripts/deploy",
    deployments: "./deployments",
  },

  mocha: {
    timeout: 60000,
  },
};

export default config;
```

---

## 第四部分：多链部署配置

### 4.1 部署配置文件

创建 `scripts/deploy/config/deploy-config.ts`：

```typescript
export interface NetworkDeployConfig {
  chainId: number;
  name: string;
  isMainnet: boolean;
  isTestnet: boolean;
  isLocal: boolean;
  confirmations: number;
  tokens: {
    stakeToken?: string;
    rewardToken?: string;
  };
}

const configs: Record<string, NetworkDeployConfig> = {
  hardhat: {
    chainId: 31337,
    name: "Hardhat Local",
    isMainnet: false,
    isTestnet: false,
    isLocal: true,
    confirmations: 0,
    tokens: {},
  },
  localhost: {
    chainId: 31337,
    name: "Localhost",
    isMainnet: false,
    isTestnet: false,
    isLocal: true,
    confirmations: 0,
    tokens: {},
  },
  bscTestnet: {
    chainId: 97,
    name: "BSC Testnet",
    isMainnet: false,
    isTestnet: true,
    isLocal: false,
    confirmations: 3,
    tokens: {},
  },
  bscMainnet: {
    chainId: 56,
    name: "BSC Mainnet",
    isMainnet: true,
    isTestnet: false,
    isLocal: false,
    confirmations: 5,
    tokens: {
      stakeToken: "0x...",  // 真实代币地址
      rewardToken: "0x...",
    },
  },
  polygon: {
    chainId: 137,
    name: "Polygon",
    isMainnet: true,
    isTestnet: false,
    isLocal: false,
    confirmations: 5,
    tokens: {
      stakeToken: "0x...",
      rewardToken: "0x...",
    },
  },
};

export function getDeployConfig(networkName: string): NetworkDeployConfig {
  const config = configs[networkName];
  if (!config) {
    throw new Error(`Unknown network: ${networkName}`);
  }
  return config;
}

export function isMainnet(networkName: string): boolean {
  return getDeployConfig(networkName).isMainnet;
}

export function isLocalNetwork(networkName: string): boolean {
  return getDeployConfig(networkName).isLocal;
}

export function getConfirmations(networkName: string): number {
  return getDeployConfig(networkName).confirmations;
}
```

### 4.2 部署脚本模板

创建 `scripts/deploy/002_deploy_staking.ts`：

```typescript
import { HardhatRuntimeEnvironment } from "hardhat/types";
import { DeployFunction } from "hardhat-deploy/types";
import { getDeployConfig, isLocalNetwork, isMainnet, getConfirmations } from "./config/deploy-config";

const func: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments, getNamedAccounts, network, ethers } = hre;
  const { deploy, get, log } = deployments;
  const { deployer } = await getNamedAccounts();

  const config = getDeployConfig(network.name);

  log("\n========================================");
  log(`🚀 Deploying TokenStaking to ${config.name}`);
  log("========================================\n");

  // 1. 获取或部署 Mock 代币
  let stakeTokenAddress: string;
  let rewardTokenAddress: string;

  if (isLocalNetwork(network.name) || !isMainnet(network.name)) {
    // 本地/测试网：部署 Mock 代币
    const mockStakeToken = await deploy("MockERC20_Stake", {
      from: deployer,
      contract: "MockERC20",
      args: ["Mock Stake Token", "MST", 18],
      log: true,
      waitConfirmations: getConfirmations(network.name),
    });
    stakeTokenAddress = mockStakeToken.address;

    const mockRewardToken = await deploy("MockERC20_Reward", {
      from: deployer,
      contract: "MockERC20",
      args: ["Mock Reward Token", "MRT", 18],
      log: true,
      waitConfirmations: getConfirmations(network.name),
    });
    rewardTokenAddress = mockRewardToken.address;
  } else {
    // 主网：使用配置的真实代币地址
    stakeTokenAddress = config.tokens.stakeToken!;
    rewardTokenAddress = config.tokens.rewardToken!;
  }

  // 2. 部署质押合约
  const staking = await deploy("TokenStaking", {
    from: deployer,
    args: [stakeTokenAddress, rewardTokenAddress],
    log: true,
    waitConfirmations: getConfirmations(network.name),
  });

  log(`\n✅ TokenStaking deployed at: ${staking.address}\n`);

  // 3. 验证合约（仅非本地网络）
  if (!isLocalNetwork(network.name) && staking.newlyDeployed) {
    log("🔍 Verifying contract on explorer...");
    try {
      await hre.run("verify:verify", {
        address: staking.address,
        constructorArguments: [stakeTokenAddress, rewardTokenAddress],
      });
      log("✅ Contract verified!");
    } catch (error: any) {
      if (error.message.includes("Already Verified")) {
        log("Contract already verified");
      } else {
        log(`⚠️ Verification failed: ${error.message}`);
      }
    }
  }
};

export default func;

func.id = "deploy_token_staking";
func.tags = ["TokenStaking", "Staking", "core"];
func.dependencies = [];
```

---

## 第五部分：前端配置自动更新

### 5.1 自动更新脚本

创建 `scripts/deploy/999_update_frontend_config.ts`（编号 999 确保最后执行）：

```typescript
import { HardhatRuntimeEnvironment } from "hardhat/types";
import { DeployFunction } from "hardhat-deploy/types";
import * as fs from "fs";
import * as path from "path";

const config = {
  autoUpdate: process.env.AUTO_UPDATE_FRONTEND !== "false",
  frontendConfigPath: "../frontend/src/config/contracts.ts",
  contracts: [
    { deploymentName: "TokenStaking", configName: "TokenStaking", hasDecimals: false },
    { deploymentName: "MockERC20_Stake", configName: "StakeToken", hasDecimals: true },
    { deploymentName: "MockERC20_Reward", configName: "RewardToken", hasDecimals: true },
  ],
};

const networkConfig: Record<string, { chainId: number }> = {
  localhost: { chainId: 31337 },
  hardhat: { chainId: 31337 },
  bscTestnet: { chainId: 97 },
  bscMainnet: { chainId: 56 },
  polygon: { chainId: 137 },
  polygonAmoy: { chainId: 80002 },
  base: { chainId: 8453 },
};

const func: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments, network } = hre;
  const { log } = deployments;

  if (!config.autoUpdate) {
    log("⏭️ Skipping frontend config update");
    return;
  }

  const netConfig = networkConfig[network.name];
  if (!netConfig) {
    log(`⚠️ Unknown network: ${network.name}`);
    return;
  }

  const configPath = path.join(__dirname, "../..", config.frontendConfigPath);

  if (!fs.existsSync(configPath)) {
    log(`⚠️ Frontend config not found: ${configPath}`);
    return;
  }

  let content = fs.readFileSync(configPath, "utf-8");
  let updatedCount = 0;

  for (const contract of config.contracts) {
    try {
      const deployment = await deployments.get(contract.deploymentName);
      // 替换地址逻辑...
      updatedCount++;
      log(`✅ Updated ${contract.configName}: ${deployment.address}`);
    } catch {
      log(`⚠️ ${contract.deploymentName} not deployed, skipping`);
    }
  }

  if (updatedCount > 0) {
    fs.writeFileSync(configPath, content, "utf-8");
    log(`\n✅ Updated ${updatedCount} contract addresses in frontend config`);
  }
};

export default func;

func.id = "update_frontend_config";
func.tags = ["frontend", "config", "post-deploy"];
func.dependencies = [];
```

---

## 第六部分：NPM Scripts

### 6.1 推荐脚本配置

```json
{
  "scripts": {
    "compile": "hardhat compile",
    "build": "hardhat compile",
    "test": "hardhat test",
    "test:coverage": "hardhat coverage",
    "test:gas": "REPORT_GAS=true hardhat test",
    "clean": "hardhat clean && rm -rf cache artifacts typechain-types coverage deployments",
    "node": "hardhat node",
    "deploy:local": "hardhat deploy --network hardhat",
    "deploy:testnet": "hardhat deploy --network sepolia",
    "deploy:bsc-testnet": "hardhat deploy --network bscTestnet",
    "deploy:bsc": "hardhat deploy --network bscMainnet",
    "deploy:polygon": "hardhat deploy --network polygon",
    "verify": "hardhat verify",
    "size": "hardhat size-contracts",
    "lint": "solhint 'contracts/**/*.sol'",
    "lint:fix": "solhint 'contracts/**/*.sol' --fix",
    "format": "prettier --write 'contracts/**/*.sol' 'test/**/*.ts' 'scripts/**/*.ts'",
    "format:check": "prettier --check 'contracts/**/*.sol' 'test/**/*.ts' 'scripts/**/*.ts'"
  }
}
```

### 6.2 使用示例

```bash
# 本地开发
pnpm compile              # 编译合约
pnpm test                 # 运行测试
pnpm test:gas            # 带 Gas 报告的测试
pnpm deploy:local        # 本地部署

# 测试网部署
pnpm deploy:bsc-testnet  # 部署到 BSC 测试网

# 主网部署
pnpm deploy:bsc          # 部署到 BSC 主网

# 按标签部署特定合约
pnpm exec hardhat deploy --tags TokenStaking --network bscTestnet
```

---

## 第七部分：技能模板化

### 7.1 创建 hardhat-init 技能

将最佳实践封装为 Claude Code 技能，便于新项目快速初始化。

技能位置：`~/.claude/skills/hardhat-init/` 或 `~/Projects/my_skils/library/hardhat-init/`

技能结构：
```
hardhat-init/
├── SKILL.md              # 技能文档
└── assets/
    ├── package.json      # 模板 package.json
    ├── hardhat.config.ts # 模板配置文件
    ├── tsconfig.json
    ├── .env.example
    ├── .solhint.json
    ├── .solhintignore
    ├── .prettierrc
    └── .prettierignore
```

### 7.2 技能触发词

在 SKILL.md 的 frontmatter 中定义：

```yaml
---
name: hardhat-init
description: |
  Hardhat 智能合约项目初始化与最佳实践配置。当用户需要：
  (1) 初始化新的 Hardhat 项目
  (2) 优化现有 Hardhat 项目配置
  (3) 添加多链部署支持
  (4) 配置合约开发工具链
  (5) 设置 hardhat-deploy 部署脚本结构
  时使用此技能。
---
```

### 7.3 快速初始化命令

```bash
# 创建目录结构
mkdir -p contracts/{core,interfaces,libraries,mocks}
mkdir -p test/{unit,integration}
mkdir -p scripts/{deploy,interact}
mkdir -p scripts/deploy/config
touch contracts/{core,interfaces,libraries,mocks}/.gitkeep
touch test/{unit,integration}/.gitkeep
touch scripts/{deploy,interact}/.gitkeep
```

---

## 常见问题

### Q1: hardhat-deploy 报错 "invalid object key - customData"

**原因**：hardhat-deploy 0.14.x 与 ethers v6 不兼容

**解决**：
```bash
pnpm add -D hardhat-deploy@^1.0.4
```

### Q2: Polygon Amoy 测试网验证失败

**原因**：Polygon Amoy 不在 hardhat-verify 默认支持列表

**解决**：在 `etherscan.customChains` 中添加配置：
```typescript
customChains: [
  {
    network: "polygonAmoy",
    chainId: 80002,
    urls: {
      apiURL: "https://api-amoy.polygonscan.com/api",
      browserURL: "https://amoy.polygonscan.com",
    },
  },
],
```

### Q3: 合约大小超过 24KB 限制

**解决方案**：
1. 启用优化器：`optimizer: { enabled: true, runs: 200 }`
2. 启用 IR 编译：`viaIR: true`
3. 拆分合约为多个小合约
4. 使用库来复用代码

### Q4: 如何只部署特定合约？

使用 tags 系统：
```bash
# 只部署带有 TokenStaking 标签的脚本
pnpm exec hardhat deploy --tags TokenStaking --network bscTestnet
```

---

## 参考资源

- [Hardhat 官方文档](https://hardhat.org/docs)
- [hardhat-deploy 文档](https://github.com/wighawag/hardhat-deploy)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts)
- [hardhat-init 技能模板](https://github.com/your-org/skills-library)
