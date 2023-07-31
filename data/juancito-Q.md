# QA Report

## Low Severity Issues

### L-01 - Contracts that enable the Guardian on their constructor won't be able to transfer the token later

On the `LensProfiles` contract, the functions `enableTokenGuardian()` and `DANGER__disableTokenGuardian()` have a modifier `onlyEOA` to only allow EOAs to execute them. The `onlyEOA` modifier uses OpenZeppelin library to check `msg.sender.isContract()`.

This function only checks that the contract code is zero, which is also true during the constructor phase.

A contract may acquire a token on its `constructor` function, enable the guardian and continue with its initialization (code length will be zero, so it will pass the check).

#### Impact

The token will be stuck on the contract.

#### Proof of Concept

The reason is that the `_beforeTokenTransfer()` will revert and the guardian can't be disabled. This is because the `onlyEOA` is applied to `DANGER__disableTokenGuardian`:

```solidity
24: import {Address} from '@openzeppelin/contracts/utils/Address.sol';

27:    using Address for address;

50:    modifier onlyEOA() {
51:        if (msg.sender.isContract()) {
52:            revert Errors.NotEOA();
53:        }
54:        _;
55:    }

63:    function DANGER__disableTokenGuardian() external onlyEOA {

77:    function enableTokenGuardian() external onlyEOA {

165:    function _beforeTokenTransfer(
166:        address from,
167:        address to,
168:        uint256 tokenId
169:    ) internal override whenNotPaused {
170:        if (from != address(0) && _hasTokenGuardianEnabled(from)) {
171:            // Cannot transfer profile if the guardian is enabled, except at minting time.
172:            revert Errors.GuardianEnabled();
173:        }
```

- [LensProfiles.sol#L24](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L24)
- [LensProfiles.sol#L27](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L27)
- [LensProfiles.sol#L50-L55](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L50-L55)
- [LensProfiles.sol#L63](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L63)
- [LensProfiles.sol#L77](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L77)
- [LensProfiles.sol#L165-L173](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L165-L173)

#### Recommended Mitigation Steps

Allow anyone to disable their token guardian regardless of their account type. The `DANGER__disableTokenGuardian()` function already checks that the token was previously enabled, so this won't affect any other current behaviour.

### L-02 - `unlink()` can be called for any unlinked handles by providing `tokenId = 0`

`TokenHandleRegistry::unlink()` can be called with any unlinked `handleId`, and `tokenId = 0`.

This is because `handleToToken[_handleHash(handle)]` will returned an empty struct, `tokenPointedByHandle.id` will be zero, and the if statement `if (tokenPointedByHandle.id != tokenId) { revert RegistryErrors.NotLinked(); }` will not enter the condition.

An event will still be emitted in `_unlink()`.

Note: `tokenId = 0` is not a valid token value, as it starts on 1.

```solidity
    function unlink(uint256 handleId, uint256 tokenId) external {
        // We revert here only in the case if both tokens exists and the caller is not the owner of any of them
        if (
            ILensHandles(LENS_HANDLES).exists(handleId) &&
            ILensHandles(LENS_HANDLES).ownerOf(handleId) != msg.sender &&
            ILensHub(LENS_HUB).exists(tokenId) &&
            ILensHub(LENS_HUB).ownerOf(tokenId) != msg.sender
        ) {
            revert RegistryErrors.NotHandleNorTokenOwner();
        }

        RegistryTypes.Handle memory handle = RegistryTypes.Handle({collection: LENS_HANDLES, id: handleId});
        RegistryTypes.Token memory tokenPointedByHandle = handleToToken[_handleHash(handle)];

        // We check if the tokens are (were) linked for the case if some of them doesn't exist
@>      if (tokenPointedByHandle.id != tokenId) { // @audit-info tokenId == 0 for unlinked handles
            revert RegistryErrors.NotLinked();
        }
        _unlink(handle, tokenPointedByHandle);
    }

    function _unlink(RegistryTypes.Handle memory handle, RegistryTypes.Token memory token) internal {
        delete handleToToken[_handleHash(handle)];
        // tokenToHandle is removed too, as the first version linkage is one-to-one.
        delete tokenToHandle[_tokenHash(token)];
@>      emit RegistryEvents.HandleUnlinked(handle, token, block.timestamp);
    }
```

[TokenHandleRegistry.sol#L72-L91](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol#L72-L91)
[TokenHandleRegistry.sol#L164](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol#L164)

#### Recommended Mitigation Steps

Revert if `tokenPointedByHandle.id` is 0, as it will indicate that the handle is not linked.

### L-03 - MetadataURI has no validation for max length

Profile Image URI has a max size of 6000 as defined via `MAX_PROFILE_IMAGE_URI_LENGTH = 6000;`, but the Metadata URI doesn't have any related validation. This can arise the same problems with large URLs as with the profile image.

- [ProfileLib.sol#L14](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol#L14)
- [ProfileLib.sol#L105-L108](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol#L105-L108)

#### Recommended Mitigation Steps

Apply a validation for max length to Metadata URI as well.

### L-04 - No min length for Lens Handles

Lens Handles has a `MAX_HANDLE_LENGTH`, but it is lacking a `MIN_HANDLE_LENGTH` to prevent users from claiming short handles.

Currently on Lens V1, handles with a length shorter than 5 characters can't be minted.

#### Recommended Mitigation Steps

Implement a min handle length for Lens Handles, not dependant on whitelisted registrators.