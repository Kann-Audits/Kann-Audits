# [H-01] LP Fees Unclaimable via launchV4Pool()

## Severity

High

## Description

The function launchV4Pool() allows custom pool creation and token graduation (with manually set ticks and spacing). However, it does not call lpLocker.setTokenParams(), which is required to configure the LP locker for fee claiming.

Without calling setTokenParams(), fees earned from the pool become unclaimable, rendering the fee distribution logic non-functional for tokens launched via launchV4Pool().

In contrast, the graduateToken() function does invoke lpLocker.setTokenParams(), meaning fee claiming only works when using that path. This makes the launchV4Pool() function incomplete and effectively useless for real-world deployments where fees matter.


## Team Response

Fixed.

# [M-01] Bonding Curve Check  stepsize * numSteps May Not Match curveSupply

## Severity

Medium

## Description

When launching a token with bonding curve parameters, there is no validation ensuring that stepsize * numSteps == curveSupply. If this is not aligned:

If stepsize * numSteps > curveSupply, the bonding curve will appear to offer more tokens than were actually allocated, leading to potential inconsistencies (e.g., using LP pool tokens to fulfill purchases).

If stepsize * numSteps < curveSupply, part of the curve supply will become unreachable via the bonding curve, and remain unused.

While this behavior may be intentional to allow flexible bonding curve shapes, without an explicit check or warning, it can lead to unintended launch behavior due to misconfiguration.


## Team Response

Fixed.

# [M-02] Incorrect ETH Transfer to State Manager — Full msg.value Sent Instead of Used Amount

## Severity

Medium

## Description

When a user purchases a launch token using ETH (base token), the acceptAmount() function correctly calculates the amount used (amountInUsed) and refunds the excess ETH back to the user. However, despite the refund, the entire msg.value is still forwarded to the StateManager.buyToken() call, rather than just the used portion.

This results in the StateManager contract receiving more ETH than it should, creating incorrect accounting and potential fund mismanagement.


## Team Response

Fixed.

# [M-03] Excess Base Tokens Not Refunded for ERC20 tokens

## Severity

Medium

## Description

When a user purchases tokens using an ERC20 base token, the function _getBuyQuoteAndFees() correctly calculates the actual amount used (amountInUsed) and the associated fees. However, if the user sends more than amountInUsed, the excess tokens are not refunded.

In contrast, when using ETH as the base token, any overpayment is explicitly refunded. This inconsistent behavior creates a silent loss of funds for users interacting via ERC20 tokens.

## Team Response

Fixed.

# [M-04] ERC20 Transfer — Not All Tokens Return Boolean

## Severity

Medium

## Description

Uses ERC20(token).transfer(...) and checks whether it returns true to confirm success. However, some widely-used tokens like USDT do not return any value on transfer, which causes the boolean check to fail and revert even if the transfer was successful.

## Team Response

Fixed.

# [L-01] LP Fees Unclaimable via launchV4Pool()

## Severity

Low

## Description

Although the deployer can set allowForcedGraduation to false, graduateToken() and _launchV4Pool() do not validate this setting, allowing owner of deployer and creator to forcibly graduate a token even when it was explicitly disabled.

## Team Response

Fixed.

# [I-01] Missing Validation — numSteps and prices.length Mismatch

## Severity

Informational

## Description

In the PriceCurve bondingCurveParams, no check ensures that numSteps equals prices.length. A mismatch could result in unexpected pricing behavior or runtime errors.

## Team Response

Fixed.