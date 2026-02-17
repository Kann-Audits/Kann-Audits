# [M-01] Slippage Validation Failures

## Severity

Medium

## Description

Slippage values are calculated and displayed to users but never enforced during swap execution, leading to potential silent losses and misleading quotes.

Code Example:

```ts
// Quote calculation
slippageLegOne: jupQuote.slippageBps,
slippageLegTwo: 50,

// Swap execution – missing slippageBps
orderUrl.searchParams.set("inputMint", inputMint);
orderUrl.searchParams.set("outputMint", outputMint);
orderUrl.searchParams.set("amount", amount);
```

## Impact:

Users may receive significantly less than quoted amounts. Silent failures may occur with no warning to users. Misleading quotes could harm user trust.

## Recommendation:

Pass slippageBps to Jupiter API.

orderUrl.searchParams.set("slippageBps", slippageLegOne.toString());


Implement post-execution validation.

const actualOutput = await getTransactionOutput(signature);
if (BigInt(actualOutput) < BigInt(orderData.otherAmountThreshold)) {
    throw new Error("Slippage exceeded: expected, got less");
}

## Team Response

Fixed.

# [M-02] Kafka Consumer Retry Mechanism
## Severity

Medium

## Description

A single shared Kafka consumer instance (kafkaConsumer) across three consumers prevents proper error recovery, causing retries to fail and manual restarts to be required.

Code Example:

// Shared consumer
export const kafkaConsumer = kafka.consumer({
    groupId: "consumer-group",
    sessionTimeout: 120 * 1000,
    heartbeatInterval: 10000,
});

// All three consumers call kafkaConsumer.run() independently


## Impact:

Failed consumers cannot recover automatically. The retry mechanism fails repeatedly every 500ms. System availability and reliability are reduced.

## Recommendation:

Option 1 (recommended) is to create separate consumer instances.

export const preDepositConsumer = kafka.consumer({
    groupId: "predeposit-consumer-group",
});

export const postDepositConsumer = kafka.consumer({
    groupId: "postdeposit-consumer-group",
});

export const solverConsumer = kafka.consumer({
    groupId: "solver-consumer-group",
});


Option 2 is to use Promise.allSettled() instead of Promise.all() to handle failures without retrying the entire group.

## Team Response

Fixed.

# [I-01] Kafka Missing SSL/TLS Encryption and No Authentication
## Severity

Informational

## Description

All Kafka connections operate without SSL/TLS encryption and lack authentication mechanisms.

Code Example:

// Kafka server and consumer configuration
// No SSL/TLS or SASL auth enabled


## Impact:

All messages are transmitted in plaintext. Wallet addresses, transaction IDs, and order details are exposed to potential network eavesdropping.

## Recommendation:

Enable SSL/TLS encryption and implement SASL authentication for all Kafka connections.

## Team Response

Fixed.

# [I-02] Incomplete Balance Check in SolanaSolver
## Severity

Informational

## Description

The isSolvable() function performs an incomplete balance check and does not account for optional gas deposit transfers.

Code Example:

async isSolvable(params: OrderParams): Promise<boolean> {
    const balance = await this.getConnection().getBalance(
        new PublicKey(currentPool)
    );
    const requiredAmount = BigInt(params.amount); // Missing gas
    return BigInt(balance) >= requiredAmount;
}


## Impact:

Orders may be marked as solvable despite insufficient funds. Transactions can fail after acceptance, resulting in wasted RPC calls and fees.

## Recommendation:

Include gas deposit in balance validation.

let requiredAmount = BigInt(params.amount);

if (params.destinationGasDepositAddress) {
    requiredAmount += BigInt(DESTINATION_GAS_AMOUNT_LAMPORTS);
}

return availableBalance >= requiredAmount;

## Team Response

Fixed.