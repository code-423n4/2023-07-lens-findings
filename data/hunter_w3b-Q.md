## [L-01] Name Squatting Risk in Profile Creation

The creation of profiles through the createProfile function. This vulnerability allows users, especially whitelisted profile creators, to create profiles without incurring any cost, leading to the practice of "name squatting." Name squatting refers to the act of reserving desirable handles (profiles) with the intention of selling or profiting from them later. This can result in a negative experience for high-profile users who are unable to acquire their desired handles due to them being already taken by name squatters.


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L89-L102

```solidity
  function createProfile(Types.CreateProfileParams calldata createProfileParams)
        external
        override
        whenNotPaused
        returns (uint256)
    {
        ValidationLib.validateProfileCreatorWhitelisted(msg.sender);
        unchecked {
            uint256 profileId = ++_profileCounter;
            _mint(createProfileParams.to, profileId);
            ProfileLib.createProfile(createProfileParams, profileId);
            return profileId;
        }
    }
```
The createProfile function allows users to create profiles without incurring any costs, and it performs a validation to check if the creator is whitelisted. However, this lack of cost for profile creation can lead to name squatting behavior.


Name Squatting: Whitelisted profile creators can exploit the free profile creation process to create multiple profiles with desirable handles, even if they do not intend to use them. This practice of name squatting can lead to an artificial scarcity of high-demand handles.

User Experience: Legitimate high-profile users may face difficulties in acquiring their desired handles, as they might already be taken by name squatters.


## [L-02] Contracts are vulnerable to fee-on-transfer token related accounting issues

Without measuring the balance before and after the transfer, there's no way to ensure that enough tokens were transferred, in the cases where the token has a fee-on-transfer mechanic. If there are latent funds in the contract, subsequent transfers will succeed.

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58-L59


```solidity
File: /misc/ProfileCreationProxy.sol


       // Transfer the handle & profile to the destination
        LENS_HANDLES.transferFrom(address(this), destination, handleId);
        ILensHub(LENS_HUB).transferFrom(address(this), destination, profileId);

```


## [L-03] Lack of `ERC20` Return Value Check

The smart contract ProfileCreationProxy.sol is susceptible to a vulnerability due to the lack of checking return values when using the transferFrom function of ERC20 tokens. Specifically, some tokens (e.g., USDT) do not conform to the EIP20 standard and return void instead of a successful boolean value. As a result, calling these functions with the correct EIP20 function signatures will always result in a revert.

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58-L59


```solidity
File: /misc/ProfileCreationProxy.sol


       // Transfer the handle & profile to the destination
        LENS_HANDLES.transferFrom(address(this), destination, handleId);
        ILensHub(LENS_HUB).transferFrom(address(this), destination, profileId);

```
In this code, the transferFrom functions of two ERC20 tokens (LENS_HANDLES and ILensHub) are used to transfer handles and profiles to the destination address. However, there is no check performed on the return values of these transferFrom functions, which could lead to reverting transactions when dealing with tokens that do not comply with the EIP20 standard.


Reverted Transactions: Transactions involving the transferFrom function for non-compliant ERC20 tokens will always revert, leading to failed transactions and potential loss of gas fees.

Incomplete Profile Creation: Due to reverting transactions, the profile creation process may be incomplete, leading to unintended consequences or incomplete user profiles.



## [L-04] Some ERC20 revert on `zero value transfer`


Consider checking that the sended value is not zero. Example: https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers.



https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58-L59


```solidity
File: /misc/ProfileCreationProxy.sol


       // Transfer the handle & profile to the destination
        LENS_HANDLES.transferFrom(address(this), destination, handleId);
        ILensHub(LENS_HUB).transferFrom(address(this), destination, profileId);

```



## [L-05] Lack of Input Validation in `LensHub::setProfileMetadataURI` and `LensHub::setProfileMetadataURIWithSig` Functions

`setProfileMetadataURI` and `setProfileMetadataURIWithSig`, which allow users to set profile metadata URIs. However, these functions lack proper input validation for the metadataURI paramete. Without input validation, users can set arbitrary and potentially malicious metadata URIs,metadataURI conforms to the required format and constraints. As a result, invalid metadata URIs might be set.


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L109-L126
```solidity
    function setProfileMetadataURI(uint256 profileId, string calldata metadataURI)
        external
        override
        whenNotPaused
        onlyProfileOwnerOrDelegatedExecutor(msg.sender, profileId)
    {
        ProfileLib.setProfileMetadataURI(profileId, metadataURI);
    }

    /// @inheritdoc ILensProtocol
    function setProfileMetadataURIWithSig(
        uint256 profileId,
        string calldata metadataURI,
        Types.EIP712Signature calldata signature
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(signature.signer, profileId) {
        MetaTxLib.validateSetProfileMetadataURISignature(signature, profileId, metadataURI);
        ProfileLib.setProfileMetadataURI(profileId, metadataURI);
    }
```

## [L-06] Missing Event for `initialize`

Description: Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip

#### Recommendation: Add Event-Emit

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L48-L56


```solidity
  function initialize(uint256 profileId) external override {
        // This is called right after deployment by the LensHub, so we can skip the onlyHub check.
        if (_initialized) {
            revert Errors.Initialized();
        }
        _initialized = true;
        _followedProfileId = profileId;
        _setRoyalty(1000); // 10% of royalties
    }
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol#L45-L52

```solidity

function initialize(uint256 profileId, uint256 pubId) external override {
        if (_initialized) revert Errors.Initialized();
        _initialized = true;
        _setRoyalty(1000); // 10% of royalties
        _profileId = profileId;
        _pubId = pubId;
        // _name and _symbol remain uninitialized because we override the getters below
    }
```

## [L-07]  Use safeTransferOwnership instead of transferOwnership function

transferOwnership function is used to change Ownership from Ownable.sol.

Use a 2 structure transferOwnership which is safer. `safeTransferOwnership`, use is more secure due to 2-stage ownership transfer.

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L21-L23

```solidity
 constructor(address owner) Ownable() {
        _transferOwnership(owner);
    }
``` 
## [L-08] Redundant ownership transfers

In the ControllableByContract.sol contract when ownership is transferred to the owner. By inheriting from Ownable, this will happen automatically.

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L22

```solidity
    constructor(address owner) Ownable() {
        _transferOwnership(owner);
    }
```

