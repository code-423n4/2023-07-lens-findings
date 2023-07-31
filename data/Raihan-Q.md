# low

# summary
|      |  issue   |  instance  |
|------|----------|------------|
|[G-01]|Missing Event for initialize|2|
|[G-02]|Use safeTransfer, safeTransferFrom instead of transfer, transferFrom when transferring|2|
|[G-03]|Events.GovernanceSet emits wrong caller address|4|
|[G-04]|Signature use at deadlines should be allowed|1|
|[G-05]|Functions calling contracts/addresses with transfer hooks are missing reentrancy guards|8|
|[G-06]|Missing checks for address(0x0) in the constructor|4|
|[G-07]|Emitting storage values instead of the memory one|2|
|[G-08]|Missing checks for address(0x0) when updating address state variables|4|
|[G-09]|Use of ecrecover is susceptible to signature malleability|1|
|[L-10]| approve can revert if the current approval is not zero|1|



## [L-01] Missing Event for initialize
Description: Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip

```solidity
48 function initialize(uint256 profileId) external override {
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/contracts/FollowNFT.sol#L48

```solidity
File: contracts/misc/LegacyCollectNFT.sol
45 function initialize(uint256 profileId, uint256 pubId) external 
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol#L45

Recommendation: Add Event-Emit

## [L-02] Use safeTransfer, safeTransferFrom instead of transfer, transferFrom when transferring
Unsafe transfer does guarantee the transfer was fully complete only when it is implemented to fail whenever transfers can't be processed fully. Not all ERC20 implementations behave that way, it isn't required.

Due to that while having unsafe transfer for protocol controlled tokens produces no issues, using it for any external controlled token poses a vulnerability as long as this token implementation do not always fail for unsuccessful transfer, returning false instead, or a token can be upgraded to start behaving this way.


```solidity
File:  contracts/misc/ProfileCreationProxy.sol
58      LENS_HANDLES.transferFrom(address(this), destination, handleId);

59      ILensHub(LENS_HUB).transferFrom(address(this), destination, profileId);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58



## [L-03] Events.GovernanceSet emits wrong caller address
In the GovernanceLib.setState function the Events.GovernanceSet is emitted ([Link](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol#L19)).

It is defined as:
[Link](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Events.sol#L41-L46)
```solidity
 event GovernanceSet(
        address indexed caller,
        address indexed prevGovernance,
        address indexed newGovernance,
        uint256 timestamp
    );
```
So the first parameter should be the caller address.

When the event is emitted the first parameter is msg.sender. The issue is that the to address and msg.sender can be different. So the event in some cases contains wrong information.

same issue

```solidity
File: contracts/libraries/GovernanceLib.sol
30        emit Events.EmergencyAdminSet(msg.sender, prevEmergencyAdmin, newEmergencyAdmin, block.timestamp);

61        emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);

67        emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol#L30

## [L‑04] Signature use at deadlines should be allowed
According to EIP-2612, signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.
note: this is missed from bots
```solidity
File: contracts/base/LensProfiles.sol
146    block.timestamp < StorageLib.tokenGuardianDisablingTimestamp()[wallet]);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L146


## [L‑05] Functions calling contracts/addresses with transfer hooks are missing reentrancy guards
Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to read-only reentrancies with no way to protect against it, except by block-listing the whole protocol.
```solidity
File:   contracts/base/LensProfiles.sol
178     super._beforeTokenTransfer(from, to, tokenId);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L178 

```solidity
File:  contracts/FollowNFT.sol
446     super._beforeTokenTransfer(from, to, followTokenId);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L446

```solidity
File:  contracts/base/LensBaseERC721.sol
228   _transfer(from, to, tokenId);

239    safeTransferFrom(from, to, tokenId, '');

254   _safeTransfer(from, to, tokenId, _data);

308   _transfer(from, to, tokenId);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L228


```solidity
File:    contracts/misc/ProfileCreationProxy.sol
58             LENS_HANDLES.transferFrom(address(this), destination, handleId);

59            ILensHub(LENS_HUB).transferFrom(address(this), destination, profileId);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58


## [L‑06] Missing checks for address(0x0) in the constructor
note: these missid from bots
```solidity 
File:  contracts/misc/access/ProxyAdmin.sol
18     previousImplementation = previousImplementation_;
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ProxyAdmin.sol#L18

```solidity
File:   contracts/misc/LensV2UpgradeContract.sol
   PROXY_ADMIN = ProxyAdmin(proxyAdminAddress);
   GOVERNANCE = Governance(governanceAddress);
   newImplementation = newImplementationAddress;     
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol#L33-L35

## [L-07] Emitting storage values instead of the memory one.
Emitted values should not be read from storage again. Instead, the existing values from memory should be used.

```solidity
File:   contracts/misc/access/ControllableByContract.sol
26      emit ControllerContractUpdated(controllerContract, address(0));

31      emit ControllerContractUpdated(controllerContract, newControllerContract);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L26


## [L‑08] Missing checks for address(0x0) when updating address state variables

```solidity
File:  contracts/misc/ModuleGlobals.sol
97     _governance = newGovernance;

104    _treasury = newTreasury;

111     _treasuryFee = newTreasuryFee;
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol#L97

```solidity
File:  contracts/misc/access/ControllableByContract.sol
32    controllerContract = newControllerContract;
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L32


## [L-09] Use of ecrecover is susceptible to signature malleability
The built-in EVM precompile ecrecover is susceptible to signature malleability, which could lead to [replay attacks](https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57).
Consider using OpenZeppelin’s [ECDSA library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L125-L128) instead of the built-in function.

```solidity
File: contracts/libraries/MetaTxLib.sol
478    address recoveredAddress = ecrecover(digest, signature.v, signature.r, signature.s);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol#L478

## [L-10] approve can revert if the current approval is not zero
Some tokens like USDT check for the current approval, and they revert if it's not zero. While Tether is known to do this, it applies to other tokens as well, which are trying to protect against this attack vector.

```solidity
File:  contracts/base/LensProfiles.sol
117        super.approve(to, tokenId);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L117

```solidity
File: contracts/namespaces/LensHandles.sol
144        super.approve(to, tokenId);
```
https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol#L144