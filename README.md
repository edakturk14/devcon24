# Gasless NFT minting app w/ScaffoldETH + Coinbase Smart Wallet

*For Devcon 24' workshop, built by [Shiv Bhonde](https://github.com/technophile-04) & Eda Akturk*

In this workshop, we’ll build a **gasless NFT minting app** using **Scaffold-ETH 2**. By the end, you'll have a NFT minting app that allows users to mint NFTs without paying for gas, using the Coinbase Smart Wallet.

![Screenshot 2024-11-21 at 11.29.56](https://hackmd.io/_uploads/SkCyxdhzke.png)

### Tools & Resources We'll Use

- [Scaffold-ETH 2](https://scaffoldeth.io/): A developer toolset for building decentralized apps quickly.
- Foundry: A blazing-fast, modular toolkit for Ethereum smart contract development.
- [EIP-721](https://eips.ethereum.org/EIPS/eip-721): Standard for NFT contracts.
- [EIP-4337](https://www.erc4337.io/) (Account Abstraction): This EIP allows smart contracts to function as wallets, enabling advanced features like batching and gas sponsorship, which are essential for user-friendly dApps.
- [EIP-5792](https://eips.ethereum.org/EIPS/eip-5792) (Wallet call api): Methods for batching transactions and using paymasters to cover gas fees, streamlining wallet interactions for better efficiency and experience.
- [EIP-7677](https://eips.ethereum.org/EIPS/eip-7677) (Paymaster Contract): Standardizes gas sponsorship, making it easier for dApps to cover user transaction fees, enhancing the user experience.
- DaisyUI: A Tailwind CSS-based component library for styling.
- [Coinbase Smart Wallet](https://www.coinbase.com/en-tr/wallet/smart-wallet)

---

## Step-by-step

### 1. Set Up an SE2 App

- **Prerequisites**: Make sure you have **Foundry** installed.

```
npx create-eth@latest
```

- **Note**: When creating the app, **do not add extra spaces** in the title.
- Choose foundry as your smart contract development environment

  > ? What solidity framework do you want to use? foundry"

Once downloaded, run your app:

```
yarn chain // start your local anvil env
yarn deploy // deploys a test smart contract to the local network
yarn start // start your NextJS app
```

Visit: http://localhost:3000

_Features to explore:_

- Burner wallets
- Built in faucet
- Debug contracts tab

### 2. Add a new NFT Contract

- Go to "https://wizard.openzeppelin.com/#erc721" to get a template for an ERC721 contract.

  - Make the contract Mintable and Enumerable
  - Fill the baseURI with an image for your NFT. (idea: generate an image with Dall-e)

- Create a new file called `NFT.sol` in the `packages/foundry/contracts` folder. Add you contract here.

Some changes to the wizard generated contract:

- Make `_nextTokenId` public so that we count remaining supply
- Add `uint256 public maxSupply = 10;` to limit the number of NFTs that can be minted.
- Create `mapping(address => bool) public alreadyMinted;`
- Update safeMint with extra require statements + make alreadyMinted[to] = true
  ```solidity
  function safeMint(address to) public {
    require(_nextTokenId < maxSupply, "Max supply reached");
    require(!alreadyMinted[to], "Already minted");
    alreadyMinted[to] = true; // add this
    uint256 tokenId = _nextTokenId++;
    _safeMint(to, tokenId);
    }
  ```

### 3. Create a new deployment script and deploy the NFT Contract

- Create a new deployment script in the `packages/foundry/script` folder. Check the contents of the `DeployYourContract.s` and make the adjusments for the new NFT contract.

  - Change import instead import NFT.sol
  - Change the line of YourContract ⇒ `NFT nft = new NFT()` (we had removed the deployer from constructor)

- Add the new deployment script to `Deploy.s.sol`. And deploy the contracts by running:

```
yarn deploy --reset
```

- Go to the **Debug** tab in your SE2 app to review the contract details and confirm the deployment.

### 4. Start Building the Frontend

- Create a new folder named `/nft` in `packages/nextjs/app` and a page.tsx file inside it.

_We'll use **DaisyUI components** to build a pretty NFT minting interface. Check them out here: https://daisyui.com/components/card/_

Steps:

- Get max supply for the NFT collection and tokenID with the useScaffoldReadContract Hooks.

  ```
    const { data: supply } = useScaffoldReadContract({
      contractName: "NFT",
      functionName: "maxSupply",
    });

    const { data: tokenID } = useScaffoldReadContract({
      contractName: "NFT",
      functionName: "_nextTokenId",
    });
  ```

- Get the NFT's left to mint.

  ```
  const [remaining, setRemaining] = useState<number | null>(null);
  useEffect(() => {
    if (supply !== undefined) {
      setRemaining(Number(supply) - (tokenID ? Number(tokenID) : 0));
    }
  }, [tokenID, supply]);
  ```

- Create a mint button with the useScaffoldWriteContract hooks.

  ```
  const { writeContractAsync: writeScaffoldContractAsync } = useScaffoldWriteContract("NFT");

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
  ```

- Create a new component to display the NFT's that the user has. Here is how the page should be: `app/nft/_components/MyHoldings.tsx`. We're using the same [component](https://github.com/scaffold-eth/se-2-challenges/blob/challenge-0-simple-nft/packages/nextjs/app/myNFTs/_components/MyHoldings.tsx) from the simple NFT challange of [speedrunethereum](https://speedrunethereum.com/). Import this into  `app/nft/page.tsx`

Run your app to make sure all is working.

### 5. Create a link in the header

Create a link in `Header.tsx` to navigate to the NFT minting page.

```
{
label: "NFT",
href: "/nft",
},
```

### 6. Add coinbase smart wallet + gasless transaction

- Modify your SE2 app to support only the **Coinbase Smart Wallet**. Go `wagmiConnectors.ts` remove other wallet settings, and add the following:

```
coinbaseWallet.preference = "smartWalletOnly";

const wallets = [coinbaseWallet];
```

- We will use wagmis experimental hook called [`useWriteContracts`](https://wagmi.sh/react/api/hooks/useWriteContracts), and add it to our `page.tsx`

```
// wagmi hook to batch write to multiple contracts (EIP-5792 specific)
const { writeContractsAsync } = useWriteContracts();
// add this later to read address and contract abi
const { data: NFT } = useDeployedContractInfo("NFT");
```

- Create a new `Gasless mint button`. This will use a paymaster to cover the transaction cost.

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

- Get the PAYMASTER URL from [Coinbase CDP](https://www.coinbase.com/en-tr/developer-platform) and add it to env `NEXT_PUBLIC_PAYMASTER_URL`
  - onchain tools ⇒ Paymaster ⇒ configuration (make sure you switch to base sepolia)
  - Create a `.env.local` and add `NEXT_PUBLIC_PAYMASTER_URL`

### 7. Deploy your contract to Base Sepolia

- Run `yarn generate` to create an account for deployment.
- Update the `.env` inside foundry with `scaffold-eth-custom`
- Send some base sepolia funds to the deployer account `yarn account`
- Deploy your contract with `yarn deploy --network baseSepolia`

### 8. Update your app to point to Base Sepolia and Ship it

- Update `scaffold.config.ts`:

```
targetNetworks: [chains.baseSepolia],

pollingInterval: 3000,
```

- Ship to Vercel: `yarn vercel:yolo`
- Go to vercel deployment UI -> add enviornment variable -> paste the `NEXT_PUBLIC_PAYMASTER_URL` and then run `yarn vercel:yolo` again

There you go, a gassless nft minting app! 🚀
