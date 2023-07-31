## Finding Summary 

| ID | Description | Severity |
| - | - | :-: |
| [L-01](#l-01-avoid-directly-minting-follow-nfts-to-the-profile-owner-in-processblock) | Avoid directly minting follow NFTs to the profile owner in `processBlock()` | Low |
| [L-02](#l-02-regular-approvals-are-not-used-in-access-controls-in-follownftsol) | Regular approvals are not used in access controls in `FollowNFT.sol` | Low |
| [L-03](#l-03-users-have-no-way-of-revoking-their-signatures-in-lenshub) | Users have no way of revoking their signatures in `LensHub` | Low |
| [L-04](#l-04-removing-erc721enumerable-functionality-might-break-composability-with-other-protocols) | Removing `ERC721Enumerable` functionality might break composability with other protocols | Low |
| [L-05](#l-05-delegated-executor-configs-are-not-cleared-on-transfer) | Delegated executor configs are not cleared on transfer | Low |
| [L-06](#l-06-act-in-actionlibsol-doesnt-check-if-the-target-publication-is-a-mirror) | `act()` in `ActionLib.sol` doesn't check if the target publication is a mirror | Low |
| [L-07](#l-07-lenshub-contract-cannot-be-unpaused-if-emergency-admin-and-governance-is-the-same-address) | `LensHub` contract cannot be unpaused if emergency admin and governance is the same address | Low |
| [L-08](#l-08-only-255-action-modules-can-ever-be-whitelisted) | Only 255 action modules can ever be whitelisted | Low |
| [L-09](#l-09-setting-transactionexecutor-to-msgsender-in-createprofile-might-limit-functionality) | Setting `transactionExecutor` to `msg.sender` in `createProfile()` might limit functionality | Low |
| [L-10](#l-10-un-migratable-handles-might-exist-in-lens-protocol-v1) | Un-migratable handles might exist in Lens Protocol V1 | Low |
| [L-11](#l-11-avoid-minting-handles-that-have-a-tokenid-of-0-in-minthandle) | Avoid minting handles that have a `tokenId` of 0 in `mintHandle()` | Low |
| [L-12](#l-12-relayer-can-choose-amount-of-gas-when-calling-function-in-meta-transactions) | Relayer can choose amount of gas when calling function in meta-transactions | Low |
| [N-01](#n-01-approvefollow-should-allow-followerprofileid--0-to-cancel-follow-approvals) | `approveFollow()` should allow `followerProfileId = 0` to cancel follow approvals | Non-Critical |
| [N-02](#n-02-redundant-check-in-_isapprovedorowner-can-be-removed) | Redundant check in `_isApprovedOrOwner()` can be removed | Non-Critical |
| [N-03](#n-03-whennotpaused-modifier-is-redundant-for-burn-in-lensprofilessol) | `whenNotPaused` modifier is redundant for `burn()` in `LensProfiles.sol` | Non-Critical |
| [N-04](#n-04-unfollow-should-be-allowed-for-burnt-profiles) | `unfollow()` should be allowed for burnt profiles | Non-Critical |



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

## [L-10] Un-migratable handles might exist in Lens Protocol V1

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
* Only contains alphanumeric characters, '-' or "_".

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
* Only contains alphanumeric characters, '-' or "_".
* Ends with the ".lens" suffix.

## [L-11] Avoid minting handles that have a `tokenId` of 0 in `mintHandle()`

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

## [L-12] Relayer can choose amount of gas when calling function in meta-transactions

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
