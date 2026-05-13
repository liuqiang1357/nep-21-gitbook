# API Reference

The NEP-21 provider interface is exposed as `IDapiProvider`.

```ts
interface IDapiProvider {
  name: string;
  version: Version;
  dapiVersion: Version;
  compatibility: string[];
  connected: boolean;
  network: Network;
  supportedNetworks: Network[];
  icon: string;
  website: string;
  extra: any;

  on(event: EventName, listener: (e: CustomEvent) => void): void;
  removeListener(event: EventName, listener: (e: CustomEvent) => void): void;

  authenticate(payload: AuthenticationChallengePayload): Promise<AuthenticationResponsePayload>;
  getAccounts(): Promise<Account[]>;
  pickAddress(prompt?: string): Promise<Address>;
  getBalance(asset: UInt160, account: UInt160): Promise<Integer>;
  send(asset: UInt160, from: UInt160, to: UInt160, amount: Integer, data?: Argument): Promise<UInt256>;
  call(invocation: InvocationArguments): Promise<InvocationResult>;
  invoke(invocations: InvocationArguments[], signers?: Signer[], attributes?: TransactionAttribute[], options?: TransactionOptions): Promise<UInt256>;
  makeTransaction(invocations: InvocationArguments[], signers?: Signer[], attributes?: TransactionAttribute[], options?: TransactionOptions): Promise<ContractParametersContext>;
  sign(context: ContractParametersContext): Promise<ContractParametersContext>;
  signMessage(message: string | Base64Encoded, account?: UInt160, options?: SignOptions): Promise<SignedMessage>;
  relay(context: ContractParametersContext): Promise<UInt256>;
  getBlock(hash: UInt256): Promise<Block>;
  getBlock(index: number): Promise<Block>;
  getBlockCount(): Promise<number>;
  getTransaction(txid: UInt256): Promise<Transaction>;
  getApplicationLog(txid: UInt256): Promise<ApplicationLog>;
  getStorage(hash: UInt160, key: Base64Encoded): Promise<Base64Encoded>;
  getTokenInfo(hash: UInt160): Promise<Token>;
}
```

## Provider Discovery

### `Neo.DapiProvider.ready`

Wallets dispatch this event when a provider is ready.

| Field | Type | Description |
| --- | --- | --- |
| `detail.provider` | `IDapiProvider` | Provider instance. |

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.ready", {
  detail: { provider },
}));
```

### `Neo.DapiProvider.request`

dApps dispatch this event to request a compatible provider.

| Field | Type | Description |
| --- | --- | --- |
| `detail.version` | `Version` | Expected dAPI version, currently `"1.0"`. |

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.request", {
  detail: { version: "1.0" },
}));
```

## Properties

| Property | Type | Description |
| --- | --- | --- |
| `name` | `string` | Provider name. |
| `version` | `Version` | Provider version. |
| `dapiVersion` | `Version` | dAPI version. Must currently be `"1.0"`. |
| `compatibility` | `string[]` | Supported standards. Compatible providers should include `"NEP-21"`. |
| `connected` | `boolean` | Whether the wallet is connected to the dApp. |
| `network` | `Network` | Active N3 network magic number. |
| `supportedNetworks` | `Network[]` | Networks supported by the provider. |
| `icon` | `string` | Wallet icon URL. Use `https` or `data` URLs. |
| `website` | `string` | Wallet website URL. |
| `extra` | `any` | Wallet-specific extension data. |

## Events

### `on(event, listener)`

Registers a provider event listener.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `event` | `EventName` | Yes | `"accountchanged"` or `"networkchanged"`. |
| `listener` | `(e: CustomEvent) => void` | Yes | Listener function. |

```ts
provider.on("networkchanged", (event: CustomEvent) => {
  console.log(event.detail.network);
});
```

### `removeListener(event, listener)`

Removes a provider event listener.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `event` | `EventName` | Yes | `"accountchanged"` or `"networkchanged"`. |
| `listener` | `(e: CustomEvent) => void` | Yes | Listener previously registered with `on`. |

```ts
provider.removeListener("networkchanged", onNetworkChanged);
```

## Account and Authentication Methods

### `authenticate(payload)`

Requests an authentication signature.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `payload` | `AuthenticationChallengePayload` | Yes | Authentication challenge. |

Returns `AuthenticationResponsePayload`.

Possible errors: `UNSUPPORTED`, `INVALID`, `TIMEOUT`, `CANCELED`.

```ts
const response = await provider.authenticate({
  action: "Authentication",
  grant_type: "Signature",
  allowed_algorithms: ["ECDSA-P256"],
  domain: window.location.host,
  networks: [894710606],
  nonce: crypto.randomUUID(),
  timestamp: Date.now(),
});
```

### `getAccounts()`

Gets the connected accounts.

Returns `Account[]`.

```ts
const accounts = await provider.getAccounts();
```

### `pickAddress(prompt?)`

Asks the user to choose an address.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `prompt` | `string` | No | Optional text displayed by the wallet. |

Returns `Address`.

Possible errors: `CANCELED`.

```ts
const address = await provider.pickAddress("Select a reward address.");
```

### `signMessage(message, account?, options?)`

Signs a message with ECDSA SHA-256.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `message` | `string \| Base64Encoded` | Yes | Message to sign. |
| `account` | `UInt160` | No | Account script hash. |
| `options` | `SignOptions` | No | Encoding and compatibility options. |

Returns `SignedMessage`.

Possible errors: `INVALID`, `NOTFOUND`, `TIMEOUT`, `CANCELED`.

```ts
const signed = await provider.signMessage(
  "Sign in to example.app",
  account.hash,
  { isBase64Encoded: false },
);
```

## Asset and Contract Methods

### `getBalance(asset, account)`

Gets an account balance for a token contract.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `asset` | `UInt160` | Yes | Token contract hash. |
| `account` | `UInt160` | Yes | Account script hash. |

Returns `Integer`.

Possible errors: `INVALID`, `NOTFOUND`, `FAILED`, `RPC_ERROR`.

```ts
const balance = await provider.getBalance(GAS, account.hash);
```

### `getTokenInfo(hash)`

Gets token metadata.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `hash` | `UInt160` | Yes | Token contract hash. |

Returns `Token`.

Possible errors: `INVALID`, `NOTFOUND`, `FAILED`, `RPC_ERROR`.

```ts
const token = await provider.getTokenInfo(GAS);
```

### `call(invocation)`

Executes a contract call offchain.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `invocation` | `InvocationArguments` | Yes | Contract call definition. |

Returns `InvocationResult`.

Possible errors: `INVALID`, `RPC_ERROR`.

```ts
const result = await provider.call({
  hash: GAS,
  operation: "symbol",
  args: [],
});
```

## Transaction Methods

### `send(asset, from, to, amount, data?)`

Sends assets and returns the transaction hash.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `asset` | `UInt160` | Yes | Token contract hash. |
| `from` | `UInt160` | Yes | Sender account script hash. |
| `to` | `UInt160` | Yes | Recipient account script hash. |
| `amount` | `Integer` | Yes | Transfer amount. |
| `data` | `Argument` | No | Optional transfer data. |

Returns `UInt256`.

Possible errors: `INVALID`, `NOTFOUND`, `FAILED`, `TIMEOUT`, `CANCELED`, `INSUFFICIENT_FUNDS`, `RPC_ERROR`.

```ts
const txid = await provider.send(
  GAS,
  account.hash,
  recipientHash,
  "100000000",
  { type: "Any" },
);
```

### `invoke(invocations, signers?, attributes?, options?)`

Builds, signs, and relays one or more contract invocations.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `invocations` | `InvocationArguments[]` | Yes | Contract calls. |
| `signers` | `Signer[]` | No | Transaction signers. |
| `attributes` | `TransactionAttribute[]` | No | Transaction attributes. |
| `options` | `TransactionOptions` | No | Fee and validity options. |

Returns `UInt256`.

Possible errors: `INVALID`, `FAILED`, `TIMEOUT`, `CANCELED`, `RPC_ERROR`.

```ts
const txid = await provider.invoke(invocations, [
  { account: account.hash, scopes: "CalledByEntry" },
]);
```

### `makeTransaction(invocations, signers?, attributes?, options?)`

Builds a transaction context without relaying it.

Returns `ContractParametersContext`.

Possible errors: `INVALID`, `FAILED`, `TIMEOUT`, `CANCELED`, `RPC_ERROR`.

```ts
const context = await provider.makeTransaction(invocations, signers);
```

### `sign(context)`

Signs a transaction context with the wallet.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `context` | `ContractParametersContext` | Yes | Transaction context. |

Returns `ContractParametersContext`.

Possible errors: `UNSUPPORTED`, `INVALID`, `NOTFOUND`, `TIMEOUT`, `CANCELED`.

```ts
const signedContext = await provider.sign(context);
```

### `relay(context)`

Relays a signed transaction context.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `context` | `ContractParametersContext` | Yes | Signed transaction context. |

Returns `UInt256`.

Possible errors: `INVALID`, `TIMEOUT`, `CANCELED`, `INSUFFICIENT_FUNDS`, `RPC_ERROR`.

```ts
const txid = await provider.relay(signedContext);
```

## Chain Data Methods

### `getBlock(hashOrIndex)`

Gets a block by hash or index.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `hashOrIndex` | `UInt256 \| number` | Yes | Block hash or block index. |

Returns `Block`.

Possible errors: `INVALID`, `NOTFOUND`, `RPC_ERROR`.

```ts
const block = await provider.getBlock(1_000_000);
```

### `getBlockCount()`

Gets the current block count.

Returns `number`.

Possible errors: `RPC_ERROR`.

```ts
const count = await provider.getBlockCount();
```

### `getTransaction(txid)`

Gets a transaction by hash.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `txid` | `UInt256` | Yes | Transaction hash. |

Returns `Transaction`.

Possible errors: `INVALID`, `NOTFOUND`, `RPC_ERROR`.

```ts
const transaction = await provider.getTransaction(txid);
```

### `getApplicationLog(txid)`

Gets the application log for a transaction.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `txid` | `UInt256` | Yes | Transaction hash. |

Returns `ApplicationLog`.

Possible errors: `INVALID`, `RPC_ERROR`.

```ts
const log = await provider.getApplicationLog(txid);
```

### `getStorage(hash, key)`

Gets a contract storage value.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `hash` | `UInt160` | Yes | Contract hash. |
| `key` | `Base64Encoded` | Yes | Storage key encoded as base64. |

Returns `Base64Encoded`.

Possible errors: `INVALID`, `NOTFOUND`, `RPC_ERROR`.

```ts
const value = await provider.getStorage(contractHash, "dG9rZW4w");
```
