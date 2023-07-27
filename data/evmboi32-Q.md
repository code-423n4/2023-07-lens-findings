# `StateSet` emits the event twice

The `setState` function emits the `StateSet` event twice. Once in the internal call of the `_setState` function, and another time in the `setState` function. Should emit it only once.

The following code is inside the `GovernanceLib.sol`  
https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/GovernanceLib.sol#L50-L69

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
    // @audit emits the StateSet event twice, once in `_setState` and once here.
    emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);
}

function _setState(Types.ProtocolState newState) private returns (Types.ProtocolState) {
    Types.ProtocolState prevState = StorageLib.getState();
    StorageLib.setState(newState);
    emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);
    return prevState;
}
```


# Not following the EIP712  standard correctly
If we take a look at the EIP712 standard https://eips.ethereum.org/EIPS/eip-712 it states the following

`
The array values are encoded as the keccak256 hash of the concatenated encodeData of their contents (i.e. the encoding of SomeType[5] is identical to that of a struct containing five members of type SomeType).
`

There are a few examples in the ```MetaTxLib.sol``` that don't follow this convenction.

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MetaTxLib.sol#L103-L104

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MetaTxLib.sol#L147

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MetaTxLib.sol#L242-L243

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MetaTxLib.sol#L245

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MetaTxLib.sol#L272-L273

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MetaTxLib.sol#L275

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MetaTxLib.sol#L297-L298

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/libraries/MetaTxLib.sol#L437-L438

According to the EIP712 standard arrays should be encoded as: ```keccak256(abi.encode(array))```.