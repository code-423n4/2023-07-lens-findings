# low

| no  | Issue   | Instance |
|------|---------|----------|
|[L-01]|Missing checks for address(0x0) in the constructor|7|
|[L-02]|Signature use at deadlines should be allowed|2|
|[L-03]|Missing checks for address(0x0) when updating address state variables|4|
|[L-04]|Emitting storage values instead of the memory one.|2|
|[L-05]|Functions calling contracts/addresses with transfer hooks are missing reentrancy|8|
|[L-06]|Missing Event for initialize|2|
|[L-07]|Use safeTransfer, safeTransferFrom instead of transfer, transferFrom when |2|





##  [L‑01] Missing checks for address(0x0) in the constructor

```solidity 

18     previousImplementation = previousImplementation_;

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ProxyAdmin.sol#L18



```solidity
file:    contracts/misc/LensV2UpgradeContract.sol

36        oldFollowModulesToUnwhitelist = oldFollowModulesToUnwhitelist_;
37        newFollowModulesToWhitelist = newFollowModulesToWhitelist_;
38        oldReferenceModulesToUnwhitelist = oldReferenceModulesToUnwhitelist_;
39        newReferenceModulesToWhitelist = newReferenceModulesToWhitelist_;
40        oldCollectModulesToUnwhitelist = oldCollectModulesToUnwhitelist_;
41        newActionModulesToWhitelist = newActionModulesToWhitelist_;

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L36

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L37

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L38

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L39

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L40

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L41




## [L‑02] Signature use at deadlines should be allowed


According to EIP-2612, signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.

```solidity

470    if (signature.deadline < block.timestamp) revert Errors.SignatureExpired();

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L470

```solidity

146    block.timestamp < StorageLib.tokenGuardianDisablingTimestamp()[wallet]);

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L146




##  [L‑03] Missing checks for address(0x0) when updating address state variables

```solidity

97     _governance = newGovernance;

104    _treasury = newTreasury;

111     _treasuryFee = newTreasuryFee;

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol#L97

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol#L104

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol#L111


```solidity

32    controllerContract = newControllerContract;

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L32



##  [L-04] Emitting storage values instead of the memory one.


```solidity

26      emit ControllerContractUpdated(controllerContract, address(0));

31      emit ControllerContractUpdated(controllerContract, newControllerContract);


```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L26

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L31




##  [L‑05] Functions calling contracts/addresses with transfer hooks are missing reentrancy guards



```solidity

178     super._beforeTokenTransfer(from, to, tokenId);

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L178 

```solidity

446     super._beforeTokenTransfer(from, to, followTokenId);

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L446

```solidity

228   _transfer(from, to, tokenId);

239    safeTransferFrom(from, to, tokenId, '');

254   _safeTransfer(from, to, tokenId, _data);

308   _transfer(from, to, tokenId);

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L228


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L239


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L254


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L308

```solidity

58             LENS_HANDLES.transferFrom(address(this), destination, handleId);

59            ILensHub(LENS_HUB).transferFrom(address(this), destination, profileId);

```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L59

## [L-06] Missing Event for initialize


solidity
48 function initialize(uint256 profileId) external override {
        // This is called right after deployment by the LensHub, so we can skip the onlyHub check.
        if (_initialized) {
            revert Errors.Initialized();
        }
        _initialized = true;
        _followedProfileId = profileId;
        _setRoyalty(1000); // 10% of royalties
    }
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/contracts/FollowNFT.sol#L48-L56

solidity
45 function initialize(uint256 profileId, uint256 pubId) external override {
        if (_initialized) revert Errors.Initialized();
        _initialized = true;
        _setRoyalty(1000); // 10% of royalties
        _profileId = profileId;
        _pubId = pubId;
        // _name and _symbol remain uninitialized because we override the getters below
    }
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol#L45-L52



## [L-07] Use safeTransfer, safeTransferFrom instead of transfer, transferFrom when 




solidity
58      LENS_HANDLES.transferFrom(address(this), destination, handleId);

59      ILensHub(LENS_HUB).transferFrom(address(this), destination, profileId);
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L59
