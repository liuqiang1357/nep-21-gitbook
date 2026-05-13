# API Reference

This page documents the NEP-21 `IDapiProvider` interface in a method-by-method format.

## Provider Discovery

dApps obtain a provider instance through browser events.

### `Neo.DapiProvider.ready`

Wallet providers must dispatch this event when a compatible provider is ready.

#### Event Detail

| Parameter | Type | Description |
| --- | --- | --- |
| `provider` | `IDapiProvider` | Provider instance exposed by the wallet. |

#### Wallet Example

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.ready", {
  detail: {
    provider,
  },
}));
```

#### dApp Example

```ts
window.addEventListener("Neo.DapiProvider.ready", (event) => {
  const provider = (event as CustomEvent).detail.provider;
  console.log("NEP-21 provider ready:", provider.name);
});
```

### `Neo.DapiProvider.request`

dApps may dispatch this event to ask wallets or extensions to inject or announce a provider.

#### Event Detail

| Parameter | Type | Description |
| --- | --- | --- |
| `version` | `Version` | dAPI version expected by the dApp. For NEP-21, use `"1.0"`. |

#### dApp Example

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.request", {
  detail: {
    version: "1.0",
  },
}));
```

#### Wallet Example

```ts
window.addEventListener("Neo.DapiProvider.request", (event) => {
  const version = (event as CustomEvent).detail?.version;
  if (version === "1.0") {
    // Inject provider or dispatch Neo.DapiProvider.ready.
  }
});
```

## Provider Metadata

NEP-21 exposes provider metadata as properties on the provider object.

### Input Arguments

None.

### Success Response

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `string` | Provider name. |
| `version` | `Version` | Provider version. |
| `dapiVersion` | `Version` | dAPI version. It must currently be `"1.0"`. |
| `compatibility` | `string[]` | Supported standards. Should include `"NEP-21"` when compatible. |
| `connected` | `boolean` | Whether the user has connected the wallet to the dApp. |
| `network` | `Network` | Current N3 network magic number. |
| `supportedNetworks` | `Network[]` | Networks supported by the provider. |
| `icon` | `string` | Provider icon URL. Should use `https` or `data` URL. |
| `website` | `string` | Provider website. |
| `extra` | `any` | Provider-specific extension data. |

### Error Response

None.

### Example

```ts
const provider = await waitForNeoDapiProvider();

const {
  name,
  version,
  dapiVersion,
  compatibility,
  connected,
  network,
  supportedNetworks,
  icon,
  website,
  extra,
} = provider;

console.log("Provider name:", name);
console.log("Provider version:", version);
console.log("dAPI version:", dapiVersion);
console.log("Compatibility:", compatibility);
console.log("Connected:", connected);
console.log("Network:", network);
console.log("Supported networks:", supportedNetworks);
console.log("Icon:", icon);
console.log("Website:", website);
console.log("Extra:", extra);
```

### Example Response

```json
{
  "name": "Example Wallet",
  "version": "1.2.0",
  "dapiVersion": "1.0",
  "compatibility": ["NEP-11", "NEP-17", "NEP-21"],
  "connected": true,
  "network": 860833102,
  "supportedNetworks": [860833102, 894710606],
  "icon": "https://example.com/icon.png",
  "website": "https://example.com",
  "extra": {}
}
```

## Event Methods

### `on(event, listener)`

Adds an event handler for a provider event.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `event` | `EventName` | Event name. Supported values: `"accountchanged"`, `"networkchanged"`. |
| `listener` | `(e: CustomEvent) => void` | Event handler. |

#### Success Response

None.

#### Error Response

None defined by the spec.

#### Example

```ts
provider.on("accountchanged", (event: CustomEvent) => {
  console.log(event.detail.accounts);
});

provider.on("networkchanged", (event: CustomEvent) => {
  console.log(event.detail.network);
});
```

### `removeListener(event, listener)`

Removes a previously registered event handler.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `event` | `EventName` | Event name. Supported values: `"accountchanged"`, `"networkchanged"`. |
| `listener` | `(e: CustomEvent) => void` | Event handler to remove. |

#### Success Response

None.

#### Error Response

None defined by the spec.

#### Example

```ts
const onNetworkChanged = (event: CustomEvent) => {
  console.log(event.detail.network);
};

provider.on("networkchanged", onNetworkChanged);
provider.removeListener("networkchanged", onNetworkChanged);
```

## Methods

### `authenticate(payload)`

Requests authentication. This is usually used to log in to a website. The authentication flow is described by NEP-20.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `payload` | `AuthenticationChallengePayload` | Authentication challenge provided by the dApp or backend. |

#### Success Response

| Parameter | Type | Description |
| --- | --- | --- |
| `algorithm` | `"ECDSA-P256"` | Signature algorithm. |
| `network` | `Network` | Network used for the authentication response. |
| `pubkey` | `ECPoint` | Public key used to sign the response. |
| `address` | `Address` | Address associated with the signer. |
| `nonce` | `string` | Nonce from the challenge. |
| `timestamp` | `number` | Response timestamp. |
| `signature` | `Base64Encoded` | Base64-encoded signature. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10001` | `UNSUPPORTED` | Authentication is not supported. |
| `10002` | `INVALID` | Payload is invalid. |
| `10005` | `TIMEOUT` | Request timed out. |
| `10006` | `CANCELED` | User canceled the request. |

#### Example

```ts
const response = await provider.authenticate({
  action: "Authentication",
  grant_type: "Signature",
  allowed_algorithms: ["ECDSA-P256"],
  domain: window.location.host,
  networks: [860833102],
  nonce: crypto.randomUUID(),
  timestamp: Date.now(),
});

console.log("Authenticated address:", response.address);
```

#### Example Response

```json
{
  "algorithm": "ECDSA-P256",
  "network": 860833102,
  "pubkey": "03b209fd4f53a7170ea4444e0cb0a6bb6a53c2bd016926989cf85f9b0fba17a70c",
  "address": "NSiVJYZej4XsxG5CUpdwn7VRQk8iiiDMPM",
  "nonce": "b80fbb9e-1a39-4fb2-90f6-a431a57b1854",
  "timestamp": 1778640000000,
  "signature": "base64-signature"
}
```

### `getAccounts()`

Gets the current connected accounts in the wallet.

#### Input Arguments

None.

#### Success Response

| Type | Description |
| --- | --- |
| `Account[]` | Connected wallet accounts. |

#### Error Response

No method-specific errors are listed by the spec.

#### Example

```ts
const accounts = await provider.getAccounts();

for (const account of accounts) {
  console.log(account.address, account.hash, account.label);
}
```

#### Example Response

```json
[
  {
    "hash": "0x682cca3ebdc66210e5847d7f8115846586079d4a",
    "address": "NSiVJYZej4XsxG5CUpdwn7VRQk8iiiDMPM",
    "label": "Main Account",
    "extra": {}
  }
]
```

### `pickAddress(prompt?)`

Prompts the user to select an address from the wallet and returns the selected address.

#### Input Arguments

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `prompt` | `string` | No | Message shown by the wallet when asking the user to select an address. |

#### Success Response

| Type | Description |
| --- | --- |
| `Address` | Selected N3 address. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10006` | `CANCELED` | User canceled the address selection. |

#### Example

```ts
const address = await provider.pickAddress("Select an address to receive rewards.");
console.log("Selected address:", address);
```

#### Example Response

```json
"NSiVJYZej4XsxG5CUpdwn7VRQk8iiiDMPM"
```

### `getBalance(asset, account)`

Gets the balance of an asset for an account. The account can be in the wallet or an arbitrary address represented by script hash.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `asset` | `UInt160` | Asset contract script hash. |
| `account` | `UInt160` | Account script hash. |

#### Success Response

| Type | Description |
| --- | --- |
| `Integer` | Balance as a number or string. Use string handling for large values. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Asset or account hash is invalid. |
| `10003` | `NOTFOUND` | Asset or account data was not found. |
| `10004` | `FAILED` | Contract execution failed. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const GAS = "0xd2a4cff31913016155e38e474a2c06d08be276cf";
const [account] = await provider.getAccounts();

const balance = await provider.getBalance(GAS, account.hash);
console.log("GAS balance:", balance);
```

#### Example Response

```json
"100000000"
```

### `send(asset, from, to, amount, data?)`

Sends an asset to an account and returns the transaction hash.

The `from` account must be in the wallet. The `to` account can be in the wallet or an arbitrary account. The spec text says the wallet may select a sender when `from` is not specified, but the current interface defines `from` as a positional parameter. dApps should pass `from` explicitly for consistent cross-wallet behavior.

#### Input Arguments

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `asset` | `UInt160` | Yes | Asset contract script hash. |
| `from` | `UInt160` | Yes | Sender account script hash. |
| `to` | `UInt160` | Yes | Recipient account script hash. |
| `amount` | `Integer` | Yes | Amount to send. |
| `data` | `Argument` | No | Additional data for the transfer. |

#### Success Response

| Type | Description |
| --- | --- |
| `UInt256` | Transaction hash. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Input is invalid. |
| `10003` | `NOTFOUND` | Asset or account was not found. |
| `10004` | `FAILED` | Contract execution failed. |
| `10005` | `TIMEOUT` | Request timed out. |
| `10006` | `CANCELED` | User canceled the request. |
| `10007` | `INSUFFICIENT_FUNDS` | Sender balance is insufficient. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const GAS = "0xd2a4cff31913016155e38e474a2c06d08be276cf";
const [account] = await provider.getAccounts();

const txid = await provider.send(
  GAS,
  account.hash,
  "0x1111111111111111111111111111111111111111",
  "100000000",
  { type: "Any" },
);

console.log("Transaction hash:", txid);
```

#### Example Response

```json
"0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15"
```

### `call(invocation)`

Calls a contract offchain and returns the execution result.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `invocation` | `InvocationArguments` | Contract call definition. |

#### Success Response

| Type | Description |
| --- | --- |
| `InvocationResult` | VM execution result. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Invocation is invalid. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const result = await provider.call({
  hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
  operation: "decimals",
  args: [],
});

console.log(result.stack[0]);
```

#### Example Response

```json
{
  "script": "base64-script",
  "state": "HALT",
  "gasconsumed": "984060",
  "notifications": [],
  "stack": [
    {
      "type": "Integer",
      "value": "8"
    }
  ]
}
```

### `invoke(invocations, signers?, attributes?, options?)`

Calls one or more contracts onchain and returns the transaction hash.

#### Input Arguments

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `invocations` | `InvocationArguments[]` | Yes | Contract calls to include in the transaction. |
| `signers` | `Signer[]` | No | Transaction signers and witness scopes. |
| `attributes` | `TransactionAttribute[]` | No | Transaction attributes. |
| `options` | `TransactionOptions` | No | Fee and validity options. |

#### Success Response

| Type | Description |
| --- | --- |
| `UInt256` | Transaction hash. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Invocation, signer, attribute, or option is invalid. |
| `10004` | `FAILED` | Contract execution failed. |
| `10005` | `TIMEOUT` | Request timed out. |
| `10006` | `CANCELED` | User canceled the request. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const txid = await provider.invoke(
  [
    {
      hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
      operation: "transfer",
      args: [
        { type: "Hash160", value: "0x682cca3ebdc66210e5847d7f8115846586079d4a" },
        { type: "Hash160", value: "0x1111111111111111111111111111111111111111" },
        { type: "Integer", value: "100000000" },
        { type: "Any" },
      ],
      abortOnFail: true,
    },
  ],
  [
    {
      account: "0x682cca3ebdc66210e5847d7f8115846586079d4a",
      scopes: "CalledByEntry",
    },
  ],
  [],
  {
    extraSystemFee: "1000000",
  },
);

console.log("Transaction hash:", txid);
```

#### Example Response

```json
"0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15"
```

### `makeTransaction(invocations, signers?, attributes?, options?)`

Calls one or more contracts onchain and returns a transaction context without relaying it.

This is useful for multi-signature flows or applications that need to collect signatures before broadcasting.

#### Input Arguments

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `invocations` | `InvocationArguments[]` | Yes | Contract calls to include in the transaction. |
| `signers` | `Signer[]` | No | Transaction signers and witness scopes. |
| `attributes` | `TransactionAttribute[]` | No | Transaction attributes. |
| `options` | `TransactionOptions` | No | Fee and validity options. |

#### Success Response

| Type | Description |
| --- | --- |
| `ContractParametersContext` | Transaction context to sign or relay later. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Invocation, signer, attribute, or option is invalid. |
| `10004` | `FAILED` | Contract execution failed. |
| `10005` | `TIMEOUT` | Request timed out. |
| `10006` | `CANCELED` | User canceled the request. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const context = await provider.makeTransaction(
  [
    {
      hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
      operation: "transfer",
      args: [
        { type: "Hash160", value: "0x682cca3ebdc66210e5847d7f8115846586079d4a" },
        { type: "Hash160", value: "0x1111111111111111111111111111111111111111" },
        { type: "Integer", value: "100000000" },
        { type: "Any" },
      ],
      abortOnFail: true,
    },
  ],
  [{ account: "0x682cca3ebdc66210e5847d7f8115846586079d4a", scopes: "CalledByEntry" }],
);

console.log("Unsigned transaction context:", context.hash);
```

#### Example Response

```json
{
  "type": "Neo.Network.P2P.Payloads.Transaction",
  "hash": "0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15",
  "data": "base64-transaction",
  "items": {
    "0x682cca3ebdc66210e5847d7f8115846586079d4a": {
      "script": "base64-verification-script",
      "parameters": [],
      "signatures": {}
    }
  },
  "network": 860833102
}
```

### `sign(context)`

Signs a transaction context with the current wallet. This is usually used for multi-signature transactions.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `context` | `ContractParametersContext` | Transaction context to sign. |

#### Success Response

| Type | Description |
| --- | --- |
| `ContractParametersContext` | Transaction context containing the wallet's signatures. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10001` | `UNSUPPORTED` | Signing the context is not supported. |
| `10002` | `INVALID` | Context is invalid. |
| `10003` | `NOTFOUND` | Required account or signing data was not found. |
| `10005` | `TIMEOUT` | Request timed out. |
| `10006` | `CANCELED` | User canceled the request. |

#### Example

```ts
const signedContext = await provider.sign(context);
console.log("Signed context:", signedContext);
```

#### Example Response

```json
{
  "type": "Neo.Network.P2P.Payloads.Transaction",
  "hash": "0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15",
  "data": "base64-transaction",
  "items": {
    "0x682cca3ebdc66210e5847d7f8115846586079d4a": {
      "script": "base64-verification-script",
      "parameters": [],
      "signatures": {
        "03b209fd4f53a7170ea4444e0cb0a6bb6a53c2bd016926989cf85f9b0fba17a70c": "base64-signature"
      }
    }
  },
  "network": 860833102
}
```

### `signMessage(message, account?, options?)`

Signs a message with the specified account. The algorithm is ECDSA with SHA-256.

If `account` is omitted, the wallet should automatically select an account or prompt the user to select one.

#### Input Arguments

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `message` | `string \| Base64Encoded` | Yes | Message to sign. |
| `account` | `UInt160` | No | Account script hash used for signing. |
| `options` | `SignOptions` | No | Message encoding and compatibility options. |

#### Success Response

| Parameter | Type | Description |
| --- | --- | --- |
| `payload` | `Base64Encoded` | Base64-encoded payload that was signed. |
| `signature` | `Base64Encoded` | Signature. |
| `account` | `UInt160` | Account used to sign the message. |
| `pubkey` | `ECPoint` | Public key of the signing account. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Message, account, or options are invalid. |
| `10003` | `NOTFOUND` | Account was not found. |
| `10005` | `TIMEOUT` | Request timed out. |
| `10006` | `CANCELED` | User canceled the request. |

#### Example

```ts
const [account] = await provider.getAccounts();

const signed = await provider.signMessage(
  "Sign in to example.com",
  account.hash,
  {
    isBase64Encoded: false,
    isTypedData: false,
    isLedgerCompatible: false,
  },
);

console.log("Signature:", signed.signature);
```

#### Example Response

```json
{
  "payload": "U2lnbiBpbiB0byBleGFtcGxlLmNvbQ==",
  "signature": "base64-signature",
  "account": "0x682cca3ebdc66210e5847d7f8115846586079d4a",
  "pubkey": "03b209fd4f53a7170ea4444e0cb0a6bb6a53c2bd016926989cf85f9b0fba17a70c"
}
```

### `relay(context)`

Relays a transaction context and returns the transaction hash.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `context` | `ContractParametersContext` | Signed transaction context to relay. |

#### Success Response

| Type | Description |
| --- | --- |
| `UInt256` | Transaction hash. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Context is invalid. |
| `10005` | `TIMEOUT` | Request timed out. |
| `10006` | `CANCELED` | User canceled the request. |
| `10007` | `INSUFFICIENT_FUNDS` | Transaction cannot be relayed due to insufficient balance. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const txid = await provider.relay(signedContext);
console.log("Relayed transaction:", txid);
```

#### Example Response

```json
"0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15"
```

### `getBlock(hashOrIndex)`

Gets a block by hash or index.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `hashOrIndex` | `UInt256 \| number` | Block hash or block index. |

#### Success Response

| Type | Description |
| --- | --- |
| `Block` | Block data. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Hash or index is invalid. |
| `10003` | `NOTFOUND` | Block was not found. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const block = await provider.getBlock(1000);
console.log(block.hash, block.tx.length);
```

#### Example Response

```json
{
  "hash": "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "size": 1200,
  "confirmations": 20,
  "version": 0,
  "previousBlockHash": "0xbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
  "merkleRoot": "0xcccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc",
  "time": 1778640000,
  "nonce": "0000000000000000",
  "index": 1000,
  "primary": 0,
  "nextConsensus": "0x682cca3ebdc66210e5847d7f8115846586079d4a",
  "tx": []
}
```

### `getBlockCount()`

Gets the number of blocks in the blockchain.

#### Input Arguments

None.

#### Success Response

| Type | Description |
| --- | --- |
| `number` | Current block count. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const blockCount = await provider.getBlockCount();
console.log("Block count:", blockCount);
```

#### Example Response

```json
1234567
```

### `getTransaction(txid)`

Gets a transaction by hash.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `txid` | `UInt256` | Transaction hash. |

#### Success Response

| Type | Description |
| --- | --- |
| `Transaction` | Transaction data. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Transaction hash is invalid. |
| `10003` | `NOTFOUND` | Transaction was not found. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const tx = await provider.getTransaction("0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15");
console.log(tx.sender, tx.signers);
```

#### Example Response

```json
{
  "hash": "0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15",
  "size": 256,
  "blockHash": "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "blockTime": 1778640000,
  "confirmations": 12,
  "version": 0,
  "nonce": 123456,
  "systemFee": "984060",
  "networkFee": "123456",
  "validUntilBlock": 1235000,
  "sender": "0x682cca3ebdc66210e5847d7f8115846586079d4a",
  "signers": [],
  "attributes": [],
  "script": "base64-script"
}
```

### `getApplicationLog(txid)`

Gets the application log for a transaction.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `txid` | `UInt256` | Transaction hash. |

#### Success Response

| Type | Description |
| --- | --- |
| `ApplicationLog` | Application execution log. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Transaction hash is invalid. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const log = await provider.getApplicationLog("0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15");
console.log(log.executions[0]?.vmstate);
```

#### Example Response

```json
{
  "txid": "0x1f4d1defa46faa5e7b9b8d3f79a06bec777d7c26c4aa5f6f5899a291daa87c15",
  "executions": [
    {
      "trigger": "Application",
      "vmstate": "HALT",
      "gasconsumed": "984060",
      "stack": [],
      "notifications": []
    }
  ]
}
```

### `getStorage(hash, key)`

Gets a storage entry from a contract.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `hash` | `UInt160` | Contract script hash. |
| `key` | `Base64Encoded` | Storage key encoded as base64. |

#### Success Response

| Type | Description |
| --- | --- |
| `Base64Encoded` | Storage value encoded as base64. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Contract hash or key is invalid. |
| `10003` | `NOTFOUND` | Storage entry was not found. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const value = await provider.getStorage(
  "0x0123456789abcdef0123456789abcdef01234567",
  "dG9rZW4w",
);

console.log("Storage value:", value);
```

#### Example Response

```json
"wYCMqLCTIUiax57E8Zd/O9xN3l8="
```

### `getTokenInfo(hash)`

Gets token metadata for a contract.

#### Input Arguments

| Parameter | Type | Description |
| --- | --- | --- |
| `hash` | `UInt160` | Token contract script hash. |

#### Success Response

| Parameter | Type | Description |
| --- | --- | --- |
| `symbol` | `string` | Token symbol. |
| `decimals` | `number` | Number of decimals. |
| `totalSupply` | `Integer` | Total supply. |

#### Error Response

| Code | Name | Description |
| --- | --- | --- |
| `10002` | `INVALID` | Contract hash is invalid. |
| `10003` | `NOTFOUND` | Token contract was not found. |
| `10004` | `FAILED` | Contract execution failed. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

#### Example

```ts
const token = await provider.getTokenInfo("0xd2a4cff31913016155e38e474a2c06d08be276cf");
console.log(token.symbol, token.decimals, token.totalSupply);
```

#### Example Response

```json
{
  "symbol": "GAS",
  "decimals": 8,
  "totalSupply": "10000000000000000"
}
```
