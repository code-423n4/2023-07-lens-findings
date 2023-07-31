# Low 001
## Title
First follower is spending more gas

## Impact 
the first follower of a profile is spending more gas than subsequent followers. This will be unfair to them.

## Proof of Concept
As we can see in [`FollowLib#_follow`](), if that profile has no followNFT, the first follower does the deployment, which will consume more gas.
```solidity
function _follow()private returns (uint256){
    ...
    address followNFT = _profileToFollow.followNFT;
    if (followNFT == address(0)) {
        followNFT = _deployFollowNFT(idOfProfileToFollow);
        _profileToFollow.followNFT = followNFT;
    }
    ...
}
```
## Tools Used
Manual Review

## Recommendation
Consider deploying a followNFT at profile creation, so profile owner pays for that, while all followers spend same amount of gas to follow.


# Low 002
## Title
Non EOA is allowed to have a profile and follow other profiles

## Impact
I think this is an issue because a user that wants a lot of followers can have many contract accounts follow him. Also, this would lead to bad UX in frontend as there will be a lot of fake profiles.

Given the peculiarity of this protocol, I consider this to be a valid issue.

## Proof of Concept
Looking at LensHub#createProfile function, we can see that there is no check that prevents a non EOA from being the `createProfileParams.to`.
This will allow minting of profiles to a non EOA account.

## Tools Used
Manual Review

## Recommendation
Consider adding an onlyEOA modifier to LensHub#createProfile

