**contracts/libraries/MetaTxLib.sol**
- L169/345/346/347/368/390/391/493 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/base/HubRestricted.sol**
- L24 - No validation is performed in the constructor and the HUB variable is immutable, therefore they should be validated before setting the variable to != 0x.


**contracts/base/LensImplGetters.sol**
- L11 - No validation is performed in the constructor and the variables are immutables, therefore they should be validated before setting the variable to != 0x.


**contracts/base/LensProfiles.sol**
- L33 - No validation is performed in the constructor and the variables are immutables, therefore they should be validated before setting the variable to != 0x.


**contracts/misc/ImmutableOwnable.sol**
- L26 - No validation is performed in the constructor and the variables are immutables, therefore they should be validated before setting the variable to != 0x.

- L26 - You should check that owner != lensHub, otherwise you would enter an incorrect state of the contract.


**contracts/misc/LensV2Migration.sol**
- L26/27/28/29/30 - No validation is performed in the constructor and the variables are immutables, therefore they should be validated before setting the variable to != 0x.


**contracts/misc/LensV2UpgradeContract.sol**
- L33/34/35 - No validation is performed in the constructor and the variables are immutables, therefore they should be validated before setting the variable to != 0x.


**contracts/misc/ProfileCreationProxy.sol**
- L29/30 - No validation is performed in the constructor and the variables are immutables, therefore they should be validated before setting the variable to != 0x.


**contracts/misc/access/Governance.sol**
- L16 - No validation is performed in the constructor and the LENS_HUB variable is immutable, therefore they should be validated before setting the variable to != 0x.


**contracts/misc/access/ProxyAdmin.sol**
- L17 - No validation is performed in the constructor and the LENS_HUB variable is immutable, therefore they should be validated before setting the variable to != 0x.


**contracts/namespaces/TokenHandleRegistry.sol**
- L43/44 - No validation is performed in the constructor and the variables are immutables, therefore they should be validated before setting the variable to != 0x.
