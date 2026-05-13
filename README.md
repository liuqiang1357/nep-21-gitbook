# NEP-21 dAPI

NEP-21 defines a common browser dAPI for N3 dApps and external wallet providers. It gives dApps a predictable way to find a wallet, read connected accounts, request signatures, build transactions, relay transactions, and subscribe to account or network changes.

The goal is simple: a dApp should not need a custom integration for every wallet, and a wallet should not need a different API shape for every dApp.

## Who Should Read This

| Reader | Start here |
| --- | --- |
| dApp developers | [dApp Developers](dapp-developers.md) |
| Wallet providers | [Wallet Providers](wallet-providers.md) |
| SDK and tooling authors | [API Reference](api-reference.md) and [Types and Errors](types-and-errors.md) |
| Integration reviewers | [Interoperability](interoperability.md) |

## What NEP-21 Standardizes

| Area | What the dAPI provides |
| --- | --- |
| Provider discovery | Browser events for requesting and announcing an `IDapiProvider` instance. |
| Provider metadata | Wallet name, version, dAPI version, compatibility list, active network, supported networks, icon, and website. |
| Accounts | Read connected accounts and ask the user to select an address. |
| Authentication | Request an authentication signature compatible with the NEP-20 authentication flow. |
| Assets and contracts | Read balances, call contracts offchain, and request onchain invocations. |
| Transactions | Build, sign, relay, or directly submit transactions. |
| Chain data | Read blocks, transactions, application logs, storage values, and token metadata. |
| Events | Subscribe to account and network changes. |
| Errors | Handle failures through stable numeric error codes. |

## Core Concepts

### Provider

The provider is the wallet object exposed to the page. dApps use it as the single entry point for wallet-mediated actions.

```ts
type Provider = IDapiProvider;
```

### Account

An account represents a wallet-controlled N3 account. NEP-21 exposes both the address and script hash.

```ts
type Account = {
  hash: string;
  address: string;
  label?: string;
};
```

### Network

Networks are represented by N3 network magic numbers.

| Network | Magic |
| --- | --- |
| MainNet | `860833102` |
| TestNet | `894710606` |

Before signing or sending a transaction, dApps should verify that `provider.network` matches the network expected by the application.

### Invocation

An invocation describes a contract call.

```ts
const invocation = {
  hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
  operation: "symbol",
  args: [],
};
```

`call` executes an invocation offchain. `invoke` asks the wallet to create, sign, and relay an onchain transaction.

## Basic Integration

```ts
const provider = await waitForNeoDapiProvider();

if (!provider.compatibility.includes("NEP-21")) {
  throw new Error("The selected wallet does not support NEP-21.");
}

const accounts = await provider.getAccounts();
const account = accounts[0];

const result = await provider.call({
  hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
  operation: "symbol",
  args: [],
});

console.log(account.address, result.stack);
```

## Provider Discovery

Wallets announce a provider with `Neo.DapiProvider.ready`.

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.ready", {
  detail: { provider },
}));
```

dApps can also request a provider with `Neo.DapiProvider.request`.

```ts
window.dispatchEvent(new CustomEvent("Neo.DapiProvider.request", {
  detail: { version: "1.0" },
}));
```

The recommended dApp pattern is to listen for `ready`, dispatch `request`, filter providers by compatibility, and then let the user choose when multiple compatible wallets respond.

## Recommended Reading Order

1. Read [dApp Developers](dapp-developers.md) if you are integrating a wallet into an application.
2. Read [Wallet Providers](wallet-providers.md) if you are exposing a wallet implementation to dApps.
3. Use [API Reference](api-reference.md) while implementing individual methods.
4. Use [Types and Errors](types-and-errors.md) when wiring TypeScript types and error handling.
5. Use [Interoperability](interoperability.md) before shipping a production integration.

## Reference Materials

* [NEP-21 specification](https://github.com/neo-project/proposals/blob/master/nep-21.mediawiki)
* [Neo developer documentation](https://developers.neo.org/docs/n3/develop/network/testnet)
