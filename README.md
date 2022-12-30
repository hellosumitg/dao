# Building a DAO

![](https://i.imgur.com/6uXR2G9.png)

## What is a DAO?

DAO stands for **D**ecentralized **A**utonomous **O**rganization. You can think of DAOs as analogous to companies in the real world. Essentially, DAOs allow for members to create and vote on governance decisions.

In traditional companies, when a decision needs to be made, the board of directors or executives of the company are in charge of making that decision. In a DAO, however, this process is democratized, and any member can create a proposal, and all other members can vote on it. Each proposal created has a deadline for voting, and after the deadline the decision is made in favour of the voting outcome (YES or NO).

Membership in DAOs is typically restricted either by ownership of ERC20 tokens, or by ownership of NFTs. Examples of DAOs where membership and voting power is proportional to how many tokens you own include [Uniswap](https://uniswap.org) and [ENS](https://ens.domains). Examples of DAOs where they are based on NFTs include [Meebits DAO](https://www.meebitsdao.world/).

## Building our DAO

We want to launch a DAO for holders of our `CryptoDevs` NFTs. From the ETH that was gained through the ICO, we built up a DAO Treasury. The DAO now has a lot of ETH, but currently does nothing with it.

You want to allow your NFT holders to create and vote on proposals to use that ETH for purchasing other NFTs from an NFT marketplace, and speculate on price. Maybe in the future when you sell the NFT back, you split the profits among all members of the DAO.

## Requirements

- Anyone with a `CryptoDevs` NFT can create a proposal to purchase a different NFT from an NFT marketplace
- Everyone with a `CryptoDevs` NFT can vote for or against the active proposals
- Each NFT counts as one vote for each proposal
- Voter cannot vote multiple times on the same proposal with the same NFT
- If majority of the voters vote for the proposal by the deadline, the NFT purchase is automatically executed

## What we will make

- To be able to purchase NFTs automatically when a proposal is passed, you need an on-chain NFT marketplace that you can call a `purchase()` function on. There exist a lot of NFT marketplaces out there, but to avoid overcomplicating things, we will create a simplified fake NFT marketplace for this tutorial as the focus is on the DAO.
- We will also make the actual DAO smart contract using Hardhat.
- We will make the website using Next.js to allow users to create and vote on proposals

## Prerequisites

- You should complete transaction on the [nft_collection](https://github.com/sumitdevtech/nft_collection) and for doing so you must have whitelisted your addresses by using [whitelist_dapp](https://github.com/sumitdevtech/whitelist-dapp)
- You must have some ETH to give to the DAO Treasury

## BUIDL IT

### Smart Contract Development

We will start off with first creating the smart contracts. We will be making two smart contracts:

- `FakeNFTMarketplace.sol`
- `CryptoDevsDAO.sol`

To do so, we will use the [Hardhat](https://hardhat.org) development framework we have been using for the last few tutorials.

- Create a folder for this project named `DAO`, and open up a Terminal window in that folder.
- Setup a new hardhat project by running the following commands in your terminal:

  ```bash
  mkdir hardhat
  cd hardhat
  npm init --yes
  or
  yarn init --yes
  npm install --save-dev hardhat
  or
  yarn add hardhat --dev
  ```

  Now that you have installed Hardhat, we can setup a project. Execute the following command in your terminal.

- In the same directory where you installed Hardhat run:

  ```bash
  npx hardhat
  ```

  - Select `Create a basic sample project`
  - Press enter for the already specified `Hardhat Project root`
  - Press enter for the question on if you want to add a `.gitignore`
  - Press enter for `Do you want to install this sample project's dependencies with npm (@nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers)?`

    Now you have a hardhat project ready to go!

    If you are not on mac, please do this extra step and install these libraries as well :)

    ```bash
    npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers
    or
    yarn add @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers --dev

    ```

  and press `Enter` for all the questions (Choose the `Create a basic sample project`) option.

- Now, let's install the `@openzeppelin/contracts` package from NPM as we will be using [OpenZeppelin's Ownable Contract](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol) for the DAO contract.
  ```bash
  npm install @openzeppelin/contracts
  or
  yarn add @openzeppelin/contracts
  ```
- First, let's make a simple Fake NFT Marketplace. Create a file named `FakeNFTMarketplace.sol` under the `contracts` directory within `hardhat`, and add the codes as shown in this repo...

- The `FakeNFTMarketplace` exposes some basic functions that we will be using from the DAO contract to purchase NFTs if a proposal is passed. A real NFT marketplace would be more complicated - as not all NFTs have the same price.
- Let's make sure everything compiles before we start writing the DAO Contract. Run the following command inside the `hardhat` folder from your Terminal.
  ```bash
  npx hardhat compile
  ```
  and make sure there are no compilation errors.
- Now, we will start writing the `CryptoDevsDAO` contract. Since this is mostly a completely custom contract, and relatively more complicated than what we have done so far, we will explain this one, bit-by-bit.
- First, let's write the boilerplate code for the contract. Create a new file named `CryptoDevsDAO.sol` under the `contracts` directory in `hardhat` and add the code as shown in this repo...

- Also, we will need to call functions on the `FakeNFTMarketplace` contract, and our previously deployed `CryptoDevs NFT` contract. Recall from the `Advanced Solidity Topics` tutorial that we need to provide an interface for those contracts, so this contract knows which functions are available to call and what they take as parameters and what they return.
- Add the two interfaces one for `FakeNFTMarketplace` and one for `CryptoDevsNFT` to your code as shown in the repo...

- Now, let's think about what functionality we need in the DAO contract.

  - Store created proposals in contract state
  - Allow holders of the CryptoDevs NFT to create new proposals
  - Allow holders of the CryptoDevs NFT to vote on proposals, given they haven't already voted, and that the proposal hasn't passed it's deadline yet
  - Allow holders of the CryptoDevs NFT to execute a proposal after it's deadline has been exceeded, triggering an NFT purchase in case it passed

- Let's start off by creating a `struct` representing a `Proposal`, in our contract, as shown in this repo...

- Let's also create a mapping from Proposal IDs to Proposals to hold all created proposals, and a counter to count the number of proposals that exist as shown in this repo...

- Now, since we will be calling functions on the `FakeNFTMarketplace` and `CryptoDevsNFT` contract, let's initialize variables for those contracts.

  ```solidity
  IFakeNFTMarketplace nftMarketplace;
  ICryptoDevsNFT cryptoDevsNFT;
  ```

- Create a `constructor` function that will initialize those contract variables, and also accept an ETH deposit from the deployer to fill the DAO ETH treasury. (In the background, since we imported the `Ownable` contract, this will also set the contract deployer as the owner of this contract)

- Now, since we want pretty much all of our other functions to only be called by those who own NFTs from the `CryptoDevs NFT` contract, we will create a `modifier` to avoid duplicating code. As shown in this repo

- We now have enough to write our `createProposal` function, which will allow members to create new proposals as shown in our repo...

- Now, to vote on a proposal, we want to add an additional restriction that the proposal being voted on must not have had it's deadline exceeded. To do so, we will create a second modifier `activeProposalOnly(uint256 proposalIndex)` as shown in this repo.

  Note how this modifier takes a parameter!

- Additionally, since a vote can only be one of two values (YAY or NAY) - we can create an `enum` representing possible options as shown in the repo.

- Let's write the `voteOnProposal` function as shown in this repo...

- We're almost done! To execute a proposal whose deadline has exceeded, we will create our final modifier `inactiveProposalOnly(uint256 proposalIndex)`.

  Note this modifier also takes a parameter!

- Let's write the code for `executeProposal` as shown in this repo...

- We have at this point implemented all the core functionality. However, there are a couple of additional features we could and should implement.
  - Allow the contract owner to withdraw the ETH from the DAO if needed
  - Allow the contract to accept further ETH deposits
- The `Ownable` contract we inherit from contains a modifier `onlyOwner` which restricts a function to only be able to be called by the contract owner. Let's implement `withdrawEther` using that modifier.

  ```solidity
  /// @dev withdrawEther allows the contract owner (deployer) to withdraw the ETH from the contract
  function withdrawEther() external onlyOwner {
      payable(owner()).transfer(address(this).balance);
  }
  ```

  This will transfer the entire ETH balance of the contract to the owner address

- Finally, to allow for adding more ETH deposits to the DAO treasury, we need to add some special functions. Normally, contract addresses cannot accept ETH sent to them, unless it was through a `payable` function. But we don't want users to call functions just to deposit money, they should be able to tranfer ETH directly from their wallet. For that, let's add these two functions:

  ```solidity
  // The following two functions allow the contract to accept ETH deposits
  // directly from a wallet without calling a function
  receive() external payable {}

  fallback() external payable {}
  ```

### Smart Contract Deployment

Now that we have written both our contracts, let's deploy them to the [Goerli Testnet](https://goerli.etherscan.io). Ensure you have some ETH on the Goerli Testnet.

- Install the `dotenv` package from NPM to be able to use environment variables specified in `.env` files in the `hardhat.config.js`. Execute the following command in your Terminal in the `hardhat` directory.
  ```bash
  npm install dotenv
  or
  yarn add dotenv
  ```
- Now create a `.env` file in the `hardhat` directory and set the following two environment variables. Follow the instructions to get their values. Make sure the Goerli private key you use is has ETH on the Goerli Testnet as shown in this repo...

- Now, let's write a deployment script to automatically deploy both our contracts for us. Create a new file named `deploy.js` under `hardhat/scripts`, and add the codes to it from this repo...

- As you may have noticed, `deploy.js` imports a variable called `CRYPTODEVS_NFT_CONTRACT_ADDRESS` from a file named `constants`. Let's make that. Create a new file named `constants.js` in the `hardhat` directory as shown in this repo...

- Now, let's add the Goerli Network to your Hardhat Config so we can deploy to Goerli. Open your `hardhat.config.js` file and replace it with the codes as shown in this repo...

- Let's make sure everything compiles before proceeding. Execute the following command from your Terminal within the `hardhat` folder.
  ```bash
  npx hardhat compile
  ```
  and make sure there are no compilation errors.
  If you do face compilation errors, watch the code in this repo...
- Let's deploy! Execute the following command in your Terminal from the `hardhat` directory
  ```bash
  npx hardhat run scripts/deploy.js --network Goerli
  ```
- Save the `FakeNFTMarketplace` and `CryptoDevsDAO` contract addresses that get printed in your Terminal. You will need those later.

### Frontend Development

Whew! So much coding!

We've successfully developed and deployed our contracts to the Goerli Testnet. Now, it's time to build the Frontend interface so users can create and vote on proposals from the website.

To develop the website, we will be using [Next.js](https://nextjs.org/) as we have so far, which is a meta-framework built on top of [React](https://reactjs.org/).

- Let's get started by creating a new `next` app. Your folder structure should look like this after setting up the `next` app:
  ```bash
  - DAO
      - hardhat
      - next-app
  ```
- To create `next-app`, execute the following command in your Terminal within the `DAO-Tutorial` directory
  ```bash
  npx create-next-app@latest
  ```
  and press `Enter` for all the question prompts. This should create the `next-app` folder and setup a basic Next.js project.
- Let's see if everything works. Run the following in your Terminal
  ```bash
  cd next-app
  npm run dev
  or
  yarn dev
  ```
- Your website should be up and running at `http://localhost:3000`. However, this is a basic starter Next.js project and we need to add code for it to do what we want.
- Let's install the `web3modal` and `ethers` library. Web3Modal will allow us to support connecting to wallets in the browser, and Ethers will be used to interact with the blockchain. Run this in your Terminal from the `next-app` directory.
  ```bash
  npm install web3modal ethers
  or
  yarn add web3modal ethers
  ```
- Download and save the following file as `0.svg` in `next-app/public/cryptodevs`. We will display this image on the webpage. NOTE: You need to create the `cryptodevs` folder inside `public`.
  [Download Image](https://github.com/hellosumitg/nft_collection/blob/main/next-app/public/cryptodevs/0.svg)
- Add the CSS styles to the `next-app/styles/Home.modules.css` as shown in this repo...

- The website also needs to read/write data from two smart contracts - `CryptoDevsDAO` and `CryptoDevsNFT`. Let's store their contract addresses and ABIs in a constants file. Create a `constants.js` file in the `next-app` directory as shown in this repo...

  ```javascript

  ```

- Replace the contract address and ABI values with your relevant contract addresses and ABIs.
- Now for the actual cool website code. Open up `next-app/pages/index.js` and write the codes from this repo. Explanation of the code can be found in the comments.

- Let's run it! In your terminal, from the `next-app` directory, execute:
  ```bash
  npm run dev
  or
  yarn dev
  ```
  to see your website in action. It should look like the screenshot at the beginning of this tutorial.

Congratulations! Your CryptoDevs DAO website should now be working.

### Testing

- Create a couple of proposals
- Try voting `YAY` on one, and `NAY` on the other
- Wait for 5 minutes for their deadlines to pass
- Execute both of them.
- Watch the balance of the DAO Treasury go down by `0.1 ETH` due to the proposal which passed as it bought an NFT upon execution.

### Push to Github

Make sure to push all this code to Github before proceeding to the next step.

### Website Deployment

What good is a website if you cannot share it with others? Let's work on deploying your dApp to the world so you can share it with all your LearnWeb3DAO frens.

- Go to [Vercel Dashboard](https://vercel.com) and sign in with your GitHub account.
- Click on the `New Project` button and select your `DAO-Tutorial` repo.
- When configuring your new project, Vercel will allow you to customize your `Root Directory`
- Since our Next.js application is within a subfolder of the repo, we need to modify it.
- Click `Edit` next to `Root Directory` and set it to `next-app`.
- Select the framework as `Next.js`
- Click `Deploy`

![](https://i.imgur.com/YIOtTTR.png)

- Now you can see your deployed website by going to your Vercel Dashboard, selecting your project, and copying the domain from there!

The working link for this project is here [DAO](https://dao-hellosumitg.vercel.app/)

Special thanks to team@LearnWeb3DAO
