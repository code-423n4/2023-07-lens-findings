---
sponsor: "Lens Protocol"
slug: "2023-07-lens"
date: "2023-12-11"
title: "Lens Protocol V2"
findings: "https://github.com/code-423n4/2023-07-lens-findings/issues"
contest: 263
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Lens Protocol V2 smart contract system written in Solidity. The audit took place between July 17—July 31 2023.

## Wardens

26 Wardens contributed reports to Lens Protocol V2:

  1. [MiloTruck](https://code4rena.com/@MiloTruck)
  2. [juancito](https://code4rena.com/@juancito)
  3. [Emmanuel](https://code4rena.com/@Emmanuel)
  4. [evmboi32](https://code4rena.com/@evmboi32)
  5. [klau5](https://code4rena.com/@klau5)
  6. [maanas](https://code4rena.com/@maanas)
  7. [Limbooo](https://code4rena.com/@Limbooo)
  8. [Prestige](https://code4rena.com/@Prestige)
  9. [Kaysoft](https://code4rena.com/@Kaysoft)
  10. [AlexCzm](https://code4rena.com/@AlexCzm)
  11. [MohammedRizwan](https://code4rena.com/@MohammedRizwan)
  12. [BugzyVonBuggernaut](https://code4rena.com/@BugzyVonBuggernaut)
  13. [Sathish9098](https://code4rena.com/@Sathish9098)
  14. [ihtishamsudo](https://code4rena.com/@ihtishamsudo)
  15. [tnquanghuy0512](https://code4rena.com/@tnquanghuy0512)
  16. [adeolu](https://code4rena.com/@adeolu)
  17. [Rolezn](https://code4rena.com/@Rolezn)
  18. [mrudenko](https://code4rena.com/@mrudenko)
  19. [fatherOfBlocks](https://code4rena.com/@fatherOfBlocks)
  20. [Iurii3](https://code4rena.com/@Iurii3)
  21. [0xAnah](https://code4rena.com/@0xAnah)
  22. [descharre](https://code4rena.com/@descharre)
  23. [Stormreckson](https://code4rena.com/@Stormreckson)
  24. [Bughunter101](https://code4rena.com/@Bughunter101)
  25. [DavidGiladi](https://code4rena.com/@DavidGiladi)
  26. [ginlee](https://code4rena.com/@ginlee)

This audit was judged by [Picodes](https://code4rena.com/@Picodes).

Final report assembled by [liveactionllama](https://twitter.com/liveactionllama).

# Summary

The C4 analysis yielded an aggregated total of 11 unique vulnerabilities. Of these vulnerabilities, 0 received a risk rating in the category of HIGH severity and 11 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 23 reports detailing issues with a risk rating of LOW severity or non-critical.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Lens Protocol V2 audit repository](https://github.com/code-423n4/2023-07-lens), and is composed of 63 smart contracts written in the Solidity programming language and includes 4,108 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **0x6980-bot** from warden 0x6980, generated the [Automated Findings report](https://gist.github.com/thebrittfactor/3dc2ed1320de95e111a25c1744f4d33e), and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# Medium Risk Findings (11)
## [[M-01] Identifying publications using its ID makes the protocol vulnerable to blockchain re-orgs](https://github.com/code-423n4/2023-07-lens-findings/issues/148)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/148)*

In the protocol, publications are uniquely identified through the publisher's profile ID and the publication's ID. For example, when a user calls `act()`, the publication being acted on is determined by `publicationActedProfileId` and `publicationActedId`:

[ActionLib.sol#L23-L26](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol#L23-L26)

```solidity
        Types.Publication storage _actedOnPublication = StorageLib.getPublication(
            publicationActionParams.publicationActedProfileId,
            publicationActionParams.publicationActedId
        );
```

However, as publication IDs are not based on the publication's data, this could cause users to act on the wrong publication in the event a blockchain re-org occurs.

For example:

*   Assume the following transactions occur in separate blocks:
    *   Block 1: Alice calls `post()` to create a post; its publication ID is 20.
    *   Block 2: Bob is interested in the post, he calls `act()` with `publicationActedId = 20` to act on the post.
    *   Block 3: Alice calls `comment()` separately, which creates another publication; its publication ID is 21.
*   A blockchain re-org occurs; block 1 is dropped in place of block 3:
    *   Alice's comment now has the publication ID 20 instead of 21.
*   Bob's call to `act()` in block 2 is applied on top of the re-orged blockchain:
    *   This causes him to act on the comment instead of the post he intended to, as it now has the publication ID 20.

In this scenario, due to the blockchain re-org, Bob calls `act()` on a different publication than the one he wanted. This could have severe impacts depending on the action module being called; if the action module is used to collect and pay fees to the publisher and referrals (eg. [`MultirecipientFeeCollectModule.sol`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/modules/act/collect/MultirecipientFeeCollectModule.sol)), Bob could have lost funds.

Note that this also applies to [`comment()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L247-L255), [`mirror`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L270-L278) and [`quote()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L293-L301), as they can be called with reference modules with sensitive logic as well.

### Impact

If a blockchain re-org occurs, users could potentially act/comment/mirror/quote on the wrong publication, which has varying impacts depending on the action or reference module being used, such as a loss of funds due to paying fees.

Given that Lens Protocol is deployed on Poylgon, which has [experienced large re-orgs in the past](https://forum.polygon.technology/t/157-block-reorg-at-block-height-39599624/11388), the likelihood of the scenario described above occuring due to a blockchain re-org is not low.

### Recommended Mitigation

Consider identifying publications with a method that is dependent on its contents. For example, users could be expected to provide the `keccak256` hash of a publication's contents alongside its publication ID.

This would prevent users from acting on the wrong publication should a publication's contents change despite having the same ID.

**[donosonaumczuk (Lens) disputed and commented](https://github.com/code-423n4/2023-07-lens-findings/issues/148#issuecomment-1669778308):**
 > We disagree with the validity of this issue.
> 
> Transactions will have a nonce (even when meta-txs), so each profile will already have a specific order for each of their publications, which means that the IDs of the publications will be asigned correctly to those profiles, whenever the transactions get confirmed.
> 
> What can happen, is that the re-org ends up executing the "act" transaction before the "post/quote/comment" that is being acted on is created, leading the "act" transaction to revert. It shouldn't be likely to occur.
> 
> For the described issue to happen, a really weird edge case needs to occur, a re-org that also includes a transaction replacement (override) for the "post/quote/comment". This is very unlikely and the harm caused is also not clear.

**[Picodes (judge) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/148#issuecomment-1695853088):**
 > > *Transactions will have a nonce (even when meta-txs), so each profile will already have a specific order for each of their publications, which means that the IDs of the publications will be asigned correctly to those profiles, whenever the transactions get confirmed.*
> 
> As it may happen that multiple addresses have the right to post (for example delegated executors and owners), I think the described scenario is valid and it's possible to have multiple publications being reordered in a different order. So to me this finding is valid. Let me know if I am missing something!

**[donosonaumczuk (Lens) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/148#issuecomment-1699465260):**
 > > *As it may happen that multiple addresses have the right to post (for example delegated executors and owners) I think the described scenario is valid.*
> 
> Yes, I did not consider that detail. What you have said is correct and then the issue is valid.



***

## [[M-02] `tryMigrate()` doesn't ensure that `followerProfileId` isn't already following](https://github.com/code-423n4/2023-07-lens-findings/issues/146)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/146), also found by [Emmanuel](https://github.com/code-423n4/2023-07-lens-findings/issues/117)*

In `FollowNFT.sol`, the `tryMigrate()` function is used to migrate users who were following before the V2 upgrade. It does so by updating `_followTokenIdByFollowerProfileId` and `_followDataByFollowTokenId`, which are state variables introduced in the V2 upgrade:

[FollowNFT.sol#L510-L516](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L510-L516)

```solidity
        _followTokenIdByFollowerProfileId[followerProfileId] = followTokenId;

        uint48 mintTimestamp = uint48(StorageLib.getTokenData(followTokenId).mintTimestamp);

        _followDataByFollowTokenId[followTokenId].followerProfileId = uint160(followerProfileId);
        _followDataByFollowTokenId[followTokenId].originalFollowTimestamp = mintTimestamp;
        _followDataByFollowTokenId[followTokenId].followTimestamp = mintTimestamp;
```

Since `_followTokenIdByFollowerProfileId` is a new state variable, it will be set to 0 for users who were following before the V2 upgrade. This allows old followers to call `follow()` to follow the profile again before `tryMigrate()` is called:

[FollowNFT.sol#L59-L66](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L59-L66)

```solidity
    function follow(
        uint256 followerProfileId,
        address transactionExecutor,
        uint256 followTokenId
    ) external override onlyHub returns (uint256) {
        if (_followTokenIdByFollowerProfileId[followerProfileId] != 0) {
            revert AlreadyFollowing();
        }
```

Even if `tryMigrate()` is called by the protocol team immediately after the V2 upgrade, a malicious user can still call `follow()` before `tryMigrate()` by:

*   Front-running the migration transaction.
*   Holding his profile and follow NFT in different addresses, which causes `tryMigrate()` to return [here](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L501-L504).

As a profile should not be able to follow the same profile twice, `tryMigrate()` should then revert for old followers who have called `follow()`. However, this isn't enforced by `tryMigrate()` as there is no check that `_followDataByFollowTokenId[followerProfileId]` is 0.

As a result, if `tryMigrate()` is called after `follow()`, `_followerCount` will be incremented twice for a single profile:

[FollowNFT.sol#L506-L510](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L506-L510)

```solidity
        unchecked {
            ++_followerCount;
        }

        _followTokenIdByFollowerProfileId[followerProfileId] = followTokenId;
```

Additionally, even though `_followTokenIdByFollowerProfileId` points to a new `followTokenId`, `_followDataByFollowTokenId` will not be cleared for the previous follow token ID.

As the state of the  `FollowNFT` contract is now corrupt, followers can perform functions that they normally should not be able to, such as unfollowing when their profile is not a follower (`isFollowing()` returns `false`).

### Impact

Users who are followers before the V2 upgrade will be able to follow with a single profile twice, causing `followerCount` to be higher than the actual number of profiles following.

Addtionally, as `_followDataByFollowTokenId` is corrupted, followers might be able to call functions when they should not be allowed to, potentially leading to more severe impacts.

### Proof of Concept

The Foundry test below demonstrates that `tryMigrate()` can be called although the user is already following, and how `followerCount` and `_followDataByFollowTokenId` will be corrupted as a result. It can be run with the following command:

    forge test --match-test testCanMigrateWhileFollowing -vvv

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import 'test/base/BaseTest.t.sol';

contract FollowMigration_POC is BaseTest {
    address target = address(0x1337);
    address ALICE;

    uint256 targetProfileId;
    uint256 aliceProfileId;

    uint256 oldFollowTokenId;

    function setUp() public override {
        super.setUp();

        // Setup addresses for Alice
        ALICE = makeAddr("Alice");

        // Create profile for target and Alice 
        targetProfileId = _createProfile(target);
        aliceProfileId = _createProfile(ALICE);

        // Add simulateV1Follow() helper function to FollowNFT implementation
        FollowNFTHelper implementation = new FollowNFTHelper(address(hub));  
        vm.etch(hub.getFollowNFTImpl(), address(implementation).code);
        
        // Follow and unfollow to deploy target's FollowNFT contract
        vm.startPrank(defaultAccount.owner);
        hub.follow(
            defaultAccount.profileId,
            _toUint256Array(targetProfileId),
            _toUint256Array(0),
            _toBytesArray('')
        );
        hub.unfollow(defaultAccount.profileId, _toUint256Array(targetProfileId));
        vm.stopPrank();

        // Get FollowNFT contract
        address followNFTAddress = hub.getProfile(targetProfileId).followNFT; 
        followNFT = FollowNFT(followNFTAddress);

        // Alice follows target before the V2 upgrade
        oldFollowTokenId = FollowNFTHelper(followNFTAddress).simulateV1Follow(ALICE);
    }

    function testCanMigrateWhileFollowing() public {
        // After the V2 upgrade, Alice calls follow() instead of migrating her follow
        vm.startPrank(ALICE);
        uint256 followTokenId = hub.follow(
            aliceProfileId,
            _toUint256Array(targetProfileId),
            _toUint256Array(0),
            _toBytesArray('')
        )[0];

        // Alice migrates her V1 follow even though her profile is already following
        hub.batchMigrateFollows(
            _toUint256Array(aliceProfileId),
            _toUint256Array(targetProfileId),
            _toUint256Array(oldFollowTokenId)
        );

        // followTokenId's followerProfileId points to Alice's profile
        assertEq(followNFT.getFollowerProfileId(followTokenId), aliceProfileId);

        // However, Alice's _followTokenIdByFollowerProfileId points to oldFollowTokenId
        assertEq(followNFT.getFollowTokenId(aliceProfileId), oldFollowTokenId);

        // Follower count is 2 although Alice is the only follower
        assertEq(followNFT.getFollowerCount(), 2);

        // Wrap both follow tokens
        vm.startPrank(ALICE);
        followNFT.wrap(followTokenId);
        followNFT.wrap(oldFollowTokenId);

        // Alice unfollows using removeFollower()
        vm.expectEmit();
        emit Events.Unfollowed(aliceProfileId, targetProfileId, 1);
        followNFT.removeFollower(followTokenId);

        // Alice is no longer following
        assertFalse(followNFT.isFollowing(aliceProfileId));

        // However, she is still able to unfollow for a second time
        vm.expectEmit();
        emit Events.Unfollowed(aliceProfileId, targetProfileId, 1);
        followNFT.removeFollower(oldFollowTokenId);
    }
}

contract FollowNFTHelper is FollowNFT {
    constructor(address hub) FollowNFT(hub) {}

    /*
    Helper function to mimic a V1 follow, which does the following:
    - Increment _tokenIdCounter
    - Mint a followNFT
    */ 
    function simulateV1Follow(address follower) external returns (uint256 followTokenId) {
        followTokenId = ++_lastFollowTokenId;
        _mint(follower, followTokenId);
    }
}
```

</details>

### Recommended Mitigation

[FollowNFT.sol#L480-L489](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L480-L489)

```diff
    function tryMigrate(
        uint256 followerProfileId,
        address followerProfileOwner,
        uint256 idOfProfileFollowed,
        uint256 followTokenId
    ) external onlyHub returns (uint48) {
+       if (_followTokenIdByFollowerProfileId[followerProfileId] != 0) {
+           return 0;
+       }
+
        // Migrated FollowNFTs should have `originalFollowTimestamp` set
        if (_followDataByFollowTokenId[followTokenId].originalFollowTimestamp != 0) {
            return 0; // Already migrated
        }
```

### Assessed type

Upgradable

**[donosonaumczuk (Lens) confirmed](https://github.com/code-423n4/2023-07-lens-findings/issues/146#issuecomment-1668052077)**



***

## [[M-03] Users cannot unfollow if they do not own the FollowNFT of the `followTokenId` used for their profile](https://github.com/code-423n4/2023-07-lens-findings/issues/145)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/145), also found by [maanas](https://github.com/code-423n4/2023-07-lens-findings/issues/153) and [Prestige](https://github.com/code-423n4/2023-07-lens-findings/issues/89)*

If the `followTokenId` of a profile is wrapped, users will only be able to unfollow if they are either:

1.  The owner of the follow NFT.
2.  An approved operator of the follow NFT's owner.

This can be seen in the `unfollow()` function of `FollowNFT.sol`:

[FollowNFT.sol#L115-L125](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L115-L125)

```solidity
            // Follow token is wrapped.
            address unfollowerProfileOwner = IERC721(HUB).ownerOf(unfollowerProfileId);
            // Follower profile owner or its approved delegated executor must hold the token or be approved-for-all.
            if (
                (followTokenOwner != unfollowerProfileOwner) &&
                (followTokenOwner != transactionExecutor) &&
                !isApprovedForAll(followTokenOwner, transactionExecutor) &&
                !isApprovedForAll(followTokenOwner, unfollowerProfileOwner)
            ) {
                revert DoesNotHavePermissions();
            }
```

As seen from above, users that are not the owner or do not have approval for the wrapped follow NFT will not be able to unfollow. This is problematic as users are able to follow with a `followTokenId` without owning the corresponding follow NFT.

For example, someone who holds a follow NFT can call [`approveFollow()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L141-L153) for a user. The user can then call `follow()` with the corresponding `followTokenId`, which works as `_followWithWrappedToken()` checks for follow approval:

[FollowNFT.sol#L317-L327](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L317-L327)

```solidity
        bool isFollowApproved = _followApprovalByFollowTokenId[followTokenId] == followerProfileId;
        address followerProfileOwner = IERC721(HUB).ownerOf(followerProfileId);
        if (
            !isFollowApproved &&
            followTokenOwner != followerProfileOwner &&
            followTokenOwner != transactionExecutor &&
            !isApprovedForAll(followTokenOwner, transactionExecutor) &&
            !isApprovedForAll(followTokenOwner, followerProfileOwner)
        ) {
            revert DoesNotHavePermissions();
        }
```

Now, if the user wants to unfollow, he will be unable to do so by himself, and is forced to rely on the follow NFT owner to unfollow for his profile.

### Impact

Users that follow using a wrapped `followTokenId` that they do not own will not be unfollow the profile. This is incorrect as a profile owner should have full control over who the profile does/does not follow.

### Proof of Concept

The Foundry test below demonstrates that `unfollow()` will revert when users do not own the FollowNFT, even when unfollowing with their own profile. It can be run with the following command:

    forge test --match-test testCannotUnfollowWithoutFollowNFT -vvv

<details>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import 'test/base/BaseTest.t.sol';
import 'contracts/interfaces/IFollowNFT.sol';

contract Unfollow_POC is BaseTest {
    address targetProfileOwner;
    address ALICE;
    address BOB;

    uint256 targetProfileId;
    uint256 aliceProfileId;
    uint256 bobProfileId;

    function setUp() public override {
        super.setUp();

        // Setup addresses for target, Alice and Bob
        targetProfileOwner = makeAddr("Target");
        ALICE = makeAddr("Alice");
        BOB = makeAddr("Bob");

        // Create profile for target, Alice and Bob 
        targetProfileId = _createProfile(targetProfileOwner);
        aliceProfileId = _createProfile(ALICE);
        bobProfileId = _createProfile(BOB);
    }

    function testCannotUnfollowWithoutFollowNFT() public {
        // Alice follows target
        vm.startPrank(ALICE);
        uint256 followTokenId = hub.follow(
            aliceProfileId,
            _toUint256Array(targetProfileId),
            _toUint256Array(0),
            _toBytesArray('')
        )[0];

        // Get followNFT contract created
        FollowNFT followNFT = FollowNFT(hub.getProfile(targetProfileId).followNFT);

        // Alice lets Bob follow using her followTokenId
        followNFT.wrap(followTokenId);
        followNFT.approveFollow(bobProfileId, followTokenId);
        vm.stopPrank();

        // Bob follows using her followTokenId        
        vm.startPrank(BOB);
        hub.follow(
            bobProfileId,
            _toUint256Array(targetProfileId),
            _toUint256Array(followTokenId),
            _toBytesArray('')
        );
        assertTrue(followNFT.isFollowing(bobProfileId));

        // After a while, Bob wants to unfollow. 
        // However, unfollow() reverts as he doesn't own the followNFT
        vm.expectRevert(IFollowNFT.DoesNotHavePermissions.selector);
        hub.unfollow(bobProfileId, _toUint256Array(targetProfileId));
        vm.stopPrank();
    }
}
```

</details>

### Recommended Mitigation

In `unfollow()`, consider allowing the owner of `unfollowerProfileId` to unfollow as well:

[FollowNFT.sol#L115-L125](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L115-L125)

```diff
            // Follow token is wrapped.
            address unfollowerProfileOwner = IERC721(HUB).ownerOf(unfollowerProfileId);
            // Follower profile owner or its approved delegated executor must hold the token or be approved-for-all.
            if (
+               transactionExecutor != unfollowerProfileOwner && 
                (followTokenOwner != unfollowerProfileOwner) &&
                (followTokenOwner != transactionExecutor) &&
                !isApprovedForAll(followTokenOwner, transactionExecutor) &&
                !isApprovedForAll(followTokenOwner, unfollowerProfileOwner)
            ) {
                revert DoesNotHavePermissions();
            }
```

### Assessed type

Access Control

**[donosonaumczuk (Lens) disagreed with severity and commented](https://github.com/code-423n4/2023-07-lens-findings/issues/145#issuecomment-1668105153):**
 > We confirm the issue. However, we are still debating if it is a Medium severity one or if it should be classified as Low.
>
> We've marked it as `disagree with the severity`, so we can discuss it better with the judge.

**[Picodes (judge) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/145#issuecomment-1695819492):**
 > Some comments :
>  - The comments on [`approveFollow`](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/interfaces/IFollowNFT.sol#L72) and [`unfollow`](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L117) are quite clear on the fact that `approveFollow` doesn't give approval to unfollow.
>  - But the argument that a functionality is broken as the `unfollowerProfileOwner` should be able to perform the operation seems valid to me and would be of Med severity under `function of the protocol or its availability could be impacted`.
>  
>  Overall Medium severity seems appropriate here.



***

## [[M-04] Users can unfollow through `FollowNFT` contract when LensHub is paused by governance](https://github.com/code-423n4/2023-07-lens-findings/issues/144)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/144), also found by [juancito](https://github.com/code-423n4/2023-07-lens-findings/issues/108)*

When the `LensHub` contract has been paused by governance (`_state` set to `ProtocolState.Paused`), users should not be able unfollow profiles. This can be inferred as the `unfollow()` function has the `whenNotPaused` modifier:

[LensHub.sol#L368-L371](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L368-L371)

```solidity
    function unfollow(uint256 unfollowerProfileId, uint256[] calldata idsOfProfilesToUnfollow)
        external
        override
        whenNotPaused
```

However, in the `FollowNFT` contract, which is deployed for each profile that has followers, the `removeFollower()` and `burn()` functions do not check if the `LensHub` contract is paused:

[FollowNFT.sol#L131-L138](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L131-L138)

```solidity
    function removeFollower(uint256 followTokenId) external override {
        address followTokenOwner = ownerOf(followTokenId);
        if (followTokenOwner == msg.sender || isApprovedForAll(followTokenOwner, msg.sender)) {
            _unfollowIfHasFollower(followTokenId);
        } else {
            revert DoesNotHavePermissions();
        }
    }
```

[FollowNFT.sol#L255-L258](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L255-L258)

```solidity
    function burn(uint256 followTokenId) public override {
        _unfollowIfHasFollower(followTokenId);
        super.burn(followTokenId);
    }
```

As such, whenever the system has been paused by governance, users will still be able to unfollow profiles by wrapping their followNFT and then calling either `removeFollower()` or `burn()`.

### Impact

Users are able to unfollow profiles when the system is paused, which they should not be able to do.

This could be problematic if governance ever needs to temporarily pause unfollow functionality (eg. for a future upgrade, or unfollowing functionality has a bug, etc...).

### Proof of Concept

The Foundry test below demonstrates how users will still be able to unfollow profiles by calling `wrap()` and `removeFollower()`, even after the system has been paused by governance. It can be run with the following command:

    forge test --match-test testCanUnfollowWhilePaused -vvv

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import 'test/base/BaseTest.t.sol';

contract Unfollow_POC is BaseTest {
    address targetProfileOwner;
    uint256 targetProfileId;
    FollowNFT targetFollowNFT;

    address follower;
    uint256 followerProfileId;
    uint256 followTokenId;

    function setUp() public override {
        super.setUp();

        // Create profile for target
        targetProfileOwner = makeAddr("Target");
        targetProfileId = _createProfile(targetProfileOwner);

        // Create profile for follower
        follower = makeAddr("Follower");
        followerProfileId = _createProfile(follower);

        // Follower follows target
        vm.prank(follower);
        followTokenId = hub.follow(
            followerProfileId,
            _toUint256Array(targetProfileId),
            _toUint256Array(0),
            _toBytesArray('')
        )[0];
        targetFollowNFT = FollowNFT(hub.getProfile(targetProfileId).followNFT);
    }

    function testCanUnfollowWhilePaused() public {
        // Governance pauses system
        vm.prank(governance);
        hub.setState(Types.ProtocolState.Paused);
        assertEq(uint8(hub.getState()), uint8(Types.ProtocolState.Paused));

        // unfollow() reverts as system is paused
        vm.startPrank(follower);
        vm.expectRevert(Errors.Paused.selector);
        hub.unfollow(followerProfileId, _toUint256Array(targetProfileId));

        // However, follower can still unfollow through FollowNFT contract 
        targetFollowNFT.wrap(followTokenId);
        targetFollowNFT.removeFollower(followTokenId);        
        vm.stopPrank();

        // follower isn't following anymore
        assertFalse(targetFollowNFT.isFollowing(followerProfileId));
    }
}
```

### Recommended Mitigation

All `FollowNFT` contracts should check that the `LensHub` contract isn't paused before allowing `removeFollower()` or `burn()` to be called. This can be achieved by doing the following:

1.  Add a `whenNotPaused` modifier to `FollowNFT.sol`:

```solidity
modifier whenNotPaused() {
    if (ILensHub(HUB).getState() == Types.ProtocolState.Paused) {
        revert Errors.Paused();
    }
    _;
}
```

2.  Use the modifier on `removeFollower()` and `burn()`:

[FollowNFT.sol#L131-L138](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L131-L138)

```diff
-   function removeFollower(uint256 followTokenId) external override {
+   function removeFollower(uint256 followTokenId) external override whenNotPaused {
        // Some code here...
    }
```

[FollowNFT.sol#L255-L258](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L255-L258)

```diff
-   function burn(uint256 followTokenId) public override {
+   function burn(uint256 followTokenId) public override whenNotPaused {
        // Some code here...
    }
```

### Assessed type

Access Control

**[donosonaumczuk (Lens) disagreed with severity and commented](https://github.com/code-423n4/2023-07-lens-findings/issues/144#issuecomment-1669794975):**
 > This report is a subset of this [issue 108](https://github.com/code-423n4/2023-07-lens-findings/issues/108).<br>
> Same resolution, we accept it but we disagree with the severity. It should be Low.

**[Picodes (judge) decreased severity to Low](https://github.com/code-423n4/2023-07-lens-findings/issues/144#issuecomment-1696117127)**

**[juancito (warden) commented via duplicate issue `#108`](https://github.com/code-423n4/2023-07-lens-findings/issues/108#issuecomment-1696721097):**
> Hi @Picodes. I'd like to ask if you could take a second look at this issue as a Medium risk finding, considering the [Docs](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization#estimating-risk):
>
> > *2 — Med: Assets not at direct risk, but **the function of the protocol** or its availability **could be impacted***
>
> In this case the "function of the protocol" is impacted, and availability can also be considered as well. Not for being unavailable, but because of functions being available in moments that they shouldn't be, allowing **important actions for a social network** that users should not be able to perform, out of the control of the protocol.
> 
> When the protocol is paused, it should not allow unfollow actions (via `removeFollower()`) or new follows (via `batchMigrateProfiles()` for example, among all the other mentioned functions on the Impact section.
> 
> In DeFi protocols, missing pause modifiers having been evaluated as Medium like [here](https://github.com/code-423n4/2022-02-aave-lens-findings/issues/71), and [here](https://github.com/code-423n4/2023-03-polynomial-findings/issues/232). In the case of Lens Protocol, the functions mentioned on the Impact section should be considered of ultimate importance to have under control as a social network.

**[donosonaumczuk (Lens) commented via duplicate issue `#108`](https://github.com/code-423n4/2023-07-lens-findings/issues/108#issuecomment-1699472401):**
 > Yeah, I think it is a fair argument and can be upgraded to Medium.

**[Picodes (judge) increased severity to Medium and commented via duplicate issue `#108`](https://github.com/code-423n4/2023-07-lens-findings/issues/108#issuecomment-1701376854):**
> My view on this is that it ultimately depends on the sponsor's intent. In this case, it seems clear by the above comment and what you highlighted that the intent was to be able to totally pause follows and unfollows. So you're right and I'll upgrade this to Medium as a functionality is broken.



***

## [[M-05] Whitelisted profile creators could accidentally break migration for V1 profiles](https://github.com/code-423n4/2023-07-lens-findings/issues/143)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/143), also found by [maanas](https://github.com/code-423n4/2023-07-lens-findings/issues/170) and [juancito](https://github.com/code-423n4/2023-07-lens-findings/issues/105)*

Profiles that exist before the V2 upgrade are migrated using the [`batchMigrateProfiles()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2Migration.sol#L33-L35) function, which works by minting the profile's handle and linking it to their profile:

[MigrationLib.sol#L69-L85](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol#L69-L85)

```solidity
            string memory handle = StorageLib.getProfile(profileId).__DEPRECATED__handle;
            if (bytes(handle).length == 0) {
                return; // Already migrated
            }
            bytes32 handleHash = keccak256(bytes(handle));
            // We check if the profile is the "lensprotocol" profile by checking profileId != 1.
            // "lensprotocol" is the only edge case without the .lens suffix:
            if (profileId != LENS_PROTOCOL_PROFILE_ID) {
                assembly {
                    let handle_length := mload(handle)
                    mstore(handle, sub(handle_length, DOT_LENS_SUFFIX_LENGTH)) // Cut 5 chars (.lens) from the end
                }
            }
            // We mint a new handle on the LensHandles contract. The resulting handle NFT is sent to the profile owner.
            uint256 handleId = lensHandles.migrateHandle(profileOwner, handle);
            // We link it to the profile in the TokenHandleRegistry contract.
            tokenHandleRegistry.migrationLink(handleId, profileId);
```

For example, a profile with the handle "alice.lens" will receive an "alice" LensHandles NFT post-migration.

However, whitelisted profile creators are able to mint any handle using [`mintHandle()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L87-L94) in the `LensHandles` contract. This makes it possible for any whitelisted profile creator to mint a handle corresponding to a V1 profile before the profile is migrated.

If this occurs, `batchMigrateProfiles()` will always revert for the corresponding V1 profile as the same handle cannot be minted twice, thereby breaking migration for that profile.

### Impact

If a whitelisted profile creator accidentally mints a handle that already belongs to a V1 profile, that profile cannot be migrated.

### Proof of Concept

The Foundry test below demonstrates how `batchMigrateProfiles()` will revert if a V1 profile's handle has already been minted. It can be run with the following command:

    forge test --match-test testProfileCreatorCanBreakProfileMigration -vvv

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import 'test/base/BaseTest.t.sol';

contract ProfileMigration_POC is BaseTest {
    LensHubHelper hubProxy;

    function setUp() public override {
        super.setUp();
        
        // Add toLegacyV1Profile() function to LensHub
        LensHubHelper implementation = new LensHubHelper({
            lensHandlesAddress: address(lensHandles),
            tokenHandleRegistryAddress: address(tokenHandleRegistry),
            tokenGuardianCooldown: PROFILE_GUARDIAN_COOLDOWN
        });
        vm.prank(deployer);
        hubAsProxy.upgradeTo(address(implementation));

        // Cast proxy to LensHubHelper interface
        hubProxy = LensHubHelper(address(hubAsProxy));
    }

    function testProfileCreatorCanBreakProfileMigration() public {
        // Create a v1 profile with the handle "alice.lens"
        uint256 profileId = _createProfile(address(this));
        hubProxy.toLegacyV1Profile(profileId, "alice.lens");

        // Whitelisted profile creator accidentally mints a "alice.lens" handle
        vm.prank(lensHandles.OWNER());
        lensHandles.mintHandle(address(this), "alice");

        // V1 profile will revert when migrated as the handle already exists
        vm.expectRevert("ERC721: token already minted");
        hubProxy.batchMigrateProfiles(_toUint256Array(profileId));
    }
}

contract LensHubHelper is LensHub {
    constructor(
        address lensHandlesAddress,
        address tokenHandleRegistryAddress,
        uint256 tokenGuardianCooldown
    ) LensHub(
        address(0),
        address(0),
        address(0),
        lensHandlesAddress,
        tokenHandleRegistryAddress,
        address(0),
        address(0),
        address(0),
        tokenGuardianCooldown
    ) {}


    function toLegacyV1Profile(uint256 profileId, string memory handle) external {
        Types.Profile storage profile = StorageLib.getProfile(profileId);
        profile.__DEPRECATED__handle = handle;
        delete profile.metadataURI;
    }
}
```

### Recommended Mitigation

Ensure that the handle of a V1 profile cannot be minted through `mintHandle()`. This validation will probably have to be done off-chain, as it is unfeasible to check all existing handles on-chain with a reasonable gas cost.

### Assessed type

Upgradable

**[donosonaumczuk (Lens) confirmed commented](https://github.com/code-423n4/2023-07-lens-findings/issues/143#issuecomment-1668126368):**
 > We found this issue once the audit was already in progress, so we weren't allowed to push it, but we already mitigated it by adding this function in the LensHub:
> 
> ```solidity
>     function getProfileIdByHandleHash(bytes32 handleHash) external view returns (uint256) {
>         return StorageLib.profileIdByHandleHash()[handleHash];
>     }
> ```
> 
> And then making the ProfileCreationProxy to validate against it:
> ```solidity
>     function proxyCreateProfileWithHandle(
>         Types.CreateProfileParams memory createProfileParams,
>         string calldata handle
>     ) external onlyOwner returns (uint256, uint256) {
>         // Check if LensHubV1 already has a profile with this handle that was not migrated yet:
>         bytes32 handleHash = keccak256(bytes(string.concat(handle, '.lens')));
>         if (LensV2Migration(LENS_HUB).getProfileIdByHandleHash(handleHash) != 0) {
>             revert ProfileAlreadyExists();
>         }
>         
>         // ...
>      }
> ```
> 
> Note that we add the validation at ProfileCreationProxy instead of LensHub, as we don't want LensHub to "be aware" of the Handles, architecturally-wise.



***

## [[M-06] Inconsistent encoding of arrays in `MetaTxLib`](https://github.com/code-423n4/2023-07-lens-findings/issues/142)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/142), also found by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/140) and [evmboi32](https://github.com/code-423n4/2023-07-lens-findings/issues/190)*

<https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L143-L153><br>
<https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L100-L109>

According to the [EIP-712](https://eips.ethereum.org/EIPS/eip-712) specification, arrays are encoded by concatenating its elements and passing the result to `keccak256`:

> The array values are encoded as the `keccak256` hash of the concatenated encodeData of their contents (i.e. the encoding of `SomeType[5]` is identical to that of a struct containing five members of type `SomeType`).

An example of a correct implementation can be seen in [`validateUnfollowSignature()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L357-L376), where the `idsOfProfilesToUnfollow` array is passed to `keccak256` after using `abi.encodePacked()`:

[MetaTxLib.sol#L368](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L368)

```solidity
                        keccak256(abi.encodePacked(idsOfProfilesToUnfollow)),
```

However, some other functions in `MetaTxLib` encode arrays differently, which differs from the EIP-712 specification.

Some functions do not encode the array at all, and pass the array to `abi.encode()` alongside other arguments in its struct. For example, in `validatePostSignature()`, the `postParams.actionModules` array is not encoded by itself:

[MetaTxLib.sol#L143-L153](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L143-L153)

```solidity
                    abi.encode(
                        Typehash.POST,
                        postParams.profileId,
                        keccak256(bytes(postParams.contentURI)),
                        postParams.actionModules,
                        _hashActionModulesInitDatas(postParams.actionModulesInitDatas),
                        postParams.referenceModule,
                        keccak256(postParams.referenceModuleInitData),
                        _getAndIncrementNonce(signature.signer),
                        signature.deadline
                    )
```

Other instances of this include:

*   `mirrorParams.referrerProfileIds` and `mirrorParams.referrerPubIds` in [`validateMirrorSignature()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L297-L298)
*   `publicationActionParams.referrerProfileIds` and `publicationActionParams.referrerPubIds` in [`validateActSignature()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L437-L438)
*   All functions that use [`_abiEncode()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L190-L223), namely [`validateCommentSignature()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L225-L254) and [`validateQuoteSignature()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L256-L284)

Secondly, the `validateChangeDelegatedExecutorsConfigSignature()` function encodes the `delegatedExecutors` and `approvals` arrays using `abi.encodePacked()`, but do not pass it to `keccak256`:

[MetaTxLib.sol#L100-L109](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L100-L109)

```solidity
                    abi.encode(
                        Typehash.CHANGE_DELEGATED_EXECUTORS_CONFIG,
                        delegatorProfileId,
                        abi.encodePacked(delegatedExecutors),
                        abi.encodePacked(approvals),
                        configNumber,
                        switchToGivenConfig,
                        nonce,
                        deadline
                    )
```

### Impact

As arrays are encoded incorrectly, the signature verification in the functions listed above is not [EIP-712](https://eips.ethereum.org/EIPS/eip-712) compliant.

Contracts or dapps/backends that encode arrays according to the rules specified in [EIP-712](https://eips.ethereum.org/EIPS/eip-712) will end up with different signatures, causing any of the functions listed above to revert when called.

Moreover, the inconsistent encoding of arrays might be extremely confusing to developers who wish to use these functions to implement meta-transactions.

### Recommended Mitigation

Consider encoding arrays correctly in the functions listed above, which can be achieved by calling `abi.encodePacked()` on the array and passing its results to `keccak256`.

**[donosonaumczuk (Lens) disagreed with severity and commented](https://github.com/code-423n4/2023-07-lens-findings/issues/142#issuecomment-1669806824):**
 > We confirm it, but we think this should be Low instead.

**[Picodes (judge) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/142#issuecomment-1696111170):**
 > Following the same reasoning as in [M-07 (issue 141)](https://github.com/code-423n4/2023-07-lens-findings/issues/141), I'll keep Medium severity here as EIP compliance is of great importance for integrators and compatibility, so I consider this an instance of "function of the protocol \[is] impacted", the function being the EIP712 compliance.



***

## [[M-07] EIP-712 typehash is incorrect for several functions in `MetaTxLib`](https://github.com/code-423n4/2023-07-lens-findings/issues/141)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/141), also found by [juancito](https://github.com/code-423n4/2023-07-lens-findings/issues/107)*

In `LensHub.sol`, the second parameter of `setProfileMetadataURIWithSig()` is declared as `metadataURI`:

[LensHub.sol#L119-L123](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L119-L123)

```solidity
    function setProfileMetadataURIWithSig(
        uint256 profileId,
        string calldata metadataURI,
        Types.EIP712Signature calldata signature
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(signature.signer, profileId) {
```

However, its [EIP-712](https://eips.ethereum.org/EIPS/eip-712) typehash stores the parameter as `metadata` instead:

[Typehash.sol#L33](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Typehash.sol#L33)

```solidity
bytes32 constant SET_PROFILE_METADATA_URI = keccak256('SetProfileMetadataURI(uint256 profileId,string metadata,uint256 nonce,uint256 deadline)');
```

The `PostParams` struct (which is used for [`postWithSig()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L235-L244)) has `address[] actionModules` and `bytes[] actionModulesInitDatas` as its third and fourth fields:

[Types.sol#L178-L185](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Types.sol#L178-L185)

```solidity
    struct PostParams {
        uint256 profileId;
        string contentURI;
        address[] actionModules;
        bytes[] actionModulesInitDatas;
        address referenceModule;
        bytes referenceModuleInitData;
    }
```

However, the third and fourth fields in its typehash are declared as `address collectModule` and `bytes collectModuleInitData` instead:

[Typehash.sol#L23](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Typehash.sol#L23)

```solidity
bytes32 constant POST = keccak256('Post(uint256 profileId,string contentURI,address collectModule,bytes collectModuleInitData,address referenceModule,bytes referenceModuleInitData,uint256 nonce,uint256 deadline)');
```

This occurs for the [`commentWithSig()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L258-L267) and [`quoteWithSig()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L303-L313) functions as well:

[Typehash.sol#L25](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Typehash.sol#L25)

```solidity
bytes32 constant QUOTE = keccak256('Quote(uint256 profileId,string contentURI,uint256 pointedProfileId,uint256 pointedPubId,uint256[] referrerProfileIds,uint256[] referrerPubIds,bytes referenceModuleData,address collectModule,bytes collectModuleInitData,address referenceModule,bytes referenceModuleInitData,uint256 nonce,uint256 deadline)');
```

[Typehash.sol#L15](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Typehash.sol#L15)

```solidity
bytes32 constant COMMENT = keccak256('Comment(uint256 profileId,string contentURI,uint256 pointedProfileId,uint256 pointedPubId,uint256[] referrerProfileIds,uint256[] referrerPubIds,bytes referenceModuleData,address collectModule,bytes collectModuleInitData,address referenceModule,bytes referenceModuleInitData,uint256 nonce,uint256 deadline)');
```

The fourth and fifth fields in the `MirrorParams` struct (which is used for [`mirrorWithSig()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L281-L290)) are declared as `referrerProfileIds` and `referrerPubIds`:

[Types.sol#L282-L289](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Types.sol#L282-L289)

```solidity
    struct MirrorParams {
        uint256 profileId;
        uint256 pointedProfileId;
        uint256 pointedPubId;
        uint256[] referrerProfileIds;
        uint256[] referrerPubIds;
        bytes referenceModuleData;
    }
```

However, its EIP-712 typehash declares these fields as `referrerProfileId` and `referrerPubId` instead:

[Typehash.sol#L21](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Typehash.sol#L21)

```solidity
bytes32 constant MIRROR = keccak256('Mirror(uint256 profileId,uint256 pointedProfileId,uint256 pointedPubId,uint256[] referrerProfileId,uint256[] referrerPubId,bytes referenceModuleData,uint256 nonce,uint256 deadline)');
```

### Impact

Due to the use of incorrect typehashes, the signature verification in the functions listed above is not [EIP-712](https://eips.ethereum.org/EIPS/eip-712) compliant.

Contracts or dapps/backends that use "correct" typehashes that match the parameters of these functions will end up generating different signatures, causing them to revert when called.

### Recommended Mitigation

Amend the typehashes shown above to have matching parameters with their respective functions.

### Assessed type

Error

**[donosonaumczuk (Lens) disagreed with severity and commented](https://github.com/code-423n4/2023-07-lens-findings/issues/141#issuecomment-1669811529):**
 > We accept it, but we consider it Low severity instead. "Assets are not at risk, function incorrect as to spec".

**[Picodes (judge) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/141#issuecomment-1695727736):**
 > Keeping this under Medium severity as this breaks the EIP712 compliance, so can be seen as an instance of "function of the protocol or its availability could be impacted"



***

## [[M-08] Token guardian protection doesn't account for approved operators in `approve()`](https://github.com/code-423n4/2023-07-lens-findings/issues/136)
*Submitted by [MiloTruck](https://github.com/code-423n4/2023-07-lens-findings/issues/136), also found by [Limbooo](https://github.com/code-423n4/2023-07-lens-findings/issues/90)*

According to the [README](https://github.com/code-423n4/2023-07-lens/tree/main#definition-of-words-and-terms-in-lens-codebase), when an address has token guardian enabled, approvals should not work for the tokens owned by that address:

> **Token Guardian:** Protection mechanism for the tokens held by an address, which restricts transfers and approvals when enabled. See [LIP-4](https://github.com/lens-protocol/LIPs/blob/main/LIPs/lip-4.md) for more.

In `LensHandles.sol`, token guardian is enforced by the `_hasTokenGuardianEnabled()` check in the `approve()` function:

[LensHandles.sol#L139-L145](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L139-L145)

```solidity
    function approve(address to, uint256 tokenId) public override(IERC721, ERC721) {
        // We allow removing approvals even if the wallet has the token guardian enabled
        if (to != address(0) && _hasTokenGuardianEnabled(msg.sender)) {
            revert HandlesErrors.GuardianEnabled();
        }
        super.approve(to, tokenId);
    }
```

However, this check is inadequate as approved operators (addresses approved using `setApprovalForAll()` by the owner) are also allowed to call `approve()`. We can see this in Openzeppelin's ERC-721 implementation:

[ERC721.sol#L116-L119](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.8/contracts/token/ERC721/ERC721.sol#L116-L119)

```solidity
        require(
            _msgSender() == owner || isApprovedForAll(owner, _msgSender()),
            "ERC721: approve caller is not token owner or approved for all"
        );
```

As such, even if an owner has token guardian enabled, approvals can still be set for his tokens by other approved operators, leaving the owner's tokens vulnerable. For example:

*   Alice sets Bob as an approved operator using `setApprovalForAll()`.
*   Alice enables token guardian using `enableTokenGuardian()`.
*   If Bob wants to set approvals for Alice's tokens, he can do so by:
    *   Disabling his own token guardian using `DANGER__disableTokenGuardian()`.
    *   Calling `approve()` for Alice's tokens. This will still work, even though Alice has token guardian enabled.

Note that the `approve()` function in `LensProfiles.sol` also has the same vulnerability.

### Impact

As token guardian protection in `approve()` does not account for approved operators, although an owner has token guardian enabled, approved operators will still be able to set approvals for his tokens.

### Proof of Concept

The following Foundry test demonstrates the example above:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import 'forge-std/Test.sol';
import '../contracts/namespaces/LensHandles.sol';

contract TokenGuardian_POC is Test {
    LensHandles lensHandles;

    address ALICE;
    address BOB;
    uint256 tokenId;

    function setUp() public {
        // Setup LensHandles contract
        lensHandles = new LensHandles(address(this), address(0), 0);

        // Setup Alice and Bob addresses
        ALICE = makeAddr("Alice");
        BOB = makeAddr("Bob");

        // Mint "alice.lens" to Alice
        tokenId = lensHandles.mintHandle(ALICE, "alice");
    }
    
    function testCanApproveWhileTokenGuardianEnabled() public {
        // Alice disables tokenGuardian to set Bob as an approved operator
        vm.startPrank(ALICE);
        lensHandles.DANGER__disableTokenGuardian();
        lensHandles.setApprovalForAll(BOB, true);

        // Alice re-enables tokenGuardian
        lensHandles.enableTokenGuardian();
        vm.stopPrank();

        // Bob disables tokenGuardian for himself
        vm.startPrank(BOB);
        lensHandles.DANGER__disableTokenGuardian();

        // Alice still has tokenGuardian enabled
        assertEq(lensHandles.getTokenGuardianDisablingTimestamp(ALICE), 0);

        // However, Bob can still set approvals for Alice's handle
        lensHandles.approve(address(0x1337), tokenId);
        vm.stopPrank();
        assertEq(lensHandles.getApproved(tokenId), address(0x1337));
    }
}
```

### Recommended Mitigation

Consider checking if the token's owner has token guardian enabled as well:

[LensHandles.sol#L139-L145](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L139-L145)

```diff
    function approve(address to, uint256 tokenId) public override(IERC721, ERC721) {
        // We allow removing approvals even if the wallet has the token guardian enabled
-       if (to != address(0) && _hasTokenGuardianEnabled(msg.sender)) {
+       if (to != address(0) && (_hasTokenGuardianEnabled(msg.sender) || _hasTokenGuardianEnabled(_ownerOf(tokenId)))) {
            revert HandlesErrors.GuardianEnabled();
        }
        super.approve(to, tokenId);
    }
```

### Assessed type

Access Control

**[donosonaumczuk (Lens) commented via duplicate issue `#90`](https://github.com/code-423n4/2023-07-lens-findings/issues/90#issuecomment-1667731698):**
 > This is Medium as assets cannot be stolen directly, but it requires a hypothetical attack path with stated assumptions.
> 
> Note that the transferFrom function is not compromised itself, thus the victim could revoke the approval of the operator and the approvals from the assets before disabling the token guardian, and be safe.



***

## [[M-09] Users can self-follow via `FollowNFT::tryMigrate()` on Lens V2](https://github.com/code-423n4/2023-07-lens-findings/issues/106)
*Submitted by [juancito](https://github.com/code-423n4/2023-07-lens-findings/issues/106), also found by [evmboi32](https://github.com/code-423n4/2023-07-lens-findings/issues/60) and [Emmanuel](https://github.com/code-423n4/2023-07-lens-findings/issues/189)*

Users are not supposed to be able to self-follow on Lens v2, but they are able to bypass the restriction. This can also affect modules or newer functionalities that count on this behaviour.

Migration is an [Area of specific concern](https://github.com/code-423n4/2023-07-lens#areas-of-specific-concern) for the devs, and this can easily be prevented with a simple check.

This can't be undone without any upgrade.

### Proof of Concept

`FollowLib::follow()` has a specific restriction to revert when a user tries to self-follow on Lens v2:

```solidity
    if (followerProfileId == idsOfProfilesToFollow[i]) {
        revert Errors.SelfFollow();
    }
```

[FollowLib.sol#L35-L37](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/FollowLib.sol#L35-L37)

However, users that own a follow NFT from V1 can execute `FollowNFT::tryMigrate()` to self-follow on V2, as there is no restriction to prevent it. A test proving it can be found on the next section.

[FollowNFT.sol#L480-L520](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L480-L520)

### Coded POC

Add this test to `test/migrations/Migrations.t.sol` and run `TESTING_FORK=mainnet POLYGON_RPC_URL="https://polygon.llamarpc.com" forge test --mt "testSelfFollow"`.

Note: In case of a memory allocation error during the Forge test, please comment [these lines](https://github.com/code-423n4/2023-07-lens/blob/main/test/migrations/Migrations.t.sol#L105-L116). They are not used for the current test.

```solidity
    function testSelfFollow() public onlyFork {
        uint256 selfFollowProfileId = 3659; // juancito.lens
        uint256 selfFollowTokenId = 42;     // juancito.lens follows juancito.lens on V1

        FollowNFT nft = FollowNFT(hub.getProfile(selfFollowProfileId).followNFT);
        address user = nft.ownerOf(selfFollowTokenId); // Owner of juancito.lens

        // 1. Migrate the self-follow
        uint256[] memory selfFollowProfileIdArray = new uint256[](1);
        uint256[] memory selfFollowTokenIdArray = new uint256[](1);

        selfFollowProfileIdArray[0] = selfFollowProfileId; // 3659
        selfFollowTokenIdArray[0] = selfFollowTokenId;     // 42

        hub.batchMigrateFollows(selfFollowProfileIdArray, selfFollowProfileIdArray, selfFollowTokenIdArray);

        // 2. The user is self-following on V2
        assertTrue(nft.isFollowing(selfFollowProfileId));
    }
```

### Recommended Mitigation Steps

Add the following validation to `FollowNFT::tryMigrate()`:

```diff
+    if (followerProfileId == _followedProfileId) {
+        return 0;
+    }
```

### Assessed type

Invalid Validation

**[vicnaum (Lens) confirmed and commented](https://github.com/code-423n4/2023-07-lens-findings/issues/106#issuecomment-1668156711):**
 > This seems like a sub-case of [issue 112](https://github.com/code-423n4/2023-07-lens-findings/issues/112).<br>
> (But the mitigation is different for this case)

**[Picodes (judge) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/106#issuecomment-1695960517):**
 > The impact is the same but the issue seems different, as the mitigation suggested by `#112` wouldn't prevent this from happening.



***

## [[M-10] Users can make any user follow them via `FollowNFT::tryMigrate()` without their consent](https://github.com/code-423n4/2023-07-lens-findings/issues/104)
*Submitted by [juancito](https://github.com/code-423n4/2023-07-lens-findings/issues/104), also found by [Emmanuel](https://github.com/code-423n4/2023-07-lens-findings/issues/112)*

Follows in Lens v2 can be checked via `isFollowing()`, which returns the internal storage variable `_followTokenIdByFollowerProfileId[followerProfileId]` and can't be manipulated by an unauthorized party via any of the follow/unfollow functions, as they have proper access control checks.

This is also prevented on token transfers in Lens v2. If user A wraps their follow token and transfer it to user B, it doesn't mean that user B is following user A.

But using the `tryMigrate()` it is possible to "force" someone to follow another user without their consent.

### Impact

Users can make any other user follow them.

For a social network this could even be considered a high severity issue, as follow actions are a core component of them, and migrations were marked as an [Area of specific concern](https://github.com/code-423n4/2023-07-lens#areas-of-specific-concern) by the devs.

Unauthorized follow actions not only harm the user, by performing a crutial action without their consent, but may also affect other users.

It could be used for example to make prestigious profiles follow scam accounts, unwillingly ligitimating them.

Extra note: An adversary can execute this attack at any time. They can prevent having their follow NFT being migrated by frontrunning the migrate tx, and moving their token to another wallet. The migration will run silently with no error, and the attack can be performed later.

### Proof of Concept

Follows can be checked by the `isFollowing()` function, which is dependant on the `_followTokenIdByFollowerProfileId[followerProfileId]` mapping. `_followTokenIdByFollowerProfileId` is unset for Lens v1:

```solidity
    // Introduced in v2
    mapping(uint256 => uint256) internal _followTokenIdByFollowerProfileId;

    function isFollowing(uint256 followerProfileId) external view override returns (bool) {
        return _followTokenIdByFollowerProfileId[followerProfileId] != 0;
    }
```

*   [FollowNFT.sol#L35-L37](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L35-L37)
*   [FollowNFT.sol#L216-L218](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L216-L218)

The expected way to update `_followTokenIdByFollowerProfileId` is via the `_baseFollow()` internal function, which can only be called via other external functions protected with access control, only authorizing the profile owner who will follow or a delegate:

```solidity
    function _baseFollow(
        uint256 followerProfileId,
        uint256 followTokenId,
        bool isOriginalFollow
    ) internal {
        _followTokenIdByFollowerProfileId[followerProfileId] = followTokenId;
```

[FollowNFT.sol#L386-L391](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L386-L391)

However the `tryMigrate()` function also updates `_followDataByFollowTokenId` to help with migrations from V1, and can be called by anyone (via the hub):

```solidity
    function tryMigrate(
        uint256 followerProfileId,
        address followerProfileOwner,
        uint256 idOfProfileFollowed,
        uint256 followTokenId
    ) external onlyHub returns (uint48) {
        // ...

        _followDataByFollowTokenId[followTokenId].followerProfileId = uint160(followerProfileId);
```

[FollowNFT.sol#L514](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L514)

The only caveat is that the "ProfileNFT and FollowNFT should be in the same account":

```solidity
    address followTokenOwner = ownerOf(followTokenId);

    // ProfileNFT and FollowNFT should be in the same account
    if (followerProfileOwner != followTokenOwner) {
        return 0; // Not holding both Profile & Follow NFTs together
    }
```

[FollowNFT.sol#L501-L504](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L501-L504)

But this can bypassed by simply transfering a V1 follow NFT from anyone to the victim, who will end up following the adversary unwillingly after the execution of `tryMigrate()`.

Also, as noted on the Impact section, an adversary could prevent that someone else tries to migrate their profile, by frontrunning the tx and moving their token to some other address. It will "fail" silently because of the `return 0; // Not holding both Profile & Follow NFTs together` on the previous code snippet.

### Coded POC

Add this test to `test/migrations/Migrations.t.sol` and run `TESTING_FORK=mainnet POLYGON_RPC_URL="https://polygon.llamarpc.com" forge test --mt "testFakeFollowMigration"`.

Note: In case of a memory allocation error during the Forge test, please comment [these lines](https://github.com/code-423n4/2023-07-lens/blob/main/test/migrations/Migrations.t.sol#L105-L116). They are not used for the current test.

```solidity
    function testFakeFollowMigration() public onlyFork {
        // lensprotocol.lens will be the victim
        // juancito.lens will be the adversary
        // The adversary will make lensprotocol.lens follow them without their consent

        uint256 victimProfileId = 1;       // lensprotocol.lens
        uint256 adversaryProfileId = 3659; // juancito.lens

        uint256 followTokenId = 42;        // juancito.lens follows juancito.lens

        FollowNFT nft = FollowNFT(hub.getProfile(adversaryProfileId).followNFT);
        address adversary = nft.ownerOf(followTokenId); // Owner of juancito.lens

        // 1. Transfer the Lens v1 follow token to the victim
        // This does not automatically imply that they are following the adversary yet
        vm.startPrank(address(adversary));
        nft.transferFrom(address(adversary), hub.ownerOf(victimProfileId), followTokenId);
        assertFalse(nft.isFollowing(victimProfileId));

        // 2. Migrate the fake follow. Anyone can run this
        uint256[] memory victimProfileIdArray = new uint256[](1);
        uint256[] memory adversaryProfileIdArray = new uint256[](1);
        uint256[] memory followTokenIdArray = new uint256[](1);

        victimProfileIdArray[0] = victimProfileId;       // 1
        adversaryProfileIdArray[0] = adversaryProfileId; // 3659
        followTokenIdArray[0] = followTokenId;           // 42

        hub.batchMigrateFollows(victimProfileIdArray, adversaryProfileIdArray, followTokenIdArray);

        // 3. The victim is now following the adversary
        assertTrue(nft.isFollowing(victimProfileId));
    }
```

### Recommended Mitigation Steps

The follow migration should be reworked to account for the case of "fake" follows. Some ideas are to allow the bulk process to be run by whitelisted accounts, or by the specific token owners. A combination of both may also work, by only migrating tokens that have been minted but not transfered, for example. Or letting users opt-in for an automatic migration to give some ideas.

Additionally log an event with the reason the migration was not executed for a specific profile.

### Assessed type

Invalid Validation

**[vicnaum (Lens) confirmed](https://github.com/code-423n4/2023-07-lens-findings/issues/104#issuecomment-1670021686)**



***

## [[M-11] Blocked follower can keep follow with `batchMigrateFollows`](https://github.com/code-423n4/2023-07-lens-findings/issues/40)
*Submitted by [klau5](https://github.com/code-423n4/2023-07-lens-findings/issues/40), also found by [Emmanuel](https://github.com/code-423n4/2023-07-lens-findings/issues/192)*

You can migrate V1 followers by calling the `LensV2Migration.batchMigrateFollows` function, which can be called by anyone.

```solidity
function batchMigrateFollows(
    uint256[] calldata followerProfileIds,
    uint256[] calldata idsOfProfileFollowed,
    uint256[] calldata followTokenIds
) external {
    MigrationLib.batchMigrateFollows(followerProfileIds, idsOfProfileFollowed, followTokenIds);
}
```

<https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/LensV2Migration.sol#L37-L43>

```solidity
function _migrateFollow(
    uint256 followerProfileId,
    uint256 idOfProfileFollowed,
    uint256 followTokenId
) private {
    uint48 mintTimestamp = FollowNFT(StorageLib.getProfile(idOfProfileFollowed).followNFT).tryMigrate({
        followerProfileId: followerProfileId,
        followerProfileOwner: StorageLib.getTokenData(followerProfileId).owner,
        idOfProfileFollowed: idOfProfileFollowed,
        followTokenId: followTokenId
    });
    // `mintTimestamp` will be 0 if:
    // - Follow NFT was already migrated
    // - Follow NFT does not exist or was burnt
    // - Follower profile Owner is different from Follow NFT Owner
    if (mintTimestamp != 0) {
        emit Events.Followed({
            followerProfileId: followerProfileId,
            idOfProfileFollowed: idOfProfileFollowed,
            followTokenIdAssigned: followTokenId,
            followModuleData: '',
            processFollowModuleReturnData: '',
            timestamp: mintTimestamp // The only case where this won't match block.timestamp is during the migration
        });
    }
}
```

<https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MigrationLib.sol#L114-L139>

The `FollowNFT.tryMigrate` is where the actual migration logic proceed.  `FollowNFT.tryMigrate` does not check whether the `followerProfileId` has been blocked by the `idOfProfileFollowed`.

```solidity
function tryMigrate(
    uint256 followerProfileId,
    address followerProfileOwner,
    uint256 idOfProfileFollowed,
    uint256 followTokenId
) external onlyHub returns (uint48) {
    // Migrated FollowNFTs should have `originalFollowTimestamp` set
    if (_followDataByFollowTokenId[followTokenId].originalFollowTimestamp != 0) {
        return 0; // Already migrated
    }

    if (_followedProfileId != idOfProfileFollowed) {
        revert Errors.InvalidParameter();
    }

    if (!_exists(followTokenId)) {
        return 0; // Doesn't exist
    }

    address followTokenOwner = ownerOf(followTokenId);

    // ProfileNFT and FollowNFT should be in the same account
    if (followerProfileOwner != followTokenOwner) {
        return 0; // Not holding both Profile & Follow NFTs together
    }

    unchecked {
        ++_followerCount;
    }

    _followTokenIdByFollowerProfileId[followerProfileId] = followTokenId;

    uint48 mintTimestamp = uint48(StorageLib.getTokenData(followTokenId).mintTimestamp);

    _followDataByFollowTokenId[followTokenId].followerProfileId = uint160(followerProfileId);
    _followDataByFollowTokenId[followTokenId].originalFollowTimestamp = mintTimestamp;
    _followDataByFollowTokenId[followTokenId].followTimestamp = mintTimestamp;

    super._burn(followTokenId);
    return mintTimestamp;
}
```

<https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/FollowNFT.sol#L480-L520>

Let's think the case that the `idOfProfileFollowed` profile blocked the `followerProfileId` when the follower has not yet migrated. In this case, if the owner of the `followerProfileId` or anyone else calls `LensV2Migration.batchMigrateFollows`, then the blocked `followerProfileId` can follow the `idOfProfileFollowed`.

The following codes are the PoC codes. Add and modify <https://github.com/code-423n4/2023-07-lens/blob/main/test/migrations/Migrations.t.sol> to run PoC.

First, modify the test because it is broken due to a change of the return value of getProfile. Add the following interface at `Migrations.t.sol` test file.

```solidity
interface LensHubV1 {
    struct ProfileV1 {
        uint256 pubCount; // offset 0
        address followModule; // offset 1
        address followNFT; // offset 2
        string __DEPRECATED__handle; // offset 3
        string imageURI; // offset 4
        string __DEPRECATED__followNFTURI;
    }

    function getProfile(uint256 profileId) external view returns (ProfileV1 memory);
}
```

Also modify the following code to recover broken tests. <https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/test/migrations/Migrations.t.sol#L107>

```solidity
address followNFTAddress = LensHubV1(address(hub)).getProfile(idOfProfileFollowed).followNFT;
```

Add this test function at  `MigrationsTest` contract and run. Even after being blocked, it is possible to follow through `batchMigrateFollows`.

```solidity
function testMigrateBlockedFollowerPoC() public onlyFork {
    uint256 idOfProfileFollowed = 8;

    uint256[] memory idsOfProfileFollowed = new uint256[](10);
    uint256[] memory followTokenIds = new uint256[](10);
    bool[] memory blockStatuses = new bool[](10);

    for (uint256 i = 0; i < 10; i++) {
        uint256 followTokenId = i + 1;

        idsOfProfileFollowed[i] = idOfProfileFollowed;
        followTokenIds[i] = followTokenId;

        blockStatuses[i] = true;
    }

    // block followers
    address targetProfileOwner = hub.ownerOf(idOfProfileFollowed);
    vm.prank(targetProfileOwner);

    hub.setBlockStatus(
        idOfProfileFollowed,
        followerProfileIds,
        blockStatuses
    );

    // check block status
    assertEq(hub.isBlocked(followerProfileIds[0], idOfProfileFollowed), true);

    // migrate
    hub.batchMigrateFollows(followerProfileIds, idsOfProfileFollowed, followTokenIds);

    // check follow work 
    address followNFTAddress = LensHubV1(address(hub)).getProfile(idOfProfileFollowed).followNFT;
    assertEq(FollowNFT(followNFTAddress).isFollowing(followerProfileIds[0]), true); // blocked, but followed!
}
```

### Recommended Mitigation Steps

At `FollowNFT.tryMigrate` function, If the follower is blocked, make it unfollowed.

### Assessed type

Invalid Validation

**[vicnaum (Lens) confirmed and commented](https://github.com/code-423n4/2023-07-lens-findings/issues/40#issuecomment-1670022300):**
> This looks like a subset of [issue 112](https://github.com/code-423n4/2023-07-lens-findings/issues/112).

**[Picodes (judge) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/40#issuecomment-1695965310):**
 > Although the impact is similar, it doesn't look like a duplicate to me as this is specifically about a blocked user being able to migrate himself, whereas `#112` is about an attacker migrating someone without its consent.



***

# Low Risk and Non-Critical Issues

For this audit, 23 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-07-lens-findings/issues/168) by **MiloTruck** received the top score from the judge.

*The following wardens also submitted reports: [juancito](https://github.com/code-423n4/2023-07-lens-findings/issues/109), [Kaysoft](https://github.com/code-423n4/2023-07-lens-findings/issues/180), [AlexCzm](https://github.com/code-423n4/2023-07-lens-findings/issues/173), [MohammedRizwan](https://github.com/code-423n4/2023-07-lens-findings/issues/165), [BugzyVonBuggernaut](https://github.com/code-423n4/2023-07-lens-findings/issues/154), [Prestige](https://github.com/code-423n4/2023-07-lens-findings/issues/130), [Sathish9098](https://github.com/code-423n4/2023-07-lens-findings/issues/125), [Emmanuel](https://github.com/code-423n4/2023-07-lens-findings/issues/115), [ihtishamsudo](https://github.com/code-423n4/2023-07-lens-findings/issues/96), [tnquanghuy0512](https://github.com/code-423n4/2023-07-lens-findings/issues/76), [adeolu](https://github.com/code-423n4/2023-07-lens-findings/issues/72), [Rolezn](https://github.com/code-423n4/2023-07-lens-findings/issues/63), [evmboi32](https://github.com/code-423n4/2023-07-lens-findings/issues/55), [mrudenko](https://github.com/code-423n4/2023-07-lens-findings/issues/50), [fatherOfBlocks](https://github.com/code-423n4/2023-07-lens-findings/issues/48), [Iurii3](https://github.com/code-423n4/2023-07-lens-findings/issues/46), [0xAnah](https://github.com/code-423n4/2023-07-lens-findings/issues/41), [descharre](https://github.com/code-423n4/2023-07-lens-findings/issues/39), [Stormreckson](https://github.com/code-423n4/2023-07-lens-findings/issues/38), [Bughunter101](https://github.com/code-423n4/2023-07-lens-findings/issues/29), [DavidGiladi](https://github.com/code-423n4/2023-07-lens-findings/issues/18), and [ginlee](https://github.com/code-423n4/2023-07-lens-findings/issues/16).*

## Finding Summary 

| ID | Description | Severity |
| - | - | :-: |
| L-01 | Avoid directly minting follow NFTs to the profile owner in `processBlock()` | Low |
| L-02 | Regular approvals are not used in access controls in `FollowNFT.sol` | Low |
| L-03 | Users have no way of revoking their signatures in `LensHub` | Low |
| L-04 | Removing `ERC721Enumerable` functionality might break composability with other protocols | Low |
| L-05 | Delegated executor configs are not cleared on transfer | Low |
| L-06 | `act()` in `ActionLib.sol` doesn't check if the target publication is a mirror | Low |
| L-07 | `LensHub` contract cannot be unpaused if emergency admin and governance is the same address | Low |
| L-08 | Only 255 action modules can ever be whitelisted | Low |
| L-09 | Setting `transactionExecutor` to `msg.sender` in `createProfile()` might limit functionality | Low |
| L-10 | Avoid minting handles that have a `tokenId` of 0 in `mintHandle()` | Low |
| N-01 | `approveFollow()` should allow `followerProfileId = 0` to cancel follow approvals | Non-Critical |
| N-02 | Redundant check in `_isApprovedOrOwner()` can be removed | Non-Critical |
| N-03 | `whenNotPaused` modifier is redundant for `burn()` in `LensProfiles.sol` | Non-Critical |
| N-04 | `unfollow()` should be allowed for burnt profiles | Non-Critical |
| N-05 | Un-migratable handles might exist in Lens Protocol V1 | Non-Critical |
| N-06 | Relayer can choose amount of gas when calling function in meta-transactions | Non-Critical |

## [L-01] Avoid directly minting follow NFTs to the profile owner in `processBlock()`

In `FollowNFT.sol`, whenever a follower is blocked by a profile and his `followTokenId` is unwrapped, `processBlock()` will mint the follow NFT to the follower's address:

[FollowNFT.sol#L198-L203](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L198-L203)

```solidity
        uint256 followTokenId = _followTokenIdByFollowerProfileId[followerProfileId];
        if (followTokenId != 0) {
            if (!_isFollowTokenWrapped(followTokenId)) {
                // Wrap it first, so the user stops following but does not lose the token when being blocked.
                _mint(IERC721(HUB).ownerOf(followerProfileId), followTokenId);
            }
```

However, if the profile owner's address isn't able to follow NFTs, such as profiles held in a secure wallet (e.g. hardware wallet or multisig), the minted follow NFT would become permanently stuck.

### Recommendation

Consider allowing the follower to recover the NFT by himself by assigning `profileIdAllowedToRecover` to `followerProfileId`: 

[FollowNFT.sol#L198-L203](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L198-L203)

```diff
        uint256 followTokenId = _followTokenIdByFollowerProfileId[followerProfileId];
        if (followTokenId != 0) {
            if (!_isFollowTokenWrapped(followTokenId)) {
                // Wrap it first, so the user stops following but does not lose the token when being blocked.
-               _mint(IERC721(HUB).ownerOf(followerProfileId), followTokenId);
+               _followDataByFollowTokenId[followTokenId].profileIdAllowedToRecover = followerProfileId;
            }
```

## [L-02] Regular approvals are not used in access controls in `FollowNFT.sol`

Throughout `FollowNFT.sol`, regular ERC-721 approvals are not used for access controls. For example, these are the access control checks when `follow()` is called with a wrapped follow token: 

[FollowNFT.sol#L317-L327](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L317-L327)

```solidity
        bool isFollowApproved = _followApprovalByFollowTokenId[followTokenId] == followerProfileId;
        address followerProfileOwner = IERC721(HUB).ownerOf(followerProfileId);
        if (
            !isFollowApproved &&
            followTokenOwner != followerProfileOwner &&
            followTokenOwner != transactionExecutor &&
            !isApprovedForAll(followTokenOwner, transactionExecutor) &&
            !isApprovedForAll(followTokenOwner, followerProfileOwner)
        ) {
            revert DoesNotHavePermissions();
        }
```

Apart from follow approvals, the function only allows the token owner or approved operators (address approved using `setApprovalForAll()`). If a user is approved by the token owner using `approve()` he will be unable to call `follow()` despite having control over the token.

This isn't a major issue as the approved address can sidestep this by doing the following:
* Transfer the follow NFT to himself.
* Call `follow()`.
* Transfer the follow NFT back to the original address.

However, this could potentially be extremely inconvenient for the approved address.

### Recommendation

Consider allowing addresses approved using `approve()` to call the following functions as well:
* `follow()`
* `unfollow()`
* `removeFollower()`
* `approveFollow()`

## [L-03] Users have no way of revoking their signatures in `LensHub`

In the `LensHub` contract, users can provide signatures to allow relayers to call functions on their behalf in meta-transactions, such as `followWithSig()`.

However, there is currently no way for the user to revoke their signatures. This could become a problem is the user needs to revoke their signatures (eg. action or reference module turns malicious, user doesn't want to use the module anymore).

### Recommendation

In the `Lenshub` contract, implement a way for users to revoke their signatures. One way of achieving this is to add a function that increments the caller's nonce:

```solidity
function incrementNonce() external {
    StorageLib.nonces()[msg.sender]++;
}
```

## [L-04] Removing `ERC721Enumerable` functionality might break composability with other protocols

In `LensBaseERC721.sol`, the `ERC721Enumerable` extension is no longer supported. This can be seen in its `supportsInterface()` function, which no longer checks for `type(IERC721Enumerable).interfaceId`:

[IERC721Enumerable.interfaceId](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L84-L92)

```solidity
    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC165, IERC165) returns (bool) {
        return
            interfaceId == type(IERC721).interfaceId ||
            interfaceId == type(IERC721Timestamped).interfaceId ||
            interfaceId == type(IERC721Burnable).interfaceId ||
            interfaceId == type(IERC721MetaTx).interfaceId ||
            interfaceId == type(IERC721Metadata).interfaceId ||
            super.supportsInterface(interfaceId);
    }
```

As `LensBaseERC721` is inherited by `LensProfiles`, profiles will no longer support the `ERC721Enumerable` extension. This might break the functionality of other protocols that rely on `ERC721Enumerable`'s functions.

### Recommendation

Consider documenting/announcing that the V2 upgrade will remove `ERC721Enumerable` functionality from profiles.

## [L-05] Delegated executor configs are not cleared on transfer

When profiles are transferred, its delegated executor config is not cleared. Instead, the function switches its config to a new config:

[LensProfiles.sol#L165-L177](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L165-L177)

```solidity
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 tokenId
    ) internal override whenNotPaused {
        if (from != address(0) && _hasTokenGuardianEnabled(from)) {
            // Cannot transfer profile if the guardian is enabled, except at minting time.
            revert Errors.GuardianEnabled();
        }
        // Switches to new fresh delegated executors configuration (except on minting, as it already has a fresh setup).
        if (from != address(0)) {
            ProfileLib.switchToNewFreshDelegatedExecutorsConfig(tokenId);
        }
```

This could potentially be dangerous as users are able to switch back to previous configs using [`changeDelegatedExecutorsConfig()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L149-L163). If a previous owner had added himself to previous configs, switching back to a previous config might potentially give the previous owner the ability to steal the profile, or execute malicious functions as a delegated executor.

### Recommendation

Consider warning users about the danger of switching to previous configs in the documentation.

## [L-06] `act()` in `ActionLib.sol` doesn't check if the target publication is a mirror

According to the [Referral System Rules](https://github.com/code-423n4/2023-07-lens/tree/main#referral-system-rules), mirrors cannot be a target publication. However, the [`act()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol#L13-L63) function does not ensure that the target publication is not a mirror.

This is currently unexploitable as mirrors cannot be initialized with action modules, therefore the [`_isActionEnabled`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol#L31-L38) check will always revert when called with a mirror. However, this could potentially become exploitable if an attacker finds a way to corrupt the `enabledActionModulesBitmap` of a mirror publication.

### Recommendation

Consider validating that the target publication is not a mirror in [`act()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol#L13-L63).

## [L-07] `LensHub` contract cannot be unpaused if emergency admin and governance is the same address

In `GovernanceLib.sol`, the `setState()` function is used by both the emergency admin and governance to change the state of the contract:

[GovernanceLib.sol#L50-L60](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol#L50-L60)

```solidity
    function setState(Types.ProtocolState newState) external {
        // NOTE: This does not follow the CEI-pattern, but there is no interaction and this allows to abstract `_setState` logic.
        Types.ProtocolState prevState = _setState(newState);
        // If the sender is the emergency admin, prevent them from reducing restrictions.
        if (msg.sender == StorageLib.getEmergencyAdmin()) {
            if (newState <= prevState) {
                revert Errors.EmergencyAdminCanOnlyPauseFurther();
            }
        } else if (msg.sender != StorageLib.getGovernance()) {
            revert Errors.NotGovernanceOrEmergencyAdmin();
        }
```

As seen from above, the emergency admin can only pause further, whereas governance can set any state. 

However, if emergency admin and governance happens to be the same address, `msg.sender == StorageLib.getEmergencyAdmin()` will always be true, which only allows the system to be paused further. This could become a problem if Lens Protocol decides to set both to the same address.

### Recommendation

Consider making the following change to the if-statement:

[GovernanceLib.sol#L50-L60](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol#L50-L60)

```diff
    function setState(Types.ProtocolState newState) external {
        // NOTE: This does not follow the CEI-pattern, but there is no interaction and this allows to abstract `_setState` logic.
        Types.ProtocolState prevState = _setState(newState);
        // If the sender is the emergency admin, prevent them from reducing restrictions.
-       if (msg.sender == StorageLib.getEmergencyAdmin()) {
+       if (msg.sender != StorageLib.getGovernance() && msg.sender == StorageLib.getEmergencyAdmin()) {
-           if (newState <= prevState) {
-               revert Errors.EmergencyAdminCanOnlyPauseFurther();
-           }
-       } else if (msg.sender != StorageLib.getGovernance()) {
-           revert Errors.NotGovernanceOrEmergencyAdmin();
-       }
```

## [L-08] Only 255 action modules can ever be whitelisted

When an action module is whitelisted, `incrementMaxActionModuleIdUsed()` is called, which increments `_maxActionModuleIdUsed` and checks that it does not exceed `MAX_ACTION_MODULE_ID_SUPPORTED`:

[StorageLib.sol#L165-L175](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/StorageLib.sol#L165-L175)

```solidity
    function incrementMaxActionModuleIdUsed() internal returns (uint256) {
        uint256 incrementedId;
        assembly {
            incrementedId := add(sload(MAX_ACTION_MODULE_ID_USED_SLOT), 1)
            sstore(MAX_ACTION_MODULE_ID_USED_SLOT, incrementedId)
        }
        if (incrementedId > MAX_ACTION_MODULE_ID_SUPPORTED) {
            revert Errors.MaxActionModuleIdReached();
        }
        return incrementedId;
    }
```

`MAX_ACTION_MODULE_ID_SUPPORTED` is currently declared as `255`:

[StorageLib.sol#L47](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/StorageLib.sol#L47)

```solidity
uint256 constant MAX_ACTION_MODULE_ID_SUPPORTED = 255;
```

This becomes an issue as `_maxActionModuleIdUsed` does not changed when action modules are un-whitelisted, which means that action module IDs cannot be reused:

[GovernanceLib.sol#L105-L107](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol#L105-L107)

```solidity
            // The action module with the given address was already whitelisted before, it has an ID already assigned.
            StorageLib.actionModuleWhitelistData()[actionModule].isWhitelisted = whitelist;
            id = actionModuleWhitelistData.id;
```

This means that only 255 action modules can ever be whitelisted, and governance will not be able to whitelist action modules forever after this limit is exceeded.

## [L-09] Setting `transactionExecutor` to `msg.sender` in `createProfile()` might limit functionality

When `createProfile()` is called, the profile's follow module is initialized with `msg.sender` as its `transactionExecutor`:

[ProfileLib.sol#L56-L61](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol#L56-L61)

```solidity
            followModuleReturnData = _initFollowModule({
                profileId: profileId,
                transactionExecutor: msg.sender,
                followModule: createProfileParams.followModule,
                followModuleInitData: createProfileParams.followModuleInitData
            });
```

This could be extremely limiting if the follow module uses `transactionExecutor` during its initialization. 

For example, consider a follow module that needs to be initialized with a certain amount of tokens:
* If these tokens are transferred from `transactionExecutor`, the profile creator will bear the cost of the initialization. 
* If the profile's owner is meant to provide funds, he will have to transfer the tokens to the profile creator, which creates risk as he has to trust that the profile creator won't rug.

### Recommendation

Consider setting `transactionExecutor` to `createProfileParams.to` instead, which is the owner of the newly created profile:

[ProfileLib.sol#L56-L61](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol#L56-L61)

```diff
            followModuleReturnData = _initFollowModule({
                profileId: profileId,
-               transactionExecutor: msg.sender,
+               transactionExecutor: createProfileParams.to,
                followModule: createProfileParams.followModule,
                followModuleInitData: createProfileParams.followModuleInitData
            });
```

## [L-10] Avoid minting handles that have a `tokenId` of 0 in `mintHandle()`

When a handle is minted, its `tokenId` is computed as the `keccak256` hash of the handle name:

[LensHandles.sol#L182-L184](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L182-L184)

```solidity
    function getTokenId(string memory localName) public pure returns (uint256) {
        return uint256(keccak256(bytes(localName)));
    }
```

However, `_mintHandle()` does not ensure that the `tokenId` minted is not 0:

[LensHandles.sol#L194-L200](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L194-L200)

```solidity
    function _mintHandle(address to, string calldata localName) internal returns (uint256) {
        uint256 tokenId = getTokenId(localName);
        _mint(to, tokenId);
        _localNames[tokenId] = localName;
        emit HandlesEvents.HandleMinted(localName, NAMESPACE, tokenId, to, block.timestamp);
        return tokenId;
    }
```

This makes it theoretically possible to mint a handle with a `tokenId` of 0, which is problematic as `tokenId == 0` is treated as a default value in the protocol. For example, `getDefaultHandle()` in `TokenHandleRegistry` returns profile ID as 0 when the handle `tokenId` is 0:

[TokenHandleRegistry.sol#L100-L102](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol#L100-L102)

```solidity
        if (resolvedTokenId == 0 || !ILensHub(LENS_HUB).exists(resolvedTokenId)) {
            return 0;
        }
```

### Recommendation

In `_mintHandle()`, consider checking that `tokenId` is non-zero to protect against the extremely unlikely event where a `localName` results in `tokenId = 0`:

[LensHandles.sol#L194-L200](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L194-L200)

```diff
    function _mintHandle(address to, string calldata localName) internal returns (uint256) {
        uint256 tokenId = getTokenId(localName);
+       if (tokenId == 0) revert InvalidHandle();
        _mint(to, tokenId);
        _localNames[tokenId] = localName;
        emit HandlesEvents.HandleMinted(localName, NAMESPACE, tokenId, to, block.timestamp);
        return tokenId;
    }
```

## [N-01] `approveFollow()` should allow `followerProfileId = 0` to cancel follow approvals

In `FollowNFT.sol`, `approveFollow()` only allows profile IDs that exist to be set as `followerProfileId`:

[FollowNFT.sol#L141-L153](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L141-L144)

```solidity
    function approveFollow(uint256 followerProfileId, uint256 followTokenId) external override {
        if (!IERC721Timestamped(HUB).exists(followerProfileId)) {
            revert Errors.TokenDoesNotExist();
        }
```

As seen from above, the function does not allow `followerProfileId` to be 0. This forces users to set `followerProfileId` to their own address if they wish to cancel a follow approval, which could pose problems.

### Recommendation

Allow `approveFollow()` to be called with`followerProfileId = 0` as profiles will never have the ID 0:

[FollowNFT.sol#L141-L153](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L141-L144)

```diff
    function approveFollow(uint256 followerProfileId, uint256 followTokenId) external override {
-       if (!IERC721Timestamped(HUB).exists(followerProfileId)) {
+       if (followerProfileId != 0 && !IERC721Timestamped(HUB).exists(followerProfileId)) {
            revert Errors.TokenDoesNotExist();
        }
```

## [N-02] Redundant check in `_isApprovedOrOwner()` can be removed

In `LensBaseERC721.sol`, the `_isApprovedOrOwner()` functions contains an if-statement that checks if `tokenId` exists (`_tokenData[tokenId].owner != 0`):

[LensBaseERC721.sol#L333-L339](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L333-L339)

```solidity
    function _isApprovedOrOwner(address spender, uint256 tokenId) internal view virtual returns (bool) {
        if (!_exists(tokenId)) {
            revert Errors.TokenDoesNotExist();
        }
        address owner = ownerOf(tokenId);
        return (spender == owner || getApproved(tokenId) == spender || isApprovedForAll(owner, spender));
    }
```

However, this check is redundant as `ownerOf` also contains the same check:

[LensBaseERC721.sol#L116-L122](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L116-L122)

```solidity
    function ownerOf(uint256 tokenId) public view virtual override returns (address) {
        address owner = _tokenData[tokenId].owner;
        if (owner == address(0)) {
            revert Errors.TokenDoesNotExist();
        }
        return owner;
    }
```

### Recommendation 

Consider removing the if statement from `_isApprovedOrOwner()`:

[LensBaseERC721.sol#L333-L339](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L333-L339)

```diff
    function _isApprovedOrOwner(address spender, uint256 tokenId) internal view virtual returns (bool) {
-       if (!_exists(tokenId)) {
-           revert Errors.TokenDoesNotExist();
-       }
        address owner = ownerOf(tokenId);
        return (spender == owner || getApproved(tokenId) == spender || isApprovedForAll(owner, spender));
    }
```

## [N-03] `whenNotPaused` modifier is redundant for `burn()` in `LensProfiles.sol`

In `LensProfiles.sol`, the `burn()` function has the `whenNotPaused` modifier:

[LensProfiles.sol#L93-L96](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L93-L96)

```solidity
    function burn(uint256 tokenId)
        public
        override(LensBaseERC721, IERC721Burnable)
        whenNotPaused
```

However, the modifier is redundant as the `_beforeTokenTransfer` hook also has the `whenNotPaused` modifier:

[LensProfiles.sol#L165-L169](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L84-L92)

```solidity
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 tokenId
    ) internal override whenNotPaused {
```

### Recommendation

Consider removing the `whenNotPaused` modifier from `burn()`:

[LensProfiles.sol#L93-L96](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L93-L96)

```diff
    function burn(uint256 tokenId)
        public
        override(LensBaseERC721, IERC721Burnable)
-       whenNotPaused
```

## [N-04] `unfollow()` should be allowed for burnt profiles

In the `LensHub` contract, `unfollow()` will revert if the profile to unfollow is burnt, due to the following check:

[FollowLib.sol#L54-L62](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/FollowLib.sol#L54-L62)

```solidity
    function unfollow(
        uint256 unfollowerProfileId,
        address transactionExecutor,
        uint256[] calldata idsOfProfilesToUnfollow
    ) external {
        uint256 i;
        while (i < idsOfProfilesToUnfollow.length) {
            uint256 idOfProfileToUnfollow = idsOfProfilesToUnfollow[i];
            ValidationLib.validateProfileExists(idOfProfileToUnfollow); // auditor: This check
```

However, this implementation seems incorrect as users should be able to unfollow burnt profiles. 

This issue can be sidestepped by calling [`removeFollower()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L131-L138) or [`burn()`](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L255-L258) in the burnt profile's FollowNFT contract, but could be extremely inconvenient as users will have to first wrap their follow token.

### Recommendation

Consider removing the `validateProfileExists()` check in `unfollow()`:

[FollowLib.sol#L54-L62](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/FollowLib.sol#L54-L62)

```diff
    function unfollow(
        uint256 unfollowerProfileId,
        address transactionExecutor,
        uint256[] calldata idsOfProfilesToUnfollow
    ) external {
        uint256 i;
        while (i < idsOfProfilesToUnfollow.length) {
            uint256 idOfProfileToUnfollow = idsOfProfilesToUnfollow[i];
-           ValidationLib.validateProfileExists(idOfProfileToUnfollow); // auditor: This check
```

This is possible as the `followNFT` addrress of non-existent profiles (profiles that are not minted yet) is 0, which causes the function to revert later on during execution.

## [N-05] Un-migratable handles might exist in Lens Protocol V1

In `LensHandles.sol`, the `_validateLocalNameMigration()` function is used to validate migrated handles:

[LensHandles.sol#L210-L223](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L210-L223)

```solidity
        bytes1 firstByte = localNameAsBytes[0];
        if (firstByte == '-' || firstByte == '_') {
            revert HandlesErrors.HandleFirstCharInvalid();
        }

        uint256 i;
        while (i < localNameLength) {
            if (!_isAlphaNumeric(localNameAsBytes[i]) && localNameAsBytes[i] != '-' && localNameAsBytes[i] != '_') {
                revert HandlesErrors.HandleContainsInvalidCharacters();
            }
            unchecked {
                ++i;
            }
        }
```

As seen from above, handles can only be migrated if:
* Its first byte is alphanumeric.
* Only contains alphanumeric characters, '\-' or "\_".

Additionally, the `_migrateProfile()` function in `MigrationLib.sol` removes the last 5 bytes of each handle:

[MigrationLib.sol#L77-L80](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol#L77-L80)

```solidity
                assembly {
                    let handle_length := mload(handle)
                    mstore(handle, sub(handle_length, DOT_LENS_SUFFIX_LENGTH)) // Cut 5 chars (.lens) from the end
                }
```

This means that any V1 handle that does not contan ".lens" and is shorter than 5 bytes cannot be migrated.

However, the rules mentioned above are not strictly enforced in Lens Protocol V1:

[PublishingLogic.sol#L391](https://polygonscan.com/address/0xBA97fc9137b7cbBBC7fcB70a87DA645d917C902F#code#F8#L391)

```solidity
    function _validateHandle(string calldata handle) private pure {
        bytes memory byteHandle = bytes(handle);
        if (byteHandle.length == 0 || byteHandle.length > Constants.MAX_HANDLE_LENGTH)
            revert Errors.HandleLengthInvalid();

        uint256 byteHandleLength = byteHandle.length;
        for (uint256 i = 0; i < byteHandleLength; ) {
            if (
                (byteHandle[i] < '0' ||
                    byteHandle[i] > 'z' ||
                    (byteHandle[i] > '9' && byteHandle[i] < 'a')) &&
                byteHandle[i] != '.' &&
                byteHandle[i] != '-' &&
                byteHandle[i] != '_'
            ) revert Errors.HandleContainsInvalidCharacters();
            unchecked {
                ++i;
            }
        }
    }
```

As seen from above, V1 handles can also contain ".", and its first byte is not only restricted to alphanumeric characters. Additionally, handles do not have to end with the ".lens" suffix. This means that it might be possible to create a handle that cannot be migrated after the V2 upgrade.

### Recommendation

Ensure all profile handles obey the following rules before the V2 upgrade:

* First byte is alphanumeric.
* Only contains alphanumeric characters, '\-' or "\_".
* Ends with the ".lens" suffix.

## [N-06] Relayer can choose amount of gas when calling function in meta-transactions

The `LensHub` contract supports the relaying of calls for several functions using a supplied signature. For example, users can provide a `signature` alongside the usual parameters in `setProfileMetadataURIWithSig()` for a relayer to call the function on his behalf:

[LensHub.sol#L119-L123](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L119-L123)

```solidity
    function setProfileMetadataURIWithSig(
        uint256 profileId,
        string calldata metadataURI,
        Types.EIP712Signature calldata signature
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(signature.signer, profileId) {
```

However, the parameters above do not include a gas parameter, which means the relayer can specify any gas amount. 

If the provided gas amount is insufficient, the entire transaction will revert. However, if the function exhibits different behaviour depending on the supplied gas (which might occur in modules), a relayer could potentially influence the outcome of the call by manipulating the gas amount.

### Recommendation

For all functions used for meta-transactions, consider adding a `gas` parameter that allows users to specify the gas amount the function should be called with. This `gas` parameter should be included in the hash digest when verifying the user's signature.

**[Picodes (judge) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/168#issuecomment-1704408187):**
 > Also note among the downgraded findings that were considered in scoring:<br>
> - [#138](https://github.com/code-423n4/2023-07-lens-findings/issues/138): `transactionExecutor` is incorrectly set to relayer instead of signer in `setFollowModuleWithSig()`
> - [#139](https://github.com/code-423n4/2023-07-lens-findings/issues/139): Precomputed `LENS_HUB_CACHED_POLYGON_DOMAIN_SEPARATOR` will become incorrect if Polygon hard forks
> 
> Overall outstanding work!

**[donosonaumczuk (Lens) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/168#issuecomment-1832635699):**
> [L-06] Does not make sense, you can always add checks for things that your business logic does not allow, but what is the point? A mirror cannot initialize a publication action, and that's part of the core business logic, so no extra checks needed.<br>
> [N-06] That is perfectly fine and fair! This is not an issue.

**[donosonaumczuk (Lens) commented](https://github.com/code-423n4/2023-07-lens-findings/issues/168#issuecomment-1847520365):**
> [L-05] This is by design. Also "switching back to a previous config might potentially give the previous owner the ability to steal the profile" is invalid, as delegated executors have only rights over social operations (e.g. post, comment, follow, etc) but not over asset operations (e.g. approve, transferFrom, etc).

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
