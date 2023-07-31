# 1) Floating Pragma

## Description:
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

Lock the pragma version and also consider known bugs ([https://github.com/ethereum/solidity/releases](https://github.com/ethereum/solidity/releases)) for the compiler version that is chosen.

## Remediation:

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

Refer: [https://swcregistry.io/docs/SWC-103/](https://swcregistry.io/docs/SWC-103/)

### Contracts with Floating Pragma Versions:

1) contracts/LensHub.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L3)

2) contracts/FollowNFT.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L3)

3) contracts/libraries/ActionLib.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol#L3)

4) contracts/libraries/FollowLib.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/FollowLib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/FollowLib.sol#L3)

5) contracts/libraries/GovernanceLib.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol#L3)

6) contracts/libraries/LegacyCollectLib.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/LegacyCollectLib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/LegacyCollectLib.sol#L3)

7) contracts/libraries/MetaTxLib.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L2](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L2)

8) contracts/libraries/MigrationLib.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol#L3)

9) contracts/libraries/ProfileLib.sol  
   [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol#L3)

10) contracts/libraries/PublicationLib.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/PublicationLib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/PublicationLib.sol#L3)

11) contracts/libraries/StorageLib.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/StorageLib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/StorageLib.sol#L3)

12) contracts/libraries/ValidationLib.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/Validation

Lib.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ValidationLib.sol#L3)

13) contracts/base/ERC2981CollectionRoyalties.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/ERC2981CollectionRoyalties.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/ERC2981CollectionRoyalties.sol#L3)

14) contracts/base/HubRestricted.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/HubRestricted.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/HubRestricted.sol#L3)

15) contracts/base/LensBaseERC721.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L3)

16) contracts/base/LensGovernable.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensGovernable.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensGovernable.sol#L3)

17) contracts/base/LensHubEventHooks.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensHubEventHooks.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensHubEventHooks.sol#L3)

18) contracts/base/LensHubStorage.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensHubStorage.sol#L3C1-L4C1](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensHubStorage.sol#L3C1-L4C1)

19) contracts/base/LensImplGetters.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensImplGetters.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensImplGetters.sol#L3)

20) contracts/base/LensProfiles.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L3)

21) contracts/base/upgradeability/FollowNFTProxy.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/upgradeability/FollowNFTProxy.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/upgradeability/FollowNFTProxy.sol#L3)

22) contracts/misc/ImmutableOwnable.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ImmutableOwnable.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ImmutableOwnable.sol#L3)

23) contracts/misc/LegacyCollectNFT.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol#L3)

24) contracts/misc/LensV2Migration.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2Migration.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2Migration.sol#L3)

25) contracts/misc/LensV2UpgradeContract.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L3)

26) contracts/misc/ModuleGlobals.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol#L3)

27) contracts/misc/ProfileCreationProxy.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L3](https://github.com/code-423n

4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L3)

28) contracts/misc/access/ControllableByContract.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L3)

29) contracts/misc/access/Governance.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/Governance.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/Governance.sol#L3)

30) contracts/misc/access/ProxyAdmin.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ProxyAdmin.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ProxyAdmin.sol#L3)

31) contracts/namespaces/LensHandles.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L3)

32) contracts/namespaces/TokenHandleRegistry.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol#L3)

33) contracts/interfaces/ILegacyFollowModule.sol  
    [https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILegacyFollowModule.sol#L3](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILegacyFollowModule.sol#L3)



# 2) Use a more recent version of solidity

## Description: 
Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions. In 0.8.15, the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller. In 0.8.17, prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call. Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

**1) contracts/base/ERC2981CollectionRoyalties.sol**
[Link](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/ERC2981CollectionRoyalties.sol#L3)

**2) contracts/base/LensBaseERC721.sol**
[Link](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L3)


