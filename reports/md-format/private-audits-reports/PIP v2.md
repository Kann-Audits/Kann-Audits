# [I-01] Add Off-Chain Checks for requestWithdraw to Prevent Spam with Invalid Proofs and Potential Gas Waste for Relayers

## Severity

Informational

## Description

In the requestWithdraw process, users submit a withdrawal request by providing a proof
and public signals which get sent in a Telegram group. Relayers monitor this group and use the pro-
vided information to call the on-chain withdraw function.
However, without proper off-chain validation, bad actors can flood the Telegram group with invalid
or duplicate requests, causing relayers to waste gas attempting failed transactions.

## Recommendation

To prevent this, off-chain validation should include:
Nullifier Existence – Ensuring the provided nullifier corresponds to a valid deposit.
Correct Recipient – Verifying that the recipient address matches the expected one for the nullifier.
Withdrawal Status – Checking if the nullifier has already been used for a withdrawal.

## Team Response

Fixed.

# [I-02] Add an On-Chain View Function for Relayers to Estimate Exact Fee Rewards

## Severity

Informational

## Description

Currently, relayers calling the withdraw function lack an efficient way to determine the
exact fee they will receive for processing a withdrawal. This uncertainty may discourage participation
or lead to inefficient relaying strategies.

## Recommendation

To address this, an on-chain view function should be implemented to allow re-
layers to see the exact fee amount they will receive before executing a withdrawal transaction.

## Team Response

Fixed.
