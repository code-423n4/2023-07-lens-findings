## [L-01] setState function may not be usable by governance

[GovernanceLib.sol#L50-L62](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/GovernanceLib.sol#L50C1-L62C6)

**setState** function in GovernanceLib.sol allow Governance to set any state and EmergencyAdmin to pause further. 
Internal checks of the function made in a way that firstly msg.sender checked to be EmergencyAdmin and if no checked further to be Governance. 
It is possible that msg.sender has both these roles. It might be doe on purpose, accidentally or due to governance transition. In this rare circumstances address with Governance role won't be able to unpausa state. 

### Mitigation

Change the order of internal checks. For example,

```solidity
    function setState(Types.ProtocolState newState) external {
        // NOTE: This does not follow the CEI-pattern, but there is no interaction and this allows to abstract `_setState` logic.
        Types.ProtocolState prevState = _setState(newState);
        // If the sender is the emergency admin, prevent them from reducing restrictions.

        if (msg.sender != StorageLib.getGovernance()) {
            if (msg.sender == StorageLib.getEmergencyAdmin()) {
                if (newState <= prevState) {
                   revert Errors.EmergencyAdminCanOnlyPauseFurther();
                }
            }
            else {
                revert Errors.NotGovernanceOrEmergencyAdmin();
            }
        }
        
        emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);
    }
```

## [L-02] Same event called twice in while changing state

[GovernanceLib.sol#L61](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/GovernanceLib.sol#L61C1-L61C80)

[GovernanceLib.sol#L67](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/GovernanceLib.sol#L67C1-L67C80)

To change protocol state function **setState** of the LensGovernance contract should be called. This function calls **setState** of the GovernanceLib that further calls **_setState** of the GovernanceLib. 
Both **setState** and **_setState** function of the GovernanceLib contain the same SetState event 
```
emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);
```

During the state change two event will be emitted that might mislead users of the protocol.

### Mitigation

Delete event emitting in the setState function of the [GovernanceLib.sol](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/GovernanceLib.sol#L61C1-L61C80)

## [L-03] Wrong error message

[ValidationLib.sol#L30-L40](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/ValidationLib.sol#L30C1-L40C6)

validateAddressIsProfileOwnerOrDelegatedExecutor function checks if an address is Profile Owner or Delegated Executor. The function may revert only with **Errors.ExecutorInvalid()** inside validateAddressIsDelegatedExecutor function. But it rather should revert with **Errors.NotProfileOwnerNorExecutor()**.

### Mitigation
Add appropriate Error handle
