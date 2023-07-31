
## Low Severity Findings

### [L-01] Use cached storage variable instead of `getProfile` again

ProfileLib#createProfile: use cached `_profile` instead of querying `profileId` again:
```solidity
....
        Types.Profile storage _profile = StorageLib.getProfile(profileId);
        _profile.imageURI = createProfileParams.imageURI;
        bytes memory followModuleReturnData;
        if (createProfileParams.followModule != address(0)) {
            // Load the follow module to be used in the next assembly block.
            address followModule = createProfileParams.followModule;
            StorageLib.getProfile(profileId).followModule = followModule; //@audit use cached `_profile`
...
```

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/ProfileLib.sol#L44C11-L53


### [L-02] Remove commented code
Remove commented code from LensV2UpgradeContracts#executeLensV2Upgrade()

```solidity
    function executeLensV2Upgrade() external onlyOwner {
        // _preUpgradeChecks();
        _upgrade();
        // _postUpgradeChecks();
    }
```
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/LensV2UpgradeContract.sol#L44-L48


### [L-03] Unused return data
There are 2 cases where return data is not used:

* MigrationLib#IFollowModule(newFeeFollowModule).initializeFollowModule ignores return value
```solidity 
IFollowModule(newFeeFollowModule).initializeFollowModule({
                    profileId: profileIds[i],
                    transactionExecutor: msg.sender,
                    data: abi.encode(
                        feeFollowModuleData.currency,
                        feeFollowModuleData.amount,
                        feeFollowModuleData.recipient
                    )
                });
```
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MigrationLib.sol#L157C29-L165

* FollowNftProxy#constructor() ignores return value
```solidity
        ILensHub(msg.sender).getFollowNFTImpl().functionDelegateCall(data);
```
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/upgradeability/FollowNFTProxy.sol#L15C12-L15C12

#### Recommended Mitigation Steps
Check return value appropriately or if not, document why this is not necessary.

## Non Critical Findings

###[N-01] Duplicated natspec param
`metadataURI` @param for Types.sol#Profile struct is mentioned twice

```solidity
...
     * @param metadataURI MetadataURI is used to store the profile's metadata, for example: displayed name, description,
     * interests, etc.
     * @param metadataURI The URI to be used for the profile's metadata.
     */
```
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/constants/Types.sol#L100-L103