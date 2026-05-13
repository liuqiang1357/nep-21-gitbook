# Wallet Providers

This guide describes how a wallet should expose a NEP-21 provider to dApps.

## Provider Lifecycle

A wallet should make the provider available as soon as the page can safely use it.

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.ready", {
  detail: { provider },
}));
```

The wallet should also listen for provider requests.

```ts
window.addEventListener("Neo.DapiProvider.request", (event) => {
  const version = (event as CustomEvent).detail?.version;

  if (version === "1.0") {
    window.dispatchEvent(new CustomEvent("Neo.DapiProvider.ready", {
      detail: { provider },
    }));
  }
});
```

Dispatching `ready` more than once is acceptable. dApps should deduplicate providers, and wallets should make repeated announcements idempotent.

## Provider Metadata

Expose stable metadata as properties on the provider object.

| Property | Requirement |
| --- | --- |
| `name` | Human-readable wallet name. |
| `version` | Wallet provider version. |
| `dapiVersion` | NEP-21 dAPI version. Use `"1.0"`. |
| `compatibility` | Include `"NEP-21"` and any other standards the wallet supports. |
| `connected` | Reflect whether the dApp is connected or authorized according to wallet policy. |
| `network` | Active N3 network magic number. |
| `supportedNetworks` | Network magic numbers the wallet can use. |
| `icon` | Square `https` or `data` URL image, preferably at least 128x128 px. |
| `website` | Wallet website URL. |
| `extra` | Wallet-specific metadata. Keep standard fields stable. |

Example:

```ts
const provider = {
  name: "Example Wallet",
  version: "1.0.0",
  dapiVersion: "1.0",
  compatibility: ["NEP-21"],
  connected: false,
  network: 894710606,
  supportedNetworks: [860833102, 894710606],
  icon: "https://wallet.example/icon.png",
  website: "https://wallet.example",
  extra: {},
};
```

## Connection Policy

NEP-21 exposes a `connected` property but does not define a standard `connect` method. Wallets should make their connection policy clear and consistent.

Recommended behavior:

| Situation | Recommendation |
| --- | --- |
| Read-only methods | Allow methods such as `call`, `getBlockCount`, and `getTokenInfo` when no account permission is needed. |
| Account access | Ask for permission before exposing account data if the wallet treats accounts as private. |
| Signing and transactions | Always require user confirmation. |
| Permission revoked | Update `connected` and fire `accountchanged` with the current account set. |

If the wallet adds a custom `connect` or `disconnect` method, expose it as an extension without changing the NEP-21 method behavior.

## Events

Implement `on` and `removeListener` for provider events.

```ts
provider.on("accountchanged", listener);
provider.removeListener("accountchanged", listener);
```

### `accountchanged`

Fire this event when the connected account set changes.

```ts
dispatchProviderEvent("accountchanged", {
  accounts,
});
```

### `networkchanged`

Fire this event when the active network changes.

```ts
dispatchProviderEvent("networkchanged", {
  network,
});
```

Use the same event payload shape every time. dApps should not need wallet-specific parsing to read `event.detail.accounts` or `event.detail.network`.

## Method Implementation Guidance

| Method | Wallet behavior |
| --- | --- |
| `authenticate` | Validate the challenge, show the requesting domain, ask for approval, and return a signed response. |
| `getAccounts` | Return the accounts currently connected to the dApp. |
| `pickAddress` | Let the user choose an address and return only the selected address. |
| `getBalance` | Support wallet accounts and arbitrary account script hashes. |
| `send` | Build, sign, and relay a transfer after user approval. |
| `call` | Execute offchain without requiring a signature. |
| `invoke` | Build, sign, and relay one or more contract calls after user approval. |
| `makeTransaction` | Build a transaction context without relaying it. |
| `sign` | Add wallet signatures to a transaction context. |
| `signMessage` | Sign an explicit message with ECDSA SHA-256. |
| `relay` | Relay a signed transaction context. |
| `getBlock` | Support both block hash and block index. |
| `getBlockCount` | Return the current chain block count. |
| `getTransaction` | Return transaction data in the standard shape. |
| `getApplicationLog` | Return VM executions, stack values, and notifications. |
| `getStorage` | Accept base64 keys and return base64 values. |
| `getTokenInfo` | Return `symbol`, `decimals`, and `totalSupply`. |

## User Confirmation

Before signing, show the user enough information to understand the request.

| Request | Show at minimum |
| --- | --- |
| Message signature | Domain, account, message or payload summary, and signing mode. |
| Authentication | Domain, network, account, nonce or session context, and timestamp. |
| Asset transfer | Asset, amount, sender, recipient, network, and fees. |
| Contract invocation | Contract hash, operation, signer, witness scope, network, and estimated fees. |
| Transaction relay | Transaction hash, network, signer set, and fee summary when available. |

Treat `Global` witness scope as high risk. If it is requested, show a stronger warning and require explicit approval.

## Error Responses

Reject promises with structured errors.

```ts
throw {
  code: 10006,
  message: "The user canceled the request.",
};
```

Use numeric codes consistently.

| Code | Name | Use when |
| --- | --- | --- |
| `10000` | `UNKNOWN` | No more specific code applies. |
| `10001` | `UNSUPPORTED` | The method or option is not supported. |
| `10002` | `INVALID` | Input is malformed or semantically invalid. |
| `10003` | `NOTFOUND` | Account, contract, block, transaction, or storage was not found. |
| `10004` | `FAILED` | Contract execution failed. Include the invocation result in `data` when possible. |
| `10005` | `TIMEOUT` | The operation timed out. |
| `10006` | `CANCELED` | The user rejected or closed the wallet prompt. |
| `10007` | `INSUFFICIENT_FUNDS` | Balance cannot cover amount or fees. |
| `10008` | `RPC_ERROR` | The RPC server returned an exception. |

Avoid relying on localized `message` text for machine-readable behavior. Put structured details in `data`.

## Data Consistency

| Area | Recommendation |
| --- | --- |
| Hashes | Use `0x`-prefixed hexadecimal strings for `UInt160` and `UInt256`. |
| Integers | Accept and return strings for values that may exceed JavaScript safe integer limits. |
| Base64 | Keep `Base64Encoded` fields base64 encoded, not hex encoded. |
| Network | Make sure every transaction context includes the correct `network`. |
| Account changes | Keep `getAccounts()` and `accountchanged` payloads consistent. |
| Extensions | Put wallet-specific data under `extra` or clearly documented extension methods. |

## Provider Test Plan

Before publishing a provider, test these flows against at least one sample dApp:

| Flow | Expected result |
| --- | --- |
| Provider request | The provider responds to `Neo.DapiProvider.request`. |
| Provider metadata | `dapiVersion`, `compatibility`, `network`, and `supportedNetworks` are correct. |
| Account read | `getAccounts()` returns the same shape as `accountchanged`. |
| Network switch | `networkchanged` fires and `provider.network` updates. |
| Offchain call | `call` returns `HALT`, `FAULT`, stack, notifications, and exception data correctly. |
| Transfer | `send` prompts the user and returns a transaction hash after relay. |
| Invocation | `invoke` handles signers, witness scopes, attributes, and fee options. |
| Multisig | `makeTransaction`, `sign`, and `relay` work as separate steps. |
| Error mapping | User rejection, invalid input, contract failure, timeout, and RPC errors return stable codes. |

