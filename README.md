# Devcon 2024 Workshop

Welcome to the Devcon 2024 Workshop! In this session, we’ll build a **gasless NFT minting app** using **Scaffold-ETH 2**. You'll learn to set up your development environment, deploy an ERC721 NFT contract, and integrate a paymaster to cover gas fees for your users. By the end, you'll have a fully functional and user-friendly NFT minting app that allows users to mint NFTs without paying for gas, using the Coinbase Smart Wallet.

### Tools & Resources We'll Use

- Scaffold-ETH 2: A developer toolset for building decentralized apps quickly.
- Foundry: A blazing-fast, modular toolkit for Ethereum smart contract development.
- ERC-4337: The EIP that enables Account Abstraction, foundational for the Coinbase Smart Wallet.
- ERC-7677: Set Up Paymaster with ERC-7677 for gas sponsorship
- ERC-721: Standard for NFT contracts.
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

- Install the requirments for solmate/openzepplin:

`forge install transmissions11/solmate Openzeppelin/openzeppelin-contracts`

- Create a new contract file called `NFTContract.sol` in the `packages/foundry/contracts` folder.
- Implement an **ERC721 contract** using **Solmate**. Refer to this guide for help: [Solmate NFT Tutorial](https://book.getfoundry.sh/tutorials/solmate-nft).

```
// Example ERC721 Contract
contract MyNFT is ERC721 {
    // NFT logic here
}
```

### 4. Create a new deployment script and deploy the NFT Contract

- Create a new deployment script in the `packages/foundry/script` folder. You can check the contents of the `DeployYourContract.s`to give an idea on what the script should look like.
- _Hint:_ make sure to adjust the parameters based on your constructor.

- Add the new deployment script to `Deploy.s.sol`
- _Note: you may need to update the solidity version on the contract file._

- Go to the **Debug** tab in your SE2 app to review the contract details and confirm the deployment.

### 5. Start Building the Frontend

- Create a new folder named `/nft` in your project directory.
- Use **DaisyUI components** to build an intuitive and visually appealing NFT minting interface. Check them out here: https://daisyui.com/components/card/

```
<div className="items-center flex flex-col">

<h1: nft collection details>
<daisy ui card>

</div>
```

- **Test** the frontend to make sure it works smoothly and provides a good user experience.

### 6. Add contract function calls to the frontend

- We want to be able to mint the nft from the frontend. Here is how: https://docs.scaffoldeth.io/hooks/useScaffoldWriteContract
- Get the address of the connected account. You can use the useAccount hook in wagmi (more: https://docs.scaffoldeth.io/recipes/GetCurrentBalanceFromAccount)

### 7. Edit the NFT Contract

- Add a **maximum cap** to restrict the number of users who can mint your NFT.
- Once you've made the edits, **re-deploy the contract** to update your app.

```
uint256 public constant maxSupply = 10; // Set the max supply to 10

function mintTo(address recipient) public payable returns (uint256) {
    require(currentTokenId < maxSupply, "Max supply reached");
    uint256 newItemId = ++currentTokenId;
    _safeMint(recipient, newItemId);
    return newItemId;
}
```

### 8. Add the max supply to your frontend

- You want to have the tokenId and the max supply. Simply use the useScaffoldRead hooks for this.
- _Hint: When interacting with Ethereum smart contracts, numeric values are often returned as BigNumber so you'll need to change the values to Number(xx)_

[to add: - Add an SVG image to your NFT.
]

### 9. Enable Gasless Transactions

- We'll implement **ERC7677** to use a paymaster, allowing users to mint NFTs without gas fees (sponsored by you). Learn more here: [ERC7677](https://www.erc7677.xyz/).
- Visit **CDP** to create a paymaster URL.
- Modify your SE2 app to support only the **Coinbase Smart Wallet** for secure and gasless transactions.

### 10. Ship Your Project!

- Run `yarn generate` to create an account for deployment.
- Deploy your project with `yarn deploy --network baseSepolia`.
- **Test the app thoroughly** to ensure everything works perfectly!

There you go, a gassless nft app! 🚀
