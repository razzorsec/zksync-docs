---
title: Hardhat | Deploy Contract
---

Run the following command in your terminal to initialize the project.

<!-- TODO: @dutterbutter determine best approach to leverage zksync cli for project
bootstrapping for this guide series. -->

```sh
git clone https://github.com/dutterbutter/zksync-quickstart-guide.git
cd zksync-quickstart-guide
```

Install the dependencies:

::code-group

```bash [npm]
npm install
```

```bash [yarn]
yarn install
```

```bash [pnpm]
pnpm install
```

```bash [bun]
bun install
```

::

## Set up your wallet

:display-partial{path="quick-start/_partials/_setup-wallet"}

## Compile the CrowdfundingCampaign.sol contract

This guide introduces a crowdfunding campaign contract aimed at supporting Zeek's inventive ventures.
Let's start by reviewing the starter contract in the [`contracts/` directory](https://github.com/dutterbutter/zksync-quickstart-guide/blob/main/contracts/Crowdfund.sol).

::drop-panel
  ::panel{label="CrowdfundingCampaign.sol"}
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract CrowdfundingCampaign {
        address public owner;
        uint256 public fundingGoal;
        uint256 public totalFundsRaised;
        mapping(address => uint256) public contributions;

        event ContributionReceived(address contributor, uint256 amount);
        event GoalReached(uint256 totalFundsRaised);

        constructor(uint256 _fundingGoal) {
            owner = msg.sender;
            fundingGoal = _fundingGoal;
        }

        function contribute() public payable {
            require(msg.value > 0, "Contribution must be greater than 0");
            contributions[msg.sender] += msg.value;
            totalFundsRaised += msg.value;

            emit ContributionReceived(msg.sender, msg.value);

            if (totalFundsRaised >= fundingGoal) {
                emit GoalReached(totalFundsRaised);
            }
        }

        function withdrawFunds() public {
            require(msg.sender == owner, "Only the owner can withdraw funds");
            require(totalFundsRaised >= fundingGoal, "Funding goal not reached");

            uint256 amount = address(this).balance;
            totalFundsRaised = 0;

            (bool success, ) = payable(owner).call{value: amount}("");
            require(success, "Transfer failed.");
        }

        function getTotalFundsRaised() public view returns (uint256) {
            return totalFundsRaised;
        }

        function getFundingGoal() public view returns (uint256) {
            return fundingGoal;
        }
    }
    ```
  ::
::

The `CrowdfundingCampaign` contract is designed for project crowdfunding.
This contract features:

- A constructor to initialize the campaign's funding target.
- The `contribute` method to log funds, triggering `ContributionReceived` and `GoalReached` events.
- The `withdrawFunds` method, allowing the owner to collect accumulated funds post-goal achievement.

:display-partial{path = "/_partials/_compile-solidity-contracts"}

Upon successful compilation, you'll receive output detailing the
`zksolc` and `solc` versions used during compiling and the number
of Solidity files compiled.

```bash
Compiling contracts for zkSync Era with zksolc v1.4.0 and solc v0.8.17
Compiling 1 Solidity file
Successfully compiled 1 Solidity file
```

The compiled artifacts will be located in the `/artifacts-zk` folder.

## Deploy the contract

The deployment script is located at [`/deploy/deploy.ts`](https://github.com/dutterbutter/zksync-quickstart-guide/blob/main/deploy/deploy.ts).

```typescript
import { deployContract } from "./utils";

// An example of a basic deploy script
// It will deploy a CrowdfundingCampaign contract to selected network
// `parseEther` converts ether to wei, and `.toString()` ensures serialization compatibility.
export default async function () {
  const contractArtifactName = "CrowdfundingCampaign";
  const constructorArguments = [ethers.parseEther('.02').toString()];
  await deployContract(contractArtifactName, constructorArguments);
}
```

**Key Components:**

- **contractArtifactName:** Identifies the `CrowdfundingCampaign` contract for deployment.
- **constructorArguments:** Sets initialization parameters for the contract. In this case,
the fundraising goal, converted from ether to `wei` to match Solidity's `uint256` type.

1. Execute the deployment command corresponding to your package manager. The default command
deploys to the configured network in your Hardhat setup. For local deployment, append
`--network inMemoryNode` to deploy to the local in-memory node running.

::code-group

```bash [npm]
npm run hardhat deploy-zksync --script deploy.ts
# To deploy the contract on local in-memory node:
# npm run hardhat deploy-zksync --script deploy.ts --network inMemoryNode
```

```bash [yarn]
yarn hardhat deploy-zksync --script deploy.ts
# To deploy the contract on local in-memory node:
# yarn hardhat deploy-zksync --script deploy.ts --network inMemoryNode
```

```bash [pnpm]
pnpm run hardhat deploy-zksync --script deploy.ts
# To deploy the contract on local in-memory node:
# pnpm run hardhat deploy-zksync --script deploy.ts --network inMemoryNode
```

```bash [bun]
bun run hardhat deploy-zksync --script deploy.ts
# To deploy the contract on local in-memory node:
# bun run hardhat deploy-zksync --script deploy.ts --network inMemoryNode
```

::

Upon successful deployment, you'll receive output detailing the deployment process,
including the contract address, source, and encoded constructor arguments:

```bash
Starting deployment process of "CrowdfundingCampaign"...
Estimated deployment cost: 0.000501 ETH

"CrowdfundingCampaign" was successfully deployed:
 - Contract address: 0x4E3404F21b29d069539e15f8f9E712CeAE39d90C
 - Contract source: contracts/Crowdfund.sol:CrowdfundingCampaign
 - Encoded constructor arguments: 0x00000000000000000000000000000000000000000000000000470de4df820000

Requesting contract verification...
Your verification ID is: 10067
Contract successfully verified on zkSync block explorer!
```

🥳 Congratulations! Your smart contract is now deployed. 🚀