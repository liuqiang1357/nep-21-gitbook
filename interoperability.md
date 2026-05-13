# Interoperability

This page covers practical decisions that help dApps and wallets behave consistently across implementations.

## Keep the Provider Layer Thin

NEP-21 gives dApps wallet-mediated access to accounts, signing, transactions, and selected chain queries. For complex indexing, historical queries, analytics, or application-specific reads, dApps should still use dedicated RPC nodes, indexers, or SDKs.

Use the provider when the wallet is part of the trust or approval flow. Use normal application infrastructure when the user does not need to approve anything.

## Provider Selection

dApps should treat provider discovery as a multi-provider flow.

| Case | Recommended dApp behavior |
| --- | --- |
| No provider found | Show an install or connect-wallet state. |
| One compatible provider | Use it after validating compatibility and network. |
| Multiple compatible providers | Show a wallet selector. |
| Provider lacks `"NEP-21"` | Hide it from the NEP-21 flow or show it as unsupported. |

Wallets should avoid overwriting each other. Announce a provider through events and let the dApp decide which provider to use.

## Account State

Account state can change after a dApp loads. The user may switch accounts, lock the wallet, revoke permissions, or change the active account in the wallet UI.

dApps should:

| Practice | Reason |
| --- | --- |
| Re-read `getAccounts()` before signing. | Cached accounts can become stale. |
| Pass explicit signers to `invoke`. | The wallet can verify the requested signer. |
| Listen for `accountchanged`. | UI state should update when wallet state changes. |
| Handle empty account arrays. | A wallet may disconnect or revoke access. |

Wallets should:

| Practice | Reason |
| --- | --- |
| Keep `getAccounts()` and `accountchanged` payloads consistent. | dApps can use one data model. |
| Reject unavailable signers with `NOTFOUND` or `INVALID`. | dApps can show a precise recovery path. |
| Avoid silently replacing the requested signer. | Signing with a different account can create user-visible mistakes. |

## Network State

NEP-21 exposes `network` and `networkchanged`, but it does not define a standard network switch method.

dApps should check `provider.network` before transaction requests. If the wallet is on the wrong network, show a clear message or use a wallet-specific switch method only when the wallet documents one.

Wallets should update `provider.network` before or at the same time as firing `networkchanged`.

## Transaction Intent

For transaction requests, dApps and wallets should preserve the same intent from UI to signature.

| Data | Recommended handling |
| --- | --- |
| Sender | dApps should pass it explicitly; wallets should verify it is available. |
| Recipient | Wallets should display both script hash and resolved address when possible. |
| Amount | Preserve integer precision. |
| Contract hash | Display the hash and any verified name the wallet knows. |
| Operation | Display the operation name. |
| Witness scope | Display the requested scope and warn for broad scopes. |
| Network | Display the active network. |
| Fees | Display estimated fees before confirmation. |

## `send` Parameters

The `send` method is positional:

```ts
send(asset, from, to, amount, data?)
```

For predictable behavior, dApps should pass `from` explicitly. Wallets may still offer account selection in their UI, but they should not sign with a different sender unless the dApp request allows that behavior and the user clearly approves it.

## Optional Wallet Extensions

Some wallets may expose additional methods such as `connect`, `disconnect`, `getNetwork`, or `switchNetwork`. These can improve UX, but they are not part of the base NEP-21 surface.

Recommended extension rules:

| Rule | Reason |
| --- | --- |
| Feature-detect extension methods. | The dApp remains compatible with standard providers. |
| Keep standard methods unchanged. | Other dApps can still rely on NEP-21 behavior. |
| Document extension errors separately. | Standard error handling stays predictable. |
| Keep extension data under `extra` when it is metadata. | Standard properties keep stable shapes. |

Example:

```ts
if (typeof provider.switchNetwork === "function") {
  await provider.switchNetwork(894710606);
}
```

## Error Mapping

Use numeric error codes for application control flow.

| Scenario | Recommended code |
| --- | --- |
| User rejects a prompt | `CANCELED` (`10006`) |
| Request times out | `TIMEOUT` (`10005`) |
| Unsupported method or option | `UNSUPPORTED` (`10001`) |
| Malformed hash, argument, signer, or context | `INVALID` (`10002`) |
| Requested account, contract, block, transaction, or storage is missing | `NOTFOUND` (`10003`) |
| Contract execution fails | `FAILED` (`10004`) |
| Balance cannot cover amount or fees | `INSUFFICIENT_FUNDS` (`10007`) |
| RPC server returns an exception | `RPC_ERROR` (`10008`) |

Use `data` for structured details. Avoid requiring dApps to parse localized messages.

## Versioning

Providers should expose:

```ts
provider.dapiVersion === "1.0";
provider.compatibility.includes("NEP-21");
```

dApps should reject unsupported major behavior early, before asking users to sign or send transactions.

## Release Checklist

| Reader | Before release |
| --- | --- |
| dApp developers | Test provider discovery, account changes, network changes, transaction approval, user rejection, and RPC failures. |
| Wallet providers | Test against multiple dApps and verify every method rejects with structured errors. |
| SDK authors | Keep type definitions aligned with the standard and avoid wallet-specific assumptions in shared types. |

