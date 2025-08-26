# [C-01] First Deposit Can Result in Zero Shares Due to Direct Token Transfer

## Severity

Critical

## Description

The StakedUSH vault is vulnerable to zero share deposits when USH tokens are sent directly to the contract.
Because the vault calculates shares based on totalSupply and totalAssets, a direct token transfer increases totalAssets without increasing totalSupply.
As a result, legitimate depositor will receive 0 shares for a positive deposit, breaking fair share distribution and potentially locking user funds.

Root Cause:
The sUSH vault calculates shares using the formula:

shares = (assets * (totalSupply + decimalsOffset)) / (totalAssets + 1)

If a user directly transfers USH to the vault contract (bypassing deposit()), the vault’s totalAssets increases while totalSupply remains 0.

When the next user calls deposit(), the formula becomes:

(assets * (0 + 0)) / (totalAssets + 1) = 0

Thus, the user receives 0 shares (REVERTS), even though they tried depositing a positive amount of assets.

## Team Response

Fixed.

# [H-01] FULL-Restricted Users Can Still Deposit

## Severity

High

## Description

The deposit flow does not fully enforce the FULL_RESTRICTED_STAKER_ROLE check.
When a user with FULL restriction calls deposit(assets, receiver), the restriction logic fails to prevent the operation, allowing restricted users to bypass intended compliance rules.

Deposit flow:
deposit(ERC4626 logic) -> _deposit(override) where checks if both addresses have SOFT_RESTRICTED_STAKER_ROLE if true reverts
then logic goes to _deposit(ERC4626)

```solidity
 function _deposit(address caller, address receiver, uint256 assets, uint256 shares) internal virtual {  
        
        SafeERC20.safeTransferFrom(IERC20(asset()), caller, address(this), assets);  
        _mint(receiver, shares);  
  
        emit Deposit(caller, receiver, assets, shares);  
    }  
```

Where _mint is called.

```solidity
function _mint(address account, uint256 value) internal {  
        if (account == address(0)) {  
            revert ERC20InvalidReceiver(address(0));  
        }  
        _update(address(0), account, value);  
    }  
```
See how _mint sets first parameter for _update to be address 0.

and since we have _update override it goes to

```solidity
 function _update(address from, address to, uint256 value) internal override {  
        if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {  
            revert OperationNotAllowed();  
        }  
        if (hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {  
            revert OperationNotAllowed();  
        }  
        super._update(from, to, value);  
    }  
```

Where it fails to check if the msg.sender is restricted role

## Team Response

Fixed.

# [M-01] _currentDefaultAdmin Not set in SingleAdminAccessControl

## Severity

Medium

## Description

In StakedUshBase.sol, which imports and inherits AdminControl, the constructor calls:

```solidity
_grantRole(DEFAULT_ADMIN_ROLE, _owner);
```

This sets the default admin role to _owner. However, the contract does not initialize the _currentDefaultAdmin variable, leaving it as address(0).

Consequently, in a scenario where the current default admin attempts to transfer the admin role by calling transferAdmin, followed by _pendingDefaultAdmin executing acceptAdmin, the _grantRole override logic executes:

```solidity
_revokeRole(DEFAULT_ADMIN_ROLE, _currentDefaultAdmin);
```

Since _currentDefaultAdmin is address(0), the call to _revokeRole effectively does nothing. The function then proceeds to grant the role to the new admin via:

```solidity
super._grantRole(role, account);
```

The net effect is that both the original _owner and the new admin retain the DEFAULT_ADMIN_ROLE, resulting in two admins instead of a proper transfer.

This issue arises from the lack of proper initialization of _currentDefaultAdmin during contract deployment and highlights a gap in the role transfer logic that lead to unintended multi-admin privileges.

## Team Response

Fixed.

# [L-01] Pause bypass for approvals via permit

## Severity

Low

## Description

USHToken gates approve() with whenNotPaused, but inherits permit() from Solmate without overriding. While paused, anyone can still set/refresh allowances via permit, pre-arming drains that execute the instant the token is unpaused.

## Team Response

Fixed.

# [L-02] Burner role blocked by Auth Manager checks

## Severity

Low

## Description

USHToken.burn(from, amount) calls _checkAuthTransfer(from, address(0)). If `from` has since become banned/sanctioned (via `AuthManager`), `checkSanctioned/checkBanned` will revert. This prevents the protocol (holder of `BURNER_ROLE`) from programmatically burning balances from blocked accounts a common compliance/control action.

## Team Response

Fixed.

# [I-01] chainalysisOracleEnabled() function logic inverted

## Severity

Low

## Description

The view function chainalysissOracleEnabled() returns true when the oracle is unset (i.e., disabled), and false when the oracle is actually configured. This behavior contradicts the function name, which suggests it should return true when the oracle is enabled.

## Recommendation

Fix logic:

```solidity
function chainalysisOracleEnabled() public view returns (bool) {
    return KycManagerStorageLib.getV1().chainalysisOracle != address(0);
}
```
## Team Response

Fixed.
