# Types and Errors

This page defines the shared data shapes used by NEP-21.

## Primitive Types

| Type | Definition | Description |
| --- | --- | --- |
| `Base64Encoded` | `string` | Base64 encoded data. |
| `Address` | `string` | N3 address. |
| `UInt160` | `string` | 160-bit hash as a hexadecimal string. |
| `UInt256` | `string` | 256-bit hash as a hexadecimal string. |
| `ECPoint` | `string` | ECC public key. |
| `Integer` | `number \| string` | Integer value. Use strings for large values. |
| `HexString` | `string` | Hexadecimal string. |
| `Version` | `string` | Version string. |
| `Network` | `number` | N3 network magic number. |

## Common Network Values

| Network | Magic |
| --- | --- |
| MainNet | `860833102` |
| TestNet | `894710606` |

## Account

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

## Contract Arguments

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

export type Parameter = {
  name?: string;
  type: ContractParameterType;
};

export type Argument = {
  type: ContractParameterType;
  value?: any;
};
```

## Signers and Witness Scopes

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

export type Signer = {
  account: UInt160;
  scopes: WitnessScope;
  allowedContracts?: UInt160[];
  allowedGroups?: ECPoint[];
  rules?: WitnessRule[];
};
```

| Scope | Use |
| --- | --- |
| `None` | Sign the transaction without allowing contract witness use. |
| `CalledByEntry` | Allow only the entry contract to use the witness. This is the safest common default. |
| `CustomContracts` | Restrict witness use to specific contracts. |
| `CustomGroups` | Restrict witness use to specific contract groups. |
| `WitnessRules` | Restrict witness use with explicit witness rules. |
| `Global` | Allow witness use in all contexts. Treat this as high risk. |

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

export type WitnessRule = {
  action: "Deny" | "Allow";
  condition: WitnessCondition;
};
```

## Invocations

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

## VM Data

```ts
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

export type Notification = {
  contract: UInt160;
  eventname: string;
  state: StackItem;
};
```

## Transactions

```ts
export type TransactionOptions = {
  suggestedSystemFee?: Integer;
  extraSystemFee?: Integer;
  validUntilBlock?: number;
};

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
```

## Blocks and Logs

```ts
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

## Message Signing

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

## Error Codes

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

| Code | Name | Meaning |
| --- | --- | --- |
| `10000` | `UNKNOWN` | Unknown error. |
| `10001` | `UNSUPPORTED` | Requested feature or operation is not supported. |
| `10002` | `INVALID` | Input data is invalid. |
| `10003` | `NOTFOUND` | Requested data does not exist. |
| `10004` | `FAILED` | Contract execution failed. |
| `10005` | `TIMEOUT` | Requested operation timed out. |
| `10006` | `CANCELED` | User canceled the operation. |
| `10007` | `INSUFFICIENT_FUNDS` | Balance cannot cover amount or fees. |
| `10008` | `RPC_ERROR` | RPC server returned an exception. |

## Error Handling Pattern

```ts
try {
  return await provider.invoke(invocations, signers);
} catch (error: any) {
  if (error.code === 10006) {
    return undefined;
  }

  if (error.code === 10004 && error.data) {
    console.error("VM state:", error.data.state);
  }

  throw error;
}
```

