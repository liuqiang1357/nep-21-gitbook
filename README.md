# NEP-21 dAPI for N3

NEP-21 proposes a common browser API for N3 dApps and external wallet providers. It lets a dApp obtain a wallet provider, inspect wallet metadata, read accounts, request signatures, invoke contracts, relay transactions, and query selected chain data.

This documentation is written for two readers:

| Reader | What to look for |
| --- | --- |
| dApp developers | How to obtain a provider and call the standard methods. |
| Wallet providers | What shape the provider object, methods, events, and errors should have. |

The API pages follow the general style of wallet dAPI documentation: each method has a short description, parameters, return value, possible errors, and an example where useful. The content itself follows the current NEP-21 proposal.

## How dApps Get a Provider

NEP-21 providers are obtained through browser events.

Wallets announce a provider by dispatching `Neo.DapiProvider.ready`:

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.ready", {
  detail: {
    provider,
  },
}));
```

dApps listen for this event and read the provider from `event.detail.provider`:

```ts
window.addEventListener("Neo.DapiProvider.ready", (event) => {
  const provider = (event as CustomEvent).detail.provider;
  console.log(provider.name);
});
```

A dApp may also request a provider by dispatching `Neo.DapiProvider.request`:

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.request", {
  detail: {
    version: "1.0",
  },
}));
```

Wallets that support the requested version should respond by injecting or announcing a compatible provider.

## Provider Metadata

The provider exposes wallet metadata as properties.

```ts
const info = {
  name: provider.name,
  version: provider.version,
  dapiVersion: provider.dapiVersion,
  compatibility: provider.compatibility,
  connected: provider.connected,
  network: provider.network,
  supportedNetworks: provider.supportedNetworks,
  icon: provider.icon,
  website: provider.website,
  extra: provider.extra,
};
```

If the provider supports NEP-21, `compatibility` should include `"NEP-21"`.

## Minimal dApp Example

```ts
let provider: any;

window.addEventListener("Neo.DapiProvider.ready", (event) => {
  provider = (event as CustomEvent).detail.provider;
});

window.dispatchEvent(new CustomEvent("Neo.DapiProvider.request", {
  detail: { version: "1.0" },
}));

// After a provider is available:
const accounts = await provider.getAccounts();
const count = await provider.getBlockCount();

console.log(accounts, count);
```

## Pages

| Page | Contents |
| --- | --- |
| [API Reference](api-reference.md) | Provider discovery, properties, events, and methods. |
| [Types and Errors](types-and-errors.md) | Shared TypeScript-style types and error codes. |

## Network Values

| Network | Magic |
| --- | --- |
| MainNet | `860833102` |
| TestNet | `894710606` |

## Source

* [NEP-21 proposal PR #145](https://github.com/neo-project/proposals/pull/145)
