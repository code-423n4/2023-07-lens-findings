

###### Floating pragma is used (low)

Vulnerability details: 

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the pragma (for e.g. by not using ^ in pragma solidity ^0.8.15) 

Impact:

avoids that contracts do not accidentally get deployed using an older compiler version with unfixed bugs.

POC:
Present in all files

Tools used:
None

Recommended Mitigation Steps:

The correct will be
pragma solidity 0.8.x; (choice of team)

And should be made across all the imports. 


###### Lack of checks

Vulnerability details: 

The current implementation of our smart contract constructor does not include any form of input validation for its parameters.

Impact:

This lack of checks can potentially lead to erroneous assignments, and in worst-case scenarios, security vulnerabilities.

POC:

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L66

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2Migration.sol

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol


Tools used:
None

Recommended Mitigation Steps:

Ideally, each of the contract address parameters e.g (moduleGlobals, followNFTImpl, collectNFTImpl, lensHandlesAddress, tokenHandleRegistryAddress, legacyFeeFollowModule, legacyProfileFollowModule, newFeeFollowModule) should be validated for legitimacy. They should be non-zero addresses and should correspond to the expected contract interface.


###### Misleading Error Message

Vulnerability details: 

 In the current implementation of the unfollow function, when followNFT resolves to a zero address (address(0)), the function reverts with Errors.NotFollowing(). This error message suggests that the user is not following the profile they are attempting to unfollow. However, a zero address for followNFT is more indicative of the profile NFT not existing, rather than the user not following the profile.

Impact:
Confusing error message

POC:
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/FollowLib.sol#L54C23-L54C23

Tools used:
None

Recommended Mitigation Steps:

Replace the Errors.NotFollowing() error message with a more suitable error message such as Errors.NFTDoesNotExist()- create a new error message that accurately represents the situation. This will make the error handling more descriptive and accurate, thus improving debugging and issue resolution.


###### open TODO

Vulnerability details: 

There's an open TODO in the ProtocolState enum, concerning the reordering of enum members so that Paused becomes 0 and the default state.

Impact:

Protocol not fully implementing its required upgrades

POC:
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/constants/Types.sol#L48

Tools used:
None

Recommended Mitigation Steps:

completion of this TODO. Carefully plan the change, update the enum order, and simultaneously update any dependent code to prevent unwanted side effects. Thoroughly test the changes to ensure the protocol behaves as expected.


###### open TODO

Vulnerability details: 

The project files show inconsistencies in the specified Solidity pragma versions, with some files using ^0.8.19 and others using ^0.8.15 and ^0.8.18

Impact:

This variation could potentially lead to compatibility issues, unpredictable behavior, or deployment difficulties, especially when newer versions of Solidity introduce breaking changes or when features exclusive to certain versions are used.

POC:
^0.8.19:  https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol

^0.8.19 https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol

^0.8.18 https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol

^0.8.15, other files AFAIK

Tools used:
None

Recommended Mitigation Steps:

Standardize the Solidity version across all project files. If possible, upgrade all contracts to the latest and same version (^0.8.19 in this context). It's necessary to thoroughly test all functionalities post-upgrade to ensure no breaking changes were introduced with the newer version.


###### Open TODO in getHandle Function Regarding Non-Existent Tokens

Vulnerability details: 

An open TODO comment in the getHandle function questions whether the function should revert if a given tokenId does not exist. Currently, the function proceeds to concatenate localName and NAMESPACE, regardless of whether the tokenId exists or not.

Impact:

This could potentially lead to unexpected behavior or return misleading information if the tokenId doesn't correspond to a valid entity.

POC:
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L176C1-L180C6

Tools used:
None

Recommended Mitigation Steps:

Resolve the TODO by implementing a validation check for the existence of the tokenId before proceeding with the getLocalName function and subsequent operations. If the tokenId does not exist, it may be appropriate to revert the transaction with an explanatory error message.

