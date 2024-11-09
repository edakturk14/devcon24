# Devcon 2024 Workshop

Welcome to the Devcon 2024 Workshop! In this session, we’ll build a **gasless NFT minting app** using **Scaffold-ETH 2**. You'll learn to set up your development environment, deploy an ERC721 NFT contract, and integrate a paymaster to cover gas fees for your users. By the end, you'll have a fully functional and user-friendly NFT minting app that allows users to mint NFTs without paying for gas, using the Coinbase Smart Wallet.

---

## Step-by-step

### 1. Set Up an SE2 App

- **Prerequisites**: Ensure you have **Foundry** installed and ready to use.
- **Note**: When creating the app, **do not add extra spaces** in the title.

### 2. Add an NFT Contract

- Create a new contract file in the `app/foundry` folder.
- Implement an **ERC721 contract** using **Solmate**. Refer to this guide for help: [Solmate NFT Tutorial](https://book.getfoundry.sh/tutorials/solmate-nft).

### 3. Deploy the NFT Contract

- Navigate to the **Debug** tab in your SE2 app to review the contract details and confirm the deployment.

### 4. Edit the NFT Contract

- Add a **maximum cap** to restrict the number of users who can mint your NFT.
- **Generate an image** using [DALL-E](https://openai.com/dall-e/) and attach it to your NFT contract.
- Once you've made the edits, **re-deploy the contract** to update your app.

### 5. Start Building the Frontend

- Create a new folder named `/nft` in your project directory.
- Use **DaisyUI components** to build an intuitive and visually appealing NFT minting interface.
- **Test** the frontend to make sure it works smoothly and provides a good user experience.

### 6. Enable Gasless Transactions

- We'll implement **ERC7677** to use a paymaster, allowing users to mint NFTs without gas fees (sponsored by you). Learn more here: [ERC7677](https://www.erc7677.xyz/).
- Visit **CDP** to create a paymaster URL.
- Modify your SE2 app to support only the **Coinbase Smart Wallet** for secure and gasless transactions.

### 7. Ship Your Project!

- Run `yarn generate` to create an account for deployment.
- Deploy your project with `yarn deploy --network baseSepolia`.
- **Test the app thoroughly** to ensure everything works perfectly!

There you go, a gassless nft app! 🚀
