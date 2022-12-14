# Signature Based Minting with Next.JS

## Introduction

In this guide, we ask users to answer some questions about thirdweb before they can mint an NFT using [signature-based minting](https://portal.thirdweb.com/advanced-features/on-demand-minting)!

## Tools:

- [**Edition**](https://portal.thirdweb.com/contracts/edition): to create an ERC721 NFT Collection that our community can mint NFTs into.

- [**React SDK**](https://docs.thirdweb.com/react): to enable users to connect and disconnect their wallets to our website, using [useMetamask](https://docs.thirdweb.com/react/react.usemetamask) & [useDisconnect](https://docs.thirdweb.com/react/react.usedisconnect), and prompt them to approve transactions with MetaMask.

- [**TypeScript SDK**](https://docs.thirdweb.com/typescript): Mint new NFTs with [signature based minting](https://portal.thirdweb.com/advanced-features/on-demand-minting).

- [**Next JS API Routes**](https://nextjs.org/docs/api-routes/introduction): For us to securely generate signatures on the server-side, on behalf of our admin wallet, using our wallet's private key.

## Using This Repo

- Create a project using this example by running:

```bash
npx thirdweb create --template quiz
```

- Create your [Edition](https://portal.thirdweb.com/pre-built-contracts/edition) using the thirdweb dashboard
- Create an environment variable in a `.env.local` file with your private key, in the form `PRIVATE_KEY=xxx`, similarly to the `.env.example` file provided.
- Ensure to change the contract addresses to your own!

## What is Signature Based Minting?

[Signature-based minting](https://portal.thirdweb.com/advanced-features/on-demand-minting) allows you to grant mint signatures from an admin wallet (that has the `minter` role) in the contract, and have another wallet mint an NFT that you specify in your contract.

```tsx
import { ChainId, ThirdwebProvider } from "@thirdweb-dev/react";

// This is the chainId your dApp will work on.
const activeChainId = ChainId.Goerli;

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <ThirdwebProvider desiredChainId={activeChainId}>
      <Component {...pageProps} />
    </ThirdwebProvider>
  );
}
```

### Connecting User's Wallets

Over at the home page at `index.tsx`, we're using the [thirdweb React SDK MetaMask Connector](https://docs.thirdweb.com/react/category/wallet-connection) so our user can connect their wallet to our website.

```tsx
import {
  useAddress,
  useDisconnect,
  useMetamask,
  useNFTCollection,
} from "@thirdweb-dev/react";

  // Helpful thirdweb hooks to connect and manage the wallet from metamask.
  const address = useAddress();
  const connectWithMetamask = useMetamask();
  const isOnWrongNetwork = useNetworkMismatch();
  const [, switchNetwork] = useNetwork();

};
```

### Quiz Questions

We have two stateful variables to store which questions the user is on and if they have failed the quiz:

```jsx
const [questionNumber, setQuestionNumber] = useState(0);
const [failed, setFailed] = useState(false);
```

The [QuizContainer](./components/QuizContainer.tsx) component stores an array of questions, and their possible answers.

The current question (`questionNumber`) the user is on is shown in the [QuizQuestion](./components/QuizQuestion.tsx) component.

The user can click one of the answers to select it.

- If they get it correct, they see the next question
- If they get it wrong, they fail the quiz!

When the user finishes the final question and got all questions correct, they see the `Mint` NFT button which triggers the signature-based minting process outlined below.

### Creating NFTs with Signature-based Minting

The way that our signature-based minting process works is in 3 steps:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650958559249/S8mlZIQZm.png)

1. The connected wallet calls a [Next JS API Route](https://nextjs.org/docs/api-routes/introduction).

2. The API route generates a signature to mint an NFT based on token ID `0`'s metadata in the Edition contract.

3. Once the API function is done processing, it sends the client/user a signature. The user can call `mint` with this signature to create an NFT with the conditions provided by our server.

## API Route

To create an API Route, you'll need to create a file in the `/pages/api` directory of your project.

On the server-side API route, we can:

- Run a few checks to see if the requested NFT meets our criteria
- Generate a signature to mint the NFT if it does.
- Send the signature back to the client/user if the NFT is eligible.

**Initialize the Thirdweb SDK on the server-side**

```tsx
// Initialize the Thirdweb SDK on the serverside
const sdk = ThirdwebSDK.fromPrivateKey(
  // Your wallet private key (read it in from .env.local file)
  process.env.PRIVATE_KEY as string,
  "goerli"
);
```

**Load the NFT Collection via it's contract address using the SDK**

```tsx
const nftCollection = sdk.getEdition(
  // Replace this with your Edition contract address
  "0x000000000000000000000000000000000000000"
);
```

**Generate a signature to mint the NFT**

```tsx
// Generate the signature for the page NFT
const signedPayload = await editionContract.signature.generateFromTokenId({
  quantity: 1,
  tokenId: 0,
  to: address,
});
```

**Return the signature to the client**

```tsx
// Return back the signedPayload to the client.
res.status(200).json(signedPayload);
```

If at any point this process fails or the request is not valid, we send back an error response instead of the generated signature.

```tsx
res.status(500).json({ error: `Server error ${e}` });
```

## Making the API Request on the client

With our API route available, we make `fetch` requests to this API, and securely run that code on the server-side.

**Call the API route on the client**

```tsx
// Make a request to /api/server
const signedPayloadReq = await fetch(`/api/server`, {
  method: "POST",
  body: JSON.stringify({
    address: address, // Address of the current user
  }),
});
```

**Mint the NFT with the signature**

```tsx
// Now we can call signature.mint and pass in the signed payload that we received from the server.
// This means we provided a signature for the user to mint an NFT with.
const nft = await nftCollection?.signature.mint(signedPayload);
```

---

## Join our Discord!

For any questions, suggestions, join our discord at [https://discord.gg/thirdweb](https://discord.gg/thirdweb).
