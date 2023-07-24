# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       |isContract() can be bypassed|1|
|L-02       |`TOKEN_GUARDIAN_COOLDOWN` should be a constant of 7 days|1|

## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | Remove block.timestamp and block.number from events  | - |
|N-02       | Use storage variable instead of retrieving it from library  | - |
|N-03       | Don't write to memory when a variable is only used once  | - |
# Details
## Low Risk
## [L-01] isContract() can be bypassed
The LensProfiles modifier checks if the contract is an EOA. It uses the `isContract` function from the OpenZeppelin Adress library.

[LensProfiles.sol#L50-L55](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L50-L55)
```solidity
    modifier onlyEOA() {
        if (msg.sender.isContract()) {
            revert Errors.NotEOA();
        }
        _;
    }
```

[Address.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/utils/Address.sol#L40-L46)
```solidity
    function isContract(address account) internal view returns (bool) {
        // This method relies on extcodesize/address.code.length, which returns 0
        // for contracts in construction, since the code is only stored at the end
        // of the constructor execution.

        return account.code.length > 0;
    }
```

If an address is a contract then the size of code stored at the address will be greater than 0. However if the function is called from the constructor, the code size will still be 0. The check .isContract() will think it's an EOA when in reality it's a contract.

Instead of checking `account.code.length > 0`. Do `msg.sender != tx.origin`

## [N-02] `TOKEN_GUARDIAN_COOLDOWN` should be a constant of 7 days
The interface documentation of the function `DANGER__disableTokenGuardian` says it has a 7 day cooldown. However the `` is a immutable variable that's assigned to in the constructor. This is prone to errors and should be avoided. Since it's immutable there is no chance to change it afterwards. It's better to make it a constant with a value of 7 days so no mistake can be made by the admin.
[ILensProfiles.sol#L8-L15](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProfiles.sol#L8-L15)
```solidity
    /**
     * @notice DANGER: Triggers disabling the profile protection mechanism for the msg.sender, which will allow
     * transfers or approvals over profiles held by it.
     * Disabling the mechanism will have a 7-day timelock before it becomes effective, allowing the owner to re-enable
     * the protection back in case of being under attack.
     * The protection layer only applies to EOA wallets.
     */
    function DANGER__disableTokenGuardian() external;
```
[LensProfiles.sol#L31-L36](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol#L31-L36)
```solidity
    uint256 internal immutable TOKEN_GUARDIAN_COOLDOWN;

    constructor(address moduleGlobals, uint256 tokenGuardianCooldown) {
        MODULE_GLOBALS = IModuleGlobals(moduleGlobals);
        TOKEN_GUARDIAN_COOLDOWN = tokenGuardianCooldown;
    }
```
## Non critical
## [N-01] Remove block.timestamp and block.number from events
block.timestamp and block.number are added to event information by default so adding them manually is unnecessary and a gas waster.

This is used in almost all of the emit statements.

## [N-02] Use storage variable instead of retrieving it from library again
`getProfile()` returns a profile in storage. When this is already retrieved from the library, it should be reused instead of calling `getProfile()` again.

[ProfileLib.sol#L44-L52](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol#L44-L52)
```diff
        Types.Profile storage _profile = StorageLib.getProfile(profileId);
        _profile.imageURI = createProfileParams.imageURI;

        bytes memory followModuleReturnData;
        if (createProfileParams.followModule != address(0)) {
            // Load the follow module to be used in the next assembly block.
            address followModule = createProfileParams.followModule;

-           StorageLib.getProfile(profileId).followModule = followModule;
+           _profile.followModule = followModule;
        }
```

## [N-03] Don't write to memory when a variable is only used once
When a storage variable or a parameter is only used once, it's unnecessary to assign it to a memory variable.

[ProfileLib.sol#L265-L275](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol#L265-L275)
```diff
    if (configNumber == nextAvailableConfigNumber) {
        // The next configuration available is being changed, it must be marked.
        // Otherwise, on a profile transfer, the next owner can inherit a used/dirty configuration.
        _delegatedExecutorsConfig.maxConfigNumberSet = nextAvailableConfigNumber;
-       configSwitched = switchToGivenConfig;
-       if (configSwitched) {
+       if (switchToGivenConfig) {
            // The configuration is being switched, previous and current configuration numbers must be updated.
            _delegatedExecutorsConfig.prevConfigNumber = _delegatedExecutorsConfig.configNumber;
            _delegatedExecutorsConfig.configNumber = nextAvailableConfigNumber;
        }
    } else {
```