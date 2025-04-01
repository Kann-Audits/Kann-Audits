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