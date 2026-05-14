# Types and Errors

This page lists the shared types and error codes used by NEP-21.

## Basic Types

| Type | Definition | Description |
| --- | --- | --- |
| `Base64Encoded` | `string` | Base64 encoded data. |
| `Address` | `string` | N3 address. |
| `UInt160` | `string` | 160-bit hash as a hexadecimal string. |
| `UInt256` | `string` | 256-bit hash as a hexadecimal string. |
| `ECPoint` | `string` | ECC public key. |
| `Integer` | `number \| string` | Large integer value. |
| `HexString` | `string` | Hexadecimal string. |
| `Version` | `string` | Version string. |
| `Network` | `number` | N3 network magic number. |

## Provider Events

```ts
export type EventName =
  | "accountchanged"
  | "networkchanged";

export type ProviderReadyEvent = CustomEvent<{
  provider: IDapiProvider;
}>;

export type ProviderRequestEvent = CustomEvent<{
  version: Version;
}>;

export type AccountChangedEvent = CustomEvent<{
  accounts: Account[];
}>;

export type NetworkChangedEvent = CustomEvent<{
  network: Network;
}>;
```

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

## Contract Parameters

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

## Witnesses and Signers

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

## Transaction Context

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

## Chain Data

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

## VM Data

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

## Signing and Authentication

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

## Errors

Promises reject with an error object.

```ts
export interface Error {
  code: ErrorCode;
  message: string;
  data?: any;
}
```

| Code | Name | Description |
| --- | --- | --- |
| `10000` | `UNKNOWN` | An unknown error has occurred. |
| `10001` | `UNSUPPORTED` | The requested feature or operation is not supported. |
| `10002` | `INVALID` | The input data is in an invalid format. |
| `10003` | `NOTFOUND` | The requested data does not exist. |
| `10004` | `FAILED` | The contract execution failed. |
| `10005` | `TIMEOUT` | The requested operation was cancelled due to timeout. |
| `10006` | `CANCELED` | The requested operation was cancelled by the user. |
| `10007` | `INSUFFICIENT_FUNDS` | The requested operation failed due to insufficient balance. |
| `10008` | `RPC_ERROR` | An exception was thrown by the RPC server. |

