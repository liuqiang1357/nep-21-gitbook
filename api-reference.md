# API Reference

This page documents the `IDapiProvider` interface defined by the current NEP-21 proposal.

## Provider

### `Neo.DapiProvider.ready`

Wallet providers dispatch this browser event when an `IDapiProvider` is ready. dApps listen for it and read the provider from `event.detail.provider`.

| Field | Type | Description |
| --- | --- | --- |
| `detail.provider` | `IDapiProvider` | Provider instance exposed by the wallet. |

```ts
window.addEventListener("Neo.DapiProvider.ready", (event) => {
  const provider = (event as CustomEvent).detail.provider;
});
```

### `Neo.DapiProvider.request`

dApps may dispatch this browser event to request a provider.

| Field | Type | Description |
| --- | --- | --- |
| `detail.version` | `Version` | dAPI version requested by the dApp. |

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.request", {
  detail: {
    version: "1.0",
  },
}));
```

If multiple providers respond, the dApp may choose which provider to use.

### Provider properties

NEP-21 exposes provider information as properties on the provider object.

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Provider name. |
| `version` | `Version` | Provider version. |
| `dapiVersion` | `Version` | dAPI version. It must currently be `"1.0"`. |
| `compatibility` | `string[]` | Supported standards. It should include `"NEP-21"` when supported. |
| `connected` | `boolean` | Whether the user has connected their wallet to the dApp. |
| `network` | `Network` | Current network. |
| `supportedNetworks` | `Network[]` | Networks supported by the provider. |
| `icon` | `string` | Provider icon URL. The URL scheme should be `https` or `data`. |
| `website` | `string` | Provider website. |
| `extra` | `any` | Additional provider data. |

```ts
console.log(provider.name);
console.log(provider.compatibility);
console.log(provider.network);
```

Example provider information:

```json
{
  "name": "Example Wallet",
  "version": "1.0.0",
  "dapiVersion": "1.0",
  "compatibility": ["NEP-21"],
  "connected": true,
  "network": 894710606,
  "supportedNetworks": [860833102, 894710606],
  "icon": "https://wallet.example/icon.png",
  "website": "https://wallet.example",
  "extra": {}
}
```

## Account and Authentication

### `authenticate(payload)`

Requests authentication. This is usually used to log in to a website. The authentication process is described in NEP-20.

| Parameter | Type | Description |
| --- | --- | --- |
| `payload` | `AuthenticationChallengePayload` | Authentication challenge payload. |

Returns `AuthenticationResponsePayload`.

Possible errors: `UNSUPPORTED`, `INVALID`, `TIMEOUT`, `CANCELED`.

```ts
const result = await provider.authenticate({
  action: "Authentication",
  grant_type: "Signature",
  allowed_algorithms: ["ECDSA-P256"],
  domain: "example.com",
  networks: [894710606],
  nonce: "nonce-value",
  timestamp: Date.now(),
});
```

Example response:

```json
{
  "algorithm": "ECDSA-P256",
  "network": 894710606,
  "pubkey": "03b209fd4f53a7170ea4444e0cb0a6bb6a53c2bd016926989cf85f9b0fba17a70c",
  "address": "NSiVJYZej4XsxG5CUpdwn7VRQk8iiiDMPM",
  "nonce": "nonce-value",
  "timestamp": 1710000000000,
  "signature": "base64-signature"
}
```

### `getAccounts()`

Gets the current connected account in the wallet.

Parameters: none.

Returns `Account[]`.

```ts
const accounts = await provider.getAccounts();
```

Example response:

```json
[
  {
    "hash": "0x682cca3ebdc66210e5847d7f8115846586079d4a",
    "address": "NSiVJYZej4XsxG5CUpdwn7VRQk8iiiDMPM",
    "label": "Account 1"
  }
]
```

### `pickAddress(prompt?)`

Prompts the user to select an account from the wallet and returns the selected address.

| Parameter | Type | Description |
| --- | --- | --- |
| `prompt` | `string` | Optional prompt shown to the user. |

Returns `Address`.

Possible error: `CANCELED`.

```ts
const address = await provider.pickAddress("Select an address");
```

## Read Methods

### `getBalance(asset, account)`

Gets the balance of the specified account. The account can be in the wallet or an arbitrary address.

| Parameter | Type | Description |
| --- | --- | --- |
| `asset` | `UInt160` | Asset contract hash. |
| `account` | `UInt160` | Account script hash. |

Returns `Integer`.

Possible errors: `INVALID`, `NOTFOUND`, `FAILED`, `RPC_ERROR`.

```ts
const balance = await provider.getBalance(
  "0xd2a4cff31913016155e38e474a2c06d08be276cf",
  "0x682cca3ebdc66210e5847d7f8115846586079d4a",
);
```

### `getTokenInfo(hash)`

Gets information about a token.

| Parameter | Type | Description |
| --- | --- | --- |
| `hash` | `UInt160` | Token contract hash. |

Returns `Token`.

Possible errors: `INVALID`, `NOTFOUND`, `FAILED`, `RPC_ERROR`.

```ts
const token = await provider.getTokenInfo("0xd2a4cff31913016155e38e474a2c06d08be276cf");
```

### `call(invocation)`

Calls a contract offchain and returns the execution result.

| Parameter | Type | Description |
| --- | --- | --- |
| `invocation` | `InvocationArguments` | Contract call arguments. |

Returns `InvocationResult`.

Possible errors: `INVALID`, `RPC_ERROR`.

```ts
const result = await provider.call({
  hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
  operation: "symbol",
  args: [],
});
```

### `getBlock(hash)`

Gets a block by hash.

| Parameter | Type | Description |
| --- | --- | --- |
| `hash` | `UInt256` | Block hash. |

Returns `Block`.

Possible errors: `INVALID`, `NOTFOUND`, `RPC_ERROR`.

```ts
const block = await provider.getBlock("0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa");
```

### `getBlock(index)`

Gets a block by index.

| Parameter | Type | Description |
| --- | --- | --- |
| `index` | `number` | Block index. |

Returns `Block`.

Possible errors: `INVALID`, `NOTFOUND`, `RPC_ERROR`.

```ts
const block = await provider.getBlock(1000);
```

### `getBlockCount()`

Gets the count of blocks in the blockchain.

Parameters: none.

Returns `number`.

Possible error: `RPC_ERROR`.

```ts
const count = await provider.getBlockCount();
```

### `getTransaction(txid)`

Gets a transaction by hash.

| Parameter | Type | Description |
| --- | --- | --- |
| `txid` | `UInt256` | Transaction hash. |

Returns `Transaction`.

Possible errors: `INVALID`, `NOTFOUND`, `RPC_ERROR`.

```ts
const tx = await provider.getTransaction("0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15");
```

### `getApplicationLog(txid)`

Gets the application log of a transaction.

| Parameter | Type | Description |
| --- | --- | --- |
| `txid` | `UInt256` | Transaction hash. |

Returns `ApplicationLog`.

Possible errors: `INVALID`, `RPC_ERROR`.

```ts
const log = await provider.getApplicationLog("0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15");
```

### `getStorage(hash, key)`

Gets a storage entry.

| Parameter | Type | Description |
| --- | --- | --- |
| `hash` | `UInt160` | Contract hash. |
| `key` | `Base64Encoded` | Storage key. |

Returns `Base64Encoded`.

Possible errors: `INVALID`, `NOTFOUND`, `RPC_ERROR`.

```ts
const value = await provider.getStorage(
  "0x0123456789abcdef0123456789abcdef01234567",
  "dG9rZW4w",
);
```

## Write and Signing Methods

### `send(asset, from, to, amount, data?)`

Sends assets to an account and returns the transaction hash.

| Parameter | Type | Description |
| --- | --- | --- |
| `asset` | `UInt160` | Asset contract hash. |
| `from` | `UInt160` | Sender account script hash. |
| `to` | `UInt160` | Recipient account script hash. |
| `amount` | `Integer` | Amount to send. |
| `data` | `Argument` | Optional transfer data. |

Returns `UInt256`.

Possible errors: `INVALID`, `NOTFOUND`, `FAILED`, `TIMEOUT`, `CANCELED`, `INSUFFICIENT_FUNDS`, `RPC_ERROR`.

```ts
const txid = await provider.send(
  "0xd2a4cff31913016155e38e474a2c06d08be276cf",
  "0x682cca3ebdc66210e5847d7f8115846586079d4a",
  "0x1111111111111111111111111111111111111111",
  "100000000",
  { type: "Any" },
);
```

### `invoke(invocations, signers?, attributes?, options?)`

Calls one or more contracts onchain and returns the transaction hash.

| Parameter | Type | Description |
| --- | --- | --- |
| `invocations` | `InvocationArguments[]` | Contract calls. |
| `signers` | `Signer[]` | Optional transaction signers. |
| `attributes` | `TransactionAttribute[]` | Optional transaction attributes. |
| `options` | `TransactionOptions` | Optional transaction options. |

Returns `UInt256`.

Possible errors: `INVALID`, `FAILED`, `TIMEOUT`, `CANCELED`, `RPC_ERROR`.

```ts
const txid = await provider.invoke([
  {
    hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
    operation: "transfer",
    args: [
      { type: "Hash160", value: "0x682cca3ebdc66210e5847d7f8115846586079d4a" },
      { type: "Hash160", value: "0x1111111111111111111111111111111111111111" },
      { type: "Integer", value: "100000000" },
      { type: "Any" }
    ],
    abortOnFail: true
  }
]);
```

### `makeTransaction(invocations, signers?, attributes?, options?)`

Calls one or more contracts onchain and returns the transaction without relaying it.

| Parameter | Type | Description |
| --- | --- | --- |
| `invocations` | `InvocationArguments[]` | Contract calls. |
| `signers` | `Signer[]` | Optional transaction signers. |
| `attributes` | `TransactionAttribute[]` | Optional transaction attributes. |
| `options` | `TransactionOptions` | Optional transaction options. |

Returns `ContractParametersContext`.

Possible errors: `INVALID`, `FAILED`, `TIMEOUT`, `CANCELED`, `RPC_ERROR`.

```ts
const GAS = "0xd2a4cff31913016155e38e474a2c06d08be276cf";
const sender = "0x682cca3ebdc66210e5847d7f8115846586079d4a";
const recipient = "0x1111111111111111111111111111111111111111";

const context = await provider.makeTransaction(
  [
    {
      hash: GAS,
      operation: "transfer",
      args: [
        { type: "Hash160", value: sender },
        { type: "Hash160", value: recipient },
        { type: "Integer", value: "100000000" },
        { type: "Any" }
      ],
      abortOnFail: true
    }
  ],
  [
    {
      account: sender,
      scopes: "CalledByEntry"
    }
  ]
);
```

### `sign(context)`

Signs a transaction with the current wallet. This is usually used for multi-signature transactions.

| Parameter | Type | Description |
| --- | --- | --- |
| `context` | `ContractParametersContext` | Transaction context. |

Returns `ContractParametersContext`.

Possible errors: `UNSUPPORTED`, `INVALID`, `NOTFOUND`, `TIMEOUT`, `CANCELED`.

```ts
// context is a ContractParametersContext returned by makeTransaction.
const signedContext = await provider.sign(context);
```

### `signMessage(message, account?, options?)`

Signs a message with the specified account. The algorithm is ECDSA with SHA-256.

| Parameter | Type | Description |
| --- | --- | --- |
| `message` | `string \| Base64Encoded` | Message to sign. |
| `account` | `UInt160` | Optional account script hash. |
| `options` | `SignOptions` | Optional signing options. |

Returns `SignedMessage`.

Possible errors: `INVALID`, `NOTFOUND`, `TIMEOUT`, `CANCELED`.

```ts
const signed = await provider.signMessage(
  "hello",
  "0x682cca3ebdc66210e5847d7f8115846586079d4a",
  { isBase64Encoded: false },
);
```

### `relay(context)`

Relays a transaction and returns the transaction hash.

| Parameter | Type | Description |
| --- | --- | --- |
| `context` | `ContractParametersContext` | Transaction context. |

Returns `UInt256`.

Possible errors: `INVALID`, `TIMEOUT`, `CANCELED`, `INSUFFICIENT_FUNDS`, `RPC_ERROR`.

```ts
// signedContext is a signed ContractParametersContext.
const txid = await provider.relay(signedContext);
```

## Event Methods

### `on(event, listener)`

Adds an event handler.

| Parameter | Type | Description |
| --- | --- | --- |
| `event` | `EventName` | `"accountchanged"` or `"networkchanged"`. |
| `listener` | `(e: CustomEvent) => void` | Event handler. |

```ts
provider.on("accountchanged", (event) => {
  console.log(event.detail.accounts);
});
```

### `removeListener(event, listener)`

Removes an event handler.

| Parameter | Type | Description |
| --- | --- | --- |
| `event` | `EventName` | `"accountchanged"` or `"networkchanged"`. |
| `listener` | `(e: CustomEvent) => void` | Event handler to remove. |

```ts
const onAccountChanged = (event: CustomEvent) => {
  console.log(event.detail.accounts);
};

provider.on("accountchanged", onAccountChanged);
provider.removeListener("accountchanged", onAccountChanged);
```
