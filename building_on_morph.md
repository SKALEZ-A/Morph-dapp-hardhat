# Building a Decentralized Application on Morph Using Hardhat and Next.js

## Introduction

Morph is a Layer 2 (L2) blockchain designed to enhance Ethereum’s scalability, offering a secure, decentralized, cost-efficient, and high-performing EVM-equivalent platform. As a consumer-centric blockchain, Morph supports a growing ecosystem of decentralized applications (dApps), particularly in DeFi.

This tutorial walks you through building a simple dApp on Morph’s testnet using Hardhat for smart contract development and Next.js for the frontend. By the end, you’ll deploy a smart contract to Morph’s testnet and create a web interface to interact with it.

**Prerequisites:**

- Node.js (v16 or later) and npm
- MetaMask browser extension
- Basic knowledge of Solidity, JavaScript, and React
- A code editor like VS Code
- Access to Morph’s testnet RPC and faucet for test tokens

---

## Step 1: Setting Up the Development Environment

### 1.1 Install Hardhat

```bash
mkdir morph-dapp && cd morph-dapp
npm init -y
npm install --save-dev hardhat
npx hardhat init
```

Select "Create a JavaScript project" and accept default options.

### 1.2 Install Next.js and Ethers

```bash
npx create-next-app@latest morph-frontend
cd morph-frontend
npm install ethers
```

Now you have:

- Hardhat (root folder) for smart contracts
- Next.js (morph-frontend) for frontend

### 1.3 Configure MetaMask for Morph Testnet

Add the Morph Testnet manually:

- **Network Name:** Morph Testnet
- **RPC URL:** [https://rpc-testnet.morphl2.io](https://rpc-testnet.morphl2.io)
- **Chain ID:** 2810
- **Currency Symbol:** ETH
- **Block Explorer:** [https://explorer-testnet.morphl2.io](https://explorer-testnet.morphl2.io)

Request test ETH: [Morph Faucet](https://stakely.io/en/faucet/ethereum-holesky-testnet-eth)

---

## Step 2: Writing a Smart Contract

### 2.1 Create the Contract

In the `contracts` folder, create `Counter.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Counter {
    uint256 private count;

    constructor() {
        count = 0;
    }

    function increment() public {
        count += 1;
    }

    function getCount() public view returns (uint256) {
        return count;
    }
}
```

### 2.2 Configure Hardhat for Morph

Install Hardhat Toolbox:

```bash
npm install --save-dev @nomicfoundation/hardhat-toolbox
```

Edit `hardhat.config.js`:

```js
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.20",
  networks: {
    morphTestnet: {
      url: "https://rpc-testnet.morphl2.io",
      accounts: ["YOUR_PRIVATE_KEY"], // Use environment variable in real use
      chainId: 2810,
    },
  },
};
```

### 2.3 Deploy the Contract

Create `scripts/deploy.js`:

```js
const hre = require("hardhat");

async function main() {
  const Counter = await hre.ethers.getContractFactory("Counter");
  const counter = await Counter.deploy();
  await counter.deployed();
  console.log("Counter deployed to:", counter.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Run the deployment:

```bash
npx hardhat run scripts/deploy.js --network morphTestnet
```

---

## Step 3: Building the Frontend with Next.js

### 3.1 Create the UI in `pages/index.js`

```js
import { useState, useEffect } from "react";
import { ethers } from "ethers";
import styles from "../styles/Home.module.css";

export default function Home() {
  const [count, setCount] = useState(0);
  const [contract, setContract] = useState(null);

  const contractAddress = "YOUR_CONTRACT_ADDRESS"; // Replace with deployed address
  const abi = [
    "function increment() public",
    "function getCount() public view returns (uint256)",
  ];

  useEffect(() => {
    const connect = async () => {
      if (window.ethereum) {
        const provider = new ethers.providers.Web3Provider(window.ethereum);
        await provider.send("eth_requestAccounts", []);
        const signer = provider.getSigner();
        const counter = new ethers.Contract(contractAddress, abi, signer);
        setContract(counter);
        const current = await counter.getCount();
        setCount(current.toNumber());
      }
    };
    connect();
  }, []);

  const handleIncrement = async () => {
    if (contract) {
      const tx = await contract.increment();
      await tx.wait();
      const updated = await contract.getCount();
      setCount(updated.toNumber());
    }
  };

  return (
    <div className={styles.container}>
      <h1>Morph Counter dApp</h1>
      <p>Current Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}
```

### 3.2 Add Basic Styling in `styles/Home.module.css`

```css
.container {
  padding: 0 2rem;
  text-align: center;
  margin-top: 2rem;
}
```

### 3.3 Run the Frontend

```bash
cd morph-frontend
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000) to interact with your contract.

---

## Step 4: Testing the Smart Contract

Create `test/Counter.js`:

```js
const { expect } = require("chai");

describe("Counter", function () {
  let Counter, counter;

  beforeEach(async function () {
    Counter = await ethers.getContractFactory("Counter");
    counter = await Counter.deploy();
    await counter.deployed();
  });

  it("should increment count", async function () {
    await counter.increment();
    expect(await counter.getCount()).to.equal(1);
  });
});
```

Run tests:

```bash
npx hardhat test
```

---

## Step 5: Contributing to Morph’s Documentation

1. **Fork the Repository:** [Morph Examples GitHub](https://github.com/morph-l2/morph-examples)
2. **Add Your Tutorial:** Create a new Markdown file (`building-with-hardhat-nextjs.md`) and paste this content.
3. **Submit a Pull Request:** Follow the repo’s contribution guidelines.
4. **Engage the Community:** Share your work on [MorphLayer Discord](https://discord.com/invite/MorphLayer)

---

## Best Practices

- **Security:** Use `.env` files for private keys (e.g., `dotenv` package).
- **Testing:** Write more tests to cover edge cases.
- **User Experience:** Add loading states and error handling in UI.

---

## Resources

- [Morph Examples GitHub](https://github.com/morph-l2/morph-examples)
- [Getting Started with Hardhat (YouTube)](https://www.youtube.com/watch?v=8yAx5EU1wcE)
- [Smart Contract Testing Workshop (YouTube)](https://www.youtube.com/watch?v=vljq0OnuVN8\&t=129s)
- [Morph Testnet Faucet](https://stakely.io/en/faucet/ethereum-holesky-testnet-eth)
- [Morph Developer Blog](https://morph.ghost.io/morph-developer-resources-2/)
- [Morph Community Discord](https://discord.com/invite/MorphLayer)

---

## Conclusion

You’ve now built and deployed a simple dApp on Morph using Hardhat and Next.js, created a web UI, and tested your contract. By contributing this tutorial, you help expand Morph's developer resources and support the ecosystem.

**Submission Notes:**

- **File Format:** Markdown (`building-with-hardhat-nextjs.md`)
- **Contribute via:** [Morph Documentation GitHub](https://github.com/morph-l2/morph-examples)
- **Ask for help:** [Morph Discord](https://discord.com/invite/MorphLayer)

