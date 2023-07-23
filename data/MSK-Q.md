## [LOW-01] 
###  ```FollowLib._follow``` has external calls inside a loop
#### PoC
Exernal call is being used in ```While loop``` that can cause DoS
``` solidity
 while (i < idsOfProfilesToUnfollow.length) {
            uint256 idOfProfileToUnfollow = idsOfProfilesToUnfollow[i];
            ValidationLib.validateProfileExists(idOfProfileToUnfollow);

            address followNFT = StorageLib.getProfile(idOfProfileToUnfollow).followNFT;

            if (followNFT == address(0)) {
                revert Errors.NotFollowing();
            }

            IFollowNFT(followNFT).unfollow({
                unfollowerProfileId: unfollowerProfileId,
                transactionExecutor: transactionExecutor
            });

            emit Events.Unfollowed(unfollowerProfileId, idOfProfileToUnfollow, block.timestamp);

            unchecked {
                ++i;
            }
```
#### Code Link 
[FollowLib.sol#L60](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/FollowLib.sol#L60)
#### Recommended Mitigation Step
Do not use external calls in Loops.