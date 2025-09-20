# [M-01] Missing Validation of deposit to targetvault Return Value

## Severity

Medium

## Description

The _deployFunds function forwards funds to targetVault by calling:

targetVault.deposit(deployAmount, address(this));

The _deployFunds function assumes targetVault.deposit(...) will always give the strategy a non-zero amount of vault shares. In normal, well-behaved vaults that’s true, but there are cases where this assumption can fail. For example, if a freshly launched/integrated vault has been pre-funded (someone transfers assets directly to the vault contract without calling transfer the vault’s share-calculation can be skewed and a subsequent deposit call can result in 0 shares minted for the depositor. Other edge cases could also lead to the same outcome, leaving the strategy’s assets effectively locked in the vault with no credited shares.

Recommendation
Always validate that shares were actually received:

```solidity
uint256 sharesReceived = targetVault.deposit(deployAmount, address(this));
require(sharesReceived > 0, "Deposit did not mint shares");
```

## Team Response

Fixed.


# [I-01] Redundant Balance Check and Math.min in deployFunds

## Severity

Informational

## Description

A user calls deposit()
This function is defined in TokenizedStrategy and is executed via delegatecall from the Forwarder (which inherits BaseStrategy).
As a result, storage updates happen in the Forwarder, msg.sender remains the user, and address(this) refers to the Forwarder.

Inside TokenizedStrategy.deposit:
Assets are transferred from the user to the Forwarder (_asset.safeTransferFrom(msg.sender, address(this), assets)).
then
 IBaseStrategy(address(this)).deployFunds(
            _asset.balanceOf(address(this))
        );
We call deployFunds with the balance of the forwarder = BaseStrategy  where then inside Forwarder it gets uint256 deployAmount = Math.min(_amount, balance);
Which both values  are same since _amount is address this balance of asset   and balance is address(this) balance  of the asset in the contract.

Math.min and getting the balance of the contract in _deployFunds is redudant.

## Team Response

Fixed.