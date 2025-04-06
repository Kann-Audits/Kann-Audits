# [M-01] AuctionDemo.sol, claimAuction() sends the highest bid to the wrong account.

## Severity

Medium Risk

## Description

claimAuction() is invoked by the highest bidder to claim the NFT. However, there's a critical issue where the bid amount is sent to the smart contract owner rather than the previous NFT owner. This results in a financial loss for the original NFT owner.

## Proof of Concept

```solidity
(bool success, ) = payable(owner()).call{value: highestBid}("");
```

As you can observe, the address is obtained using the owner() of the Ownable contract, which returns the smart contract owner, not the NFT owner. This leads to funds being sent to an unintended account.

## Recommendation

Send the funds to the previous owner of the NFT:

```solidity
address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
(bool success, ) = payable(ownerOfToken).call{value: highestBid}("");
IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
```

IMPORTANT! To implement this change successfully, first send the funds to the original user, and then transfer the NFT to the highest bidder. Please follow the specified order in the code above

## Team Response

Fixed.

# [M-02] Issue in listVesting and spotPurchase Function for SINGLE Listing Type

## Severity

Medium Risk

## Description

In MarketPlace.sol when User calls spotPurchase he writes from which vesting plan and listing id to buy and how much to buy - amount.
The issue comes here if the owner has set the ListingType.Single logically it should only let the User to buy the whole stock without letting him buy just a partial fragment of it. However in listVesting this logic is wrong.

The provided code consists of two primary functions within a marketplace contract: listVesting and spotPurchase. These functions are used for listing and purchasing tokens under a vesting plan, with specific rules governing how tokens can be bought based on listing types. The marketplace contract includes logic for handling different types of listings, particularly the SINGLE listing type, which is supposed to enforce the condition that the entire listing amount must be purchased in one transaction.

The core issue identified lies in the conditional logic of the listVesting function when handling the SINGLE listing type. The current condition in the function allows for the possibility of partial purchases under a SINGLE listing type, which contradicts the expected behavior where the buyer must purchase the entire available listing amount.

```solidity
require(
    _listingType != ListingType.SINGLE || (_minPurchaseAmt > 0 && _minPurchaseAmt <= _amount),
    "SS_Marketplace: Minimum Purchase Amount cannot be more than listing amount"
);
```

In this conditional statement, when the listing type is not SINGLE, it checks whether \_minPurchaseAmt > 0 && \_minPurchaseAmt <= \_amount. For SINGLE listing types, this condition only ensures that \_minPurchaseAmt is less than or equal to the listing amount (\_amount), but it does not enforce that \_minPurchaseAmt must exactly match \_amount.

Impact: Partial Purchases on SINGLE Listings Users could buy less than the total available amount of tokens listed under the SINGLE listing type. This undermines the intended behavior where the entire listing should be purchased at once.

## Recommendation

```solidity
require(
    _listingType != ListingType.SINGLE || (_minPurchaseAmt == _amount),
    "SS_Marketplace: Minimum Purchase Amount must be equal to the listing amount for SINGLE listing type"
);
```

## Team Response

Fixed.
