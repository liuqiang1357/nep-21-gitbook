# dApp Developers

This guide explains how a dApp should use a NEP-21 provider in the browser.

## Integration Flow

A production integration usually follows this sequence:

1. Discover compatible providers.
2. Select a provider.
3. Verify dAPI compatibility and active network.
4. Read connected accounts.
5. Subscribe to account and network changes.
6. Use `call` for read-only contract execution.
7. Use `send`, `invoke`, `makeTransaction`, `sign`, or `relay` for wallet-mediated actions.
8. Handle errors by numeric code.

## Discover Providers

NEP-21 providers are discovered through browser events. Listen for `Neo.DapiProvider.ready`, then dispatch `Neo.DapiProvider.request` to ask installed wallets to announce themselves.

```ts
export async function getNeoDapiProviders(timeoutMs = 3000) {
  return new Promise<any[]>((resolve) => {
    const providers: any[] = [];

    const onReady = (event: Event) => {
      const provider = (event as CustomEvent).detail?.provider;
      if (!provider) return;

      const alreadyAdded = providers.some((item) => item.name === provider.name);
      if (!alreadyAdded && provider.compatibility?.includes("NEP-21")) {
        providers.push(provider);
      }
    };

    window.addEventListener("Neo.DapiProvider.ready", onReady);

    window.dispatchEvent(new CustomEvent("Neo.DapiProvider.request", {
      detail: { version: "1.0" },
    }));

    window.setTimeout(() => {
      window.removeEventListener("Neo.DapiProvider.ready", onReady);
      resolve(providers);
    }, timeoutMs);
  });
}
```

If more than one provider responds, show a wallet selector. Do not silently pick a wallet unless your product has a clear reason to do so.

## Validate the Provider

Check the standard version, compatibility, and network before using the provider.

```ts
const MAINNET = 860833102;
const TESTNET = 894710606;

function assertCompatibleProvider(provider: any, expectedNetwork: number) {
  if (provider.dapiVersion !== "1.0") {
    throw new Error(`Unsupported dAPI version: ${provider.dapiVersion}`);
  }

  if (!provider.compatibility?.includes("NEP-21")) {
    throw new Error("This wallet does not support NEP-21.");
  }

  if (!provider.supportedNetworks?.includes(expectedNetwork)) {
    throw new Error("This wallet does not support the required network.");
  }

  if (provider.network !== expectedNetwork) {
    throw new Error("The wallet is connected to a different network.");
  }
}
```

NEP-21 does not define a standard `switchNetwork` method. If your dApp needs a network switch, use wallet-specific extensions only after detecting and documenting them.

## Read Accounts

Use `getAccounts()` to read the wallet accounts currently connected to the dApp.

```ts
const accounts = await provider.getAccounts();

if (accounts.length === 0) {
  throw new Error("No connected account is available.");
}

const account = accounts[0];
```

When a user action depends on a signer, re-read accounts near the action instead of relying only on cached state.

```ts
async function getCurrentAccount(provider: any) {
  const accounts = await provider.getAccounts();
  const account = accounts[0];

  if (!account) {
    throw new Error("Connect an account before continuing.");
  }

  return account;
}
```

## Subscribe to Wallet Changes

Wallet state can change outside the dApp. Subscribe to account and network changes when the page is active.

```ts
const onAccountChanged = (event: CustomEvent) => {
  const accounts = event.detail.accounts;
  console.log("Accounts changed:", accounts);
};

const onNetworkChanged = (event: CustomEvent) => {
  const network = event.detail.network;
  console.log("Network changed:", network);
};

provider.on("accountchanged", onAccountChanged);
provider.on("networkchanged", onNetworkChanged);

// Later, during cleanup:
provider.removeListener("accountchanged", onAccountChanged);
provider.removeListener("networkchanged", onNetworkChanged);
```

## Read Token Data

Use `getTokenInfo` for basic token metadata and `getBalance` for account balances.

```ts
const GAS = "0xd2a4cff31913016155e38e474a2c06d08be276cf";
const account = await getCurrentAccount(provider);

const token = await provider.getTokenInfo(GAS);
const balance = await provider.getBalance(GAS, account.hash);

console.log(token.symbol, token.decimals, balance);
```

`Integer` values may be strings. Treat balances and supplies as arbitrary precision values.

## Call Contracts Offchain

Use `call` for read-only contract execution. It does not ask the user to sign a transaction.

```ts
const result = await provider.call({
  hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
  operation: "decimals",
  args: [],
});

if (result.state !== "HALT") {
  throw new Error(result.exception ?? "Contract call failed.");
}

console.log(result.stack[0]);
```

Use `abortOnFail` only for invocations that become transactions. It is most useful when the wallet builds an onchain transaction and should fail the whole transaction if a contract returns `false`.

## Send Assets

Use `send` for a simple asset transfer.

```ts
const GAS = "0xd2a4cff31913016155e38e474a2c06d08be276cf";
const account = await getCurrentAccount(provider);

const txid = await provider.send(
  GAS,
  account.hash,
  "0x1111111111111111111111111111111111111111",
  "100000000",
  { type: "Any" },
);

console.log("Transaction:", txid);
```

Pass the sender explicitly when possible. Although a wallet may help choose an account in some flows, explicit sender selection produces more predictable behavior across wallets.

## Invoke Contracts Onchain

Use `invoke` when the dApp needs one or more contract calls in an onchain transaction.

```ts
const account = await getCurrentAccount(provider);

const txid = await provider.invoke(
  [
    {
      hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
      operation: "transfer",
      args: [
        { type: "Hash160", value: account.hash },
        { type: "Hash160", value: "0x1111111111111111111111111111111111111111" },
        { type: "Integer", value: "100000000" },
        { type: "Any" },
      ],
      abortOnFail: true,
    },
  ],
  [
    {
      account: account.hash,
      scopes: "CalledByEntry",
    },
  ],
);

console.log("Transaction:", txid);
```

Prefer `CalledByEntry` as the default witness scope. Use broader scopes only when the contract flow requires them and the user can understand the risk.

## Build, Sign, and Relay Separately

Use `makeTransaction`, `sign`, and `relay` when the transaction needs a separate signing or broadcasting flow.

```ts
const context = await provider.makeTransaction(invocations, signers);
const signedContext = await provider.sign(context);
const txid = await provider.relay(signedContext);
```

This pattern is useful for multi-signature transactions, batched signing flows, and tools that preview or export transaction contexts before broadcast.

## Sign Messages

Use `signMessage` when the dApp needs a wallet-controlled signature outside a transaction.

```ts
const account = await getCurrentAccount(provider);

const signed = await provider.signMessage(
  "Sign in to example.app",
  account.hash,
  {
    isBase64Encoded: false,
    isTypedData: false,
    isLedgerCompatible: false,
  },
);

console.log(signed.account, signed.pubkey, signed.signature);
```

For login flows, prefer `authenticate` when your backend follows the NEP-20 authentication payload format.

## Handle Errors

Promise rejections should include `{ code, message, data? }`. Branch by `code`, not by wallet-specific text.

```ts
try {
  const txid = await provider.invoke(invocations, signers);
  console.log(txid);
} catch (error: any) {
  switch (error.code) {
    case 10006:
      // User canceled the wallet prompt.
      break;
    case 10007:
      // The account cannot cover the amount or fees.
      break;
    case 10008:
      // RPC server error.
      console.error(error.data);
      break;
    default:
      console.error(error);
      break;
  }
}
```

## Production Checklist

| Area | Checklist |
| --- | --- |
| Provider selection | Handle zero, one, or multiple compatible providers. |
| Compatibility | Check `dapiVersion` and `compatibility`. |
| Network | Check `provider.network` before signing or relaying. |
| Accounts | Re-read accounts before signer-sensitive actions. |
| Signers | Pass explicit `Signer[]` for contract invocations. |
| Witness scope | Start with `CalledByEntry`; avoid `Global` unless required. |
| Integers | Treat balances, fees, and supplies as arbitrary precision values. |
| Errors | Use numeric error codes for control flow. |
| Cleanup | Remove event listeners when the page or component unmounts. |

