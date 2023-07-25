# Lens Non-Critical Findings

## [N-01] Interfaces should be defined in separate files from their usage
The interfaces below should be defined in separate files, so that it's easier for future projects to import them, and to avoid duplication later on if they need to be used elsewhere in the project.

### 2 Instances
- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol#L13-#L21
- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/Governance.sol#L8-#L10



## [N-02] Empty bytes check is missing
When developing smart contracts in Solidity, it's crucial to validate the inputs of your functions. This includes ensuring that the bytes parameters are not empty, especially when they represent crucial data such as addresses, identifiers, or raw data that the contract needs to process.
Missing empty bytes checks can lead to unexpected behaviour in your contract. For instance, certain operations might fail, produce incorrect results, or consume unnecessary gas when performed with empty bytes. Moreover, missing input validation can potentially expose your contract to malicious activity, including exploitation of unhandled edge cases.
To mitigate these issues, always validate that bytes parameters are not empty when the logic of your contract requires it.

### 3 Instances
- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L129
```solidity
129:    function setFollowModule(
130:        uint256 profileId,
131:        address followModule,
132:        bytes calldata followModuleInitData
134:    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(msg.sender, profileId) {
135:        ProfileLib.setFollowModule(profileId, followModule, followModuleInitData);
136:    }
```
The `setFollowModule()` function above should implement a check for the possibility of empty bytes data being passed to the function as this could lead to uncertain scenarios. The function could be refactored to:
```diff
    function setFollowModule(
        uint256 profileId,
        address followModule,
        bytes calldata followModuleInitData
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(msg.sender, profileId) {
+       if (followModuleInitData.length == 0)
+           revert emptyData();        
        ProfileLib.setFollowModule(profileId, followModule, followModuleInitData);
    }
```

- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L138
```solidity
137:    /// @inheritdoc ILensProtocol
138:    function setFollowModuleWithSig(
139:        uint256 profileId,
140:        address followModule,
141:        bytes calldata followModuleInitData,
142:        Types.EIP712Signature calldata signature
143:    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(signature.signer, profileId) {
144:        MetaTxLib.validateSetFollowModuleSignature(signature, profileId, followModule, followModuleInitData);
145:        ProfileLib.setFollowModule(profileId, followModule, followModuleInitData);
146:    }
```
The `setFollowModuleWithSig()` function above should implement a check for the possibility of `followModuleInitData` being empty (i.e if its length is zero)  as this could lead to uncertain scenarios. The function could be refactored to:
```diff
/// @inheritdoc ILensProtocol
    function setFollowModuleWithSig(
        uint256 profileId,
        address followModule,
        bytes calldata followModuleInitData,
        Types.EIP712Signature calldata signature
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(signature.signer, profileId) {
+       if (followModuleInitData.length == 0)
+           revert emptyData();
        MetaTxLib.validateSetFollowModuleSignature(signature, profileId, followModule, followModuleInitData);
        ProfileLib.setFollowModule(profileId, followModule, followModuleInitData);
    }
```

- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L245
```solidity
245:    function safeTransferFrom(
246:        address from,
247:        address to,
248:        uint256 tokenId,
249:        bytes memory _data
250:    ) public virtual override {
251:        if (!_isApprovedOrOwner(msg.sender, tokenId)) {
252:            revert Errors.NotOwnerOrApproved();
253:        }
254:        _safeTransfer(from, to, tokenId, _data);
255:    }
```
The `safeTransferFrom` function above should implement a check for the possibility of empty bytes data being passed to the function as this could lead to uncertain scenarios. The function could be refactored to:
```diff
    function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId,
        bytes memory _data
    ) public virtual override {
+        if (_data.length == 0)
+            revert emptyData();
        if (!_isApprovedOrOwner(msg.sender, tokenId)) {
            revert Errors.NotOwnerOrApproved();
        }
        _safeTransfer(from, to, tokenId, _data);
    }
```

## [N-03] Multiple mappings can be replaced with a single struct mapping
Using a single struct mapping in place of multiple defined mappings in a Solidity contract can lead to improved code organization, better readability, and easier maintainability. By consolidating related data into a single struct, developers can create a more cohesive data structure that logically groups together relevant pieces of information, thus reducing redundancy and clutter. This approach simplifies the codebase, making it easier to understand, navigate, and modify.

### 3 Instances
- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L42&&#L48
```solidty
40:    // Mapping from token ID to token Data (owner address and mint timestamp uint96), this
41:    // replaces the original mapping(uint256 => address) private _owners;
42:    mapping(uint256 => Types.TokenData) private _tokenData;
43:
.
.
.
47:    // Mapping from token ID to approved address
48:    mapping(uint256 => address) private _tokenApprovals;
```
The `_tokenData` mapping on line 42 and the `_tokenApprovals` mapping on line 48 could be combined to a single uint256 to struct mapping to better enhance code redability, maintainability and organization. The code could be refactored to:
```diff
+    struct TokenInfo{
+       Types.TokenData tokenData;
+       address tokenAddress;
+    }
+
+    mapping (uint256 tokenID => TokenInfo tokenInfo) private _tokens;

-    mapping(uint256 => Types.TokenData) private _tokenData;
-    mapping(uint256 => address) private _tokenApprovals;
```

- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensHubStorage.sol#L36-#L38
- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L36-#L38



## [N-04] Typographical Errors

### Instances
- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Events.sol#L312
```diff
-     * @param followModuleData The data to passed to the follow module, if any.
+     * @param followModuleData The data to pass to the follow module, if any.
```

- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L156
```diff
-     * Comments can have referrers (e.g. publications or profiles that allowed to discover the pointed publication).
+     * Comments can have referrers (e.g. publications or profiles that are allowed to discover the pointed publication).
```

## [N-05]  Else-block not required
One level of nesting can be removed by not having an else block when the if-block returns or reverts.
### 1 Instance
- https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/ERC2981CollectionRoyalties.sol#L46
```solidity
    function _setRoyalty(uint256 royaltiesInBasisPoints) internal virtual {
        if (royaltiesInBasisPoints > BASIS_POINTS) {
            revert Errors.InvalidParameter();
        } else {
            _storeRoyaltiesInBasisPoints(royaltiesInBasisPoints);
        }
    }
```
In the function above the else block of the if-else statement could be removed and the code refactored to:
```diff
     function _setRoyalty(uint256 royaltiesInBasisPoints) internal virtual {
         if (royaltiesInBasisPoints > BASIS_POINTS) {
             revert Errors.InvalidParameter();
-        } else {
        _storeRoyaltiesInBasisPoints(royaltiesInBasisPoints);
-        }
     }
```
