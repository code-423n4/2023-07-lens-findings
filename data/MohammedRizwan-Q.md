## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | TOKEN_GUARDIAN_COOLDOWN must be 7 days per proposal documentation | 1 |
| [L&#x2011;02] | mintTimestampOf() returns incorrect mintTimestamp data type | 1 |
| [L&#x2011;03] | isContract() can be bypassed in onlyEOA and isContract() is removed by openzeppelin | 1 |
| [L&#x2011;04] | call should handle with default gas | 1 |
| [L&#x2011;05] | Avoid emitting block.timestamp with event | Contracts |
| [L&#x2011;06] | Prevent immutable addresses setting zero address | 2 |
| [L&#x2011;07] | Use latest version of openzeppelin i.e v4.9.3 | 1 |

### [L&#x2011;01]  TOKEN_GUARDIAN_COOLDOWN must be 7 days per proposal documentation
In LensProfiles.sol contract, TOKEN_GUARDIAN_COOLDOWN is passed in constructor, however it is not checked and it can be passed as 0 making the DANGER__disableTokenGuardian() and enableTokenGuardian() functions fail. 

Per LensProfiles interface,
```
* Disabling the mechanism will have a 7-day timelock before it becomes effective, allowing the owner to re-enable
```

Per further confirmation with sponsor, TOKEN_GUARDIAN_COOLDOWN is 7 days. Howevever it is not validated while setting in constructor. In addition this TOKEN_GUARDIAN_COOLDOWN is immutable meaning once set can not be re-set. Recommend validating this value in constructor.

Proposal reference, [here](https://github.com/lens-protocol/LIPs/blob/main/LIPs/lip-4.md#2-introduce-a-7-day-security-cooldown-period-for-eoa-wallets)

There is 1 instance of this issue:

```Solidity
File: contracts/base/LensProfiles.sol

33    constructor(address moduleGlobals, uint256 tokenGuardianCooldown) {
34        MODULE_GLOBALS = IModuleGlobals(moduleGlobals);
35        TOKEN_GUARDIAN_COOLDOWN = tokenGuardianCooldown;
36    }
```

### Recommended Mitigation steps
```Solidity
File: contracts/base/LensProfiles.sol

    constructor(address moduleGlobals, uint256 tokenGuardianCooldown) {
+       require(tokenGuardianCooldown == 7 days, "invalid cooldown period");
        MODULE_GLOBALS = IModuleGlobals(moduleGlobals);
        TOKEN_GUARDIAN_COOLDOWN = tokenGuardianCooldown;
    }
```

### [L&#x2011;02]   mintTimestampOf() returns incorrect mintTimestamp data type
In LensBaseERC721.sol, mintTimestampOf() function incorrectly returns the uint size of mintTimestamp variable.

```Solidity
File: 

>>  function mintTimestampOf(uint256 tokenId) public view virtual override returns (uint256) {
>>      uint96 mintTimestamp = _tokenData[tokenId].mintTimestamp;
        if (mintTimestamp == 0) {
            revert Errors.TokenDoesNotExist();
        }
        return mintTimestamp;
    }
```

mintTimestamp is stored in uint96 whereas it returns uint256. This can be further verified from TokenData struct.

```Solidity
    struct TokenData {
        address owner;
>>      uint96 mintTimestamp;
    }
```

### Recommended Mitigation steps
1) If the mintTimestamp is required to be downcasted then use openzeppelin safecast library.
2) OR change the function return type to uint96.

```Solidity
-   function mintTimestampOf(uint256 tokenId) public view virtual override returns (uint256) {
+   function mintTimestampOf(uint256 tokenId) public view virtual override returns (uint96) {

        uint96 mintTimestamp = _tokenData[tokenId].mintTimestamp;
        if (mintTimestamp == 0) {
            revert Errors.TokenDoesNotExist();
        }
        return mintTimestamp;
    }
```
### [L&#x2011;03]  isContract() can be bypassed in onlyEOA and isContract() is removed by openzeppelin
In LensProfiles.sol and LensHandles.sol contracts has used onlyEOA() modifier which is given as below,

```Solidity
    modifier onlyEOA() {
        if (msg.sender.isContract()) {
            revert Errors.NotEOA();
        }
        _;
    }
```
This is mainly used for DANGER__disableTokenGuardian() and enableTokenGuardian() in both contracts. It is designed to restict certain operations to Externally Owned account(EOA). However, the vulnerability exist that may allow the malicious contract to bypass this restriction.

The onlyEOA modifier in the contract used openzeppelin Address.isContract() to check whether the address is EOA address or contract address.(openzeppelin officially removed isContract() in latest Address.sol).

Under the hood, it uses ```msg.sender.code.length > 0``` which will be true for the contract address and false for the EOA address.

**References:-**
Openzeppelin has removed isContract() from Address.sol citing potential misuse.
Link for reference- https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3417

Check the Address.sol, isContract function is removed and does not present in contract.
Link for reference- https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol

[Reference to check this Bypass Contract Size Check POC](https://solidity-by-example.org/hacks/contract-size/)

[Reference of Consensys where it also says to follow below recommendation](https://consensys.net/blog/developers/solidity-best-practices-for-smart-contract-security/)

### Recommended Mitigation steps

```Solidity
    modifier onlyEOA() {
-       if (msg.sender.isContract()) {
-           revert Errors.NotEOA();
-       }
+        require(msg.sender == tx.origin, "only EOA allowed");
        _;
    }
```

### [L&#x2011;04]  call should handle with default gas
In Governance.sol, while passing value with call, gasleft() is used. However per solidity documentation, gas limits should not be hardcoded. gasleft() is proposed to be removed in [proposal](https://github.com/ethereum/solidity/issues/8363) however this issue is temporarity inactive now. 
Further discussions on removal of gas opcode [here](https://ethereum-magicians.org/t/eip-2489-deprecate-the-gas-opcode/3958/8).

There is [1](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/access/Governance.sol#L82) instance of this issue:

### Recommended Mitigation steps
```Solidity
File: contracts/misc/access/Governance.sol

    function executeAsGovernance(address target, bytes calldata data)
        external
        payable
        onlyOwnerOrControllerContract
        returns (bytes memory)
    {

        // Some code

-        (bool success, bytes memory returnData) = target.call{gas: gasleft(), value: msg.value}(data);
+        (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

        // Some code

        }
```

### [L&#x2011;05]  Avoid emitting block.timestamp with event
In lens contracts, It is seen that block.timestamp is also emitted with other index parameters, however events are bydefault emits block.timestamp. Therefore there is no need to use block.timestamp in event emits.

For example:
```Solidity
File: contracts/base/LensHubEventHooks.sol

    function emitUnfollowedEvent(uint256 unfollowerProfileId, uint256 idOfProfileUnfollowed) external override {

       // Some code

>>        emit Events.Unfollowed(unfollowerProfileId, idOfProfileUnfollowed, block.timestamp);
    }

    function emitCollectNFTTransferEvent(
        uint256 profileId,
        uint256 pubId,
        uint256 collectNFTId,
        address from,
        address to
    ) external {

       // Some code

>>        emit Events.CollectNFTTransferred(profileId, pubId, collectNFTId, from, to, block.timestamp);
    }
```

### Recommended Mitigation steps
Remove block.timestamp from event emit.

### [L&#x2011;06]  Prevent immutable addresses setting zero address
ImmutableOwnable.sol contract is used to set the immutable address variables. Once these address variables are set they can not be changed due to being immutable. It is recommended to add the zero address check in constructor.

There are [2](https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/misc/ImmutableOwnable.sol#L26-L29) instances of this issue:

```Solidity
File: contracts/misc/ImmutableOwnable.sol

    constructor(address owner, address lensHub) {
        OWNER = owner;
        LENS_HUB = lensHub;
    }
```

### Recommended Mitigation Steps
```Solidity
File: contracts/misc/ImmutableOwnable.sol

    constructor(address owner, address lensHub) {
+       require(owner != address(0), "invalid address");
+       require(lensHub != address(0), "invalid address");
        OWNER = owner;
        LENS_HUB = lensHub;
    }
```
### [L&#x2011;07]  Use latest version of openzeppelin i.e v4.9.3
The lens contracts has used old version of openzeppelin v4.8 as identified from package.json. This openzeppelin version has Medium severity and other bugs which are fixed by openzeppelin in latest versions. When external libraries are used in contracts, It is import that they are free from bugs and should not have vulnerabilities.

### Recommended Mitigation steps
Update the openzeppelin version to v4.9.3
