## [NC-1] No need to emit `block.timestamp` in an event since it is part of a transaction
 There is no need to emit `block.timestamp` in an event since it is part of a transaction

File: 
- https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensProfiles.sol#L72
- https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensProfiles.sol#L86

```
 /// @inheritdoc ILensProfiles
    function DANGER__disableTokenGuardian() external onlyEOA {
        if (StorageLib.tokenGuardianDisablingTimestamp()[msg.sender] != 0) {
            revert Errors.DisablingAlreadyTriggered();
        }
        StorageLib.tokenGuardianDisablingTimestamp()[msg.sender] = block.timestamp + TOKEN_GUARDIAN_COOLDOWN;
        emit Events.TokenGuardianStateChanged({
            wallet: msg.sender,
            enabled: false,
            tokenGuardianDisablingTimestamp: block.timestamp + TOKEN_GUARDIAN_COOLDOWN,
            timestamp: block.timestamp//@audit no need to emit block.timestamp
        });
    }
```

#### Recommended Mitigation Steps
Consider removing block.timestamp from events because it is already in the transaction details.

## [NC-2] Remove commented code
Commented code sometimes is an indication of unfinished work. Consider removing it.

File: https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/LensV2UpgradeContract.sol#L45

```
function executeLensV2Upgrade() external onlyOwner {
        // _preUpgradeChecks(); //@audit commmented code.
        _upgrade();
        // _postUpgradeChecks(); //@audit commmented code.
    }
```
#### Recommended Mitigation Steps
Consider removing all commented codes.

## [NC-3] `owner` parameter of `ControlByContract.sol::contstructor` shadows `Ownable.owner`
The owner parameter of the constructor of ControlByContract.sol contract shadows the `owner` state variable of the parent `Ownable` contract.

File: https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/access/ControllableByContract.sol#L21

```
constructor(address owner) Ownable() {
        _transferOwnership(owner); //@audit shadowing Ownable.owner
    }
```
#### Recommended Mitigation Steps
Consider renaming the `owner` parameter of the ControlByContract.sol to avoid variable shadowing.
