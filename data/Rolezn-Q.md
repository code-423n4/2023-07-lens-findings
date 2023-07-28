## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#low1-add-to-blacklist-function) | Add to `blacklist` function | 1 |
| [LOW&#x2011;2](#low2-ierc20-approve-is-deprecated) | IERC20 `approve()` Is Deprecated | 2 |
| [LOW&#x2011;3](#low3-minting-tokens-to-the-zero-address-should-be-avoided) | Minting tokens to the zero address should be avoided | 2 |
| [LOW&#x2011;4](#low4-nft-contract-redefines-_mint_safemint-but-not-both) | NFT contract redefines `_mint()`/`_safeMint()`, but not both | 2 |
| [LOW&#x2011;5](#low5-remove-unused-code) | Remove unused code | 1 |
| [LOW&#x2011;6](#low6-should-an-airdrop-token-arrive-on-contract-it-will-be-stuck) | Should an airdrop token arrive on contract, it will be stuck | 1 |
| [LOW&#x2011;7](#low7-no-storage-gap-for-upgradeable-contracts) | No Storage Gap For Upgradeable Contracts | 1 |
| [LOW&#x2011;8](#low8-use-safecast-to-safely-downcast-variables) | Use SafeCast to safely downcast variables | 13 |

Total: 23 contexts over 8 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#nc1-the-call-abiencodewithselector-is-not-type-safe) | The call `abi.encodeWithSelector` is not type safe | 1 |
| [NC&#x2011;2](#nc2-consider-adding-a-denylist) | Consider adding a deny-list | 3 |
| [NC&#x2011;3](#nc3-declare-interfaces-on-separate-files) | Declare interfaces on separate files | 1 |
| [NC&#x2011;4](#nc4-elseblock-not-required) | `else`-block not required | 1 |
| [NC&#x2011;5](#nc5-blocktimestamp-is-already-used-when-emitting-events-no-need-to-input-timestamp) | `block.timestamp` is already used when emitting events, no need to input timestamp | 30 |
| [NC&#x2011;6](#nc6-empty-bytes-check-is-missing) | Empty `bytes` check is missing | 20 |
| [NC&#x2011;7](#nc7-some-functions-contain-the-same-exact-logic) | Some functions contain the same exact logic | 2 |
| [NC&#x2011;8](#nc8-generate-perfect-code-headers-for-better-solidity-code-layout-and-readability) | Generate perfect code headers for better solidity code layout and readability | 4 |
| [NC&#x2011;9](#nc9-internal-functions-not-called-by-the-contract-should-be-removed) | `internal` functions not called by the contract should be removed | 9 |
| [NC&#x2011;10](#nc10-consider-moving-msgsender-checks-to-a-common-authorization-modifier) | Consider moving `msg.sender` checks to a common authorization `modifier` | 4 |
| [NC&#x2011;11](#nc11-using--without-specifying-an-upper-bound-is-unsafe) | Using `>`/`>=` without specifying an upper bound is unsafe | 38 |
| [NC&#x2011;12](#nc12-project-should-include-an-emergencystop-pattern) | Project should include an `EmergencyStop` pattern | 3 |
| [NC&#x2011;13](#nc23-use-eip5267-to-describe-eip712-domains) | Use EIP-5267 to describe EIP-712 domains | 2 |
| [NC&#x2011;14](#nc27-use-smtchecker) | Use SMTChecker | 1 |
| [NC&#x2011;15](#nc15-visibility-should-be-set-explicitly-rather-than-defaulting-to-internal) | Visibility should be set explicitly rather than defaulting to `internal` | 22 |
| [NC&#x2011;16](#nc16-zero-as-a-function-argument-should-have-a-descriptive-meaning) | Zero as a function argument should have a descriptive meaning | 21 |

Total: 162 contexts over 16 issues

## Low Risk Issues

### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.



### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> IERC20 `approve()` Is Deprecated

`approve()` is subject to a known front-running attack. It is deprecated in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#IERC20-approve-address-uint256-

#### <ins>Proof Of Concept</ins>

```solidity
File: LensProfiles.sol

117: super.approve(to, tokenId);
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensProfiles.sol#L117

```solidity
File: LensHandles.sol

144: super.approve(to, tokenId);
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/LensHandles.sol#L144



#### <ins>Recommended Mitigation Steps</ins>

Consider using `safeIncreaseAllowance()` / `safeDecreaseAllowance()` instead.





### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Minting tokens to the zero address should be avoided

The core function `mint` is used by users to mint an option position by providing token1 as collateral and borrowing the max amount of liquidity. `Address(0)` check is missing in both this function and the internal function `_mint`, which is triggered to mint the tokens to the to address. Consider applying a check in the function to ensure tokens aren't minted to the zero address.

#### <ins>Proof Of Concept</ins>


```solidity
File: LegacyCollectNFT.sol


function mint(address to) external override returns (uint256) {
        if (msg.sender != HUB) revert Errors.NotHub();
        unchecked {
            uint256 tokenId = ++_tokenIdCounter;
            _mint(to, tokenId);
            return tokenId;
        }
    }

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/LegacyCollectNFT.sol#L55

```solidity
File: LensHandles.sol


function mintHandle(address to, string calldata localName)
        external
        onlyOwnerOrWhitelistedProfileCreator
        returns (uint256)
    {
        _validateLocalName(localName);
        return _mintHandle(to, localName);
    }

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/LensHandles.sol#L87




### <a href="#qa-summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> NFT contract redefines `_mint()`/`_safeMint()`, but not both

If one of the functions is re-implemented, or has new arguments, the other should be as well. The `_mint()` variant is supposed to skip `onERC721Received()` checks, whereas `_safeMint()` does not. Not having both points to a possible issue with spec-compatability.

#### <ins>Proof Of Concept</ins>

```solidity
File: LensBaseERC721.sol

127: function mintTimestampOf(uint256 tokenId) public view virtual override returns (uint256) {

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensBaseERC721.sol#L127

```solidity
File: LegacyCollectNFT.sol

55: function mint(address to) external override returns (uint256) {

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/LegacyCollectNFT.sol#L55




### <a href="#qa-summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Remove unused code
This code is not used in the main project contract files, remove it or add event-emit
Code that is not in use, suggests that they should not be present and could potentially contain insecure functionalities.

#### <ins>Proof Of Concept</ins>


```solidity
File: FollowNFTProxy.sol

function _implementation
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/upgradeability/FollowNFTProxy.sol#L18



### <a href="#qa-summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Should an airdrop token arrive on contract, it will be stuck

With the `_wrap()` function, NFTs are transferred to the contract and in case of airdrop due to these NFTs, it will be stuck in the contract as there is no function to take these airdrop tokens from the contract.

Important NFT project owners are given airdrops, especially since the project includes NFTs such as BAYC, Moonbirds, Doodles, Azuki, there is a high probability of receiving Airdrops, but there is no function to withdraw incoming airdrop tokens, so airdrop tokens will be stuck in the contract.

A common method for airdrops is to collect airdrops with `claim`, so the contract can be considered upgradagable, adding a function to make `claim`.

#### <ins>Proof Of Concept</ins>


```solidity
File: FollowNFT.sol

168: function _wrap(uint256 followTokenId, address wrappedTokenReceiver) internal {
        if (_isFollowTokenWrapped(followTokenId)) {
            revert AlreadyWrapped();
        }
        uint256 followerProfileId = _followDataByFollowTokenId[followTokenId].followerProfileId;
        if (followerProfileId == 0) {
            followerProfileId = _followDataByFollowTokenId[followTokenId].profileIdAllowedToRecover;
            if (followerProfileId == 0) {
                revert FollowTokenDoesNotExist();
            }
            delete _followDataByFollowTokenId[followTokenId].profileIdAllowedToRecover;
        }
        address followerProfileOwner = IERC721(HUB).ownerOf(followerProfileId);
        if (msg.sender != followerProfileOwner) {
            revert DoesNotHavePermissions();
        }
        _mint(wrappedTokenReceiver == address(0) ? followerProfileOwner : wrappedTokenReceiver, followTokenId);
    }

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L168



#### <ins>Recommended Mitigation Steps</ins>
Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```


### <a href="#qa-summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> No storage gap for upgradeable contract might lead to storage slot collision

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts.

Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

#### <ins>Proof Of Concept</ins>

However, the contract doesn't contain a storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

```solidity
File: ProxyAdmin.sol

ProxyAdmin.sol
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/access/ProxyAdmin.sol#L5



#### <ins>Recommended Mitigation Steps</ins>
Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. Please reference OpenZeppelin upgradeable contract templates.
```solidity
    uint256[50] private __gap;
```



### <a href="#qa-summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Use SafeCast to safely downcast variables

Downcasting `int`/`uint`s in Solidity can be unsafe due to the potential for data loss and unintended behavior. When downcasting a larger integer type to a smaller one (e.g., `uint256` to `uint128`), the value may exceed the range of the target type, leading to truncation and loss of significant digits. This data loss can result in unexpected state changes, incorrect calculations, or other contract vulnerabilities, ultimately compromising the contracts functionality and reliability. To prevent these risks, developers should carefully consider the range of values their variables may hold and ensure that proper checks are in place to prevent out-of-range values before performing downcasting. Also consider using OZ SafeCast functionality.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: FollowNFT.sol

392: _followDataByFollowTokenId[followTokenId].followerProfileId = uint160(followerProfileId);
514: _followDataByFollowTokenId[followTokenId].followerProfileId = uint160(followerProfileId);
393: _followDataByFollowTokenId[followTokenId].followTimestamp = uint48(block.timestamp);
396: _followDataByFollowTokenId[followTokenId].originalFollowTimestamp = uint48(block.timestamp);
402: uint48 mintTimestamp = uint48(StorageLib.getTokenData(followTokenId).mintTimestamp);
512: uint48 mintTimestamp = uint48(StorageLib.getTokenData(followTokenId).mintTimestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L392

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L514

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L393

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L396

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L402

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L512



```solidity
File: LensBaseERC721.sol

365: _tokenData[tokenId].mintTimestamp = uint96(block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensBaseERC721.sol#L365

```solidity
File: GovernanceLib.sol

100: uint248(id),

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L100

```solidity
File: PublicationLib.sol

173: if (uint8(pubType) == 0) {

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/PublicationLib.sol#L173



</details>



## Non Critical Issues

### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> The call `abi.encodeWithSelector` is not type safe

The call `abi.encodeCall` should be used instead. Note `abi.encodeCall` is only safe from typographical errors in solidity versions 0.8.13 and above

#### <ins>Proof Of Concept</ins>


```solidity
File: FollowLib.sol

91: bytes memory functionData = abi.encodeWithSelector(IFollowNFT.initialize.selector, profileId);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/FollowLib.sol#L91



### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Consider adding a deny-list

Doing so will significantly increase centralization, but will help to prevent hackers from using stolen tokens

#### <ins>Proof Of Concept</ins>


```solidity
File: FollowNFT.sol

18: contract FollowNFT is HubRestricted, LensBaseERC721, ERC2981CollectionRoyalties, IFollowNFT

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L18

```solidity
File: LegacyCollectNFT.sol

23: contract LegacyCollectNFT is LensBaseERC721, ERC2981CollectionRoyalties, ICollectNFT

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/LegacyCollectNFT.sol#L23

```solidity
File: LensHandles.sol

25: contract LensHandles is ERC721, ImmutableOwnable, ILensHandles

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/LensHandles.sol#L25




### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Declare interfaces on separate files

Interfaces should be declared on a separate file and not on the same file where the contract resides in.

#### <ins>Proof Of Concept</ins>

```solidity
File: Governance.sol

8: interface ILensHub_V1 {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/access/Governance.sol#L8



### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> `else`-block not required

One level of nesting can be removed by not having an `else` block when the `if`-block returns, and `if (foo) { return 1; } else { return 2; }` becomes `if (foo) { return 1; } return 2;`

#### <ins>Proof Of Concept</ins>


```solidity
File: PublicationLib.sol

196: if (pubType == Types.PublicationType.Mirror) {
            return StorageLib.getPublication(_publication.pointedProfileId, _publication.pointedPubId).contentURI;
        } else {
            return StorageLib.getPublication(profileId, pubId).contentURI;
        }

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/PublicationLib.sol#L196






### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> block.timestamp is already used when emitting events, no need to input timestamp

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LensHubEventHooks.sol

17: emit Events.Unfollowed(unfollowerProfileId, idOfProfileUnfollowed, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensHubEventHooks.sol#L17

```solidity
File: LensHubEventHooks.sol

36: emit Events.CollectNFTTransferred(profileId, pubId, collectNFTId, from, to, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensHubEventHooks.sol#L36

```solidity
File: ActionLib.sol

60: emit Events.Acted(publicationActionParams, actionModuleReturnData, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ActionLib.sol#L60

```solidity
File: FollowLib.sol

75: emit Events.Unfollowed(unfollowerProfileId, idOfProfileToUnfollow, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/FollowLib.sol#L75

```solidity
File: FollowLib.sol

93: emit Events.FollowNFTDeployed(profileId, followNFT, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/FollowLib.sol#L93

```solidity
File: GovernanceLib.sol

19: emit Events.GovernanceSet(msg.sender, prevGovernance, newGovernance, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L19

```solidity
File: GovernanceLib.sol

30: emit Events.EmergencyAdminSet(msg.sender, prevEmergencyAdmin, newEmergencyAdmin, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L30

```solidity
File: GovernanceLib.sol

61: emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L61

```solidity
File: GovernanceLib.sol

67: emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L67

```solidity
File: GovernanceLib.sol

73: emit Events.ProfileCreatorWhitelisted(profileCreator, whitelist, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L73

```solidity
File: GovernanceLib.sol

78: emit Events.FollowModuleWhitelisted(followModule, whitelist, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L78

```solidity
File: GovernanceLib.sol

83: emit Events.ReferenceModuleWhitelisted(referenceModule, whitelist, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L83

```solidity
File: GovernanceLib.sol

109: emit Events.ActionModuleWhitelisted(actionModule, id, whitelist, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L109

```solidity
File: LegacyCollectLib.sol

142: emit Events.CollectNFTDeployed(profileId, pubId, collectNFT, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/LegacyCollectLib.sol#L142

```solidity
File: ProfileLib.sol

102: emit Events.FollowModuleSet(profileId, followModule, followModuleReturnData, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L102

```solidity
File: ProfileLib.sol

107: emit Events.ProfileMetadataSet(profileId, metadataURI, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L107

```solidity
File: ProfileLib.sol

125: emit Events.ProfileImageURISet(profileId, imageURI, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L125

```solidity
File: ProfileLib.sol

151: emit Events.Unfollowed(idOfProfileToSetBlockStatus, byProfileId, block.timestamp);
156: emit Events.Blocked(byProfileId, idOfProfileToSetBlockStatus, block.timestamp);
158: emit Events.Unblocked(byProfileId, idOfProfileToSetBlockStatus, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L151

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L156

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L158



```solidity
File: PublicationLib.sol

128: emit Events.MirrorCreated(mirrorParams, pubIdAssigned, referenceModuleReturnData, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/PublicationLib.sol#L128

```solidity
File: ModuleGlobals.sol

98: emit Events.ModuleGlobalsGovernanceSet(prevGovernance, newGovernance, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/ModuleGlobals.sol#L98

```solidity
File: ModuleGlobals.sol

105: emit Events.ModuleGlobalsTreasurySet(prevTreasury, newTreasury, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/ModuleGlobals.sol#L105

```solidity
File: ModuleGlobals.sol

112: emit Events.ModuleGlobalsTreasuryFeeSet(prevTreasuryFee, newTreasuryFee, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/ModuleGlobals.sol#L112

```solidity
File: ModuleGlobals.sol

119: emit Events.ModuleGlobalsCurrencyWhitelisted(currency, prevWhitelisted, toWhitelist, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/ModuleGlobals.sol#L119

```solidity
File: LensHandles.sol

198: emit HandlesEvents.HandleMinted(localName, NAMESPACE, tokenId, to, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/LensHandles.sol#L198

```solidity
File: TokenHandleRegistry.sol

141: emit RegistryEvents.HandleLinked(handle, token, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/TokenHandleRegistry.sol#L141

```solidity
File: TokenHandleRegistry.sol

148: emit RegistryEvents.HandleUnlinked(handle, tokenPointedByHandle, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/TokenHandleRegistry.sol#L148

```solidity
File: TokenHandleRegistry.sol

156: emit RegistryEvents.HandleUnlinked(handlePointedByToken, token, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/TokenHandleRegistry.sol#L156

```solidity
File: TokenHandleRegistry.sol

164: emit RegistryEvents.HandleUnlinked(handle, token, block.timestamp);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/TokenHandleRegistry.sol#L164



</details>




### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Empty `bytes` check is missing

To avoid mistakenly accepting empty `bytes` as a valid value, consider to check for empty `bytes`. This helps ensure that empty `bytes` are not treated as valid inputs, preventing potential issues.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: FollowNFT.sol

263: function supportsInterface(bytes4 interfaceId)
        public
        view
        virtual
        override(LensBaseERC721, ERC2981CollectionRoyalties)
        returns (bool)
    {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L263

```solidity
File: LensHub.sol

130: function setFollowModule(
        uint256 profileId,
        address followModule,
        bytes calldata followModuleInitData
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(msg.sender, profileId) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/LensHub.sol#L130

```solidity
File: LensHub.sol

139: function setFollowModuleWithSig(
        uint256 profileId,
        address followModule,
        bytes calldata followModuleInitData,
        Types.EIP712Signature calldata signature
    ) external override whenNotPaused onlyProfileOwnerOrDelegatedExecutor(signature.signer, profileId) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/LensHub.sol#L139

```solidity
File: ERC2981CollectionRoyalties.sol

17: function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/ERC2981CollectionRoyalties.sol#L17

```solidity
File: LensBaseERC721.sol

79: function tokenURI(uint256 tokenId) external view virtual returns (string memory);
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensBaseERC721.sol#L79

```solidity
File: LensBaseERC721.sol

246: function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId,
        bytes memory _data
    ) public virtual override {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensBaseERC721.sol#L246

```solidity
File: LensBaseERC721.sol

303: function _safeTransfer(
        address from,
        address to,
        uint256 tokenId,
        bytes memory _data
    ) internal virtual {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensBaseERC721.sol#L303

```solidity
File: LensBaseERC721.sol

470: function _checkOnERC721Received(
        address from,
        address to,
        uint256 tokenId,
        bytes memory _data
    ) private returns (bool) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensBaseERC721.sol#L470

```solidity
File: LensProfiles.sol

131: function supportsInterface(bytes4 interfaceId)
        public
        view
        virtual
        override(LensBaseERC721, ERC2981CollectionRoyalties, IERC165)
        returns (bool)
    {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensProfiles.sol#L131

```solidity
File: FollowLib.sol

99: function _follow(
        uint256 followerProfileId,
        address transactionExecutor,
        uint256 idOfProfileToFollow,
        uint256 followTokenId,
        bytes calldata followModuleData
    ) private returns (uint256) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/FollowLib.sol#L99

```solidity
File: MetaTxLib.sol

65: function validateSetFollowModuleSignature(
        Types.EIP712Signature calldata signature,
        uint256 profileId,
        address followModule,
        bytes calldata followModuleInitData
    ) external {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/MetaTxLib.sol#L65

```solidity
File: MetaTxLib.sol

469: function _validateRecoveredAddress(bytes32 digest, Types.EIP712Signature calldata signature) private view {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/MetaTxLib.sol#L469

```solidity
File: MetaTxLib.sol

492: function _calculateDigest(bytes32 hashedMessage) private view returns (bytes32) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/MetaTxLib.sol#L492

```solidity
File: ProfileLib.sol

93: function setFollowModule(
        uint256 profileId,
        address followModule,
        bytes calldata followModuleInitData
    ) external {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L93

```solidity
File: ProfileLib.sol

111: function _initFollowModule(
        uint256 profileId,
        address transactionExecutor,
        address followModule,
        bytes memory followModuleInitData
    ) private returns (bytes memory) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L111

```solidity
File: PublicationLib.sol

522: function _initPubReferenceModule(
        uint256 profileId,
        address transactionExecutor,
        uint256 pubId,
        address referenceModule,
        bytes memory referenceModuleInitData
    ) private returns (bytes memory) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/PublicationLib.sol#L522

```solidity
File: LegacyCollectNFT.sol

91: function supportsInterface(bytes4 interfaceId)
        public
        view
        virtual
        override(ERC2981CollectionRoyalties, LensBaseERC721)
        returns (bool)
    {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/LegacyCollectNFT.sol#L91

```solidity
File: Governance.sol

73: function executeAsGovernance(address target, bytes calldata data)
        external
        payable
        onlyOwnerOrControllerContract
        returns (bytes memory)
    {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/access/Governance.sol#L73

```solidity
File: ProxyAdmin.sol

47: function proxy_upgradeAndCall(address newImplementation, bytes calldata data)
        external
        onlyOwnerOrControllerContract
    {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/access/ProxyAdmin.sol#L47

```solidity
File: LensHandles.sol

249: function _isAlphaNumeric(bytes1 char) internal pure returns (bool) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/LensHandles.sol#L249



</details>





### <a href="#qa-summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Some functions contain the same exact logic

These functions might be a problem if the logic changes before the contract is deployed, as the developer must remember to syncronize the logic between all the function instances.

Consider using a single function instead of duplicating the code, for example by using a `library`, or through inheritance.

#### <ins>Proof Of Concept</ins>


```solidity
File: FollowNFT.sol

467: function _getRoyaltiesInBasisPointsSlot() internal pure override returns (uint256) {
        uint256 slot;
        assembly {
            slot := _royaltiesInBasisPoints.slot
        }
        return slot;
    }

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L467

```solidity
File: LegacyCollectNFT.sol

116: function _getRoyaltiesInBasisPointsSlot() internal pure override returns (uint256) {
        uint256 slot;
        assembly {
            slot := _royaltiesInBasisPoints.slot
        }
        return slot;
    }

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts\misc\LegacyCollectNFT.sol#L116



### <a href="#qa-summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Generate perfect code headers for better solidity code layout and readability

It is recommended to use pre-made headers for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```


#### <ins>Proof Of Concept</ins>


```solidity
File: FollowNFT.sol

11: FollowNFT.sol

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L11

```solidity
File: LensImplGetters.sol

5: LensImplGetters.sol

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensImplGetters.sol#L5

```solidity
File: LensProfiles.sol

8: LensProfiles.sol

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensProfiles.sol#L8

```solidity
File: ModuleGlobals.sol

7: ModuleGlobals.sol

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/ModuleGlobals.sol#L7


### <a href="#qa-summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> `internal` functions not called by the contract should be removed

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: FollowNFT.sol

function _beforeTokenTransfer(
        address from,
        address to,
        uint256 followTokenId
    ) internal override {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L436

```solidity
File: FollowNFT.sol

function _getReceiver(
        uint256 
    ) internal view override returns (address) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L449

```solidity
File: FollowNFT.sol

function _beforeRoyaltiesSet(
        uint256 
    ) internal view override {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L455

```solidity
File: FollowNFT.sol

function _getRoyaltiesInBasisPointsSlot() internal pure override returns (uint256) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L467

```solidity
File: FollowNFTProxy.sol

function _implementation() internal view override returns (address) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/upgradeability/FollowNFTProxy.sol#L18

```solidity
File: LegacyCollectNFT.sol

function _getReceiver(
        uint256 
    ) internal view override returns (address) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/LegacyCollectNFT.sol#L102

```solidity
File: LegacyCollectNFT.sol

function _beforeRoyaltiesSet(
        uint256 
    ) internal view override {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/LegacyCollectNFT.sol#L108

```solidity
File: LegacyCollectNFT.sol

function _getRoyaltiesInBasisPointsSlot() internal pure override returns (uint256) {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/LegacyCollectNFT.sol#L116

```solidity
File: LensHandles.sol

function _beforeTokenTransfer(
        address from,
        address to,
        uint256 ,
        uint256 batchSize
    ) internal override {
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/LensHandles.sol#L260



</details>



### <a href="#qa-summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Consider moving `msg.sender` checks to a common authorization `modifier`

#### <ins>Proof Of Concept</ins>


```solidity
File: FollowNFT.sol

133: if (followTokenOwner == msg.sender || isApprovedForAll(followTokenOwner, msg.sender)) {

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L133

```solidity
File: LensBaseERC721.sol

201: if (operator == msg.sender) {

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensBaseERC721.sol#L201

```solidity
File: GovernanceLib.sol

54: if (msg.sender == StorageLib.getEmergencyAdmin()) {

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/GovernanceLib.sol#L54

```solidity
File: Governance.sol

79: if (msg.sender == controllerContract && target == address(LENS_HUB)) {

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/access/Governance.sol#L79




### <a href="#qa-summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> Using `>`/`>=` without specifying an upper bound is unsafe

There will be breaking changes in future versions of solidity, and at that point your code will no longer be compatable. While you may have the specific version to use in a configuration file, others that include your source files may not.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: ICollectModule.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ICollectModule.sol#L3

```solidity
File: ICollectNFT.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ICollectNFT.sol#L3

```solidity
File: IERC721Burnable.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/IERC721Burnable.sol#L3

```solidity
File: IERC721MetaTx.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/IERC721MetaTx.sol#L3

```solidity
File: IERC721Timestamped.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/IERC721Timestamped.sol#L3

```solidity
File: IFollowModule.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/IFollowModule.sol#L3

```solidity
File: IFollowNFT.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/IFollowNFT.sol#L3

```solidity
File: ILegacyCollectModule.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILegacyCollectModule.sol#L3

```solidity
File: ILegacyCollectNFT.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILegacyCollectNFT.sol#L3

```solidity
File: ILegacyReferenceModule.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILegacyReferenceModule.sol#L3

```solidity
File: ILensERC721.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensERC721.sol#L3

```solidity
File: ILensGovernable.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensGovernable.sol#L3

```solidity
File: ILensHandles.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensHandles.sol#L3

```solidity
File: ILensHub.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensHub.sol#L3

```solidity
File: ILensHubEventHooks.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensHubEventHooks.sol#L3

```solidity
File: ILensHubInitializable.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensHubInitializable.sol#L3

```solidity
File: ILensImplGetters.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensImplGetters.sol#L3

```solidity
File: ILensProfiles.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensProfiles.sol#L3

```solidity
File: ILensProtocol.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ILensProtocol.sol#L3

```solidity
File: IModuleGlobals.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/IModuleGlobals.sol#L3

```solidity
File: IPublicationActionModule.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/IPublicationActionModule.sol#L3

```solidity
File: IReferenceModule.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/IReferenceModule.sol#L3

```solidity
File: ITokenHandleRegistry.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/interfaces/ITokenHandleRegistry.sol#L3

```solidity
File: Errors.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/constants/Errors.sol#L3

```solidity
File: Errors.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/constants/Errors.sol#L3

```solidity
File: Events.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/constants/Events.sol#L3

```solidity
File: Events.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/constants/Events.sol#L3

```solidity
File: Typehash.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/constants/Typehash.sol#L3

```solidity
File: Types.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/constants/Types.sol#L3

```solidity
File: Types.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/constants/Types.sol#L3

```solidity
File: Errors.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/modules/constants/Errors.sol#L3

```solidity
File: Errors.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/modules/constants/Errors.sol#L3

```solidity
File: Errors.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/constants/Errors.sol#L3

```solidity
File: Errors.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/constants/Errors.sol#L3

```solidity
File: Events.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/constants/Events.sol#L3

```solidity
File: Events.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/constants/Events.sol#L3

```solidity
File: Types.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/constants/Types.sol#L3

```solidity
File: Types.sol

3: pragma solidity >=0.6.0;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/constants/Types.sol#L3



</details>



### <a href="#qa-summary">[NC&#x2011;12]</a><a name="NC&#x2011;12"> Project should include an `EmergencyStop` pattern

At the initialization of the project's contracts, the system may need to be stopped or upgraded. It is suggested to have a script beforehand and add an `EmergencyStop` pattern. This can also be called an <a href="Circut Breaker Pattern">https://consensys.github.io/smart-contract-best-practices/development-recommendations/precautions/circuit-breakers/</a>


#### <ins>Proof Of Concept</ins>


```solidity
File: FollowNFT.sol

48: function initialize(

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L48

```solidity
File: LensBaseERC721.sol

71: function _initialize(

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensBaseERC721.sol#L71

```solidity
File: LegacyCollectNFT.sol

45: function initialize(

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/misc/LegacyCollectNFT.sol#L45




### <a href="#qa-summary">[NC&#x2011;13]</a><a name="NC&#x2011;13"> Use EIP-5267 to describe EIP-712 domains

EIP-5267 is a standard which allows for the retrieval and description of EIP-712 hash domains. This enable external tools to allow users to view the fields and values that describe their domain.

This is especially useful when a project may exist on multiple chains and or in multiple contracts, and allows users/tools to verify that the signature is for the right fork, chain, version, contract, etc.

#### <ins>Proof Of Concept</ins>


```solidity
File: MetaTxLib.sol

30: *         keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)'),
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/MetaTxLib.sol#L30

```solidity
File: Typehash.sol

17: bytes32 constant EIP712_DOMAIN = keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)');
```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/constants/Typehash.sol#L17





### <a href="#qa-summary">[NC&#x2011;14]</a><a name="NC&#x2011;14"> Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19



### <a href="#qa-summary">[NC&#x2011;15]</a><a name="NC&#x2011;15"> Visibility should be set explicitly rather than defaulting to `internal`

#### <ins>Proof Of Concept</ins>


```solidity
File: ProfileLib.sol

14: uint16 constant MAX_PROFILE_IMAGE_URI_LENGTH = 6000;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/ProfileLib.sol#L14

```solidity
File: StorageLib.sol

11: uint256 constant TOKEN_DATA_MAPPING_SLOT = 2;
19: uint256 constant SIG_NONCES_MAPPING_SLOT = 10;
20: uint256 constant LAST_INITIALIZED_REVISION_SLOT = 11; // VersionedInitializable's `lastInitializedRevision` field.
21: uint256 constant PROTOCOL_STATE_SLOT = 12;
22: uint256 constant PROFILE_CREATOR_WHITELIST_MAPPING_SLOT = 13;
23: uint256 constant FOLLOW_MODULE_WHITELIST_MAPPING_SLOT = 14;
24: uint256 constant ACTION_MODULE_WHITELIST_DATA_MAPPING_SLOT = 15;
25: uint256 constant REFERENCE_MODULE_WHITELIST_MAPPING_SLOT = 16;
27: uint256 constant PROFILE_ID_BY_HANDLE_HASH_MAPPING_SLOT = 18; // Deprecated slot, but still needed for V2 migration.
28: uint256 constant PROFILES_MAPPING_SLOT = 19;
29: uint256 constant PUBLICATIONS_MAPPING_SLOT = 20;
31: uint256 constant PROFILE_COUNTER_SLOT = 22;
32: uint256 constant GOVERNANCE_SLOT = 23;
33: uint256 constant EMERGENCY_ADMIN_SLOT = 24;
37: uint256 constant TOKEN_GUARDIAN_DISABLING_TIMESTAMP_MAPPING_SLOT = 25;
41: uint256 constant DELEGATED_EXECUTOR_CONFIG_MAPPING_SLOT = 26;
42: uint256 constant BLOCKED_STATUS_MAPPING_SLOT = 27;
43: uint256 constant ACTION_MODULES_SLOT = 28;
44: uint256 constant MAX_ACTION_MODULE_ID_USED_SLOT = 29;
45: uint256 constant PROFILE_ROYALTIES_BPS_SLOT = 30;
47: uint256 constant MAX_ACTION_MODULE_ID_SUPPORTED = 255;

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L11

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L19

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L20

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L21

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L22

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L23

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L24

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L25

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L27

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L28

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L29

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L31

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L32

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L33

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L37

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L41

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L42

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L43

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L44

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L45

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L47








### <a href="#qa-summary">[NC&#x2011;16]</a><a name="NC&#x2011;16"> Zero as a function argument should have a descriptive meaning

Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: FollowNFT.sol

331: _approveFollow(0, followTokenId);
444: _approveFollow(0, followTokenId);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L331

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/FollowNFT.sol#L444


```solidity
File: LensProfiles.sol

85: tokenGuardianDisablingTimestamp: 0,

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/base/LensProfiles.sol#L85

```solidity
File: StorageLib.sol

55: mstore(0, profileId)
65: mstore(0, profileId)
57: mstore(32, keccak256(0, 64))
58: mstore(0, pubId)
59: _publication.slot := keccak256(0, 64)

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L55-L59

```solidity
File: StorageLib.sol

65: mstore(0, profileId)
67: _profiles.slot := keccak256(0, 64)

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L65

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L67



```solidity
File: StorageLib.sol

77: mstore(0, delegatorProfileId)
79: _delegatedExecutorsConfig.slot := keccak256(0, 64)

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L77

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L79



```solidity
File: StorageLib.sol

95: mstore(0, tokenId)
97: _tokenData.slot := keccak256(0, 64)

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L95

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L97



```solidity
File: StorageLib.sol

107: mstore(0, blockerProfileId)
109: _blockedStatus.slot := keccak256(0, 64)

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L107

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/libraries/StorageLib.sol#L109



```solidity
File: LensHandles.sol

134: tokenGuardianDisablingTimestamp: 0,

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/LensHandles.sol#L134

```solidity
File: LensHandles.sol

271: super._beforeTokenTransfer(from, to, 0, batchSize);

```

https://github.com/code-423n4/2023-07-lens/tree/main/contracts/namespaces/LensHandles.sol#L271



</details>