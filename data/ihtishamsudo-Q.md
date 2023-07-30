## [L-01] Using External Call In Loop Might Result In DOS Attack 
#### Description
In ```MigrationLib.sol``` There is an external call to the ```initializeFollowModule``` function of the ```IFollowModule``` contract, which is invoked for each ```profileId``` in function ```batchMigrateFollowModules```.
#### Impact 
The external call could consume a significant amount of gas, and processing a large array of ```profileIds``` could  results in a considerable gas cost or function may exceed block gas limit.
#### Vulnerable Code Snippet 
``` solidity
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
#### Vulnerable Code Links
[MigrationLib.sol#L141-L175](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MigrationLib.sol#L141-L175)
[FollowLib.sol#L54-L81](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/FollowLib.sol#L54-L81)
#### Tools Used 
Manual Review / Solidity Visual Developer 
#### Recommended Mitigation Step 
Functions having external calls in loop can batch multiple profileIds per transaction, estimating gas usage before making external calls, and thoroughly testing under various scenarios.
## [L-02] Avoid Using Block.timestamp 
#### Description
Using ```Block.timestamp``` is not a good practice and must be avoided because it can easily be manipulated by attackers
#### Impact 
By manipulating ```Block.timestamp``` attackers can break functionality of smart contract.
#### Vulnerable Code Snippet 
``` solidity 
    function _hasTokenGuardianEnabled(address wallet) internal view returns (bool) {
        return
            !wallet.isContract() &&
            (StorageLib.tokenGuardianDisablingTimestamp()[wallet] == 0 ||
                block.timestamp < StorageLib.tokenGuardianDisablingTimestamp()[wallet]);
    }
```
#### Vulnerable Code Link
[LensProfiles.sol#L142-L147](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensProfiles.sol#L142-L147)
[LensHandles.sol#L149](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L149)
[LensHandles.sol#L253](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L253)
[LensHandles.sol#L266](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/namespaces/LensHandles.sol#L266)
#### Tools Used 
Manual Detection
#### Recommended Mitigation Steps
Avoid Relying on ```block.timestamp```
## [L-03] State variables That Are Used But Never Initialized 
#### Description
Uninitialized state variables.
#### Vulnerable Code Snippet
```mapping(uint256 id => address actionModule) internal _actionModules``` is never initialized but used in ```LensHub.getActionModuleById``` as below
``` solidity
function getActionModuleById(uint256 id) external view override returns (address) {
        return _actionModules[id];
    }
```
#### Vulnerable Code Link
[LensHubStorage.sol#L61](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensHubStorage.sol#L61)
[LensHub.sol#L567](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L567)
#### Tools Used
Static Analyzer
#### Recommended Mitigation Steps
Initialize all the variables. If a variable is meant to be initialized to zero, explicitly set it to zero to improve code readability.
## [NC-01] Variables are not mixedCase
[mapping(address => mapping(uint256 => uint256)) private __DEPRECATED__ownedTokens](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensBaseERC721.sol#L54) is not mixedCase 
