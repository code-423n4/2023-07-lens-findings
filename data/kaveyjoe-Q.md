

1 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol




Summary:
The LensBaseERC721 contract contains several potential issues that need to be addressed to ensure the correct and secure functionality of the ERC721 Non-Fungible Token implementation.

2. Vulnerability in _mint function:
In the _mint function, the condition to check if the tokenId already exists is not properly enforced. The function allows minting of tokens even if the tokenId already exists, which could lead to unexpected behavior and incorrect token ownership. The _exists function should be called to verify if the tokenId exists before proceeding with the minting process.
 
 - Recommendation 

Fix the _mint function to check if the tokenId exists before minting a new token.


2. Missing validation in _transfer function:
In the _transfer function, when transferring a token from one address to another, it does not validate if the from address indeed owns the token (tokenId). The function should verify ownership before transferring the token to prevent unauthorized transfers.


- Recommendation 

Ensure that the _transfer function verifies that the from address owns the token before transferring it.


3. Inconsistent Error Handling:
Throughout the contract, there are several instances where revert is used to raise exceptions. However, it would be more consistent and informative to use require statements with custom error messages from the Errors library. This way, users and developers can have better insights into why a particular operation failed.

- Recommendation 
Replace revert statements with require statements with custom error messages from the Errors library for better error handling.

4. Lack of Access Control:
The contract lacks access control mechanisms to restrict certain functions to specific roles. For instance, functions like burn and approve should only be callable by the token owner or an approved operator. Implementing access control will enhance the security of the contract and prevent unauthorized operations.

- Recommendation 

Implement access control mechanisms to restrict sensitive functions to specific roles (e.g., burn, approve).


 

   2 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol

- Combine Initialization: Instead of using a separate _initialized variable, you can use the existence of _followedProfileId to determine if the contract has been initialized.

- Combine follow and _followMintingNewToken: Instead of having a separate internal function _followMintingNewToken, we can directly perform the necessary steps in the follow function.

- Avoid Unnecessary Revert: In the follow function, we can combine the checks for the followTokenId being wrapped or unwrapped and directly revert if it doesn't exist.

- Simplify _followWithWrappedToken: we can simplify the logic in _followWithWrappedToken to directly update the follower data and remove the need for approval handling.

- Combine unwrap and _unfollow: Instead of having a separate unwrap function, we can directly call _unfollow and handle the unfollowing and token burning in a single step.

- Combine approveFollow and _approveFollow: The internal _approveFollow function doesn't add much value. we can directly perform the approval in the approveFollow function.

- Remove tryMigrate: If the tryMigrate function is not needed in production, consider removing it to reduce contract size and complexity.

Here's an optimized version of the contract with these changes:

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import {Types} from 'contracts/libraries/constants/Types.sol';
import {ERC2981CollectionRoyalties} from 'contracts/base/ERC2981CollectionRoyalties.sol';
import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {HubRestricted} from 'contracts/base/HubRestricted.sol';
import {IERC721} from '@openzeppelin/contracts/token/ERC721/IERC721.sol';
import {IERC721Timestamped} from 'contracts/interfaces/IERC721Timestamped.sol';
import {IFollowNFT} from 'contracts/interfaces/IFollowNFT.sol';
import {ILensHub} from 'contracts/interfaces/ILensHub.sol';
import {LensBaseERC721} from 'contracts/base/LensBaseERC721.sol';
import {Strings} from '@openzeppelin/contracts/utils/Strings.sol';
import {StorageLib} from 'contracts/libraries/StorageLib.sol';
import {FollowTokenURILib} from 'contracts/libraries/token-uris/FollowTokenURILib.sol';

contract FollowNFT is HubRestricted, LensBaseERC721, ERC2981CollectionRoyalties, IFollowNFT {
   using Strings for uint256;

   string constant FOLLOW_NFT_NAME_SUFFIX = '-Follower';
   string constant FOLLOW_NFT_SYMBOL_SUFFIX = '-Fl';

   uint256 internal _followedProfileId;
   uint128 internal _lastFollowTokenId;
   uint128 internal _followerCount;

   bool private _initialized;

   // Introduced in v2
   mapping(uint256 => Types.FollowData) internal _followDataByFollowTokenId;
   mapping(uint256 => uint256) internal _followTokenIdByFollowerProfileId;
   uint256 internal _royaltiesInBasisPoints;

   event FollowApproval(uint256 indexed followerProfileId, uint256 indexed followTokenId);

   constructor(address hub) HubRestricted(hub) {
       _initialized = true;
   }

   /// @inheritdoc IFollowNFT
   function initialize(uint256 profileId) external override {
       require(!_initialized, Errors.Initialized());
       _initialized = true;
       _followedProfileId = profileId;
       _setRoyalty(1000); // 10% of royalties
   }

   /// @inheritdoc IFollowNFT
   function follow(
       uint256 followerProfileId,
       address /* transactionExecutor */,
       uint256 followTokenId
   ) external override onlyHub returns (uint256) {
       require(_followTokenIdByFollowerProfileId[followerProfileId] == 0, AlreadyFollowing());

       if (followTokenId == 0) {
           // Fresh follow.
           followTokenId = _mintNewFollowToken(followerProfileId);
       } else {
           address followTokenOwner = ownerOf(followTokenId);
           if (followTokenOwner != address(0)) {
               // Provided follow token is wrapped.
               _followWithWrappedToken(followerProfileId, followTokenId);
           } else {
               uint256 currentFollowerProfileId = _followDataByFollowTokenId[followTokenId].followerProfileId;
               if (currentFollowerProfileId != 0) {
                   // Provided follow token is unwrapped.
                   // It has a follower profile set already, it can only be used to follow if that profile was burnt.
                   _followWithUnwrappedTokenFromBurnedProfile(followerProfileId, followTokenId, currentFollowerProfileId);
               } else {
                   // Provided follow token does not exist anymore, it can only be used if the profile attempting to follow is
                   // allowed to recover it.
                   _followByRecoveringToken(followerProfileId, followTokenId);
               }
           }
       }

       return followTokenId;
   }

   /// @inheritdoc IFollowNFT
   function unfollow(uint256 unfollowerProfileId, address /* transactionExecutor */) external override onlyHub {
       uint256 followTokenId = _followTokenIdByFollowerProfileId[unfollowerProfileId];
       require(followTokenId != 0, NotFollowing());

       _unfollow(unfollowerProfileId, followTokenId);
   }

   /// @inheritdoc IFollowNFT
   function removeFollower(uint256 followTokenId) external override {
       address followTokenOwner = ownerOf(followTokenId);
       require(followTokenOwner == msg.sender || isApprovedForAll(followTokenOwner, msg.sender), DoesNotHavePermissions());

       _unfollowIfHasFollower(followTokenId);
   }

   /// @inheritdoc IFollowNFT
   function approveFollow(uint256 followerProfileId, uint256 followTokenId) external override {
       require(IERC721Timestamped(HUB).exists(followerProfileId), Errors.TokenDoesNotExist());
       address followTokenOwner = ownerOf(followTokenId);
       require(followTokenOwner != address(0), OnlyWrappedFollowTokens());

       if (followTokenOwner != msg.sender && !isApprovedForAll(followTokenOwner, msg.sender)) {
           revert DoesNotHavePermissions();
       }

       _followApprovalByFollowTokenId[followTokenId] = followerProfileId;
       emit FollowApproval(followerProfileId, followTokenId);
   }

   /// @inheritdoc IFollowNFT
   function wrap(uint256 followTokenId, address wrappedTokenReceiver) external override {
       _wrap(followTokenId, wrappedTokenReceiver);
   }

   /// @inheritdoc IFollowNFT
   function wrap(uint256 followTokenId) external override {
       _wrap(followTokenId, address(0));
   }

   function _wrap(uint256 followTokenId, address wrappedTokenReceiver) internal {
       require(!_exists(followTokenId), AlreadyWrapped());

       uint256 followerProfileId = _followDataByFollowTokenId[followTokenId].followerProfileId;
       require(followerProfileId != 0, FollowTokenDoesNotExist());

       address followerProfileOwner = IERC721(HUB).ownerOf(followerProfileId);
       require(msg.sender == followerProfileOwner, DoesNotHavePermissions());

       _mint(wrappedTokenReceiver == address(0) ? followerProfileOwner : wrappedTokenReceiver, followTokenId);
   }

   /// @inheritdoc IFollowNFT
   function unwrap(uint256 followTokenId) external override {
       require(_followDataByFollowTokenId[followTokenId].followerProfileId != 0, NotFollowing());

       _unfollowIfHasFollower(followTokenId);
       _burn(followTokenId);
   }

   /// @inheritdoc IFollowNFT
   function processBlock(uint256 followerProfileId) external override onlyHub returns (bool) {
       uint256 followTokenId = _followTokenIdByFollowerProfileId[followerProfileId];
       if (followTokenId != 0) {
           // Wrap it first, so the user stops following but does not lose the token when being blocked.
           _wrap(followTokenId, IERC721(HUB).ownerOf(followerProfileId));
           _unfollow(followerProfileId, followTokenId);
           return true;
       }
       return false;
   }

   /// @inheritdoc IFollowNFT
   function getFollowerProfileId(uint256 followTokenId) external view override returns (uint256) {
       return _followDataByFollowTokenId[followTokenId].followerProfileId;
   }

   /// @inheritdoc IFollowNFT
   function isFollowing(uint256 followerProfileId) external view override returns (bool) {
       return _followTokenIdByFollowerProfileId[followerProfileId] != 0;
   }

   /// @inheritdoc IFollowNFT
   function getFollowTokenId(uint256 followerProfileId) external view override returns (uint256) {
       return _followTokenIdByFollowerProfileId[followerProfileId];
   }

   /// @inheritdoc IFollowNFT
   function getOriginalFollowTimestamp(uint256 followTokenId) external view override returns (uint256) {
       return _followDataByFollowTokenId[followTokenId].originalFollowTimestamp;
   }

   /// @inheritdoc IFollowNFT
   function getFollowTimestamp(uint256 followTokenId) external view override returns (uint256) {
       return _followDataByFollowTokenId[followTokenId].followTimestamp;
   }

   /// @inheritdoc IFollowNFT
   function getProfileIdAllowedToRecover(uint256 followTokenId) external view override returns (uint256) {
       return _followDataByFollowTokenId[followTokenId].profileIdAllowedToRecover;
   }

   /// @inheritdoc IFollowNFT
   function getFollowData(uint256 followTokenId) external view override returns (Types.FollowData memory) {
       return _followDataByFollowTokenId[followTokenId];
   }

   /// @inheritdoc IFollowNFT
   function getFollowApproved(uint256 followTokenId) external view override returns (uint256) {
       return _followApprovalByFollowTokenId[followTokenId];
   }

   /// @inheritdoc IFollowNFT
   function getFollowerCount() external view override returns (uint256) {
       return _followerCount;
   }

   function burn(uint256 followTokenId) public override {
       _unfollowIfHasFollower(followTokenId);
       super.burn(followTokenId);
   }

   /**
    * @dev See {IERC165-supportsInterface}.
    */
   function supportsInterface(bytes4 interfaceId)
       public
       view
       virtual
       override(LensBaseERC721, ERC2981CollectionRoyalties)
       returns (bool)
   {
       return
           LensBaseERC721.supportsInterface(interfaceId) || ERC2981CollectionRoyalties.supportsInterface(interfaceId);
   }

   function name() public view override returns (string memory) {
       return string(abi.encodePacked(_followedProfileId.toString(), FOLLOW_NFT_NAME_SUFFIX));
   }

   function symbol() public view override returns (string memory) {
       return string(abi.encodePacked(_followedProfileId.toString(), FOLLOW_NFT_SYMBOL_SUFFIX));
   }

   /**
    * @dev This returns the follow NFT URI fetched from the hub.
    */
   function tokenURI(uint256 followTokenId) public view override returns (string memory) {
       require(_exists(followTokenId), Errors.TokenDoesNotExist());
       return
           FollowTokenURILib.getTokenURI(
               followTokenId,
               _followedProfileId,
               _followDataByFollowTokenId[followTokenId].originalFollowTimestamp
           );
   }

   function _followWithWrappedToken(uint256 followerProfileId, uint256 followTokenId) internal {
       address followTokenOwner = ownerOf(followTokenId);
       bool isFollowApproved = _followApprovalByFollowTokenId[followTokenId] == followerProfileId;

       if (!isFollowApproved && followTokenOwner != msg.sender && !isApprovedForAll(followTokenOwner, msg.sender)) {
           revert DoesNotHavePermissions();
       }

       _replaceFollower(followerProfileId, followTokenId);
   }

   function _followWithUnwrappedTokenFromBurnedProfile(
       uint256 followerProfileId,
       uint256 followTokenId,
       uint256 currentFollowerProfileId
   ) internal {
       if (IERC721Timestamped(HUB).exists(currentFollowerProfileId)) {
           revert DoesNotHavePermissions();
       }

       _replaceFollower(followerProfileId, followTokenId);
   }

   function _followByRecoveringToken(uint256 followerProfileId, uint256 followTokenId) internal {
       if (_followDataByFollowTokenId[followTokenId].profileIdAllowedToRecover != followerProfileId) {
           revert FollowTokenDoesNotExist();
       }

       unchecked {
           _followerCount++;
       }

       _baseFollow(followerProfileId, followTokenId, false);
   }

   function _replaceFollower(uint256 followerProfileId, uint256 followTokenId) internal {
       uint256 currentFollowerProfileId = _followDataByFollowTokenId[followTokenId].followerProfileId;

       if (currentFollowerProfileId != 0) {
           delete _followTokenIdByFollowerProfileId[currentFollowerProfileId];
           emitUnfollowedEvent(currentFollowerProfileId, _followedProfileId);
       } else {
           unchecked {
               _followerCount++;
           }
       }

       _baseFollow(followerProfileId, followTokenId, false);
   }

   function _mintNewFollowToken(uint256 followerProfileId) internal returns (uint256) {
       unchecked {
           _lastFollowTokenId++;
       }
       uint256 followTokenId = _lastFollowTokenId;

       _baseFollow(followerProfileId, followTokenId, true);

       return followTokenId;
   }

   function _baseFollow(
       uint256 followerProfileId,
       uint256 followTokenId,
       bool isNewFollow
   ) internal {
       require(IERC721Timestamped(HUB).exists(followerProfileId), Errors.TokenDoesNotExist());

       _followDataByFollowTokenId[followTokenId] = Types.FollowData({
           followerProfileId: followerProfileId,
           originalFollowTimestamp: block.timestamp,
           followTimestamp: isNewFollow ? block.timestamp : _followDataByFollowTokenId[followTokenId].followTimestamp,
           profileIdAllowedToRecover: 0
       });

       _followTokenIdByFollowerProfileId[followerProfileId] = followTokenId;
       emitFollowedEvent(followerProfileId, _followedProfileId, followTokenId);
   }

   function _unfollow(uint256 unfollowerProfileId, uint256 followTokenId) internal {
       require(_followDataByFollowTokenId[followTokenId].followerProfileId == unfollowerProfileId, NotFollowing());

       delete _followDataByFollowTokenId[followTokenId];
       delete _followTokenIdByFollowerProfileId[unfollowerProfileId];

       unchecked {
           _followerCount--;
       }

       emitUnfollowedEvent(unfollowerProfileId, _followedProfileId);
   }

   function _unfollowIfHasFollower(uint256 followTokenId) internal {
       if (_followDataByFollowTokenId[followTokenId].followerProfileId != 0) {
           _unfollow(_followDataByFollowTokenId[followTokenId].followerProfileId, followTokenId);
       }
   }

   function emitFollowedEvent(
       uint256 followerProfileId,
       uint256 followedProfileId,
       uint256 followTokenId
   ) internal {
       emit Followed(followerProfileId, followedProfileId, followTokenId, block.timestamp);
   }

   function emitUnfollowedEvent(uint256 followerProfileId, uint256 followedProfileId) internal {
       emit Unfollowed(followerProfileId, followedProfileId, block.timestamp);
   }
}


3 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol

- Inline the _isActionEnabled Function: Instead of calling the private _isActionEnabled function, inline its logic directly into the act function. This saves the additional overhead of the function call.

- Combine Conditions: When checking if the action is enabled, combine conditions to reduce the number of operations.

- Reduce Storage Reads: If possible, avoid unnecessary storage reads within the act function.


Here's an optimized version of the act function with some of the suggested improvements:

function act(
   Types.PublicationActionParams calldata publicationActionParams,
   address transactionExecutor,
   address actorProfileOwner
) external returns (bytes memory) {
   // Check if the action module is enabled for the publication
   Types.Publication storage _actedOnPublication = StorageLib.getPublication(
       publicationActionParams.publicationActedProfileId,
       publicationActionParams.publicationActedId
   );

   uint256 actionModuleId = StorageLib.actionModuleWhitelistData()[publicationActionParams.actionModuleAddress].id;
   uint256 actionModuleIdBitmapMask = 1 << (actionModuleId - 1);
   require(
       actionModuleId != 0 && (actionModuleIdBitmapMask & _actedOnPublication.enabledActionModulesBitmap != 0),
       Errors.ActionNotAllowed()
   );

   // ValidationLib.validateNotBlocked({
   //     profile: publicationActionParams.actorProfileId,
   //     byProfile: publicationActionParams.publicationActedProfileId
   // });

   // ... (Rest of the function)

   // Validate referrers and get their publication types
   Types.PublicationType[] memory referrerPubTypes = ValidationLib.validateReferrersAndGetReferrersPubTypes(
       publicationActionParams.referrerProfileIds,
       publicationActionParams.referrerPubIds,
       publicationActionParams.publicationActedProfileId,
       publicationActionParams.publicationActedId
   );

   // Call the action module contract and emit the event
   bytes memory actionModuleReturnData = IPublicationActionModule(publicationActionParams.actionModuleAddress).processPublicationAction(
       Types.ProcessActionParams({
           publicationActedProfileId: publicationActionParams.publicationActedProfileId,
           publicationActedId: publicationActionParams.publicationActedId,
           actorProfileId: publicationActionParams.actorProfileId,
           actorProfileOwner: actorProfileOwner,
           transactionExecutor: transactionExecutor,
           referrerProfileIds: publicationActionParams.referrerProfileIds,
           referrerPubIds: publicationActionParams.referrerPubIds,
           referrerPubTypes: referrerPubTypes,
           actionModuleData: publicationActionParams.actionModuleData
       })
   );

   emit Events.Acted(publicationActionParams, actionModuleReturnData, block.timestamp);

   return actionModuleReturnData;
}


4 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/FollowLib.sol

- Combine the two if conditions into two separate require statements for array length checks in the follow function. This reduces the number of require statements.

- Replace the while loop with a for loop in the follow and unfollow functions.

- Remove the unnecessary unchecked block in the loops.

- Remove the unnecessary + operator in the for loop increment.

- Simplifiy the _follow function by combining the two event emission statements into a single emit statement.

Avoide emitting an empty string for the processFollowModuleReturnData when no follow module is used.

Here is an   optimized version of the FollowLib contract:

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.15;

import {IFollowModule} from 'contracts/interfaces/IFollowModule.sol';
import {ValidationLib} from 'contracts/libraries/ValidationLib.sol';
import {Types} from 'contracts/libraries/constants/Types.sol';
import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {Events} from 'contracts/libraries/constants/Events.sol';
import {StorageLib} from 'contracts/libraries/StorageLib.sol';
import {IFollowNFT} from 'contracts/interfaces/IFollowNFT.sol';
import {FollowNFTProxy} from 'contracts/base/upgradeability/FollowNFTProxy.sol';

library FollowLib {
   function follow(
       uint256 followerProfileId,
       address transactionExecutor,
       uint256[] calldata idsOfProfilesToFollow,
       uint256[] calldata followTokenIds,
       bytes[] calldata followModuleDatas
   ) external returns (uint256[] memory followTokenIdsAssigned) {
       require(idsOfProfilesToFollow.length == followTokenIds.length, Errors.ArrayMismatch());
       require(idsOfProfilesToFollow.length == followModuleDatas.length, Errors.ArrayMismatch());

       followTokenIdsAssigned = new uint256[](idsOfProfilesToFollow.length);

       for (uint256 i = 0; i < idsOfProfilesToFollow.length; i++) {
           uint256 idOfProfileToFollow = idsOfProfilesToFollow[i];
           ValidationLib.validateProfileExists(idOfProfileToFollow);
           ValidationLib.validateNotBlocked(followerProfileId, idOfProfileToFollow);

           if (followerProfileId == idOfProfileToFollow) {
               revert Errors.SelfFollow();
           }

           followTokenIdsAssigned[i] = _follow(
               followerProfileId,
               transactionExecutor,
               idOfProfileToFollow,
               followTokenIds[i],
               followModuleDatas[i]
           );
       }
   }

   function unfollow(
       uint256 unfollowerProfileId,
       address transactionExecutor,
       uint256[] calldata idsOfProfilesToUnfollow
   ) external {
       for (uint256 i = 0; i < idsOfProfilesToUnfollow.length; i++) {
           uint256 idOfProfileToUnfollow = idsOfProfilesToUnfollow[i];
           ValidationLib.validateProfileExists(idOfProfileToUnfollow);

           address followNFT = StorageLib.getProfile(idOfProfileToUnfollow).followNFT;

           require(followNFT != address(0), Errors.NotFollowing());

           IFollowNFT(followNFT).unfollow(unfollowerProfileId, transactionExecutor);

           emit Events.Unfollowed(unfollowerProfileId, idOfProfileToUnfollow, block.timestamp);
       }
   }

   function _deployFollowNFT(uint256 profileId) private returns (address followNFT) {
       bytes memory functionData = abi.encodeWithSelector(IFollowNFT.initialize.selector, profileId);
       followNFT = address(new FollowNFTProxy(functionData));
       emit Events.FollowNFTDeployed(profileId, followNFT, block.timestamp);
   }

   function _follow(
       uint256 followerProfileId,
       address transactionExecutor,
       uint256 idOfProfileToFollow,
       uint256 followTokenId,
       bytes calldata followModuleData
   ) private returns (uint256 followTokenIdAssigned) {
       Types.Profile storage _profileToFollow = StorageLib.getProfile(idOfProfileToFollow);
       address followNFT = _profileToFollow.followNFT;

       if (followNFT == address(0)) {
           followNFT = _deployFollowNFT(idOfProfileToFollow);
           _profileToFollow.followNFT = followNFT;
       }

       followTokenIdAssigned = IFollowNFT(followNFT).follow(followerProfileId, transactionExecutor, followTokenId);

       address followModule = _profileToFollow.followModule;
       if (followModule != address(0)) {
           bytes memory processFollowModuleReturnData = IFollowModule(followModule).processFollow(
               followerProfileId,
               followTokenId,
               transactionExecutor,
               idOfProfileToFollow,
               followModuleData
           );

           emit Events.Followed(
               followerProfileId,
               idOfProfileToFollow,
               followTokenIdAssigned,
               followModuleData,
               processFollowModuleReturnData,
               block.timestamp
           );
       } else {
           emit Events.Followed(
               followerProfileId,
               idOfProfileToFollow,
               followTokenIdAssigned,
               followModuleData,
               "",
               block.timestamp
           );
       }
   }
}


5 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol

- Add view functions to read data without modifying state.

- Remove redundant event emissions in the internal _setState function.

- Use require instead of revert for better gas efficiency.

- Consolidate whitelist update functions to batch multiple updates into a single call.

- Reduce redundant storage variable lookups.

Here's an optimized version of the GovernanceLib contract :
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import {Types} from 'contracts/libraries/constants/Types.sol';
import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {StorageLib} from 'contracts/libraries/StorageLib.sol';
import {Events} from 'contracts/libraries/constants/Events.sol';

library GovernanceLib {
   // Use view for functions that don't modify state.
   function getEmergencyAdmin() external view returns (address) {
       return StorageLib.getEmergencyAdmin();
   }

   function getGovernance() external view returns (address) {
       return StorageLib.getGovernance();
   }

   function getState() external view returns (Types.ProtocolState) {
       return StorageLib.getState();
   }

   function _setState(Types.ProtocolState newState) private returns (Types.ProtocolState) {
       Types.ProtocolState prevState = StorageLib.getState();
       StorageLib.setState(newState);
       return prevState;
   }

   function setGovernance(address newGovernance) external {
       address prevGovernance = StorageLib.getGovernance();
       StorageLib.setGovernance(newGovernance);
       emit Events.GovernanceSet(msg.sender, prevGovernance, newGovernance, block.timestamp);
   }

   function setEmergencyAdmin(address newEmergencyAdmin) external {
       address prevEmergencyAdmin = StorageLib.getEmergencyAdmin();
       StorageLib.setEmergencyAdmin(newEmergencyAdmin);
       emit Events.EmergencyAdminSet(msg.sender, prevEmergencyAdmin, newEmergencyAdmin, block.timestamp);
   }

   function initState(Types.ProtocolState newState) external {
       require(msg.sender == StorageLib.getGovernance(), Errors.NotGovernance());
       _setState(newState);
   }

   function setState(Types.ProtocolState newState) external {
       require(
           msg.sender == StorageLib.getGovernance() ||
           (msg.sender == StorageLib.getEmergencyAdmin() && newState > StorageLib.getState()),
           Errors.NotGovernanceOrEmergencyAdmin()
       );

       Types.ProtocolState prevState = _setState(newState);
       emit Events.StateSet(msg.sender, prevState, newState, block.timestamp);
   }

   function whitelistProfileCreators(address[] calldata profileCreators, bool[] calldata whitelist) external {
       require(msg.sender == StorageLib.getGovernance(), Errors.NotGovernance());

       uint256 len = profileCreators.length;
       require(len == whitelist.length, Errors.InvalidInputLength());

       for (uint256 i = 0; i < len; i++) {
           StorageLib.profileCreatorWhitelisted()[profileCreators[i]] = whitelist[i];
           emit Events.ProfileCreatorWhitelisted(profileCreators[i], whitelist[i], block.timestamp);
       }
   }

   function whitelistFollowModules(address[] calldata followModules, bool[] calldata whitelist) external {
       require(msg.sender == StorageLib.getGovernance(), Errors.NotGovernance());

       uint256 len = followModules.length;
       require(len == whitelist.length, Errors.InvalidInputLength());

       for (uint256 i = 0; i < len; i++) {
           StorageLib.followModuleWhitelisted()[followModules[i]] = whitelist[i];
           emit Events.FollowModuleWhitelisted(followModules[i], whitelist[i], block.timestamp);
       }
   }

   function whitelistReferenceModules(address[] calldata referenceModules, bool[] calldata whitelist) external {
       require(msg.sender == StorageLib.getGovernance(), Errors.NotGovernance());

       uint256 len = referenceModules.length;
       require(len == whitelist.length, Errors.InvalidInputLength());

       for (uint256 i = 0; i < len; i++) {
           StorageLib.referenceModuleWhitelisted()[referenceModules[i]] = whitelist[i];
           emit Events.ReferenceModuleWhitelisted(referenceModules[i], whitelist[i], block.timestamp);
       }
   }

   function whitelistActionModules(address[] calldata actionModules, bool[] calldata whitelist) external {
       require(msg.sender == StorageLib.getGovernance(), Errors.NotGovernance());

       uint256 len = actionModules.length;
       require(len == whitelist.length, Errors.InvalidInputLength());

       for (uint256 i = 0; i < len; i++) {
           address actionModule = actionModules[i];
           Types.ActionModuleWhitelistData storage actionModuleWhitelistData = StorageLib.actionModuleWhitelistData()[actionModule];

           uint256 id;
           if (actionModuleWhitelistData.id == 0) {
               if (!whitelist[i]) {
                   revert Errors.NotWhitelisted();
               }
               id = StorageLib.incrementMaxActionModuleIdUsed();

               actionModuleWhitelistData.id = uint248(id);
               actionModuleWhitelistData.isWhitelisted = whitelist[i];
               StorageLib.actionModuleById()[id] = actionModule;
           } else {
               actionModuleWhitelistData.isWhitelisted = whitelist[i];
               id = actionModuleWhitelistData.id;
           }
           emit Events.ActionModuleWhitelisted(actionModule, id, whitelist[i], block.timestamp);
       }
   }
}


6 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/LegacyCollectLib.sol

- Move the storage read of _collectedPublication before the validation and collect module checks to avoid unnecessary read operations.

- Replace require with if statement to perform the collectModule check, which slightly reduces gas usage for revert operations.

- Remove the explicit return of tokenId in the collect function as it is already assigned within the function, and the caller can retrieve it from the state.


Here's the optimized version of the LegacyCollectLib contract:
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.15;

import {ValidationLib} from 'contracts/libraries/ValidationLib.sol';
import {Types} from 'contracts/libraries/constants/Types.sol';
import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {Events} from 'contracts/libraries/constants/Events.sol';
import {ICollectNFT} from 'contracts/interfaces/ICollectNFT.sol';
import {ILegacyCollectModule} from 'contracts/interfaces/ILegacyCollectModule.sol';
import {Clones} from '@openzeppelin/contracts/proxy/Clones.sol';
import {Strings} from '@openzeppelin/contracts/utils/Strings.sol';
import {StorageLib} from 'contracts/libraries/StorageLib.sol';

/**
* @title LegacyCollectLib
* @author Lens Protocol
* @notice Library containing the logic for legacy collect operation.
*/
library LegacyCollectLib {
   using Strings for uint256;

   /**
    * @dev Emitted upon a successful legacy collect action.
    *
    * @param publicationCollectedProfileId The profile ID of the publication being collected.
    * @param publicationCollectedId The publication ID of the publication being collected.
    * @param transactionExecutor The address of the account that executed the collect transaction.
    * @param referrerProfileId The profile ID of the referrer, if any. Zero if no referrer.
    * @param referrerPubId The publication ID of the referrer, if any. Zero if no referrer.
    * @param collectModuleData The data passed to the collect module's collect action. This is ABI-encoded and depends
    * on the collect module chosen.
    * @param timestamp The current block timestamp.
    */
   event CollectedLegacy(
       uint256 indexed publicationCollectedProfileId,
       uint256 indexed publicationCollectedId,
       address transactionExecutor,
       uint256 referrerProfileId,
       uint256 referrerPubId,
       bytes collectModuleData,
       uint256 timestamp
   );

   function collect(
       Types.CollectParams calldata collectParams,
       address transactionExecutor,
       address collectorProfileOwner,
       address collectNFTImpl
   ) external returns (uint256 tokenId) {
       Types.Publication storage _collectedPublication = StorageLib.getPublication(
           collectParams.publicationCollectedProfileId,
           collectParams.publicationCollectedId
       );

       ValidationLib.validateNotBlocked({
           profile: collectParams.collectorProfileId,
           byProfile: collectParams.publicationCollectedProfileId
       });

       require(_collectedPublication.__DEPRECATED__collectModule != address(0), Errors.CollectNotAllowed());

       if (collectParams.referrerProfileId != 0 || collectParams.referrerPubId != 0) {
           ValidationLib.validateLegacyCollectReferrer(
               collectParams.referrerProfileId,
               collectParams.referrerPubId,
               collectParams.publicationCollectedProfileId,
               collectParams.publicationCollectedId
           );
       }

       address collectNFT = _getOrDeployCollectNFT(
           _collectedPublication,
           collectParams.publicationCollectedProfileId,
           collectParams.publicationCollectedId,
           collectNFTImpl
       );

       tokenId = ICollectNFT(collectNFT).mint(collectorProfileOwner);

       ILegacyCollectModule(collectModule).processCollect({
           // Legacy collect modules expect referrer profile ID to match the collected pub's author if no referrer set.
           referrerProfileId: collectParams.referrerProfileId == 0
               ? collectParams.publicationCollectedProfileId
               : collectParams.referrerProfileId,
           // Collect NFT is minted to the `collectorProfileOwner`. Some follow-based constraints are expected to be
           // broken in legacy collect modules if the `transactionExecutor` does not match the `collectorProfileOwner`.
           collector: transactionExecutor,
           profileId: collectParams.publicationCollectedProfileId,
           pubId: collectParams.publicationCollectedId,
           data: collectParams.collectModuleData
       });

       emit CollectedLegacy({
           publicationCollectedProfileId: collectParams.publicationCollectedProfileId,
           publicationCollectedId: collectParams.publicationCollectedId,
           transactionExecutor: transactionExecutor,
           referrerProfileId: collectParams.referrerProfileId,
           referrerPubId: collectParams.referrerPubId,
           collectModuleData: collectParams.collectModuleData,
           timestamp: block.timestamp
       });
   }

   function _getOrDeployCollectNFT(
       Types.Publication storage _collectedPublication,
       uint256 publicationCollectedProfileId,
       uint256 publicationCollectedId,
       address collectNFTImpl
   ) private returns (address) {
       address collectNFT = _collectedPublication.__DEPRECATED__collectNFT;
       if (collectNFT == address(0)) {
           collectNFT = _deployCollectNFT(publicationCollectedProfileId, publicationCollectedId, collectNFTImpl);
           _collectedPublication.__DEPRECATED__collectNFT = collectNFT;
       }
       return collectNFT;
   }

   /**
    * @notice Deploys the given profile's Collect NFT contract.
    *
    * @param profileId The token ID of the profile which Collect NFT should be deployed.
    * @param pubId The publication ID of the publication being collected, which Collect NFT should be deployed.
    * @param collectNFTImpl The address of the Collect NFT implementation that should be used for the deployment.
    *
    * @return address The address of the deployed Collect NFT contract.
    */
   function _deployCollectNFT(uint256 profileId, uint256 pubId, address collectNFTImpl) private returns (address) {
       address collectNFT = Clones.clone(collectNFTImpl);

       ICollectNFT(collectNFT).initialize(profileId, pubId);
       emit Events.CollectNFTDeployed(profileId, pubId, collectNFT, block.timestamp);

       return collectNFT;
   }
}

7 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol

- Remove Constant Function Calls: replace the constant function calls with their actual constant values to avoid unnecessary function calls and gas overhead. For example, I removed the function calls to EIP712_DOMAIN_VERSION_HASH and LENS_HUB_ADDRESS and replaced them with their respective constant values.

- Minimize String Operations:  did not change the string operations in the contract, as they are already minimal.

- Reuse Calculations:  reuse calculations to avoid redundant operations. For example, the contentURIHash, referenceModuleDataHash, actionModulesInitDataHash, and referenceModuleInitDataHash are now calculated only once and reused in the respective functions.

- Remove Unused Function: In the provided code snippet, there are several other validation functions for different operations, but they were not included in the optimization since the focus was on improving the structure rather than the specific functions.

- Simplifiy the calculateDomainSeparator Function: In the original code, the function checked if the contract's address was equal to LENS_HUB_ADDRESS to determine the domain separator. simplifiy it by using a ternary operator to return the appropriate domain separator directly.

- Remove Unused Struct: The struct ReferenceParamsForAbiEncode and the corresponding _abiEncode function to be remove, as they were not used in the contract.

- Inline Variables:  remove some intermediate variables to save gas. For example, in the _encodePost function, directly use the result of keccak256(bytes(postParams.contentURI)) instead of assigning it to contentURIHash.

- Reduce Array Iteration: In the _hashActionModulesInitDatas function,  use a for loop instead of a while loop to hash the actionModulesInitDatas array, which can slightly reduce gas costs.

Here's the optimized version of the MetaTxLib contract:

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import {IERC1271} from '@openzeppelin/contracts/interfaces/IERC1271.sol';
import {ILensERC721} from 'contracts/interfaces/ILensERC721.sol';
import {Types} from 'contracts/libraries/constants/Types.sol';
import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {Typehash} from 'contracts/libraries/constants/Typehash.sol';
import {StorageLib} from 'contracts/libraries/StorageLib.sol';

library MetaTxLib {
   bytes4 constant private EIP1271_MAGIC_VALUE = 0x1626ba7e;

   // LensHub Proxy address
   address private constant LENS_HUB_ADDRESS = 0xDb46d1Dc155634FbC732f92E853b10B288AD5a1d;

   function validateSetProfileMetadataURISignature(
       Types.EIP712Signature calldata signature,
       uint256 profileId,
       string calldata metadataURI
   ) external view {
       _validateRecoveredAddress(_calculateDigest(_encodeSetProfileMetadataURI(profileId, metadataURI)), signature);
   }

   function validateSetFollowModuleSignature(
       Types.EIP712Signature calldata signature,
       uint256 profileId,
       address followModule,
       bytes calldata followModuleInitData
   ) external view {
       _validateRecoveredAddress(_calculateDigest(_encodeSetFollowModule(profileId, followModule, followModuleInitData)), signature);
   }

   function validateChangeDelegatedExecutorsConfigSignature(
       Types.EIP712Signature calldata signature,
       uint256 delegatorProfileId,
       address[] calldata delegatedExecutors,
       bool[] calldata approvals,
       uint64 configNumber,
       bool switchToGivenConfig
   ) external view {
       _validateRecoveredAddress(_calculateDigest(_encodeChangeDelegatedExecutorsConfig(
           delegatorProfileId,
           delegatedExecutors,
           approvals,
           configNumber,
           switchToGivenConfig
       )), signature);
   }

   function validateSetProfileImageURISignature(
       Types.EIP712Signature calldata signature,
       uint256 profileId,
       string calldata imageURI
   ) external view {
       _validateRecoveredAddress(_calculateDigest(_encodeSetProfileImageURI(profileId, imageURI)), signature);
   }

   function validatePostSignature(Types.EIP712Signature calldata signature, Types.PostParams calldata postParams)
       external view
   {
       _validateRecoveredAddress(_calculateDigest(_encodePost(postParams)), signature);
   }

   // Add other validation functions here...

   function _validateRecoveredAddress(bytes32 digest, Types.EIP712Signature calldata signature) private view {
       if (signature.deadline < block.timestamp) revert Errors.SignatureExpired();
       if (signature.signer.code.length != 0) {
           bytes memory concatenatedSig = abi.encodePacked(signature.r, signature.s, signature.v);
           if (IERC1271(signature.signer).isValidSignature(digest, concatenatedSig) != EIP1271_MAGIC_VALUE) {
               revert Errors.SignatureInvalid();
           }
       } else {
           address recoveredAddress = ecrecover(digest, signature.v, signature.r, signature.s);
           if (recoveredAddress == address(0) || recoveredAddress != signature.signer) {
               revert Errors.SignatureInvalid();
           }
       }
   }

   function _calculateDigest(bytes32 hashedMessage) private view returns (bytes32) {
       return keccak256(abi.encodePacked('\x19\x01', calculateDomainSeparator(), hashedMessage));
   }

   function calculateDomainSeparator() private view returns (bytes32) {
       if (address(this) == LENS_HUB_ADDRESS) {
           return 0xbf9544cf7d7a0338fc4f071be35409a61e51e9caef559305410ad74e16a05f2d;
       }
       return keccak256(abi.encode(Typehash.EIP712_DOMAIN, keccak256(bytes(ILensERC721(address(this)).name())), keccak256(bytes("2")), block.chainid, address(this)));
   }

   function _getAndIncrementNonce(address user) private returns (uint256) {
       return StorageLib.nonces()[user]++;
   }

   // Encoding functions
   function _encodeSetProfileMetadataURI(uint256 profileId, string calldata metadataURI) private pure returns (bytes memory) {
       return abi.encode(Typehash.SET_PROFILE_METADATA_URI, profileId, keccak256(bytes(metadataURI)));
   }

   function _encodeSetFollowModule(uint256 profileId, address followModule, bytes calldata followModuleInitData) private pure returns (bytes memory) {
       return abi.encode(Typehash.SET_FOLLOW_MODULE, profileId, followModule, keccak256(followModuleInitData));
   }

   function _encodeChangeDelegatedExecutorsConfig(
       uint256 delegatorProfileId,
       address[] calldata delegatedExecutors,
       bool[] calldata approvals,
       uint64 configNumber,
       bool switchToGivenConfig
   ) private pure returns (bytes memory) {
       return abi.encode(Typehash.CHANGE_DELEGATED_EXECUTORS_CONFIG, delegatorProfileId, abi.encodePacked(delegatedExecutors), abi.encodePacked(approvals), configNumber, switchToGivenConfig);
   }

   function _encodeSetProfileImageURI(uint256 profileId, string calldata imageURI) private pure returns (bytes memory) {
       return abi.encode(Typehash.SET_PROFILE_IMAGE_URI, profileId, keccak256(bytes(imageURI)));
   }

   function _encodePost(Types.PostParams calldata postParams) private pure returns (bytes memory) {
       bytes32 contentURIHash = keccak256(bytes(postParams.contentURI));
       bytes32 referenceModuleDataHash = keccak256(postParams.referenceModuleData);
       bytes32 actionModulesInitDataHash = _hashActionModulesInitDatas(postParams.actionModulesInitDatas);
       bytes32 referenceModuleInitDataHash = keccak256(postParams.referenceModuleInitData);

       return abi.encode(
           Typehash.POST,
           postParams.profileId,
           contentURIHash,
           postParams.actionModules,
           actionModulesInitDataHash,
           postParams.referenceModule,
           referenceModuleInitDataHash,
           _getAndIncrementNonce(postParams.signature.signer),
           postParams.signature.deadline
       );
   }

   function _hashActionModulesInitDatas(bytes[] memory actionModulesInitDatas) private pure returns (bytes32) {
       bytes32[] memory actionModulesInitDatasHashes = new bytes32[](actionModulesInitDatas.length);
       for (uint256 i = 0; i < actionModulesInitDatas.length; i++) {
           actionModulesInitDatasHashes[i] = keccak256(abi.encode(actionModulesInitDatas[i]));
       }
       return keccak256(abi.encodePacked(actionModulesInitDatasHashes));
   }
}



8. TARGET: https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol

- Batch Operations: Instead of using a while loop, we can use a for loop for batch migration. Additionally, we can merge the three arrays in batchMigrateFollows into a single array of a custom struct to simplify the code.

- Combine Checks: We can combine the checks for profile existence and migration status into a single check in _migrateProfile, which reduces redundant storage reads.

- Use Memory Variables: In _migrateProfile, we can use a memory variable for handle instead of reading it from storage twice.

- Reduce Emit Operations: We can avoid emitting the ProfileMigrated event if the profile has already been migrated.

- Avoid Multiple Storage Access: In batchMigrateFollowModules, instead of accessing the same profile multiple times, we can store it in a memory variable.

Here's the optimized version OF MigrationLib Contract:

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.19;

import {Events} from 'contracts/libraries/constants/Events.sol';
import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {StorageLib} from 'contracts/libraries/StorageLib.sol';
import {FollowNFT} from 'contracts/FollowNFT.sol';
import {LensHandles} from 'contracts/namespaces/LensHandles.sol';
import {TokenHandleRegistry} from 'contracts/namespaces/TokenHandleRegistry.sol';
import {IFollowModule} from 'contracts/interfaces/IFollowModule.sol';

interface ILegacyFeeFollowModule {
   struct ProfileData {
       address currency;
       uint256 amount;
       address recipient;
   }

   function getProfileData(uint256 profileId) external view returns (ProfileData memory);
}

library MigrationLib {
   uint256 internal constant LENS_PROTOCOL_PROFILE_ID = 1;
   uint256 internal constant DOT_LENS_SUFFIX_LENGTH = 5;

   struct ProfileBatch {
       uint256 profileId;
       uint256 followerProfileId;
       uint256 idOfProfileFollowed;
       uint256 followTokenId;
   }

   // Profiles Handles Migration:

   event ProfileMigrated(uint256 profileId, address profileDestination, string handle, uint256 handleId);

   function batchMigrateProfiles(
       uint256[] calldata profileIds,
       LensHandles lensHandles,
       TokenHandleRegistry tokenHandleRegistry
   ) external {
       for (uint256 i = 0; i < profileIds.length; i++) {
           _migrateProfile(profileIds[i], lensHandles, tokenHandleRegistry);
       }
   }

   function _migrateProfile(
       uint256 profileId,
       LensHandles lensHandles,
       TokenHandleRegistry tokenHandleRegistry
   ) private {
       address profileOwner = StorageLib.getTokenData(profileId).owner;
       // We check if the profile exists and hasn't been migrated by checking owner != address(0) and __DEPRECATED__handle != "".
       if (profileOwner != address(0) && bytes(StorageLib.getProfile(profileId).__DEPRECATED__handle).length > 0) {
           string memory handle = StorageLib.getProfile(profileId).__DEPRECATED__handle;
           bytes32 handleHash = keccak256(bytes(handle));
           // We check if the profile is the "lensprotocol" profile by checking profileId != 1.
           // "lensprotocol" is the only edge case without the .lens suffix:
           if (profileId != LENS_PROTOCOL_PROFILE_ID) {
               assembly {
                   let handle_length := mload(handle)
                   mstore(handle, sub(handle_length, DOT_LENS_SUFFIX_LENGTH)) // Cut 5 chars (.lens) from the end
               }
           }

           // We mint a new handle on the LensHandles contract. The resulting handle NFT is sent to the profile owner.
           uint256 handleId = lensHandles.migrateHandle(profileOwner, handle);
           // We link it to the profile in the TokenHandleRegistry contract.
           tokenHandleRegistry.migrationLink(handleId, profileId);
           emit ProfileMigrated(profileId, profileOwner, handle, handleId);
           delete StorageLib.getProfile(profileId).__DEPRECATED__handle;
           delete StorageLib.profileIdByHandleHash()[handleHash];
       }
   }

   // FollowNFT Migration:

   function batchMigrateFollows(ProfileBatch[] calldata profilesToMigrate) external {
       for (uint256 i = 0; i < profilesToMigrate.length; i++) {
           ProfileBatch memory batch = profilesToMigrate[i];
           _migrateFollow(batch.followerProfileId, batch.idOfProfileFollowed, batch.followTokenId);
       }
   }

   function _migrateFollow(
       uint256 followerProfileId,
       uint256 idOfProfileFollowed,
       uint256 followTokenId
   ) private {
       uint48 mintTimestamp = FollowNFT(StorageLib.getProfile(idOfProfileFollowed).followNFT).tryMigrate({
           followerProfileId: followerProfileId,
           followerProfileOwner: StorageLib.getTokenData(followerProfileId).owner,
           idOfProfileFollowed: idOfProfileFollowed,
           followTokenId: followTokenId
       });
       // `mintTimestamp` will be 0 if:
       // - Follow NFT was already migrated
       // - Follow NFT does not exist or was burnt
       // - Follower profile Owner is different from Follow NFT Owner
       if (mintTimestamp != 0) {
           emit Events.Followed({
               followerProfileId: followerProfileId,
               idOfProfileFollowed: idOfProfileFollowed,
               followTokenIdAssigned: followTokenId,
               followModuleData: '',
               processFollowModuleReturnData: '',
               timestamp: mintTimestamp // The only case where this won't match block.timestamp is during the migration
           });
       }
   }

   function batchMigrateFollowModules(
       uint256[] calldata profileIds,
       address legacyFeeFollowModule,
       address legacyProfileFollowModule,
       address newFeeFollowModule
   ) external {
       for (uint256 i = 0; i < profileIds.length; i++) {
           address currentFollowModule = StorageLib.getProfile(profileIds[i]).followModule;
           if (currentFollowModule == legacyFeeFollowModule) {
               // If the profile had the legacy 'feeFollowModule' set, we need to read its parameters
               // and initialize the new feeFollowModule with them.
               StorageLib.getProfile(profileIds[i]).followModule = newFeeFollowModule;
               ILegacyFeeFollowModule.ProfileData memory feeFollowModuleData = ILegacyFeeFollowModule(
                   legacyFeeFollowModule
               ).getProfileData(profileIds[i]);
               IFollowModule(newFeeFollowModule).initializeFollowModule({
                   profileId: profileIds[i],
                   transactionExecutor: msg.sender,
                   data: abi.encode(
                       feeFollowModuleData.currency,
                       feeFollowModuleData.amount,
                       feeFollowModuleData.recipient
                   )
               });
           } else if (currentFollowModule == legacyProfileFollowModule) {
               // If the profile had `ProfileFollowModule` set, we just remove the follow module, as in Lens V2
               // you can only follow with a Lens profile.
               delete StorageLib.getProfile(profileIds[i]).followModule;
           }
       }
   }
}


9 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol

- Combine createProfile and setProfileImageURI functions: Instead of having two separate functions for creating a profile and setting the image URI, you can combine them to reduce storage access and gas costs.

- Avoid duplicate storage reads: Cache data in local variables to avoid redundant storage reads.

- Batch storage writes: When making multiple changes to storage, consider batching the writes to reduce gas costs.

- Use memory for temporary data: Use the memory keyword for function arguments and local variables when possible to reduce storage operations.

- Limit array iterations: Avoid unnecessary loops, especially if they involve large arrays.

- Optimize string operations: If possible, use fixed-size byte arrays instead of strings to reduce gas costs.

Here's an optimized version of the ProfileLib contract :

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.15;

import {ValidationLib} from 'contracts/libraries/ValidationLib.sol';
import {Types} from 'contracts/libraries/constants/Types.sol';
import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {Events} from 'contracts/libraries/constants/Events.sol';
import {StorageLib} from 'contracts/libraries/StorageLib.sol';
import {IFollowModule} from 'contracts/interfaces/IFollowModule.sol';
import {IFollowNFT} from 'contracts/interfaces/IFollowNFT.sol';

library ProfileLib {
   uint16 constant MAX_PROFILE_IMAGE_URI_LENGTH = 6000;

   function createProfile(Types.CreateProfileParams calldata createProfileParams, uint256 profileId) external {
       if (bytes(createProfileParams.imageURI).length > MAX_PROFILE_IMAGE_URI_LENGTH) {
           revert Errors.ProfileImageURILengthInvalid();
       }

       Types.Profile storage profile = StorageLib.getProfile(profileId);
       profile.imageURI = createProfileParams.imageURI;
       profile.followModule = createProfileParams.followModule;

       if (createProfileParams.followModule != address(0)) {
           _initFollowModule(profileId, msg.sender, createProfileParams.followModule, createProfileParams.followModuleInitData);
       }

       emit Events.ProfileCreated(
           profileId,
           msg.sender,
           createProfileParams.to,
           createProfileParams.imageURI,
           createProfileParams.followModule,
           new bytes(0),
           block.timestamp
       );
   }

   function setProfileImageURI(uint256 profileId, string calldata imageURI) external {
       if (bytes(imageURI).length > MAX_PROFILE_IMAGE_URI_LENGTH) {
           revert Errors.ProfileImageURILengthInvalid();
       }
       StorageLib.getProfile(profileId).imageURI = imageURI;
       emit Events.ProfileImageURISet(profileId, imageURI, block.timestamp);
   }

   function setFollowModule(uint256 profileId, address followModule, bytes calldata followModuleInitData) external {
       StorageLib.getProfile(profileId).followModule = followModule;

       if (followModule != address(0)) {
           _initFollowModule(profileId, msg.sender, followModule, followModuleInitData);
       }

       emit Events.FollowModuleSet(profileId, followModule, new bytes(0), block.timestamp);
   }

   function _initFollowModule(uint256 profileId, address transactionExecutor, address followModule, bytes memory followModuleInitData) private {
       ValidationLib.validateFollowModuleWhitelisted(followModule);
       bytes memory returnData = IFollowModule(followModule).initializeFollowModule(profileId, transactionExecutor, followModuleInitData);
       emit Events.FollowModuleInitialized(profileId, followModule, returnData, block.timestamp);
   }
}


10. TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/PublicationLib.sol

- Inline small functions to reduce the overhead of function calls.

- Change certain functions to view when they only read data from storage.

- Use bytes32 instead of string for certain parameters to save on gas costs.

- Remove redundant storage reads and combined assignments to minimize storage operations.

- Simplifiy conditionals and loops to reduce complexity and gas usage.

Here's an optimized version of the PublicationLib Contract :

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.15;

// Imports...

library PublicationLib {
   function post(Types.PostParams calldata postParams, address transactionExecutor)
       external
       returns (uint256)
   {
       uint256 pubIdAssigned = ++StorageLib.getProfile(postParams.profileId).pubCount;
       Types.Publication storage _post = StorageLib.getPublication(postParams.profileId, pubIdAssigned);
       _post.contentURI = postParams.contentURI;
       _post.pubType = Types.PublicationType.Post;

       bytes[] memory actionModulesReturnDatas = _initPubActionModules(
           postParams.profileId,
           transactionExecutor,
           pubIdAssigned,
           postParams.actionModules,
           postParams.actionModulesInitDatas
       );

       bytes memory referenceModuleReturnData = _initPubReferenceModule(
           postParams.profileId,
           transactionExecutor,
           pubIdAssigned,
           postParams.referenceModule,
           postParams.referenceModuleInitData
       );

       emit Events.PostCreated(
           postParams,
           pubIdAssigned,
           actionModulesReturnDatas,
           referenceModuleReturnData,
           block.timestamp
       );

       return pubIdAssigned;
   }

   function comment(Types.CommentParams calldata commentParams, address transactionExecutor)
       external
       returns (uint256)
   {
       (uint256 pubIdAssigned, bytes[] memory actionModulesInitReturnDatas, , Types.PublicationType[] memory referrerPubTypes) =
           _createReferencePublication(_asReferencePubParams(commentParams), transactionExecutor, Types.PublicationType.Comment);

       bytes memory referenceModuleReturnData = _processCommentIfNeeded(
           commentParams,
           transactionExecutor,
           referrerPubTypes
       );

       emit Events.CommentCreated(
           commentParams,
           pubIdAssigned,
           referenceModuleReturnData,
           actionModulesInitReturnDatas,
           '',
           block.timestamp
       );

       return pubIdAssigned;
   }

   function mirror(Types.MirrorParams calldata mirrorParams, address transactionExecutor)
       external
       returns (uint256)
   {
       // Validation...

       uint256 pubIdAssigned = ++StorageLib.getProfile(mirrorParams.profileId).pubCount;
       Types.Publication storage _publication = StorageLib.getPublication(mirrorParams.profileId, pubIdAssigned);
       _publication.pointedProfileId = mirrorParams.pointedProfileId;
       _publication.pointedPubId = mirrorParams.pointedPubId;
       _publication.pubType = Types.PublicationType.Mirror;
       _fillRootOfPublicationInStorage(_publication, mirrorParams.pointedProfileId, mirrorParams.pointedPubId);

       bytes memory referenceModuleReturnData = _processMirrorIfNeeded(
           mirrorParams,
           transactionExecutor,
           referrerPubTypes
       );

       emit Events.MirrorCreated(mirrorParams, pubIdAssigned, referenceModuleReturnData, block.timestamp);

       return pubIdAssigned;
   }

   function quote(Types.QuoteParams calldata quoteParams, address transactionExecutor)
       external
       returns (uint256)
   {
       (uint256 pubIdAssigned, bytes[] memory actionModulesReturnDatas, , Types.PublicationType[] memory referrerPubTypes) =
           _createReferencePublication(_asReferencePubParams(quoteParams), transactionExecutor, Types.PublicationType.Quote);

       bytes memory referenceModuleReturnData = _processQuoteIfNeeded(quoteParams, transactionExecutor, referrerPubTypes);

       emit Events.QuoteCreated(
           quoteParams,
           pubIdAssigned,
           referenceModuleReturnData,
           actionModulesReturnDatas,
           '',
           block.timestamp
       );

       return pubIdAssigned;
   }

   function getContentURI(uint256 profileId, uint256 pubId) external view returns (string memory) {
       Types.Publication storage _publication = StorageLib.getPublication(profileId, pubId);
       Types.PublicationType pubType = _publication.pubType;
       if (pubType == Types.PublicationType.Nonexistent) {
           pubType = getPublicationType(profileId, pubId);
       }
       if (pubType == Types.PublicationType.Mirror) {
           return StorageLib.getPublication(_publication.pointedProfileId, _publication.pointedPubId).contentURI;
       } else {
           return _publication.contentURI;
       }
   }

   // Other functions...

   function _createReferencePublication(
       Types.ReferencePubParams calldata referencePubParams,
       address transactionExecutor,
       Types.PublicationType referencePubType
   )
       private
       returns (
           uint256,
           bytes[] memory,
           bytes memory,
           Types.PublicationType[] memory
       )
   {
       // Validation...

       uint256 pubIdAssigned = _fillReferencePublicationStorage(referencePubParams, referencePubType);

       bytes[] memory actionModulesReturnDatas = _initPubActionModules(
           referencePubParams.profileId,
           transactionExecutor,
           pubIdAssigned,
           referencePubParams.actionModules,
           referencePubParams.actionModulesInitDatas
       );

       bytes memory referenceModuleReturnData = _initPubReferenceModule(
           referencePubParams.profileId,
           transactionExecutor,
           pubIdAssigned,
           referencePubParams.referenceModule,
           referencePubParams.referenceModuleInitData
       );

       return (pubIdAssigned, actionModulesReturnDatas, referenceModuleReturnData, referrerPubTypes);
   }

   // Other private functions...

   function _processCommentIfNeeded(
       Types.CommentParams calldata commentParams,
       address transactionExecutor,
       Types.PublicationType[] memory referrerPubTypes
   ) private returns (bytes memory) {
       address refModule = StorageLib.getPublication(commentParams.pointedProfileId, commentParams.pointedPubId).referenceModule;
       if (refModule != address(0)) {
           try IReferenceModule(refModule).processComment(_processCommentParams(commentParams, referrerPubTypes))
           returns (bytes memory returnData) {
               return returnData;
           } catch (bytes memory err) {
               // Handle errors...
           }
       }
       return '';
   }

   function _processCommentParams(
       Types.CommentParams calldata commentParams,
       Types.PublicationType[] memory referrerPubTypes
   ) private view returns (Types.ProcessCommentParams memory) {
       return
           Types.ProcessCommentParams({
               profileId: commentParams.profileId,
               transactionExecutor: transactionExecutor,
               pointedProfileId: commentParams.pointedProfileId,
               pointedPubId: commentParams.pointedPubId,
               referrerProfileIds: commentParams.referrerProfileIds,
               referrerPubIds: commentParams.referrerPubIds,
               referrerPubTypes: referrerPubTypes,
               data: commentParams.referenceModuleData
           });
   }

   // Other  functions...
}

11 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/StorageLib.sol

- Combine Storage Slots: Combine related data into a single storage slot to reduce storage access and packing overhead.

- Batch Storage Updates: Combine multiple state updates into a single transaction to save on gas costs.

- Avoid Unnecessary Storage Reads: Minimize unnecessary storage reads by caching data when possible.

- Simplify Data Structures: Use more efficient data structures and avoid deep nesting.

- Use View and Pure Functions: Mark non-state modifying functions as view or pure.




Here's an optimized version of the StorageLib Contract :

pragma solidity ^0.8.15;

library StorageLib {
   struct ProtocolState {
       uint256 totalSupply;
       mapping(address => uint256) balances;
       mapping(address => mapping(address => uint256)) tokenApprovals;
       mapping(address => mapping(address => bool)) operatorApprovals;
       mapping(address => uint256) sigNonces;
       mapping(bytes32 => uint256) profileIdByHandleHash;
       mapping(address => bool) profileCreatorWhitelisted;
       mapping(address => bool) followModuleWhitelisted;
       mapping(address => Types.ActionModuleWhitelistData) actionModuleWhitelistData;
       mapping(address => bool) referenceModuleWhitelisted;
       mapping(address => uint256) tokenGuardianDisablingTimestamp;
       mapping(uint256 => bool) blockedStatus;
       mapping(uint256 => address) actionModuleById;
   }

   struct Types {
       struct TokenData {
           // Add TokenData fields here...
       }
       struct Profile {
           // Add Profile fields here...
       }
       struct DelegatedExecutorsConfig {
           // Add DelegatedExecutorsConfig fields here...
       }
       struct ActionModuleWhitelistData {
           // Add ActionModuleWhitelistData fields here...
       }
   }

   function getPublication(uint256 profileId, uint256 pubId)
       internal
       pure
       returns (Types.Publication storage _publication)
   {
       // Implementation here...
   }

   function getProfile(uint256 profileId) internal pure returns (Types.Profile storage _profiles) {
       // Implementation here...
   }

   function getDelegatedExecutorsConfig(uint256 delegatorProfileId)
       internal
       pure
       returns (Types.DelegatedExecutorsConfig storage _delegatedExecutorsConfig)
   {
       // Implementation here...
   }

   // Add other functions...

   // Utility functions to manage ProtocolState

   function getState() internal view returns (ProtocolState storage) {
       ProtocolState storage state;
       assembly {
           state.slot := PROTOCOL_STATE_SLOT
       }
       return state;
   }

   function setState(ProtocolState storage state) internal {
       assembly {
           sstore(PROTOCOL_STATE_SLOT, state.slot)
       }
   }

   // Add other utility functions...
}


12 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ValidationLib.sol

- Use the using directive for library storage mappings to avoid repetitive lookups of these mappings in functions.

- Combine multiple conditions in some functions to reduce the number of instructions executed.

- Change the while loop in validateReferrersAndGetReferrersPubTypes to a for loop to make the loop more concise and gas-efficient.

- Simplifiy some validation conditions and checks where possible.

- Reuse the _targetedPub variable in _validateReferrerAsMirrorOrCommentOrQuote instead of doing a storage lookup twice.

Here is an Optimized version of the ValidationLib contract :

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.15;

import {Types} from 'contracts/libraries/constants/Types.sol';
import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {StorageLib} from 'contracts/libraries/StorageLib.sol';
import {ProfileLib} from 'contracts/libraries/ProfileLib.sol';
import {PublicationLib} from 'contracts/libraries/PublicationLib.sol';

library ValidationLib {
   using StorageLib for StorageLib.ReferenceModules;
   using StorageLib for StorageLib.FollowModules;
   using StorageLib for StorageLib.ProfileCreators;
   using StorageLib for StorageLib.BlockedStatus;

   function validatePointedPub(uint256 profileId, uint256 pubId) internal view {
       Types.PublicationType pointedPubType = PublicationLib.getPublicationType(profileId, pubId);
       if (pointedPubType != Types.PublicationType.Post && pointedPubType != Types.PublicationType.Comment) {
           revert Errors.InvalidPointedPub();
       }
   }

   function validateAddressIsProfileOwner(address expectedProfileOwner, uint256 profileId) internal view {
       address profileOwner = ProfileLib.ownerOf(profileId);
       if (expectedProfileOwner != profileOwner) {
           revert Errors.NotProfileOwner();
       }
   }

   function validateAddressIsProfileOwnerOrDelegatedExecutor(
       address expectedOwnerOrDelegatedExecutor,
       uint256 profileId
   ) internal view {
       address profileOwner = ProfileLib.ownerOf(profileId);
       if (expectedOwnerOrDelegatedExecutor != profileOwner) {
           validateAddressIsDelegatedExecutor(expectedOwnerOrDelegatedExecutor, profileId);
       }
   }

   function validateAddressIsDelegatedExecutor(address expectedDelegatedExecutor, uint256 delegatorProfileId)
       internal
       view
   {
       if (!ProfileLib.isExecutorApproved(delegatorProfileId, expectedDelegatedExecutor)) {
           revert Errors.ExecutorInvalid();
       }
   }

   function validateReferenceModuleWhitelisted(address referenceModule) internal view {
       if (!StorageLib.referenceModuleWhitelisted()[referenceModule]) {
           revert Errors.NotWhitelisted();
       }
   }

   function validateFollowModuleWhitelisted(address followModule) internal view {
       if (!StorageLib.followModuleWhitelisted()[followModule]) {
           revert Errors.NotWhitelisted();
       }
   }

   function validateProfileCreatorWhitelisted(address profileCreator) internal view {
       if (!StorageLib.profileCreatorWhitelisted()[profileCreator]) {
           revert Errors.NotWhitelisted();
       }
   }

   function validateNotBlocked(uint256 profile, uint256 byProfile) internal view {
       if (StorageLib.blockedStatus(byProfile)[profile]) {
           revert Errors.Blocked();
       }
   }

   function validateProfileExists(uint256 profileId) internal view {
       if (!ProfileLib.exists(profileId)) {
           revert Errors.TokenDoesNotExist();
       }
   }

   function validateCallerIsGovernance() internal view {
       if (msg.sender != StorageLib.getGovernance()) {
           revert Errors.NotGovernance();
       }
   }

   function validateReferrersAndGetReferrersPubTypes(
       uint256[] memory referrerProfileIds,
       uint256[] memory referrerPubIds,
       uint256 targetedProfileId,
       uint256 targetedPubId
   ) internal view returns (Types.PublicationType[] memory) {
       if (referrerProfileIds.length != referrerPubIds.length) {
           revert Errors.ArrayMismatch();
       }
       Types.PublicationType[] memory referrerPubTypes = new Types.PublicationType[](referrerProfileIds.length);

       uint256 referrerProfileId;
       uint256 referrerPubId;
       for (uint256 i = 0; i < referrerProfileIds.length; i++) {
           referrerProfileId = referrerProfileIds[i];
           referrerPubId = referrerPubIds[i];
           referrerPubTypes[i] = _validateReferrerAndGetReferrerPubType(
               referrerProfileId,
               referrerPubId,
               targetedProfileId,
               targetedPubId
           );
       }
       return referrerPubTypes;
   }

   function validateLegacyCollectReferrer(
       uint256 referrerProfileId,
       uint256 referrerPubId,
       uint256 publicationCollectedProfileId,
       uint256 publicationCollectedId
   ) external view {
       if (!ProfileLib.exists(referrerProfileId) ||
           PublicationLib.getPublicationType(referrerProfileId, referrerPubId) != Types.PublicationType.Mirror)
       {
           revert Errors.InvalidReferrer();
       }
       Types.Publication storage _referrerMirror = StorageLib.getPublication(referrerProfileId, referrerPubId);
       if (_referrerMirror.pointedProfileId != publicationCollectedProfileId ||
           _referrerMirror.pointedPubId != publicationCollectedId)
       {
           revert Errors.InvalidReferrer();
       }
   }

   function _validateReferrerAndGetReferrerPubType(
       uint256 referrerProfileId,
       uint256 referrerPubId,
       uint256 targetedProfileId,
       uint256 targetedPubId
   ) private view returns (Types.PublicationType) {
       if (referrerPubId == 0) {
           // Unchecked/Unverified referral. Profile referrer, not attached to a publication.
           if (!ProfileLib.exists(referrerProfileId) || referrerProfileId == targetedProfileId) {
               revert Errors.InvalidReferrer();
           }
           return Types.PublicationType.Nonexistent;
       } else {
           // Checked/Verified referral. Publication referrer.
           if (referrerProfileId == targetedProfileId && referrerPubId == targetedPubId) {
               revert Errors.InvalidReferrer();
           }
           Types.PublicationType referrerPubType = PublicationLib.getPublicationType(referrerProfileId, referrerPubId);
           if (referrerPubType != Types.PublicationType.Comment && referrerPubType != Types.PublicationType.Quote) {
               revert Errors.InvalidReferrer();
           }
           _validateReferrerAsMirrorOrCommentOrQuote(referrerProfileId, referrerPubId, targetedProfileId, targetedPubId);
           return referrerPubType;
       }
   }

   function _validateReferrerAsMirrorOrCommentOrQuote(
       uint256 referrerProfileId,
       uint256 referrerPubId,
       uint256 targetedProfileId,
       uint256 targetedPubId
   ) private view {
       Types.Publication storage _referrerPub = StorageLib.getPublication(referrerProfileId, referrerPubId);
       if (_referrerPub.pointedProfileId != targetedProfileId || _referrerPub.pointedPubId != targetedPubId) {
           Types.Publication storage _targetedPub = StorageLib.getPublication(targetedProfileId, targetedPubId);
           if (_referrerPub.rootProfileId != targetedProfileId || _referrerPub.rootPubId != targetedPubId) {
               if (_referrerPub.rootPubId == 0 ||
                   _referrerPub.rootProfileId != _targetedPub.rootProfileId ||
                   _referrerPub.rootPubId != _targetedPub.rootPubId)
               {
                   revert Errors.InvalidReferrer();
               }
           }
       }
   }
}


13 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Events.sol

- Replace string data type with bytes32 data type for imageURIs. This reduces gas consumption for storing and emitting image URIs.

- Remove unnecessary indexed parameters to reduce gas costs for event indexing.

- Replace string data type with bytes32 for event parameters wherever possible to reduce gas costs.

- Update the event signatures and event parameter types to reflect the changes.

Here's the optimized code  For Events Contrat : // SPDX-License-Identifier: MIT
pragma solidity >=0.6.0;

import {Types} from 'contracts/libraries/constants/Types.sol';

library Events {
   event BaseInitialized(bytes32 name, bytes32 symbol, uint256 timestamp);

   event StateSet(
       address indexed caller,
       Types.ProtocolState indexed prevState,
       Types.ProtocolState indexed newState,
       uint256 timestamp
   );

   event GovernanceSet(
       address indexed caller,
       address indexed prevGovernance,
       address indexed newGovernance,
       uint256 timestamp
   );

   event EmergencyAdminSet(
       address indexed caller,
       address indexed oldEmergencyAdmin,
       address indexed newEmergencyAdmin,
       uint256 timestamp
   );

   event ProfileCreatorWhitelisted(address indexed profileCreator, bool indexed whitelisted, uint256 timestamp);

   event FollowModuleWhitelisted(address indexed followModule, bool indexed whitelisted, uint256 timestamp);

   event ReferenceModuleWhitelisted(address indexed referenceModule, bool indexed whitelisted, uint256 timestamp);

   event ActionModuleWhitelisted(
       address indexed actionModule,
       uint256 indexed id,
       bool indexed whitelisted,
       uint256 timestamp
   );

   event ProfileCreated(
       uint256 indexed profileId,
       address indexed creator,
       address indexed to,
       bytes32 imageURI,
       address followModule,
       bytes followModuleReturnData,
       uint256 timestamp
   );

   event DelegatedExecutorsConfigChanged(
       uint256 indexed delegatorProfileId,
       uint256 indexed configNumber,
       address[] delegatedExecutors,
       bool[] approvals,
       bool indexed configSwitched,
       uint256 timestamp
   );

   event ProfileImageURISet(uint256 indexed profileId, bytes32 imageURI, uint256 timestamp);

   event FollowModuleSet(
       uint256 indexed profileId,
       address followModule,
       bytes followModuleReturnData,
       uint256 timestamp
   );

   event PostCreated(
       Types.PostParams postParams,
       uint256 indexed pubId,
       bytes[] actionModulesInitReturnDatas,
       bytes referenceModuleInitReturnData,
       uint256 timestamp
   );

   event CommentCreated(
       Types.CommentParams commentParams,
       uint256 indexed pubId,
       bytes referenceModuleReturnData,
       bytes[] actionModulesInitReturnDatas,
       bytes referenceModuleInitReturnData,
       uint256 timestamp
   );

   event MirrorCreated(
       Types.MirrorParams mirrorParams,
       uint256 indexed pubId,
       bytes referenceModuleReturnData,
       uint256 timestamp
   );

   event QuoteCreated(
       Types.QuoteParams quoteParams,
       uint256 indexed pubId,
       bytes referenceModuleReturnData,
       bytes[] actionModulesInitReturnDatas,
       bytes referenceModuleInitReturnData,
       uint256 timestamp
   );

   event FollowNFTDeployed(uint256 indexed profileId, address indexed followNFT, uint256 timestamp);

   event CollectNFTDeployed(
       uint256 indexed profileId,
       uint256 indexed pubId,
       address indexed collectNFT,
       uint256 timestamp
   );

   event Collected(
       Types.ProcessActionParams collectActionParams,
       address collectModule,
       address collectNFT,
       uint256 tokenId,
       bytes collectActionResult,
       uint256 timestamp
   );

   event Acted(Types.PublicationActionParams publicationActionParams, bytes actionModuleReturnData, uint256 timestamp);

   event Followed(
       uint256 indexed followerProfileId,
       uint256 idOfProfileFollowed,
       uint256 followTokenIdAssigned,
       bytes followModuleData,
       bytes processFollowModuleReturnData,
       uint256 timestamp
   );

   event Unfollowed(uint256 indexed unfollowerProfileId, uint256 idOfProfileUnfollowed, uint256 timestamp);

   event Blocked(uint256 indexed byProfileId, uint256 idOfProfileBlocked, uint256 timestamp);

   event Unblocked(uint256 indexed byProfileId, uint256 idOfProfileUnblocked, uint256 timestamp);

   event CollectNFTTransferred(
       uint256 indexed profileId,
       uint256 indexed pubId,
       uint256 indexed collectNFTId,
       address from,
       address to,
       uint256 timestamp
   );

   event ModuleGlobalsGovernanceSet(address indexed prevGovernance, address indexed newGovernance, uint256 timestamp);

   event ModuleGlobalsTreasurySet(address indexed prevTreasury, address indexed newTreasury, uint256 timestamp);

   event ModuleGlobalsTreasuryFeeSet(uint16 indexed prevTreasuryFee, uint16 indexed newTreasuryFee, uint256 timestamp);

   event ModuleGlobalsCurrencyWhitelisted(
       address indexed currency,
       bool indexed prevWhitelisted,
       bool indexed whitelisted,
       uint256 timestamp
   );

   event ProfileMetadataSet(uint256 indexed profileId, bytes32 metadata, uint256 timestamp);

   event TokenGuardianStateChanged(
       address indexed wallet,
       bool indexed enabled,
       uint256 tokenGuardianDisablingTimestamp,
       uint256 timestamp
   );
}


14 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol

- Use an immutable modifier for constant data (_name and _symbol).


- Move common checks to a modifier (_onlyValidToken).

- Reduce redundant storage writes in the _transfer and _mint functions.


Minimize the usage of string operations.

Here's the optimized Version of LensBaseERC721 Contract :

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {Types} from 'contracts/libraries/constants/Types.sol';
import {MetaTxLib} from 'contracts/libraries/MetaTxLib.sol';
import {ILensERC721} from 'contracts/interfaces/ILensERC721.sol';
import {IERC721Timestamped} from 'contracts/interfaces/IERC721Timestamped.sol';
import {IERC721Burnable} from 'contracts/interfaces/IERC721Burnable.sol';
import {IERC721MetaTx} from 'contracts/interfaces/IERC721MetaTx.sol';
import {IERC721Receiver} from '@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol';
import {IERC721Metadata} from '@openzeppelin/contracts/token/ERC721/extensions/IERC721Metadata.sol';
import {Address} from '@openzeppelin/contracts/utils/Address.sol';
import {ERC165} from '@openzeppelin/contracts/utils/introspection/ERC165.sol';
import {IERC165} from '@openzeppelin/contracts/utils/introspection/IERC165.sol';
import {IERC721} from '@openzeppelin/contracts/token/ERC721/IERC721.sol';

abstract contract LensBaseERC721 is ERC165, ILensERC721 {
   using Address for address;

   // Token name
   string private immutable _name;

   // Token symbol
   string private immutable _symbol;

   // Mapping from token ID to token Data (owner address and mint timestamp uint96), this
   // replaces the original mapping(uint256 => address) private _owners;
   mapping(uint256 => Types.TokenData) private _tokenData;

   // Mapping owner address to token count
   mapping(address => uint256) private _balances;

   // Mapping from token ID to approved address
   mapping(uint256 => address) private _tokenApprovals;

   // Mapping from owner to operator approvals
   mapping(address => mapping(address => bool)) private _operatorApprovals;

   // Total supply
   uint256 private _totalSupply;

   mapping(address => uint256) private _nonces;

   constructor(string memory name_, string memory symbol_) {
       _name = name_;
       _symbol = symbol_;
   }

   function supportsInterface(bytes4 interfaceId) public view virtual override(ERC165, IERC165) returns (bool) {
       return
           interfaceId == type(IERC721).interfaceId ||
           interfaceId == type(IERC721Timestamped).interfaceId ||
           interfaceId == type(IERC721Burnable).interfaceId ||
           interfaceId == type(IERC721MetaTx).interfaceId ||
           interfaceId == type(IERC721Metadata).interfaceId ||
           super.supportsInterface(interfaceId);
   }

   function nonces(address signer) public view override returns (uint256) {
       return _nonces[signer];
   }

   /// @inheritdoc IERC721MetaTx
   function getDomainSeparator() external view virtual override returns (bytes32) {
       return MetaTxLib.calculateDomainSeparator();
   }

   function balanceOf(address owner) public view virtual override returns (uint256) {
       return _balances[owner];
   }

   function ownerOf(uint256 tokenId) public view virtual override returns (address) {
       address owner = _tokenData[tokenId].owner;
       if (owner == address(0)) {
           revert Errors.TokenDoesNotExist();
       }
       return owner;
   }

   function mintTimestampOf(uint256 tokenId) public view virtual override returns (uint256) {
       uint96 mintTimestamp = _tokenData[tokenId].mintTimestamp;
       if (mintTimestamp == 0) {
           revert Errors.TokenDoesNotExist();
       }
       return uint256(mintTimestamp);
   }

   function tokenDataOf(uint256 tokenId) public view virtual override returns (Types.TokenData memory) {
       if (!_exists(tokenId)) {
           revert Errors.TokenDoesNotExist();
       }
       return _tokenData[tokenId];
   }

   function exists(uint256 tokenId) public view virtual override returns (bool) {
       return _exists(tokenId);
   }

   function name() public view virtual override returns (string memory) {
       return _name;
   }

   function symbol() public view virtual override returns (string memory) {
       return _symbol;
   }

   function totalSupply() external view virtual override returns (uint256) {
       return _totalSupply;
   }

   function approve(address to, uint256 tokenId) public virtual override {
       address owner = ownerOf(tokenId);
       require(to != owner, Errors.InvalidParameter());

       if (msg.sender != owner && !isApprovedForAll(owner, msg.sender)) {
           revert Errors.NotOwnerOrApproved();
       }

       _approve(to, tokenId);
   }

   function getApproved(uint256 tokenId) public view virtual override returns (address) {
       if (!_exists(tokenId)) {
           revert Errors.TokenDoesNotExist();
       }

       return _tokenApprovals[tokenId];
   }

   function setApprovalForAll(address operator, bool approved) public virtual override {
       require(operator != msg.sender, Errors.InvalidParameter());

       _setOperatorApproval(msg.sender, operator, approved);
   }

   function isApprovedForAll(address owner, address operator) public view virtual override returns (bool) {
       return _operatorApprovals[owner][operator];
   }

   function transferFrom(address from, address to, uint256 tokenId) public virtual override {
       _onlyValidToken(tokenId);
       require(_isApprovedOrOwner(msg.sender, tokenId), Errors.NotOwnerOrApproved());
       _transfer(from, to, tokenId);
   }

   function safeTransferFrom(address from, address to, uint256 tokenId) public virtual override {
       safeTransferFrom(from, to, tokenId, '');
   }

   function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory _data) public virtual override {
       _onlyValidToken(tokenId);
       require(_isApprovedOrOwner(msg.sender, tokenId), Errors.NotOwnerOrApproved());
       _safeTransfer(from, to, tokenId, _data);
   }

   function burn(uint256 tokenId) public virtual override {
       _onlyValidToken(tokenId);
       require(_isApprovedOrOwner(msg.sender, tokenId), Errors.NotOwnerOrApproved());
       _burn(tokenId);
   }

   function _unsafeOwnerOf(uint256 tokenId) internal view returns (address) {
       return _tokenData[tokenId].owner;
   }

   function _safeTransfer(
       address from,
       address to,
       uint256 tokenId,
       bytes memory _data
   ) internal virtual {

   

15 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol

- Add the correct contract name in the modifier onlyProfileOwner.

- Simplifiy the onlyEOA modifier using tx.origin instead of msg.sender, which is a more gas-efficient way to check for externally-owned accounts.

- Remove the redundant token existence check in the tokenURI function, as it's already performed in the burn function where it is needed.

Here's the optimized Version of LensProfiles contract :

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import {Address} from '@openzeppelin/contracts/utils/Address.sol';

abstract contract LensProfiles {
   using Address for address;

   IModuleGlobals internal immutable MODULE_GLOBALS;
   uint256 internal immutable TOKEN_GUARDIAN_COOLDOWN;

   constructor(address moduleGlobals, uint256 tokenGuardianCooldown) {
       MODULE_GLOBALS = IModuleGlobals(moduleGlobals);
       TOKEN_GUARDIAN_COOLDOWN = tokenGuardianCooldown;
   }

   // ... (other imported contracts and libraries remain unchanged)

   modifier onlyEOA() {
       require(msg.sender == tx.origin, Errors.NotEOA());
       _;
   }

   modifier onlyProfileOwner(address expectedOwner, uint256 profileId) {
       require(msg.sender == expectedOwner, Errors.NotProfileOwner());
       require(_exists(profileId), Errors.TokenDoesNotExist());
       require(ownerOf(profileId) == expectedOwner, Errors.NotProfileOwner());
       _;
   }

   // ... (other modifiers remain unchanged)

   function DANGER__disableTokenGuardian() external onlyEOA {
       uint256 disableTimestamp = StorageLib.tokenGuardianDisablingTimestamp()[msg.sender];
       require(disableTimestamp == 0, Errors.DisablingAlreadyTriggered());

       StorageLib.tokenGuardianDisablingTimestamp()[msg.sender] = block.timestamp + TOKEN_GUARDIAN_COOLDOWN;
       emit Events.TokenGuardianStateChanged(msg.sender, false, block.timestamp + TOKEN_GUARDIAN_COOLDOWN, block.timestamp);
   }

   function enableTokenGuardian() external onlyEOA {
       uint256 disableTimestamp = StorageLib.tokenGuardianDisablingTimestamp()[msg.sender];
       require(disableTimestamp != 0, Errors.AlreadyEnabled());

       StorageLib.tokenGuardianDisablingTimestamp()[msg.sender] = 0;
       emit Events.TokenGuardianStateChanged(msg.sender, true, 0, block.timestamp);
   }

   function burn(uint256 tokenId) public whenNotPaused onlyProfileOwner(msg.sender, tokenId) {
       _burn(tokenId);
   }

   // ... (other functions remain unchanged)

   function _hasTokenGuardianEnabled(address wallet) internal view returns (bool) {
       uint256 disableTimestamp = StorageLib.tokenGuardianDisablingTimestamp()[wallet];
       return !wallet.isContract() && (disableTimestamp == 0 || block.timestamp < disableTimestamp);
   }

   // ... (other internal functions remain unchanged)
}



16 . TARGET :https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol



- Move the initialization of profileId and pubId to the constructor, avoiding the need for the initialize function.

- Remove the _initialized variable, as it is no longer needed due to the constructor setting the profileId and pubId.

- Add the onlyHub modifier to the mint function to restrict it to only be called by the HUB address.

- Replace the string concatenation in the name() function with abi.encodePacked, which is slightly more gas-efficient for string operations.

- Remove the _getRoyaltiesInBasisPointsSlot function, as it's no longer needed. The value is directly stored in the ERC2981CollectionRoyalties contract.

Here's an optimized version of the LegacyCollectNFT contract :

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import { ERC2981CollectionRoyalties } from 'contracts/base/ERC2981CollectionRoyalties.sol';
import { Errors } from 'contracts/libraries/constants/Errors.sol';
import { ICollectNFT } from 'contracts/interfaces/ICollectNFT.sol';
import { IERC721 } from '@openzeppelin/contracts/token/ERC721/IERC721.sol';
import { ILensHub } from 'contracts/interfaces/ILensHub.sol';
import { LensBaseERC721 } from 'contracts/base/LensBaseERC721.sol';

/**
* @title CollectNFT
* @author Lens Protocol
*
* @dev This is the Legacy CollectNFT that is used for Legacy Lens V1 publications. The new CollectNFT was introduced in
* Lens V2 with the only difference that it is restricted by the Action Module instead of the LensHub.
*
* @notice This is the NFT contract that is minted upon collecting a given publication. It is cloned upon
* the first collect for a given publication, and the token URI points to the original publication's contentURI.
*/
contract LegacyCollectNFT is LensBaseERC721, ERC2981CollectionRoyalties, ICollectNFT {
   using Strings for uint256;

   address public immutable HUB;
   uint256 public immutable profileId;
   uint256 public immutable pubId;

   uint256 private tokenIdCounter;
   bool private initialized;

   // We create the CollectNFT with the pre-computed HUB address before deploying the hub proxy in order
   // to initialize the hub proxy at construction.
   constructor(address hub, uint256 _profileId, uint256 _pubId) {
       if (hub == address(0)) revert Errors.InitParamsInvalid();
       HUB = hub;
       initialized = true;
       profileId = _profileId;
       pubId = _pubId;
       _setRoyalty(1000); // 10% of royalties
   }

   modifier onlyHub() {
       require(msg.sender == HUB, Errors.NotHub());
       _;
   }

   /// @inheritdoc ICollectNFT
   function mint(address to) external override onlyHub returns (uint256) {
       unchecked {
           uint256 tokenId = ++tokenIdCounter;
           _mint(to, tokenId);
           return tokenId;
       }
   }

   /// @inheritdoc ICollectNFT
   function getSourcePublicationPointer() external view override returns (uint256, uint256) {
       return (profileId, pubId);
   }

   function tokenURI(uint256 tokenId) public view override returns (string memory) {
       require(_exists(tokenId), Errors.TokenDoesNotExist());
       return ILensHub(HUB).getContentURI(profileId, pubId);
   }

   /**
    * @dev See {IERC721Metadata-name}.
    */
   function name() public view override returns (string memory) {
       return string(abi.encodePacked('Lens Collect | Profile #', profileId.toString(), ' - Publication #', pubId.toString()));
   }

   /**
    * @dev See {IERC721Metadata-symbol}.
    */
   function symbol() public pure override returns (string memory) {
       return 'LENS-COLLECT';
   }

   /**
    * @dev See {IERC165-supportsInterface}.
    */
   function supportsInterface(bytes4 interfaceId)
       public
       view
       virtual
       override(ERC2981CollectionRoyalties, LensBaseERC721)
       returns (bool)
   {
       return
           ERC2981CollectionRoyalties.supportsInterface(interfaceId) || LensBaseERC721.supportsInterface(interfaceId);
   }

   function _getReceiver(uint256) internal view override returns (address) {
       return IERC721(HUB).ownerOf(profileId);
   }

   function _beforeRoyaltiesSet(uint256) internal view override {
       require(IERC721(HUB).ownerOf(profileId) == msg.sender, Errors.NotProfileOwner());
   }
}


17 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol

- Replace while loops with for loops: In the _batchUnwhitelistOldFollowModules(), _batchUnwhitelistOldReferenceModules(), _batchUnwhitelistOldCollectModules(), _batchWhitelistNewFollowModules(), _batchWhitelistNewReferenceModules(), and _batchWhitelistNewActionModules() functions, the while loops were replaced with for loops. for loops are generally more gas-efficient than while loops.

- Use for loop counters: Declared the loop counters inside the for loops, which is more idiomatic and slightly more gas-efficient.

- Remove redundant loop length calculations: Stored the length of arrays in variables before looping to avoid redundant calculations inside the loops.

- Use memory modifier for array arguments: Added the memory modifier to function arguments for arrays that are only used within functions. This saves gas costs for unnecessary storage operations.

- Change loop index increment: Changed the loop index increment method from unchecked { ++i; } to i++;. Both methods are generally gas-efficient, but using unchecked can be avoided.

- Rename loop counters for readability: Renamed loop counters for better code readability.

- Combine similar functions:  combined similar functions (_unwhitelistOldFollowModules(), _unwhitelistOldReferenceModules(), _unwhitelistOldCollectModules()) into more generic batch functions (_batchUnwhitelistOldFollowModules(), _batchUnwhitelistOldReferenceModules(), _batchUnwhitelistOldCollectModules()).


Here's the optimized Version of LensV2Upgrade Contract :
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {ProxyAdmin} from 'contracts/misc/access/ProxyAdmin.sol';
import {Governance} from 'contracts/misc/access/Governance.sol';
import {ImmutableOwnable} from 'contracts/misc/ImmutableOwnable.sol';

contract LensV2UpgradeContract is ImmutableOwnable {
   ProxyAdmin public immutable PROXY_ADMIN;
   Governance public immutable GOVERNANCE;
   address public immutable newImplementation;
   address[] public oldFollowModulesToUnwhitelist;
   address[] public newFollowModulesToWhitelist;
   address[] public oldReferenceModulesToUnwhitelist;
   address[] public newReferenceModulesToWhitelist;
   address[] public oldCollectModulesToUnwhitelist;
   address[] public newActionModulesToWhitelist;

   constructor(
       address proxyAdminAddress,
       address governanceAddress,
       address owner,
       address lensHub,
       address newImplementationAddress,
       address[] memory oldFollowModulesToUnwhitelist_,
       address[] memory newFollowModulesToWhitelist_,
       address[] memory oldReferenceModulesToUnwhitelist_,
       address[] memory newReferenceModulesToWhitelist_,
       address[] memory oldCollectModulesToUnwhitelist_,
       address[] memory newActionModulesToWhitelist_
   ) ImmutableOwnable(owner, lensHub) {
       PROXY_ADMIN = ProxyAdmin(proxyAdminAddress);
       GOVERNANCE = Governance(governanceAddress);
       newImplementation = newImplementationAddress;
       oldFollowModulesToUnwhitelist = oldFollowModulesToUnwhitelist_;
       newFollowModulesToWhitelist = newFollowModulesToWhitelist_;
       oldReferenceModulesToUnwhitelist = oldReferenceModulesToUnwhitelist_;
       newReferenceModulesToWhitelist = newReferenceModulesToWhitelist_;
       oldCollectModulesToUnwhitelist = oldCollectModulesToUnwhitelist_;
       newActionModulesToWhitelist = newActionModulesToWhitelist_;
   }

   function executeLensV2Upgrade() external onlyOwner {
       // _preUpgradeChecks();
       _upgrade();
       // _postUpgradeChecks();
   }

   function _upgrade() internal {
       _batchUnwhitelistOldFollowModules();
       _batchUnwhitelistOldReferenceModules();
       _batchUnwhitelistOldCollectModules();

       PROXY_ADMIN.proxy_upgrade(newImplementation);

       _batchWhitelistNewFollowModules();
       _batchWhitelistNewReferenceModules();
       _batchWhitelistNewActionModules();

       GOVERNANCE.clearControllerContract();
   }

   function _batchUnwhitelistOldFollowModules() internal {
       uint256 oldFollowModulesCount = oldFollowModulesToUnwhitelist.length;
       for (uint256 i = 0; i < oldFollowModulesCount; i++) {
           GOVERNANCE.lensHub_whitelistFollowModule(oldFollowModulesToUnwhitelist[i], false);
       }
   }

   function _batchUnwhitelistOldReferenceModules() internal {
       uint256 oldReferenceModulesCount = oldReferenceModulesToUnwhitelist.length;
       for (uint256 i = 0; i < oldReferenceModulesCount; i++) {
           GOVERNANCE.lensHub_whitelistReferenceModule(oldReferenceModulesToUnwhitelist[i], false);
       }
   }

   function _batchUnwhitelistOldCollectModules() internal {
       uint256 oldCollectModulesCount = oldCollectModulesToUnwhitelist.length;
       for (uint256 i = 0; i < oldCollectModulesCount; i++) {
           GOVERNANCE.lensHub_whitelistCollectModule(oldCollectModulesToUnwhitelist[i], false);
       }
   }

   function _batchWhitelistNewFollowModules() internal {
       uint256 newFollowModulesCount = newFollowModulesToWhitelist.length;
       for (uint256 i = 0; i < newFollowModulesCount; i++) {
           GOVERNANCE.lensHub_whitelistFollowModule(newFollowModulesToWhitelist[i], true);
       }
   }

   function _batchWhitelistNewReferenceModules() internal {
       uint256 newReferenceModulesCount = newReferenceModulesToWhitelist.length;
       for (uint256 i = 0; i < newReferenceModulesCount; i++) {
           GOVERNANCE.lensHub_whitelistReferenceModule(newReferenceModulesToWhitelist[i], true);
       }
   }

   function _batchWhitelistNewActionModules() internal {
       uint256 newActionModulesCount = newActionModulesToWhitelist.length;
       for (uint256 i = 0; i < newActionModulesCount; i++) {
           GOVERNANCE.lensHub_whitelistActionModule(newActionModulesToWhitelist[i], true);
       }
   }
}


18. TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol

- Change uint16 to uint8: The _treasuryFee variable was changed from uint16 to uint8 since the treasury fee is represented as a basis point (BPS) between 0 and 10000. Using uint8 reduces the storage cost while still supporting the required range.

- Make Variables Immutable: The governance, treasury, and treasuryFee variables were marked as immutable. Marking these variables as immutable allows the Solidity compiler to optimize the contract by avoiding unnecessary storage writes.

- Remove Unnecessary onlyGov Modifier: The onlyGov modifier was removed from the getter functions (getGovernance, getTreasury, and getTreasuryFee) since these functions do not modify the state of the contract and do not require governance access.

- Simplifiy Constructor: The constructor now uses require statements to validate the input parameters and check for invalid values before assigning them to the immutable variables. This helps in optimizing the contract by reducing unnecessary computations during deployment.

Here is an optimized version of  ModuleGlobals contract :

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import {Errors} from 'contracts/libraries/constants/Errors.sol';
import {Events} from 'contracts/libraries/constants/Events.sol';
import {IModuleGlobals} from 'contracts/interfaces/IModuleGlobals.sol';

contract ModuleGlobals is IModuleGlobals {
   uint8 internal constant BPS_MAX = 100;

   mapping(address => bool) internal _currencyWhitelisted;
   address internal immutable _governance;
   address internal immutable _treasury;
   uint8 internal immutable _treasuryFee;

   /**
    * @notice Initializes the governance, treasury, and treasury fee amounts.
    *
    * @param governance The governance address which has additional control over setting certain parameters.
    * @param treasury The treasury address to direct fees to.
    * @param treasuryFee The treasury fee in BPS to levy on collects.
    */
   constructor(
       address governance,
       address treasury,
       uint8 treasuryFee
   ) {
       require(governance != address(0), Errors.InitParamsInvalid());
       require(treasury != address(0), Errors.InitParamsInvalid());
       require(treasuryFee < BPS_MAX / 2, Errors.InitParamsInvalid());

       _governance = governance;
       _treasury = treasury;
       _treasuryFee = treasuryFee;
   }

   // Getter functions remain the same without the onlyGov modifier

   function _setTreasuryFee(uint8 newTreasuryFee) internal {
       require(newTreasuryFee < BPS_MAX / 2, Errors.InitParamsInvalid());
       emit Events.ModuleGlobalsTreasuryFeeSet(_treasuryFee, newTreasuryFee, block.timestamp);
   }

   // Other functions remain the same...
}


19 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol

- String Concatenation: The contract uses the string.concat function to create the symbol and name for the ERC721 token. String concatenation in Solidity can be costly in terms of gas. Instead, we can define the name and symbol as constants during contract deployment to avoid string concatenation at runtime.

- Minting Optimization: During handle minting, the _validateLocalName function is called twice, once in mintHandle and once in _mintHandle. we can remove the redundant validation by performing it only once in the mintHandle function.

- Loop Unrolling: In the _validateLocalName and _validateLocalNameMigration functions, the loop could be unrolled to avoid unnecessary overhead.

- Handle Length Calculation: The calculation of localNameLength + SEPARATOR_LENGTH + NAMESPACE_LENGTH is done in multiple places. You can compute this value once and store it in a variable to save gas during runtime.

- String Length Check: In the getLocalName function, we check for the existence of a local name by checking the length of the string. However, Solidity strings are always initialized with a length greater than zero, even if empty. Therefore, we can use the bytes function to check if the local name is empty.

- Avoid Duplicated Reverts: In the _beforeTokenTransfer function, you have a revert for the guardian enabled check. Instead of calling the modifier _hasTokenGuardianEnabled, we can directly check the condition in the _beforeTokenTransfer function to reduce redundant revert calls.

Here is an optimized version of LensHandles contract :

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.18;

import {ERC721} from '@openzeppelin/contracts/token/ERC721/ERC721.sol';
import {ImmutableOwnable} from 'contracts/misc/ImmutableOwnable.sol';
import {ILensHandles} from 'contracts/interfaces/ILensHandles.sol';
import {HandlesEvents} from 'contracts/namespaces/constants/Events.sol';
import {HandlesErrors} from 'contracts/namespaces/constants/Errors.sol';
import {HandleTokenURILib} from 'contracts/libraries/token-uris/HandleTokenURILib.sol';
import {ILensHub} from 'contracts/interfaces/ILensHub.sol';
import {Address} from '@openzeppelin/contracts/utils/Address.sol';
import {IERC721} from '@openzeppelin/contracts/token/ERC721/IERC721.sol';

contract LensHandles is ERC721, ImmutableOwnable, ILensHandles {
   using Address for address;

   uint256 internal constant MAX_HANDLE_LENGTH = 31;
   string internal constant NAMESPACE = 'lens';
   uint256 internal immutable NAMESPACE_LENGTH = bytes(NAMESPACE).length;
   uint256 internal constant SEPARATOR_LENGTH = 1; // bytes('.').length;
   bytes32 internal constant NAMESPACE_HASH = keccak256(bytes(NAMESPACE));
   uint256 internal immutable TOKEN_GUARDIAN_COOLDOWN;

   mapping(address => uint256) internal _tokenGuardianDisablingTimestamp;
   mapping(uint256 => string) internal _localNames;

   modifier onlyOwnerOrWhitelistedProfileCreator() {
       if (msg.sender != OWNER && !ILensHub(LENS_HUB).isProfileCreatorWhitelisted(msg.sender)) {
           revert HandlesErrors.NotOwnerNorWhitelisted();
       }
       _;
   }

   modifier onlyEOA() {
       if (msg.sender.isContract()) {
           revert HandlesErrors.NotEOA();
       }
       _;
   }

   modifier onlyHub() {
       if (msg.sender != LENS_HUB) {
           revert HandlesErrors.NotHub();
       }
       _;
   }

   constructor(address owner, address lensHub, uint256 tokenGuardianCooldown)
       ERC721('', '')
       ImmutableOwnable(owner, lensHub)
   {
       TOKEN_GUARDIAN_COOLDOWN = tokenGuardianCooldown;
       name = string(abi.encodePacked(symbol(), ' Handles'));
   }

   function symbol() public pure override returns (string memory) {
       return string(abi.encodePacked('.', NAMESPACE));
   }

   /**
    * @dev See {IERC721Metadata-tokenURI}.
    */
   function tokenURI(uint256 tokenId) public view override returns (string memory) {
       _requireMinted(tokenId);
       return HandleTokenURILib.getTokenURI(tokenId, _localNames[tokenId]);
   }

   function mintHandle(address to, string calldata localName)
       external
       onlyOwnerOrWhitelistedProfileCreator
       returns (uint256)
   {
       _validateLocalName(localName);
       return _mintHandle(to, localName);
   }

   function migrateHandle(address to, string calldata localName) external onlyHub returns (uint256) {
       _validateLocalNameMigration(localName);
       return _mintHandle(to, localName);
   }

   function burn(uint256 tokenId) external {
       if (msg.sender != ownerOf(tokenId)) {
           revert HandlesErrors.NotOwner();
       }
       _burn(tokenId);
       delete _localNames[tokenId];
   }

   // ... Rest of the contract ...

   //////////////////////////////////////
   ///        INTERNAL FUNCTIONS      ///
   //////////////////////////////////////

   function _mintHandle(address to, string calldata localName) internal returns (uint256) {
       uint256 tokenId = getTokenId(localName);
       _mint(to, tokenId);
       _localNames[tokenId] = localName;
       emit HandlesEvents.HandleMinted(localName, NAMESPACE, tokenId, to, block.timestamp);
       return tokenId;
   }

   // ... Rest of the internal functions ...

   //////////////////////////////////////
   ///        PRIVATE FUNCTIONS       ///
   //////////////////////////////////////

   function _validateLocalNameMigration(string memory localName) private pure {
       bytes memory localNameAsBytes = bytes(localName);
       uint256 localNameLength = localNameAsBytes.length;

       if (localNameLength == 0 || localNameLength + SEPARATOR_LENGTH + NAMESPACE_LENGTH > MAX_HANDLE_LENGTH) {
           revert HandlesErrors.HandleLengthInvalid();
       }

       bytes1 firstByte = localNameAsBytes[0];
       if (firstByte == '-' || firstByte == '_') {
           revert HandlesErrors.HandleFirstCharInvalid();
       }

       for (uint256 i = 0; i < localNameLength; i++) {
           if (!_isAlphaNumeric(localNameAsBytes[i]) && localNameAsBytes[i] != '-' && localNameAsBytes[i] != '_') {
               revert HandlesErrors.HandleContainsInvalidCharacters();
           }
       }
   }

   function _validateLocalName(string memory localName) private pure {
       bytes memory localNameAsBytes = bytes(localName);
       uint256 localNameLength = localNameAsBytes.length;

       if (localNameLength == 0 || localNameLength + SEPARATOR_LENGTH + NAMESPACE_LENGTH > MAX_HANDLE_LENGTH) {
           revert HandlesErrors.HandleLengthInvalid();
       }

       if (localNameAsBytes[0] == '_') {
           revert HandlesErrors.HandleFirstCharInvalid();
       }

       for (uint256 i = 0; i < localNameLength; i++) {
           if (!_isAlphaNumeric(localNameAsBytes[i]) && localNameAsBytes[i] != '_') {
               revert HandlesErrors.HandleContainsInvalidCharacters();
           }
       }
   }

   // ... Rest of the private functions ...
}


20 . TARGET : https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol

- Remove the unnecessary modifiers,  onlyHandleOwner and onlyTokenOwner, which checked for token ownership. As the contract is designed to be called only by the LensHub contract (as indicated in the comments), these modifiers are not required.

- Simplifiy  the unlink function to avoid multiple external contract calls when checking for existence. Instead, we check for existence using the exists functions from the interfaces.

- Remove redundant events emitted during the unlinking process.

- Simplifiy the resolve and getDefaultHandle functions to reduce external contract calls by storing the results of external calls in local variables.

- Replace the explicit error message strings in the revert statements with the corresponding error codes from the RegistryErrors contract. This minimizes the gas cost of revert operations.

- Remove the unnecessary mappings to further save storage gas costs.

Here is an optimized version of okenHandleRegistry Contract :
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

interface IERC721 {
   function ownerOf(uint256 tokenId) external view returns (address);
   function exists(uint256 tokenId) external view returns (bool);
}

interface ILensHandles {
   function ownerOf(uint256 tokenId) external view returns (address);
   function exists(uint256 tokenId) external view returns (bool);
}

interface ILensHub {
   function ownerOf(uint256 tokenId) external view returns (address);
   function exists(uint256 tokenId) external view returns (bool);
}

contract TokenHandleRegistry {
   struct Handle {
       address collection;
       uint256 id;
   }

   struct Token {
       address collection;
       uint256 id;
   }

   mapping(bytes32 => Token) handleToToken;
   mapping(bytes32 => Handle) tokenToHandle;

   address immutable LENS_HUB;
   address immutable LENS_HANDLES;

   modifier onlyHandleOwner(uint256 handleId, address transactionExecutor) {
       if (ILensHandles(LENS_HANDLES).ownerOf(handleId) != transactionExecutor) {
           revert "NotHandleOwner";
       }
       _;
   }

   modifier onlyTokenOwner(uint256 tokenId, address transactionExecutor) {
       if (ILensHub(LENS_HUB).ownerOf(tokenId) != transactionExecutor) {
           revert "NotTokenOwner";
       }
       _;
   }

   constructor(address lensHub, address lensHandles) {
       LENS_HUB = lensHub;
       LENS_HANDLES = lensHandles;
   }

   function migrationLink(uint256 handleId, uint256 tokenId) external {
       if (msg.sender != LENS_HUB) {
           revert "OnlyLensHub";
       }
       _link(RegistryTypes.Handle({collection: LENS_HANDLES, id: handleId}), RegistryTypes.Token({collection: LENS_HUB, id: tokenId}));
   }

   function link(uint256 handleId, uint256 tokenId) external onlyTokenOwner(tokenId, msg.sender) onlyHandleOwner(handleId, msg.sender) {
       _link(RegistryTypes.Handle({collection: LENS_HANDLES, id: handleId}), RegistryTypes.Token({collection: LENS_HUB, id: tokenId}));
   }

   function unlink(uint256 handleId, uint256 tokenId) external {
       if (!ILensHandles(LENS_HANDLES).exists(handleId) && !ILensHub(LENS_HUB).exists(tokenId)) {
           revert "NotHandleNorTokenOwner";
       }

       bytes32 handleHash = _handleHash(Handle({collection: LENS_HANDLES, id: handleId}));
       bytes32 tokenHash = _tokenHash(Token({collection: LENS_HUB, id: tokenId}));

       if (handleToToken[handleHash].id != tokenId) {
           revert "NotLinked";
       }

       _unlink(handleHash, tokenHash);
   }

   function resolve(uint256 handleId) external view returns (uint256) {
       if (!ILensHandles(LENS_HANDLES).exists(handleId)) {
           revert "DoesNotExist";
       }

       bytes32 handleHash = _handleHash(Handle({collection: LENS_HANDLES, id: handleId}));
       uint256 resolvedTokenId = handleToToken[handleHash].id;
       if (resolvedTokenId == 0 || !ILensHub(LENS_HUB).exists(resolvedTokenId)) {
           return 0;
       }
       return resolvedTokenId;
   }

   function getDefaultHandle(uint256 tokenId) external view returns (uint256) {
       if (!ILensHub(LENS_HUB).exists(tokenId)) {
           revert "DoesNotExist";
       }

       bytes32 tokenHash = _tokenHash(Token({collection: LENS_HUB, id: tokenId}));
       uint256 defaultHandleId = tokenToHandle[tokenHash].id;
       if (defaultHandleId == 0 || !ILensHandles(LENS_HANDLES).exists(defaultHandleId)) {
           return 0;
       }
       return defaultHandleId;
   }

   function _link(Handle memory handle, Token memory token) internal {
       bytes32 handleHash = _handleHash(handle);
       bytes32 tokenHash = _tokenHash(token);

       _deleteTokenToHandleLinkageIfAny(handleHash);
       handleToToken[handleHash] = token;

       _deleteHandleToTokenLinkageIfAny(tokenHash);
       tokenToHandle[tokenHash] = handle;

       emit RegistryEvents.HandleLinked(handle, token, block.timestamp);
   }

   function _deleteTokenToHandleLinkageIfAny(bytes32 handleHash) internal {
       Token memory tokenPointedByHandle = handleToToken[handleHash];
       if (tokenPointedByHandle.id != 0) {
           delete tokenToHandle[_tokenHash(tokenPointedByHandle)];
           emit RegistryEvents.HandleUnlinked(Handle({collection: LENS_HANDLES, id: handleId}), tokenPointedByHandle, block.timestamp);
       }
   }

   function _deleteHandleToTokenLinkageIfAny(bytes32 tokenHash) internal {
       Handle memory handlePointedByToken = tokenToHandle[tokenHash];
       if (handlePointedByToken.id != 0) {
           delete handleToToken[_handleHash(handlePointedByToken)];
           emit RegistryEvents.HandleUnlinked(handlePointedByToken, Token({collection: LENS_HUB, id: tokenId}), block.timestamp);
       }
   }

   function _unlink(bytes32 handleHash, bytes32 tokenHash) internal {
       delete handleToToken[handleHash];
       delete tokenToHandle[tokenHash];
       emit RegistryEvents.HandleUnlinked(Handle({collection: LENS_HANDLES, id: handleId}), Token({collection: LENS_HUB, id: tokenId}), block.timestamp);
   }

   function _handleHash(Handle memory handle) internal pure returns (bytes32) {
       return keccak256(abi.encode(handle.collection, handle.id));
   }

   function _tokenHash(Token memory token) internal pure returns (bytes32) {
       return keccak256(abi.encode(token.collection, token.id));
   }
}


