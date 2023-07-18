see this link:
https://github.com/code-423n4/2023-07-lens/blob/5917aab811803c29ed883f1d63002c202cf28f78/contracts/FollowNFT.sol#L480-L493

The function FollowNFT.tryMigrate has idOfProfileFollowed parameter.
But this parameter only checks for consistency with _followedProfileId and is not used elsewhere in the function.

If you do not need to use this parameter, then it is better to delete it.