
# 1) setBlockStatus() and setBlockStatusWithSig() dont have return types defined but have return statements in logic. 

**Lines of code**
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L397-L403
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L406-L414


**Proof of Concept**
```
    /// @inheritdoc ILensProtocol
    function setBlockStatus(
        uint256 byProfileId,
        uint256[] calldata idsOfProfilesToSetBlockStatus,
        bool[] calldata blockStatus
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(msg.sender, byProfileId) {
        return ProfileLib.setBlockStatus(byProfileId, idsOfProfilesToSetBlockStatus, blockStatus);
    }

    /// @inheritdoc ILensProtocol
    function setBlockStatusWithSig(
        uint256 byProfileId,
        uint256[] calldata idsOfProfilesToSetBlockStatus,
        bool[] calldata blockStatus,
        Types.EIP712Signature calldata signature
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(signature.signer, byProfileId) {
        MetaTxLib.validateSetBlockStatusSignature(signature, byProfileId, idsOfProfilesToSetBlockStatus, blockStatus);
        return ProfileLib.setBlockStatus(byProfileId, idsOfProfilesToSetBlockStatus, blockStatus);
    }
```
**Tools used**
VS CODE 

**RECOMMENDED MITIGATION**
Since ProfileLib.setBlockStatus() dont return anything, remove the `return` keyword in their logic. 

# 2) no need to add timestamp field to events emitted as events have the timestamp field by default


**Lines of code**
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L135
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L122
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/constants/Events.sol
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/constants/Events.sol



**RECOMMENDED MITIGATION**
remove timestamp field
