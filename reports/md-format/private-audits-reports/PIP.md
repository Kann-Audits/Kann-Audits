# [M-01] Rescue Tokens: Implement a Rescue Funds Function for stucked funds

## Severity

Medium Risk

## Description

In Pip.sol, when a user deposits, they receive a nullifier code which is required when
withdrawing from a different address. However, since it is possible for a user to forget their nullifier
code, the funds could become permanently stuck in the contract.


## Recommendation

Add a rescueFunds function along with an on-chain view function to check
whether the nullifier hash (used during withdrawal) has been consumed. This would enable the
protocol to verify that the user genuinely forgot their nullifier code and facilitate the recovery of their
funds.

## Team Response

Acknowledged. 


# [M-02] Constant Fee for Relayer

## Severity

Medium Risk

## Description

Gas consumption on the chain could increase—either temporarily during high conges-
tion periods or permanently due to evolving network conditions. With the current fixed fee parame-
ters, relayers may not receive sufficient incentives to cover their gas expenses. In some cases, a relayer
might end up paying more in gas fees than the reward they receive. This imbalance not only discour-
ages relayer participation but also forces new addresses to source gas from elsewhere in order to call
the withdraw() function, which negatively impacts the protocol’s overall usability and efficiency.

## Recommendation

User-Defined Relayer Fee: Allow users to input a custom relayer fee when initi-
ating a transaction. This approach gives users the flexibility to determine what they consider a fair
incentive for the relayer, based on current network conditions and their own willingness to pay.
Dynamic Gas Fee Function: Alternatively, implement a function that can adjust the gas fee for relay-
ers. This function would allow the protocol administrator (or even potentially the community through
governance mechanisms) to update the relayer fee dynamically in response to changes in gas prices.
This ensures that the fee remains competitive and sufficient to cover relayer costs.

## Team Response

Fixed. 