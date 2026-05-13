# Wallet Provider Checklist

Use this checklist when implementing a NEP-21-compatible wallet provider.

## Provider Injection

| Item | Requirement |
| --- | --- |
| Ready event | Dispatch `Neo.DapiProvider.ready` when the provider is ready. |
| Request event | Listen for `Neo.DapiProvider.request` and respond when the requested dAPI version is supported. |
| Multiple wallets | If more than one provider exists, allow the dApp to choose from compatible providers. |
| Compatibility | Include `"NEP-21"` in `provider.compatibility`. |

## Provider Properties

| Property | Requirement |
| --- | --- |
| `name` | Human-readable wallet name. |
| `version` | Wallet provider version. |
| `dapiVersion` | Must currently be `"1.0"`. |
| `compatibility` | Include all supported standards, for example `["NEP-11", "NEP-17", "NEP-21"]`. |
| `connected` | Reflect whether the user has connected the wallet to the dApp. |
| `network` | Reflect the active network. |
| `supportedNetworks` | List supported network magic numbers. |
| `icon` | Use a square `https` or `data` URL image, at least 128x128 px. |
| `website` | Wallet website URL. |
| `extra` | Provider-specific metadata. Keep it optional for dApps. |

## Events

| Event | Requirement |
| --- | --- |
| `accountchanged` | Fire when the connected account set changes. Include `{ accounts: Account[] }` in `detail`. |
| `networkchanged` | Fire when the active network changes. Include `{ network: Network }` in `detail`. |
| Listener removal | `removeListener` must remove exactly the listener registered by `on`. |

## Method Behavior

| Method | Implementation notes |
| --- | --- |
| `authenticate` | Validate the challenge payload and ask the user to approve authentication. |
| `getAccounts` | Return currently connected accounts. Keep account fields stable across calls. |
| `pickAddress` | Show a clear account selection prompt and return only the selected address. |
| `getBalance` | Support wallet accounts and arbitrary account script hashes. |
| `send` | Require user confirmation before signing and relaying. Prefer explicit `from` handling. |
| `call` | Execute offchain and return the VM result without asking for signing. |
| `invoke` | Build, sign, and relay an onchain transaction after user approval. |
| `makeTransaction` | Build a transaction context without relaying. |
| `sign` | Add wallet signatures to a transaction context. |
| `signMessage` | Sign with ECDSA SHA-256 and return payload, signature, account, and public key. |
| `relay` | Relay a signed transaction context and return the transaction hash. |
| `getBlock` | Support both hash and index overloads. |
| `getBlockCount` | Return the current chain height/count from the configured network. |
| `getTransaction` | Return transaction data matching the NEP-21 transaction type. |
| `getApplicationLog` | Return application log executions, stack values, and notifications. |
| `getStorage` | Accept base64 storage keys and return base64 values. |
| `getTokenInfo` | Return `symbol`, `decimals`, and `totalSupply`. |

## Error Handling

| Item | Requirement |
| --- | --- |
| Structured errors | Reject promises with `{ code, message, data? }`. |
| Stable codes | Use the numeric `ErrorCode` values defined by NEP-21. |
| User rejection | Return `CANCELED` (`10006`) when the user rejects a prompt. |
| Timeout | Return `TIMEOUT` (`10005`) when a prompt or network operation times out. |
| Contract failure | Return `FAILED` (`10004`) and include `InvocationResult` in `data` when contract execution fails. |
| RPC failure | Return `RPC_ERROR` (`10008`) for RPC server exceptions. |

## Security and UX

| Item | Recommendation |
| --- | --- |
| Confirmation UI | Show contract hash, method, signer, network, asset, amount, fee, and recipient before signing. |
| Witness scope | Default to `CalledByEntry` unless the dApp explicitly requests a wider scope and the user confirms it. |
| Global scope | Treat `Global` as high risk and display a stronger warning. |
| Network mismatch | Warn or reject when the dApp expects a different network from the active provider network. |
| Account mismatch | Warn or reject when the requested signer is not available in the wallet. |
| Large integers | Preserve integer precision by accepting and returning string values. |
| Provider extensions | Put wallet-specific fields under `extra` instead of changing standard field shapes. |

## Compatibility Notes

NEP-21 does not currently define a dedicated `connect`, `disconnect`, `getNetwork`, or `switchNetwork` method. If a wallet adds those methods, expose them as provider-specific extensions and document them under `extra` or a wallet-specific page, while keeping the standard NEP-21 behavior intact.

For best interoperability, wallet providers should keep the standard surface stable and avoid relying on localized messages or wallet-specific response shapes for core flows.

