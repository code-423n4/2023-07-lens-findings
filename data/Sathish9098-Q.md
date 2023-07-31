# LOW FINDININGS 

##

## [L-1] Add to ``blacklist`` function

As stated in the project:

- Is it an NFT?: true

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
Recommended Mitigation Steps
Add to Blacklist function and modifier.

## 

## [L-2] ``tokenURI`` function should return empty string instead of revert when ``followTokenId`` not exist 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L286-L288

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensProfiles.sol#L105

##

## [L-3] Lack of access control ``createProfile() ``

The function createProfile also has a potential lack of access control. The function only validates that the sender of the transaction is whitelisted, but it does not check if the sender has the permission to create a profile. This means that any whitelisted address could create a profile, even if they do not have the permission to do so.

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L89-L102

##

## [L-4] It is not ideal to have two functions with the same name in different contracts. ``createProfile()`` function names used in different contracts ``LensHub.sol`` , ``ProfileLib.sol``

This can lead to confusion and errors. It is better to give each function a unique name that reflects its purpose

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/ProfileLib.sol#L39

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L89

##

## [L-5] Anyone can call ``initialize()`` function and set their own values this will create ``unintended consequences``

Once contract The function initialize() has a potential lack of access control. The function only checks if the contract has already been initialized, but it does not check if the sender of the transaction is the LensHub. This means that any account could call the function and set the followed profile ID and royalty, which could have unintended consequences

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L48-L56

##

## [L-6] Once contract deployed then unable to initialize ``_followedProfileId ``,``_setRoyalty(1000)`` values. Once _initialize value set to true then  initialize() function always reverts 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L44

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L48-L56

##

## [L-7] Users cannot follow profiles themselves, which could be inconvenient if only possible to called by ``onlyHub`` modifier

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L59-L63

##

## [L-8] No control for ``removeFollower(), approveFollower()``  function 

This means that any account could call the function and unfollow any profile, even if they do not own the profile or have been approved by the owner of the profile. This could have unintended consequences, such as preventing users from receiving royalties from their followers

The function only checks if the sender of the transaction is the owner of the follow token or if the sender has been approved by the owner of the follow token. However, it does not check if the sender is the owner of the profile that is being unfollowed. 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L131

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L141

##

## [L-9] It uses the override modifier, but the function does not override the ``supportsInterface()`` function in the ERC2981CollectionRoyalties contract. This means that the function will only return true if the interface ID is supported by the ``LensBaseERC721`` contract

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L263-L272

##

## [L-10] The function does not check if the ``interface ID`` is a valid interface ID

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L263-L272

##

## [L-11] Lack of consistency in ``increment operators`` . Some places`` ++X`` used and some places ``X++`` used 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L300-L301

##

## [L-12] ``followerProfileId`` validity not checked in `` _followMintingNewToken()``  when minting new token . ``followerProfileId `` can be zero value 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L297-L309

##

## [L-13] ``unchecked`` block may cause problem in future when ``_followerCount`` increased . For safety remove unchecked keyword 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L361-L363

##

## [L-14] ``currentFollowerProfileId != 0`` ``> 0`` check should be used instead of != 

The check currentFollowerProfileId > 0 may be more efficient than the check currentFollowerProfileId != 0. This is because the check currentFollowerProfileId > 0 can be evaluated more quickly than the check currentFollowerProfileId != 0.

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L373

##

## [L-15] Use sequential ``tokenId`` generation when minting  new NFT instead of user defined inputs . Use state variable to increment the ``tokenId`` values 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensBaseERC721.sol#L353

##

## [L-16] Lack of input validation for ``uint256`` when assigning critical values to immutable variables in ``constructor``

If any human errors we need to redeploy the over all contract. Add necessary sanity checks 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensProfiles.sol#L33-L36 

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L62-L68

##

## [L-17] Possible to craft a contract that could pass the ``onlyEOA`` modifier check even though it is not an ``EOA``

Create a contract that inherits from another contract that is an EOA. For example, the following contract inherits from the Ownable contract, which is an EOA.

```
contract MyContract is Ownable {
  // ...
}

```
Because the MyContract contract inherits from the Ownable contract, it will also be an EOA. This means that the onlyEOA modifier will not prevent calls to the MyContract contract

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensProfiles.sol#L50-L55

##

## [L-18] ``approve()``, ``setApprovalForAll()`` functions should use ``whenNotPaused `` modifier 

The approve and setApprovalForAll functions do need to use the whenNotPaused modifier. This is because these functions allow users to grant other users permission to spend their tokens, and it is important to ensure that these functions cannot be called when the contract is paused.

The whenNotPaused modifier is a Solidity modifier that prevents functions from being called when the contract is paused. This means that if the contract is paused, then the approve and setApprovalForAll functions will not be able to be called, even if the user has the correct permissions.

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensProfiles.sol#L112

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensProfiles.sol#L120

##

## [L-19] No clear documentation about ``executeLensV2Upgrade()`` function

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/LensV2UpgradeContract.sol#L44-L48

##

## [L-20]  ``_upgrade`` function does not perform any ``pre-upgrade checks`` and ``post-upgrade checks``, which means that there is no way to ensure that the contract is in a good state before it is upgraded. This could lead to problems if the contract is upgraded while it is in a bad state

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/LensV2UpgradeContract.sol#L44-L48

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/LensV2UpgradeContract.sol#L50-L62

##

## [L-21] The ``_validateLocalName`` and ``_validateLocalNameMigration`` functions prohibit handle names starting with underscores. Such restrictions as they might lead to unexpected limitations for users in certain cases

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L211

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L234

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L240

##

## [L-22] The contract uses custom error messages defined in ``HandlesErrors.sol``. While it's not shown in the provided code, ensure that the error messages provide sufficient information for debugging and don't leak sensitive data

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L103

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L115

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L128

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L128

##

## [L-23] The contract is only compatible with Lens Handles and Lens Hub tokens. This means that it cannot be used to link tokens from other protocols or with other types of handles

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/TokenHandleRegistry.sol#L19

##

## [L-24] The contract does not allow for multiple handles to be linked to the same token. This could be a problem if a user wants to use different handles for different purposes

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/TokenHandleRegistry.sol#L61-L69

##

## [L-25] Natspec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Recommendation
NatSpec comments should be increased in Contracts

Recommendation  Code:
```js
 /**
     * @notice Sets approval for specific withdrawer addresses
     * @dev This function updates the amount of the withdrawApproval mapping value based on the given 3 argument values
     * @param withdrawer_ Withdrawer address from state variable
     * @param token_ instance of ERC20
     * @param amount_ User amount
     * @param 

## [L-26] Reduce the inheritance list

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L45-L52



