# NEP-21 dAPI Developer Guide

NEP-21 defines a common dAPI interface for N3 dApps and external wallet providers. This guide turns the specification into developer-oriented Markdown that can be imported into GitBook directly.

The API reference follows the structure used by NeoLine dAPI documentation: method description, input arguments, success response, error response, example, and example response.

## Who This Is For

This guide is written for:

| Role | Use this guide to |
| --- | --- |
| dApp developers | discover a wallet provider, read accounts, sign messages, send assets, and invoke contracts |
| wallet providers | expose a compatible `IDapiProvider` implementation to dApps |
| documentation maintainers | publish NEP-21 as GitBook, Mintlify, Docusaurus, or Nextra documentation |

## GitBook Import

Import the `nep-21-gitbook` folder into GitBook.

GitBook will use:

| File | Purpose |
| --- | --- |
| `README.md` | landing page |
| `SUMMARY.md` | sidebar navigation |
| `api-reference.md` | provider discovery, events, properties, and methods |
| `types-and-errors.md` | shared TypeScript-style types and error codes |
| `wallet-provider-checklist.md` | wallet implementation checklist |

## Quick Start

### 1. Get a Provider

Wallets implementing NEP-21 announce themselves by dispatching `Neo.DapiProvider.ready`. A dApp may also request a provider by dispatching `Neo.DapiProvider.request`.

```ts
export function waitForNeoDapiProvider(expectedVersion = "1.0", timeoutMs = 3000) {
  return new Promise((resolve, reject) => {
    let settled = false;

    const cleanup = () => {
      window.removeEventListener("Neo.DapiProvider.ready", onReady);
      clearTimeout(timer);
    };

    const finish = (provider: any) => {
      if (settled) return;
      settled = true;
      cleanup();
      resolve(provider);
    };

    const onReady = (event: Event) => {
      const provider = (event as CustomEvent).detail?.provider;
      if (provider?.compatibility?.includes("NEP-21")) {
        finish(provider);
      }
    };

    const timer = window.setTimeout(() => {
      if (settled) return;
      settled = true;
      cleanup();
      reject(new Error("No NEP-21 provider was found."));
    }, timeoutMs);

    window.addEventListener("Neo.DapiProvider.ready", onReady);

    // Ask installed wallets/extensions to inject or announce a compatible provider.
    window.dispatchEvent(new CustomEvent("Neo.DapiProvider.request", {
      detail: { version: expectedVersion },
    }));
  });
}
```

### 2. Validate Provider Metadata

NEP-21 does not define a `getProvider()` method. The provider instance exposes provider metadata as properties.

```ts
const provider: any = await waitForNeoDapiProvider();

if (provider.dapiVersion !== "1.0") {
  throw new Error(`Unsupported dAPI version: ${provider.dapiVersion}`);
}

if (!provider.compatibility.includes("NEP-21")) {
  throw new Error(`${provider.name} does not support NEP-21.`);
}

console.log({
  name: provider.name,
  version: provider.version,
  dapiVersion: provider.dapiVersion,
  network: provider.network,
  supportedNetworks: provider.supportedNetworks,
});
```

### 3. Read Accounts and Subscribe to Changes

```ts
const accounts = await provider.getAccounts();
const currentAccount = accounts[0];

provider.on("accountchanged", (event: CustomEvent) => {
  console.log("Accounts changed:", event.detail.accounts);
});

provider.on("networkchanged", (event: CustomEvent) => {
  console.log("Network changed:", event.detail.network);
});
```

### 4. Call a Contract Offchain

```ts
const result = await provider.call({
  hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
  operation: "symbol",
  args: [],
});

console.log(result.state, result.stack);
```

### 5. Invoke a Contract Onchain

```ts
const txid = await provider.invoke([
  {
    hash: "0xd2a4cff31913016155e38e474a2c06d08be276cf",
    operation: "transfer",
    args: [
      { type: "Hash160", value: currentAccount.hash },
      { type: "Hash160", value: "0x1111111111111111111111111111111111111111" },
      { type: "Integer", value: "100000000" },
      { type: "Any" },
    ],
    abortOnFail: true,
  },
], [
  {
    account: currentAccount.hash,
    scopes: "CalledByEntry",
  },
]);

console.log("Transaction hash:", txid);
```

## Integration Notes

NEP-21 intentionally standardizes the wallet-facing API surface, but production dApps should still handle wallet differences carefully:

| Topic | Recommendation |
| --- | --- |
| Provider discovery | Listen for `Neo.DapiProvider.ready` and also dispatch `Neo.DapiProvider.request`. |
| Compatibility | Check `provider.compatibility.includes("NEP-21")` before calling methods. |
| Account state | Re-read `getAccounts()` before signing or sending high-value transactions. |
| Network state | Compare `provider.network` with the network expected by the dApp. |
| `send` sender | Pass the `from` account explicitly when possible. The spec text says wallets may select it when omitted, but the current interface lists `from` as a positional parameter. |
| Error handling | Branch by numeric `error.code`, not by localized `error.message`. |

## Documentation Platform Recommendation

GitBook is a practical choice if the target audience includes non-engineering contributors and you want a hosted editor.

For developer API documentation, Mintlify is usually the most polished hosted option because it treats API references, tabs, callouts, and code examples as first-class content. For open-source docs that should live close to the repository and support versioned builds, Docusaurus or Nextra are stronger long-term choices.

For this NEP-21 document set, GitBook import is enough today. If the docs become the canonical wallet/dApp developer portal, consider Mintlify for hosted API docs or Docusaurus/Nextra for repo-native publishing.

## Sources

* [NEP-21 specification](https://github.com/neo-project/proposals/blob/master/nep-21.mediawiki)
* [NeoLine N3 dAPI documentation](https://neoline.io/dapi/N3.html#getProvider)
