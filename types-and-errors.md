# Types and Errors

This page summarizes the shared types used by the NEP-21 dAPI.

## Primitive Aliases

| Type | Definition | Description |
| --- | --- | --- |
| `Base64Encoded` | `string` | Data encoded as base64. |
| `Address` | `string` | N3 address, for example `NSiVJYZej4XsxG5CUpdwn7VRQk8iiiDMPM`. |
| `UInt160` | `string` | 160-bit hash represented by a hexadecimal string. |
| `UInt256` | `string` | 256-bit hash represented by a hexadecimal string. |
| `ECPoint` | `string` | ECC public key. |
| `Integer` | `number \| string` | Large integer. Use strings when values may exceed JavaScript safe integer limits. |
| `HexString` | `string` | Hexadecimal string. |
| `Version` | `string` | Version string, for example `"1.0.0"`. |
| `Network` | `number` | N3 network magic number. |

## Network Values

| Network | Value |
| --- | --- |
| MainNet | `860833102` |
| TestNet | `894710606` |

Always compare against the wallet's `provider.network` and `provider.supportedNetworks` before sending network-specific transactions.

## Contract Parameters

### `ContractParameterType`

```ts
export type ContractParameterType =
  | "Any"
  | "Boolean"
  | "Integer"
  | "ByteArray"
  | "String"
  | "Hash160"
  | "Hash256"
  | "PublicKey"
  | "Signature"
  | "Array"
  | "Map"
  | "InteropInterface"
  | "Void";
```

### `Argument`

```ts
export type Argument = {
  type: ContractParameterType;
  value?: any;
};
```

### `Parameter`

```ts
export type Parameter = {
  name?: string;
  type: ContractParameterType;
};
```

## Accounts

```ts
export type Account = {
  hash: UInt160;
  address: Address;
  label?: string;
  contract?: {
    script?: Base64Encoded;
    parameters: Parameter[];
    deployed: boolean;
  };
  extra?: any;
};
```

## Witness Scopes

```ts
export type WitnessScope =
  | "None"
  | "CalledByEntry"
  | "CustomContracts"
  | "CustomGroups"
  | "WitnessRules"
  | "Global"
  | "CalledByEntry, CustomContracts"
  | "CalledByEntry, CustomGroups"
  | "CalledByEntry, WitnessRules"
  | "CustomContracts, CustomGroups"
  | "CustomContracts, WitnessRules"
  | "CustomGroups, WitnessRules"
  | "CalledByEntry, CustomContracts, CustomGroups"
  | "CalledByEntry, CustomContracts, WitnessRules"
  | "CalledByEntry, CustomGroups, WitnessRules"
  | "CustomContracts, CustomGroups, WitnessRules"
  | "CalledByEntry, CustomContracts, CustomGroups, WitnessRules";
```

| Scope | Description |
| --- | --- |
| `None` | Only signs the transaction. No contracts may use the witness. |
| `CalledByEntry` | Only the entry contract may use the witness. Recommended default for dApps. |
| `CustomContracts` | Specific contracts may use the witness. |
| `CustomGroups` | Specific contract groups may use the witness. |
| `WitnessRules` | Witness use must satisfy specified witness rules. |
| `Global` | Allows witness use in all contexts. Use only for highly trusted flows. |

## Witness Rules

```ts
export type WitnessConditionType =
  | "Boolean"
  | "Not"
  | "And"
  | "Or"
  | "ScriptHash"
  | "Group"
  | "CalledByEntry"
  | "CalledByContract"
  | "CalledByGroup";

export interface WitnessCondition {
  type: WitnessConditionType;
}

export interface BooleanCondition extends WitnessCondition {
  type: "Boolean";
  expression: boolean;
}

export interface NotCondition extends WitnessCondition {
  type: "Not";
  expression: WitnessCondition;
}

export interface AndCondition extends WitnessCondition {
  type: "And";
  expressions: WitnessCondition[];
}

export interface OrCondition extends WitnessCondition {
  type: "Or";
  expressions: WitnessCondition[];
}

export interface ScriptHashCondition extends WitnessCondition {
  type: "ScriptHash";
  hash: UInt160;
}

export interface GroupCondition extends WitnessCondition {
  type: "Group";
  group: ECPoint;
}

export interface CalledByEntryCondition extends WitnessCondition {
  type: "CalledByEntry";
}

export interface CalledByContractCondition extends WitnessCondition {
  type: "CalledByContract";
  hash: UInt160;
}

export interface CalledByGroupCondition extends WitnessCondition {
  type: "CalledByGroup";
  group: ECPoint;
}

export type WitnessRule = {
  action: "Deny" | "Allow";
  condition: WitnessCondition;
};
```

## Signer

```ts
export type Signer = {
  account: UInt160;
  scopes: WitnessScope;
  allowedContracts?: UInt160[];
  allowedGroups?: ECPoint[];
  rules?: WitnessRule[];
};
```

## Invocation

```ts
export type InvocationArguments = {
  hash: UInt160;
  operation: string;
  args?: Argument[];
  abortOnFail?: boolean;
};

export type InvocationResult = {
  script: Base64Encoded;
  state: VMState;
  gasconsumed: Integer;
  exception?: string;
  notifications: Notification[];
  stack: StackItem[];
};
```

## VM and Stack Types

```ts
export type TriggerType =
  | "OnPersist"
  | "PostPersist"
  | "Verification"
  | "Application";

export type VMState =
  | "NONE"
  | "HALT"
  | "FAULT"
  | "BREAK";

export type StackItemType =
  | "Any"
  | "Pointer"
  | "Boolean"
  | "Integer"
  | "ByteString"
  | "Buffer"
  | "Array"
  | "Struct"
  | "Map"
  | "InteropInterface";

export type StackItem = {
  type: StackItemType;
  value?: any;
};
```

## Notifications and Application Logs

```ts
export type Notification = {
  contract: UInt160;
  eventname: string;
  state: StackItem;
};

export type ApplicationLog = {
  txid: UInt256;
  executions: {
    trigger: TriggerType;
    vmstate: VMState;
    exception?: string;
    gasconsumed: Integer;
    stack: StackItem[];
    notifications: Notification[];
  }[];
};
```

## Token

```ts
export type Token = {
  symbol: string;
  decimals: number;
  totalSupply: Integer;
};
```

## Transaction Options

```ts
export type TransactionOptions = {
  suggestedSystemFee?: Integer;
  extraSystemFee?: Integer;
  validUntilBlock?: number;
};
```

| Field | Description |
| --- | --- |
| `suggestedSystemFee` | Suggested system fee. If provided, wallet should use it instead of automatic calculation. |
| `extraSystemFee` | Extra system fee added to the automatically calculated system fee. Ignored when `suggestedSystemFee` is provided. |
| `validUntilBlock` | Block height until which the transaction is valid. |

## Contract Parameters Context

```ts
export type ContractParametersContext = {
  type: "Neo.Network.P2P.Payloads.Transaction";
  hash: UInt256;
  data: Base64Encoded;
  items: Record<UInt160, {
    script: Base64Encoded;
    parameters: Argument[];
    signatures: Record<ECPoint, Base64Encoded>;
  }>;
  network: Network;
};
```

## Signing

```ts
export type SignOptions = {
  isBase64Encoded?: boolean;
  isTypedData?: boolean;
  isLedgerCompatible?: boolean;
};

export type SignedMessage = {
  payload: Base64Encoded;
  signature: Base64Encoded;
  account: UInt160;
  pubkey: ECPoint;
};
```

| Option | Description |
| --- | --- |
| `isBase64Encoded` | Indicates whether `message` is already base64 encoded. Defaults to `false`. |
| `isTypedData` | Reserved for typed data. The current spec notes that this can only be `false` for now. |
| `isLedgerCompatible` | If `true`, wallet signs a Ledger-compatible wrapped payload. Defaults to `false`. |

## Authentication

```ts
export type AuthenticationChallengePayload = {
  action: "Authentication";
  grant_type: "Signature";
  allowed_algorithms: ["ECDSA-P256"];
  domain: string;
  networks: Network[];
  nonce: string;
  timestamp: number;
};

export type AuthenticationResponsePayload = {
  algorithm: "ECDSA-P256";
  network: Network;
  pubkey: ECPoint;
  address: Address;
  nonce: string;
  timestamp: number;
  signature: Base64Encoded;
};
```

## Blocks and Transactions

```ts
export type TransactionAttributeType =
  | "HighPriority"
  | "OracleResponse";

export interface TransactionAttribute {
  type: TransactionAttributeType;
}

export type Transaction = {
  hash: UInt256;
  size: number;
  blockHash: UInt256;
  blockTime: number;
  confirmations: number;
  version: number;
  nonce: number;
  systemFee: Integer;
  networkFee: Integer;
  validUntilBlock: number;
  sender: UInt160;
  signers: Signer[];
  attributes: TransactionAttribute[];
  script: Base64Encoded;
};

export type Block = {
  hash: UInt256;
  size: number;
  confirmations: number;
  nextBlockHash?: UInt256;
  version: number;
  previousBlockHash: UInt256;
  merkleRoot: UInt256;
  time: number;
  nonce: HexString;
  index: number;
  primary: number;
  nextConsensus: UInt160;
  tx: Transaction[];
};
```

## Error Model

All promise rejections should use a structured error object.

```ts
export const enum ErrorCode {
  UNKNOWN = 10000,
  UNSUPPORTED = 10001,
  INVALID = 10002,
  NOTFOUND = 10003,
  FAILED = 10004,
  TIMEOUT = 10005,
  CANCELED = 10006,
  INSUFFICIENT_FUNDS = 10007,
  RPC_ERROR = 10008,
}

export interface Error {
  code: ErrorCode;
  message: string;
  data?: any;
}

export interface FailedError extends Error {
  code: ErrorCode.FAILED;
  message: "Contract execution failed";
  data: InvocationResult;
}
```

| Code | Name | Description |
| --- | --- | --- |
| `10000` | `UNKNOWN` | Unknown error. |
| `10001` | `UNSUPPORTED` | Requested feature or operation is not supported. |
| `10002` | `INVALID` | Input data is invalid. |
| `10003` | `NOTFOUND` | Requested data does not exist. |
| `10004` | `FAILED` | Contract execution failed. |
| `10005` | `TIMEOUT` | Requested operation timed out. |
| `10006` | `CANCELED` | User canceled the operation. |
| `10007` | `INSUFFICIENT_FUNDS` | Balance is insufficient. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

## Error Handling Example

```ts
try {
  const txid = await provider.invoke(invocations, signers);
  console.log(txid);
} catch (error: any) {
  switch (error.code) {
    case 10006:
      // The user rejected the wallet prompt.
      break;
    case 10007:
      // The account cannot pay the transfer amount or fees.
      break;
    case 10008:
      // The RPC server rejected or failed the request.
      console.error(error.data);
      break;
    default:
      console.error(error);
      break;
  }
}
```

