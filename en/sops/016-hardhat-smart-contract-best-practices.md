# Hardhat Smart Contract Development Best Practices

> Build production-grade Hardhat projects from scratch, including multi-chain deployment, automated testing, ABI export, and skill templating workflow.

**Document ID**: 016
**Date**: 2025-12-30
**Tags**: `Hardhat` `Solidity` `Smart Contract` `Multi-chain` `hardhat-deploy` `Skill`

---

## Overview

This document captures best practices for using the Hardhat framework in Web3 development:

1. **Project Structure Standards** - Clear directory organization
2. **Dependency Version Locking** - Avoiding compatibility issues
3. **Multi-chain Deployment Configuration** - One codebase, multiple chains
4. **Automated Toolchain** - Linter, Formatter, Gas Reporter
5. **Skill Templating** - Encapsulating best practices as reusable skills

Based on real-world production project experience.

---

## Part 1: Project Structure

### 1.1 Recommended Directory Structure

```
contracts/
├── contracts/
│   ├── core/           # Core business contracts
│   ├── interfaces/     # Interface definitions (IXxx.sol)
│   ├── libraries/      # Reusable libraries
│   └── mocks/          # Test mock contracts
├── test/
│   ├── unit/           # Unit tests
│   └── integration/    # Integration tests
├── scripts/
│   ├── deploy/         # hardhat-deploy scripts
│   │   ├── config/     # Multi-chain configuration
│   │   ├── 001_deploy_xxx.ts
│   │   ├── 002_deploy_yyy.ts
│   │   └── 999_update_frontend_config.ts
│   └── interact/       # Interaction scripts
├── deployments/        # Deployment records (auto-generated)
├── artifacts/          # Compilation artifacts (auto-generated)
├── typechain-types/    # TypeScript types (auto-generated)
├── hardhat.config.ts
├── package.json
├── tsconfig.json
├── .env.example
├── .solhint.json
└── .prettierrc
```

### 1.2 Directory Responsibilities

| Directory | Purpose | Example Files |
|-----------|---------|---------------|
| `contracts/core/` | Core business logic | `TokenStaking.sol` |
| `contracts/interfaces/` | Interface definitions | `ITokenStaking.sol` |
| `contracts/libraries/` | Reusable utility libraries | `SafeMath.sol` |
| `contracts/mocks/` | Test mocks | `MockERC20.sol` |
| `scripts/deploy/` | Deploy scripts (numbered prefix) | `002_deploy_staking.ts` |
| `scripts/deploy/config/` | Multi-chain config | `deploy-config.ts` |

---

## Part 2: Critical Dependency Versions

### 2.1 Core Dependencies

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

### 2.2 Version Compatibility Warnings

| Dependency | Requirement | Reason |
|------------|-------------|--------|
| `hardhat` | Use 2.x | Hardhat 3's toolbox ecosystem not fully compatible yet |
| `hardhat-deploy` | **Must be ^1.0.4** | v0.14.x incompatible with ethers v6 |
| `ethers` | ^6.x | Modern project standard |

**Common Error**: Using `hardhat-deploy@0.14.x` causes:
```
Error: invalid object key - customData
```

**Solution**: Upgrade to `hardhat-deploy@^1.0.4`

---

## Part 3: Hardhat Configuration

### 3.1 Complete Configuration Template

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
  // Multi-version compiler
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

  // Multi-chain network configuration
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

  // Etherscan verification config
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

  // Contract size check
  contractSizer: {
    alphaSort: true,
    runOnCompile: true,
    strict: true,
  },

  // ABI export (auto-export to frontend on compile)
  abiExporter: {
    path: "../frontend/src/abi",
    runOnCompile: true,
    clear: true,
    flat: true,
    format: "json",
    only: ["^contracts/core/"],
  },

  // hardhat-deploy configuration
  namedAccounts: {
    deployer: { default: 0 },
  },

  // Path configuration
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

## Part 4: Multi-chain Deployment Configuration

### 4.1 Deployment Config File

Create `scripts/deploy/config/deploy-config.ts`:

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
      stakeToken: "0x...",  // Real token address
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

### 4.2 Deployment Script Template

Create `scripts/deploy/002_deploy_staking.ts`:

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

  // 1. Get or deploy mock tokens
  let stakeTokenAddress: string;
  let rewardTokenAddress: string;

  if (isLocalNetwork(network.name) || !isMainnet(network.name)) {
    // Local/Testnet: Deploy mock tokens
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
    // Mainnet: Use configured real token addresses
    stakeTokenAddress = config.tokens.stakeToken!;
    rewardTokenAddress = config.tokens.rewardToken!;
  }

  // 2. Deploy staking contract
  const staking = await deploy("TokenStaking", {
    from: deployer,
    args: [stakeTokenAddress, rewardTokenAddress],
    log: true,
    waitConfirmations: getConfirmations(network.name),
  });

  log(`\n✅ TokenStaking deployed at: ${staking.address}\n`);

  // 3. Verify contract (non-local networks only)
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

## Part 5: Frontend Config Auto-Update

### 5.1 Auto-Update Script

Create `scripts/deploy/999_update_frontend_config.ts` (number 999 ensures last execution):

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
      // Address replacement logic...
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

## Part 6: NPM Scripts

### 6.1 Recommended Script Configuration

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

### 6.2 Usage Examples

```bash
# Local development
pnpm compile              # Compile contracts
pnpm test                 # Run tests
pnpm test:gas            # Test with gas report
pnpm deploy:local        # Local deployment

# Testnet deployment
pnpm deploy:bsc-testnet  # Deploy to BSC Testnet

# Mainnet deployment
pnpm deploy:bsc          # Deploy to BSC Mainnet

# Deploy specific contract by tag
pnpm exec hardhat deploy --tags TokenStaking --network bscTestnet
```

---

## Part 7: Skill Templating

### 7.1 Creating the hardhat-init Skill

Encapsulate best practices as a Claude Code skill for quick project initialization.

Skill location: `~/.claude/skills/hardhat-init/` or `~/Projects/my_skils/library/hardhat-init/`

Skill structure:
```
hardhat-init/
├── SKILL.md              # Skill documentation
└── assets/
    ├── package.json      # Template package.json
    ├── hardhat.config.ts # Template config file
    ├── tsconfig.json
    ├── .env.example
    ├── .solhint.json
    ├── .solhintignore
    ├── .prettierrc
    └── .prettierignore
```

### 7.2 Skill Trigger Words

Define in SKILL.md frontmatter:

```yaml
---
name: hardhat-init
description: |
  Hardhat smart contract project initialization and best practices configuration.
  Use this skill when user needs to:
  (1) Initialize a new Hardhat project
  (2) Optimize existing Hardhat project configuration
  (3) Add multi-chain deployment support
  (4) Configure contract development toolchain
  (5) Set up hardhat-deploy script structure
---
```

### 7.3 Quick Initialization Commands

```bash
# Create directory structure
mkdir -p contracts/{core,interfaces,libraries,mocks}
mkdir -p test/{unit,integration}
mkdir -p scripts/{deploy,interact}
mkdir -p scripts/deploy/config
touch contracts/{core,interfaces,libraries,mocks}/.gitkeep
touch test/{unit,integration}/.gitkeep
touch scripts/{deploy,interact}/.gitkeep
```

---

## FAQ

### Q1: hardhat-deploy error "invalid object key - customData"

**Cause**: hardhat-deploy 0.14.x incompatible with ethers v6

**Solution**:
```bash
pnpm add -D hardhat-deploy@^1.0.4
```

### Q2: Polygon Amoy testnet verification fails

**Cause**: Polygon Amoy not in hardhat-verify default support list

**Solution**: Add config to `etherscan.customChains`:
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

### Q3: Contract size exceeds 24KB limit

**Solutions**:
1. Enable optimizer: `optimizer: { enabled: true, runs: 200 }`
2. Enable IR compilation: `viaIR: true`
3. Split contract into smaller contracts
4. Use libraries to reuse code

### Q4: How to deploy only specific contracts?

Use the tags system:
```bash
# Only deploy scripts with TokenStaking tag
pnpm exec hardhat deploy --tags TokenStaking --network bscTestnet
```

---

## References

- [Hardhat Official Documentation](https://hardhat.org/docs)
- [hardhat-deploy Documentation](https://github.com/wighawag/hardhat-deploy)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts)
- [hardhat-init Skill Template](https://github.com/your-org/skills-library)
