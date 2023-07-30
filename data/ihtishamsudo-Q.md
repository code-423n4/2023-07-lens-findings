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
