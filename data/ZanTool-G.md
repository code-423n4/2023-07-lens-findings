# 1. Use Assembly to Check Zero Address
## Location
contracts/base/LensBaseERC721.sol:107 118 323 416 354
contracts/base/LensProfiles.sol:114
contracts/FollowNFT.sol:74 109 146 157 184
contracts/LensHub.sol:494
contracts/libraries/FollowLib.sol:108 121 66
contracts/libraries/LegacyCollectLib.sol:66 122
contracts/libraries/MigrationLib.sol:67
contracts/libraries/ProfileLib.sol:48 99 148 25 18
contracts/libraries/PublicationLib.sol:528 321 427 374 178 183
contracts/modules/act/collect/CollectPublicationAction.sol:69 102
contracts/modules/act/seadrop/SeaDropMintPublicationAction.sol:127
contracts/namespaces/LensHandles.sol:141
contracts/namespaces/TokenHandleRegistry.sol:146 154
## Description
Using assembly to check zero address can save gas. Under Solidity compiler 0.8.x, about 18 gas
can be saved in each call.
## Recommendation
It is recommended to use assembly to check zero address.
# 2. Cache State Variables that are Read Multiple Times within A Function
## Location
contracts/FollowNFT.sol:172 174
contracts/namespaces/LensHandles.sol:256 257
contracts/namespaces/LensHandles.sol:114 117
## Description
When a state variable like _followDataByFollowTokenId, _tokenGuardianDisablingTimestamp[wallet] and _tokenGuardianDisablingTimestamp[msg.sender] are read multiple times in a function, using a local variable to cache the state variable can avoid frequently reading data from storage, thereby saving gas.
## Recommendation
It is recommended to use a local variable to cache the state variables that are read multiple times in a function.
# 3. Prefer uint256
## Location
contracts/base/LensBaseERC721.sol:128
## Description
It is recommended to replace integer types that are not 32 bytes in size and cannot be combined with other storage with uint256 to avoid the gas overhead caused by filling 32 bytes in operation.
```
uint96 mintTimestamp = _tokenData[tokenId].mintTimestamp;
```
# 4. Use CustomError Instead of String
## Location
contracts/base/LensBaseERC721.sol:483
contracts/libraries/PublicationLib.sol:343 449 396
contracts/misc/access/Governance.sol:87
## Description
When using revert, using CustomError is more gas efficient than string description. Because the error message described using CustomError is only compiled into four bytes. Under Solidity compiler 0.8.x, in a reverted transaction, when compiler optimization is turned off, about 250-270 gas can be saved. When compiler optimization is turned on, about 60-80 gas can be saved.
## Recommendation
When reverting, it is recommended to use CustomError instead of ordinary strings to describe the
error message.
# 5. Get Contract Balance of ETH in Assembly
## Location
contracts/modules/act/seadrop/SeaDropMintPublicationAction.sol:128 168 185
## Description
Using the selfbalance and balance opcodes to get the ETH balance of the contract in assembly saves gas compared to getting the ETH balance through address(this).balance and xx.balance.
## Recommendation
It is recommended to get the contract ETH balance in assembly.
# 6. Variables Should Be Constants
## Location
contracts/namespaces/LensHandles.sol:30
## Description
There are unchanging state variables in the contract. It is recommended to set the variable NAMESPACE_LENGTH to constant rather than immutable.
# 7. Set the Constant to Private
## Location
contracts/modules/reference/DegreesOfSeparationReferenceModule.sol:53
## Description
For constants, if the visibility is set to public, the compiler will automatically generate a getter function for it, which will consume more gas during deployment than private.
## Recommendation
It is recommended to set the visibility of constants to private instead of public.