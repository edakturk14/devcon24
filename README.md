# Devcon 2024 Workshop

Welcome to the Devcon 2024 Workshop! In this session, we’ll build a **gasless NFT minting app** using **Scaffold-ETH 2**. You'll learn to set up your development environment, deploy an ERC721 NFT contract, integrate a paymaster to cover gas fees for your users. By the end, you'll have a fully functional and user-friendly NFT minting app that allows users to mint NFTs without paying for gas, using the Coinbase Smart Wallet.

### Tools & Resources We'll Use

- Scaffold-ETH 2: A developer toolset for building decentralized apps quickly.
- Foundry: A blazing-fast, modular toolkit for Ethereum smart contract development.
- ERC-721: Standard for NFT contracts.
- ERC-4337: The EIP that enables Account Abstraction, foundational for the Coinbase Smart Wallet.
- ERC-7677: Set Up Paymaster with ERC-7677 for gas sponsorship
- DaisyUI: A Tailwind CSS-based component library for styling.

---

## Step-by-step

### 1. Set Up an SE2 App

- **Prerequisites**: Ensure you have **Foundry** installed and ready to use.
- **Note**: When creating the app, **do not add extra spaces** in the title.

```
npx create-eth@latest
```

- Make sure to choose foundry as your smart contract development environment

  > ? What solidity framework do you want to use? foundry"

### 2. Start the app and test it out

```
yarn chain // start your local anvil env
yarn deploy // deploys a test smart contract to the local network
yarn start // start your NextJS app
```

Visit: http://localhost:3000

### 3. Add a new NFT Contract

- Go to "https://wizard.openzeppelin.com/#erc721" to get a template for an ERC721 contract.

  - Make sure to fill the baseURI with an image for your NFT.
  - rename the contract as "NFT"
  - Add a maxSupply to limit the number of NFTs that can be minted.

- Create a new file called `NFT.sol` in the `packages/foundry/contracts` folder. Here is how the contract should look like:

```
// Example ERC721 Contract
// SPDX-License-Identifier: MIT
// Compatible with OpenZeppelin Contracts ^5.0.0
pragma solidity ^0.8.22;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract NFT is ERC721, Ownable {
    uint256 public nextTokenId;
    uint256 public maxSupply = 10;

    constructor(address initialOwner)
        ERC721("MyToken", "MTK")
        Ownable(initialOwner)
    {}

    function _baseURI() internal pure override returns (string memory) {
        return "https://www.allaboutbirds.org/news/wp-content/uploads/2024/09/TOC-Autumn24-Ruby-crowned_Kinglet-Christopher_T-ML609692481-FI-480x360.jpg";
    }

    function safeMint(address to) public {
      require(nextTokenId < maxSupply, "Max supply reached");
        uint256 tokenId = nextTokenId++;
        _safeMint(to, tokenId);
    }
}
```

### 4. Create a new deployment script and deploy the NFT Contract

- Create a new deployment script in the `packages/foundry/script` folder. You can check the contents of the `DeployYourContract.s`to give an idea on what the script should look like. Make sure to adjust the parameters based on your constructor.

- Add the new deployment script to `Deploy.s.sol`. And deploy the contracts.

```
yarn deploy --reset
```

- Go to the **Debug** tab in your SE2 app to review the contract details and confirm the deployment.

### 5. Start Building the Frontend

- Create a new folder named `/nft` in your project directory.
- Use **DaisyUI components** to build an intuitive and visually appealing NFT minting interface. Check them out here: https://daisyui.com/components/card/
- Use the useScaffoldReadContract hook to read the max supply and the next token ID from the NFT contract.
- Use the useScaffoldWriteContract hook to mint the NFT.

```
"use client";

import { useEffect, useState } from "react";
import type { NextPage } from "next";
import { useAccount } from "wagmi";
import { useWriteContracts } from "wagmi/experimental";
import { useDeployedContractInfo, useScaffoldReadContract, useScaffoldWriteContract } from "~~/hooks/scaffold-eth";
import { notification } from "~~/utils/scaffold-eth";

const NFT: NextPage = () => {
  const { address: connectedAddress } = useAccount();

  const { data: supply } = useScaffoldReadContract({
    contractName: "NFT",
    functionName: "MAX_SUPPLY",
  });

  const { data: tokenID } = useScaffoldReadContract({
    contractName: "NFT",
    functionName: "nextTokenId",
  });

  const { writeContractAsync: writeScaffoldContractAsync } = useScaffoldWriteContract("NFT");

  const { data: NFT } = useDeployedContractInfo("NFT");
  // wagmi hook to batch write to multiple contracts (EIP-5792 specific)
  const { writeContractsAsync } = useWriteContracts();

  const [remaining, setRemaining] = useState<number | null>(null);
  useEffect(() => {
    if (supply !== undefined) {
      setRemaining(Number(supply) - (tokenID ? Number(tokenID) : 0));
    }
  }, [tokenID, supply]);

  return (
    <>
      <div className="flex items-center flex-col mt-10">
        <h1 className="text-center">
          <span className="block text-4xl font-bold">Mint a Bird</span>
          <span className="block text-2xl font-bold mt-1">NFTs left: {remaining}</span>
        </h1>
        <div className="card bg-base-100 w-96 shadow-xl mt-5">
          <div className="card-body items-center">
            <div className="card-actions justify-end">
              {connectedAddress ? (
                <div className="flex space-x-2">
                  <button
                    className="btn btn-primary"
                    onClick={async () => {
                      try {
                        await writeScaffoldContractAsync({
                          functionName: "safeMint",
                          args: [connectedAddress],
                        });
                      } catch (e) {
                        console.error("Error minting NFT:", e);
                      }
                    }}
                  >
                    Mint NFT
                  </button>
                </div>
              ) : (
                <span className="text-gray-500">Connect Wallet</span>
              )}
            </div>
          </div>
        </div>
      </div>
    </>
  );
};

export default NFT;

```

### 6. Create a link in the header

Create a link in `Header.tsx` to navigate to the NFT minting page.

```
  {
    label: "NFT",
    href: "/nft",
  },
```

### 7. Update the NFT Contract

- Update the contract to keep track of the addresses and the index of whom has minted the NFT.

```
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
```

### 8. Update the frontend

- Create a new compontents called "MyHoldings.tsx" to display the NFT's that the user owns.
- Add <MyHoldings /> to the page.tsx

### 9. Gassless Transactions & Coinbase Smart Wallet

- Modify your SE2 app to support only the **Coinbase Smart Wallet**. Go `wagmiConnectors.ts` remove other wallet settings, and add the following:

```
coinbaseWallet.preference = "smartWalletOnly";
const wallets = [coinbaseWallet];
```

- We'll implement **ERC7677** to use a paymaster, allowing users to mint NFTs without gas fees (sponsored by you). Learn more here: [ERC7677](https://www.erc7677.xyz/).
- Add a new gassless mint button:

```
<button
  className="btn btn-primary w-45"
  onClick={async () => {
    try {
      if (!NFT) return;
      // TODO: update the ENV
      const paymasterURL = process.env.NEXT_PUBLIC_PAYMASTER_URL;
      await writeContractsAsync({
        contracts: [
          {
            address: NFT.address,
            abi: NFT.abi,
            functionName: "safeMint",
            args: [connectedAddress],
          },
        ],
        capabilities: {
          paymasterService: {
            url: paymasterURL,
          },
        },
      });
      notification.success("NFT minted");
    } catch (e) {
      console.error("Error minting NFT:", e);
    }
  }}
>
  Gasless Mint
</button>
```

- Get the PAYMASTER URL from coinbase CDP and add it to env `NEXT_PUBLIC_PAYMASTER_URL`

### 10. Ship Your Project!

- Run `yarn generate` to create an account for deployment.
- Deploy your project with `yarn deploy --network baseSepolia`.
- **Test the app thoroughly** to ensure everything works perfectly!

There you go, a gassless nft app! 🚀

Send it out to the world: yarn vercel:yolo
