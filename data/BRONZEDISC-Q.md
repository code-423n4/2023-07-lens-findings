## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol

```solidity
// place these modifiers before the constructor
38:    modifier whenNotPaused() {
45:    modifier onlyProfileOwner(address expectedOwner, uint256 profileId) {
50:    modifier onlyEOA() {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol

```solidity
// place this struct before the first function since this contract does not have a constructor
173:    struct ReferenceParamsForAbiEncode {
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol

```solidity
// public functions coming before external ones
70:    function name() public pure override returns (string memory) {
74:    function symbol() public pure override returns (string memory) {
81:    function tokenURI(uint256 tokenId) public view override returns (string memory) {
139    function approve(address to, uint256 tokenId) public override(IERC721, ERC721) {
147:    function setApprovalForAll(address operator, bool approved) public override(IERC721, ERC721) {
168:    function getLocalName(uint256 tokenId) public view returns (string memory) {
177:    function getHandle(uint256 tokenId) public view returns (string memory) {
182:    function getTokenId(string memory localName) public pure returns (uint256) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol

```solidity
// external functions should come before public functions
99:    function getDomainSeparator() external view virtual override returns (bytes32) {
166:    function totalSupply() external view virtual override returns (uint256) {

// private function coming before internal one
469:    function _checkOnERC721Received(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ValidationLib.sol

```solidity
// external coming after internal functions
121:    function validateLegacyCollectReferrer(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/PublicationLib.sol

```solidity
// internal functions should come right before private functions
170:    function getPublicationType(uint256 profileId, uint256 pubId) internal view returns (Types.PublicationType) {
291:    function _fillRootOfPublicationInStorage(

// private function coming before external one
203:    function _asReferencePubParams(Types.QuoteParams calldata quoteParams)
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol

```solidity
// internal functions coming before external ones
16:    function ownerOf(uint256 profileId) internal view returns (address) {
24:    function exists(uint256 profileId) internal view returns (bool) {

// private functions coming before some non-private ones
110:    function _initFollowModule(
120:    function _setProfileImageURI(uint256 profileId, string calldata imageURI) private {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol

```solidity
// place these private functions for last.
60:    function _migrateProfile(
114:    function _migrateFollow(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol

```solidity
// private functions should come last
160:    function _hashActionModulesInitDatas(bytes[] memory actionModulesInitDatas) private pure returns (bytes32) {
190:    function _abiEncode(ReferenceParamsForAbiEncode memory referenceParamsForAbiEncode)
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol

```solidity
// private functions should come for last
64:    function _setState(Types.ProtocolState newState) private returns (Types.ProtocolState) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol

```solidity
// internal function coming before some externals and public ones.
168:    function _wrap(uint256 followTokenId, address wrappedTokenReceiver) internal {

// place this external function first with the others
480:    function tryMigrate(
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensHub.sol

```solidity
11:  interface ILensHub is ILensProfiles, ILensProtocol, ILensGovernable, ILensHubEventHooks, ILensImplGetters {}
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensERC721.sol

```solidity
11:  interface ILensERC721 is IERC721, IERC721Timestamped, IERC721Burnable, IERC721MetaTx, IERC721Metadata {}
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/constants/Types.sol

```solidity
10:      struct Token {
16:      struct Handle {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/constants/Events.sol

```solidity
7:  library HandlesEvents {
8:    event HandleMinted(string handle, string namespace, uint256 handleId, address to, uint256 timestamp);
27:  library RegistryEvents {
28:    event HandleLinked(RegistryTypes.Handle handle, RegistryTypes.Token token, uint256 timestamp);
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/constants/Errors.sol

```solidity
5:  library RegistryErrors {
14:  library HandlesErrors {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol

```solidity
28:    modifier onlyHandleOwner(uint256 handleId, address transactionExecutor) {
35:    modifier onlyTokenOwner(uint256 tokenId, address transactionExecutor) {
42:    constructor(address lensHub, address lensHandles) {
50:    function migrationLink(uint256 handleId, uint256 tokenId) external {
122:    function _resolveHandleToToken(
128:    function _resolveTokenToHandle(
134:    function _link(RegistryTypes.Handle memory handle, RegistryTypes.Token memory token) internal {
144:    function _deleteTokenToHandleLinkageIfAny(RegistryTypes.Handle memory handle) internal {
152:    function _deleteHandleToTokenLinkageIfAny(RegistryTypes.Token memory token) internal {
160:    function _unlink(RegistryTypes.Handle memory handle, RegistryTypes.Token memory token) internal {
167:    function _handleHash(RegistryTypes.Handle memory handle) internal pure returns (bytes32) {
171:    function _tokenHash(RegistryTypes.Token memory token) internal pure returns (bytes32) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol

```solidity
39:    modifier onlyOwnerOrWhitelistedProfileCreator() {
48:    modifier onlyEOA() {
55:    modifier onlyHub() {
62:    constructor(
70:    function name() public pure override returns (string memory) {
74:    function symbol() public pure override returns (string memory) {
96:    function migrateHandle(address to, string calldata localName) external onlyHub returns (uint256) {
101:    function burn(uint256 tokenId) external {
113:    function DANGER__disableTokenGuardian() external onlyEOA {
126:    function enableTokenGuardian() external onlyEOA {
139:    function approve(address to, uint256 tokenId) public override(IERC721, ERC721) {
147:    function setApprovalForAll(address operator, bool approved) public override(IERC721, ERC721) {
155:    function exists(uint256 tokenId) external view returns (bool) {
159:    function getNamespace() external pure returns (string memory) {
163:    function getNamespaceHash() external pure returns (bytes32) {
168:    function getLocalName(uint256 tokenId) public view returns (string memory) {
177:    function getHandle(uint256 tokenId) public view returns (string memory) {
182:    function getTokenId(string memory localName) public pure returns (uint256) {
186:    function getTokenGuardianDisablingTimestamp(address wallet) external view returns (uint256) {
194:    function _mintHandle(address to, string calldata localName) internal returns (uint256) {
202:    function _validateLocalNameMigration(string memory localName) internal view {
226:    function _validateLocalName(string memory localName) internal view {
249:    function _isAlphaNumeric(bytes1 char) internal pure returns (bool) {
260:    function _beforeTokenTransfer(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ProxyAdmin.sol

```solidity
8:  contract ProxyAdmin is ControllableByContract {
12:    constructor(
21:    function currentImplementation() external returns (address) {
29:    function rollbackLastUpgrade() external onlyOwner {
33:    function proxy_changeAdmin(address newAdmin) external onlyOwner {
41:    function proxy_upgrade(address newImplementation) external onlyOwnerOrControllerContract {
47:    function proxy_upgradeAndCall(address newImplementation, bytes calldata data)
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/Governance.sol

```solidity
8:  interface ILensHub_V1 {
9:    function whitelistCollectModule(address collectModule, bool whitelist) external;
12:  contract Governance is ControllableByContract {
15:    constructor(address lensHubAddress_, address governanceOwner_) ControllableByContract(governanceOwner_) {
23:    function lensHub_setGovernance(address newGovernance) external onlyOwner {
27:    function lensHub_setEmergencyAdmin(address newEmergencyAdmin) external onlyOwner {
35:    function lensHub_whitelistProfileCreator(address profileCreator, bool whitelist)
42:    function lensHub_whitelistFollowModule(address followModule, bool whitelist)
49:    function lensHub_whitelistReferenceModule(address referenceModule, bool whitelist)
56:    function lensHub_whitelistActionModule(address actionModule, bool whitelist)
64:    function lensHub_whitelistCollectModule(address collectModule, bool whitelist)
73:    function executeAsGovernance(address target, bytes calldata data)

```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol

```solidity
7:  contract ControllableByContract is Ownable {
8:    event ControllerContractUpdated(address previousControllerContract, address newControllerContract);
14:    modifier onlyOwnerOrControllerContract() {
21:    constructor(address owner) Ownable() {
25:    function clearControllerContract() external onlyOwnerOrControllerContract {
30:    function setControllerContract(address newControllerContract) external onlyOwner {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol

```solidity
23:    constructor(
33:    function proxyCreateProfile(Types.CreateProfileParams calldata createProfileParams)
41:    function proxyCreateProfileWithHandle(Types.CreateProfileParams memory createProfileParams, string calldata handle)
64:    function proxyCreateHandle(address to, string calldata handle) external onlyOwner returns (uint256) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol

```solidity
27:    modifier onlyGov() {
94:    function _setGovernance(address newGovernance) internal {
101:    function _setTreasury(address newTreasury) internal {
108:    function _setTreasuryFee(uint16 newTreasuryFee) internal {
115:    function _whitelistCurrency(address currency, bool toWhitelist) internal {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol

```solidity
9:  contract LensV2UpgradeContract is ImmutableOwnable {
20:    constructor(
44:    function executeLensV2Upgrade() external onlyOwner {
64:    function _unwhitelistOldFollowModules() internal {
75:    function _unwhitelistOldReferenceModules() internal {
86:    function _unwhitelistOldCollectModules() internal {
97:    function _whitelistNewFollowModules() internal {
108:    function _whitelistNewReferenceModules() internal {
119:    function _whitelistNewActionModules() internal {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2Migration.sol

```solidity
11:  contract LensV2Migration {
19:    constructor(
33:    function batchMigrateProfiles(uint256[] calldata profileIds) external {
37:    function batchMigrateFollows(
45:    function batchMigrateFollowModules(uint256[] calldata profileIds) external {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol

```solidity
// @param missing
38:    constructor(address hub) {

69:    function tokenURI(uint256 tokenId) public view override returns (string memory) {
102:    function _getReceiver(
108:    function _beforeRoyaltiesSet(
116:    function _getRoyaltiesInBasisPointsSlot() internal pure override returns (uint256) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ImmutableOwnable.sol

```solidity
12:    modifier onlyOwner() {
19:    modifier onlyOwnerOrHub() {
26:    constructor(address owner, address lensHub) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/upgradeability/FollowNFTProxy.sol

```solidity
9:  contract FollowNFTProxy is Proxy {
13:    constructor(bytes memory data) {
18:    function _implementation() internal view override returns (address) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol

```solidity
26:  abstract contract LensProfiles is LensBaseERC721, ERC2981CollectionRoyalties, ILensProfiles {
33:    constructor(address moduleGlobals, uint256 tokenGuardianCooldown) {
38:    modifier whenNotPaused() {
45:    modifier onlyProfileOwner(address expectedOwner, uint256 profileId) {
50:    modifier onlyEOA() {

// @param missing
93:    function burn(uint256 tokenId)
112:    function approve(address to, uint256 tokenId) public override(LensBaseERC721, IERC721) {
120:    function setApprovalForAll(address operator, bool approved) public override(LensBaseERC721, IERC721) {
142:    function _hasTokenGuardianEnabled(address wallet) internal view returns (bool) {
149:    function _getRoyaltiesInBasisPointsSlot() internal pure override returns (uint256) {
153:    function _getReceiver(
159:    function _beforeRoyaltiesSet(
165:    function _beforeTokenTransfer(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensImplGetters.sol

```solidity
7:  contract LensImplGetters is ILensImplGetters {
11:    constructor(address followNFTImpl, address collectNFTImpl) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensHubEventHooks.sol

```solidity
10:  abstract contract LensHubEventHooks is ILensHubEventHooks {
25:    function emitCollectNFTTransferEvent(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol

```solidity
94:    function nonces(address signer) public view override returns (uint256) {
166:    function totalSupply() external view virtual override returns (uint256) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/HubRestricted.sol

```solidity
17:    modifier onlyHub() {
24:    constructor(address hub) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/ERC2981CollectionRoyalties.sol

```solidity
43:    function _setRoyalty(uint256 royaltiesInBasisPoints) internal virtual {
51:    function _getRoyaltyAmount(
58:    function _storeRoyaltiesInBasisPoints(uint256 royaltiesInBasisPoints) internal virtual {
65:    function _loadRoyaltiesInBasisPoints() internal view virtual returns (uint256) {
74:    function _beforeRoyaltiesSet(uint256 royaltiesInBasisPoints) internal view virtual;
76:    function _getRoyaltiesInBasisPointsSlot() internal view virtual returns (uint256);
78:    function _getReceiver(uint256 tokenId) internal view virtual returns (address);
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Types.sol

```solidity
333:    struct ProcessActionParams {
346:    struct ProcessCollectParams {
358:    struct ProcessCommentParams {
369:    struct ProcessQuoteParams {
380:    struct ProcessMirrorParams {
406:    struct ActionModuleWhitelistData {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ValidationLib.sol

```solidity
16:    function validatePointedPub(uint256 profileId, uint256 pubId) internal view {
24:    function validateAddressIsProfileOwner(address expectedProfileOwner, uint256 profileId) internal view {
30:    function validateAddressIsProfileOwnerOrDelegatedExecutor(
42:    function validateAddressIsDelegatedExecutor(address expectedDelegatedExecutor, uint256 delegatorProfileId)
51:    function validateReferenceModuleWhitelisted(address referenceModule) internal view {
57:    function validateFollowModuleWhitelisted(address followModule) internal view {
63:    function validateProfileCreatorWhitelisted(address profileCreator) internal view {
69:    function validateNotBlocked(uint256 profile, uint256 byProfile) internal view {
75:    function validateProfileExists(uint256 profileId) internal view {
81:    function validateCallerIsGovernance() internal view {
87:    function validateReferrersAndGetReferrersPubTypes(
121:    function validateLegacyCollectReferrer(
143:    function _validateReferrerAndGetReferrerPubType(
181:    function _validateReferrerAsPost(

```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/StorageLib.sol

```solidity
8:  library StorageLib {
49:    function getPublication(uint256 profileId, uint256 pubId)
63:    function getProfile(uint256 profileId) internal pure returns (Types.Profile storage _profiles) {
71:    function getDelegatedExecutorsConfig(uint256 delegatorProfileId)
83:    function tokenGuardianDisablingTimestamp()
93:    function getTokenData(uint256 tokenId) internal pure returns (Types.TokenData storage _tokenData) {
101:    function blockedStatus(uint256 blockerProfileId)
113:    function nonces() internal pure returns (mapping(address => uint256) storage _nonces) {
119:    function profileIdByHandleHash()
129:    function profileCreatorWhitelisted()
139:    function followModuleWhitelisted()
149:    function actionModuleWhitelistData()
159:    function actionModuleById() internal pure returns (mapping(uint256 => address) storage _actionModules) {
165:    function incrementMaxActionModuleIdUsed() internal returns (uint256) {
177:    function referenceModuleWhitelisted()
187:    function getGovernance() internal view returns (address _governance) {
193:    function setGovernance(address newGovernance) internal {
199:    function getEmergencyAdmin() internal view returns (address _emergencyAdmin) {
205:    function setEmergencyAdmin(address newEmergencyAdmin) internal {
211:    function getState() internal view returns (Types.ProtocolState _state) {
217:    function setState(Types.ProtocolState newState) internal {
223:    function getLastInitializedRevision() internal view returns (uint256 _lastInitializedRevision) {
229:    function setLastInitializedRevision(uint256 newLastInitializedRevision) internal {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/PublicationLib.sol

```solidity
// @params missing
22:    function post(Types.PostParams calldata postParams, address transactionExecutor) external returns (uint256) {
63:    function comment(Types.CommentParams calldata commentParams, address transactionExecutor)
103:    function mirror(Types.MirrorParams calldata mirrorParams, address transactionExecutor) external returns (uint256) {
140:    function quote(Types.QuoteParams calldata quoteParams, address transactionExecutor) external returns (uint256) {

170:    function getPublicationType(uint256 profileId, uint256 pubId) internal view returns (Types.PublicationType) {
190:    function getContentURI(uint256 profileId, uint256 pubId) external view returns (string memory) {
203:    function _asReferencePubParams(Types.QuoteParams calldata quoteParams)
214:    function _asReferencePubParams(Types.CommentParams calldata commentParams)
225:    function _createReferencePublication(
272:    function _fillReferencePublicationStorage(
291:    function _fillRootOfPublicationInStorage(
313:    function _processCommentIfNeeded(
366:    function _processQuoteIfNeeded(
419:    function _processMirrorIfNeeded(
472:    function _initPubActionModules(
521:    function _initPubReferenceModule(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol

```solidity
13:  library ProfileLib {
16:    function ownerOf(uint256 profileId) internal view returns (address) {
24:    function exists(uint256 profileId) internal view returns (bool) {
105:    function setProfileMetadataURI(uint256 profileId, string calldata metadataURI) external {
110:    function _initFollowModule(
120:    function _setProfileImageURI(uint256 profileId, string calldata imageURI) private {
128:    function setBlockStatus(
166:    function switchToNewFreshDelegatedExecutorsConfig(uint256 profileId) external {
180:    function changeDelegatedExecutorsConfig(
198:    function changeGivenDelegatedExecutorsConfig(
215:    function isExecutorApproved(uint256 delegatorProfileId, address delegatedExecutor) external view returns (bool) {
222:    function _changeDelegatedExecutorsConfig(
255:    function _prepareStorageToApplyChangesUnderGivenConfig(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol

```solidity
13:  interface ILegacyFeeFollowModule {
14:    struct ProfileData {
20:    function getProfileData(uint256 profileId) external view returns (ProfileData memory);
23:   library MigrationLib {
29:    event ProfileMigrated(uint256 profileId, address profileDestination, string handle, uint256 handleId);
94:    function batchMigrateFollows(
114:    function _migrateFollow(
141:    function batchMigrateFollowModules(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol

```solidity
43:    function validateSetProfileMetadataURISignature(
64:    function validateSetFollowModuleSignature(
87:    function validateChangeDelegatedExecutorsConfigSignature(
116:    function validateSetProfileImageURISignature(
137:    function validatePostSignature(Types.EIP712Signature calldata signature, Types.PostParams calldata postParams)
160:    function _hashActionModulesInitDatas(bytes[] memory actionModulesInitDatas) private pure returns (bytes32) {
173:    struct ReferenceParamsForAbiEncode {
190:    function _abiEncode(ReferenceParamsForAbiEncode memory referenceParamsForAbiEncode)
225:    function validateCommentSignature(
256:    function validateQuoteSignature(Types.EIP712Signature calldata signature, Types.QuoteParams calldata quoteParams)
286:    function validateMirrorSignature(Types.EIP712Signature calldata signature, Types.MirrorParams calldata mirrorParams)
309:    function validateBurnSignature(Types.EIP712Signature calldata signature, uint256 tokenId) external {
320:    function validateFollowSignature(
357:    function validateUnfollowSignature(
378:    function validateSetBlockStatusSignature(
401:    function validateLegacyCollectSignature(
425:    function validateActSignature(
450:    function calculateDomainSeparator() internal view returns (bytes32) {
469:    function _validateRecoveredAddress(bytes32 digest, Types.EIP712Signature calldata signature) private view {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/LegacyCollectLib.sol

```solidity
45:    function collect(
115:    function _getOrDeployCollectNFT(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol

```solidity
10:  library GovernanceLib {
64:    function _setState(Types.ProtocolState newState) private returns (Types.ProtocolState) {
71:    function whitelistProfileCreator(address profileCreator, bool whitelist) external {
76:    function whitelistFollowModule(address followModule, bool whitelist) external {
81:    function whitelistReferenceModule(address referenceModule, bool whitelist) external {
86:    function whitelistActionModule(address actionModule, bool whitelist) external {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/FollowLib.sol

```solidity
14:  library FollowLib {
15:    function follow(
54:    function unfollow(
98:    function _follow(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol

```solidity
12:  library ActionLib {
13:    function act(
65:    function _isActionEnabled(Types.Publication storage _publication, uint256 actionModuleId)
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol

```solidity
41:    event FollowApproval(uint256 indexed followerProfileId, uint256 indexed followTokenId);
43:    constructor(address hub) HubRestricted(hub) {
168:    function _wrap(uint256 followTokenId, address wrappedTokenReceiver) internal {
255:    function burn(uint256 followTokenId) public override {
263:    function supportsInterface(bytes4 interfaceId)
274:    function name() public view override returns (string memory) {
278:    function symbol() public view override returns (string memory) {
285:    function tokenURI(uint256 followTokenId) public view override returns (string memory) {
297:    function _followMintingNewToken(uint256 followerProfileId) internal returns (uint256) {
311:    function _followWithWrappedToken(
341:    function _followWithUnwrappedTokenFromBurnedProfile(
357:    function _followByRecoveringToken(uint256 followerProfileId, uint256 followTokenId) internal returns (uint256) {
368:    function _replaceFollower(
386:    function _baseFollow(
408:    function _unfollowIfHasFollower(uint256 followTokenId) internal {
416:    function _unfollow(uint256 unfollower, uint256 followTokenId) internal {
428:    function _approveFollow(uint256 approvedProfileId, uint256 followTokenId) internal {
436:    function _beforeTokenTransfer(
449:    function _getReceiver(
455:    function _beforeRoyaltiesSet(
463:    function _isFollowTokenWrapped(uint256 followTokenId) internal view returns (bool) {
467:    function _getRoyaltiesInBasisPointsSlot() internal pure override returns (uint256) {
480:    function tryMigrate(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol

```solidity
54:    modifier onlyProfileOwnerOrDelegatedExecutor(address expectedOwnerOrDelegatedExecutor, uint256 profileId) {
59:    modifier whenPublishingEnabled() {
66:    constructor(
165:    function changeDelegatedExecutorsConfig(
566:    function getActionModuleById(uint256 id) external view override returns (address) {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ValidationLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
16:    function validatePointedPub(uint256 profileId, uint256 pubId) internal view {
24:    function validateAddressIsProfileOwner(address expectedProfileOwner, uint256 profileId) internal view {
30:    function validateAddressIsProfileOwnerOrDelegatedExecutor(
42:    function validateAddressIsDelegatedExecutor(address expectedDelegatedExecutor, uint256 delegatorProfileId)
51:    function validateReferenceModuleWhitelisted(address referenceModule) internal view {
57:    function validateFollowModuleWhitelisted(address followModule) internal view {
63:    function validateProfileCreatorWhitelisted(address profileCreator) internal view {
69:    function validateNotBlocked(uint256 profile, uint256 byProfile) internal view {
75:    function validateProfileExists(uint256 profileId) internal view {
81:    function validateCallerIsGovernance() internal view {
87:    function validateReferrersAndGetReferrersPubTypes(
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/StorageLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
49:    function getPublication(uint256 profileId, uint256 pubId)
63:    function getProfile(uint256 profileId) internal pure returns (Types.Profile storage _profiles) {
71:    function getDelegatedExecutorsConfig(uint256 delegatorProfileId)
83:    function tokenGuardianDisablingTimestamp()
93:    function getTokenData(uint256 tokenId) internal pure returns (Types.TokenData storage _tokenData) {
101:    function blockedStatus(uint256 blockerProfileId)
113:    function nonces() internal pure returns (mapping(address => uint256) storage _nonces) {
119:    function profileIdByHandleHash()
129:    function profileCreatorWhitelisted()
139:    function followModuleWhitelisted()
149:    function actionModuleWhitelistData()
159:    function actionModuleById() internal pure returns (mapping(uint256 => address) storage _actionModules) {
165:    function incrementMaxActionModuleIdUsed() internal returns (uint256) {
177:    function referenceModuleWhitelisted()
187:    function getGovernance() internal view returns (address _governance) {
193:    function setGovernance(address newGovernance) internal {
199:    function getEmergencyAdmin() internal view returns (address _emergencyAdmin) {
205:    function setEmergencyAdmin(address newEmergencyAdmin) internal {
211:    function getState() internal view returns (Types.ProtocolState _state) {
217:    function setState(Types.ProtocolState newState) internal {
223:    function getLastInitializedRevision() internal view returns (uint256 _lastInitializedRevision) {
229:    function setLastInitializedRevision(uint256 newLastInitializedRevision) internal {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/PublicationLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
170:    function getPublicationType(uint256 profileId, uint256 pubId) internal view returns (Types.PublicationType) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
16:    function ownerOf(uint256 profileId) internal view returns (address) {
24:    function exists(uint256 profileId) internal view returns (bool) {
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
450:    function calculateDomainSeparator() internal view returns (bytes32) {
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ITokenHandleRegistry.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/IReferenceModule.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/IPublicationActionModule.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/IModuleGlobals.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensImplGetters.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensHubInitializable.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensHubEventHooks.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensHub.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensHandles.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensGovernable.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensERC721.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILegacyReferenceModule.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILegacyFollowModule.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILegacyCollectNFT.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILegacyCollectModule.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/IFollowNFT.sol

```solidiity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/IFollowModule.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/IERC721Timestamped.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/IERC721MetaTx.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/IERC721Burnable.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ICollectNFT.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ICollectModule.sol

```solidity
// lock to a single version removing the >= for a safer code.
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/constants/Types.sol

```solidity
// lock to a single version removing the >= for a safer code. 
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/constants/Events.sol

```solidity
// lock to a single version removing the >= for a safer code. 
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/constants/Errors.sol

```solidity
// lock to a single version removing the >= for a safer code. 
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/TokenHandleRegistry.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.18;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/namespaces/LensHandles.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.18;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ProxyAdmin.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/Governance.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ModuleGlobals.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2UpgradeContract.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.19;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LensV2Migration.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ImmutableOwnable.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/upgradeability/FollowNFTProxy.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensProfiles.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensImplGetters.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensHubStorage.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.18;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensHubEventHooks.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensGovernable.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/HubRestricted.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/ERC2981CollectionRoyalties.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Types.sol

```solidity
// lock to a single version removing the >= for a safer code. 
3:  pragma solidity >=0.6.0;
```


https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Typehash.sol

```solidity
// lock to a single version removing the >= for a safer code. 
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Events.sol

```solidity
// lock to a single version removing the >= for a safer code. 
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/constants/Errors.sol

```solidity
// lock to a single version removing the >= for a safer code. 
3:  pragma solidity >=0.6.0;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ValidationLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/StorageLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/PublicationLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ProfileLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MigrationLib.sol

```solidity
// lock to a single version removing the ^ for a safer code. also this version differs from the rest of the repo, consider sticking to a pattern
3:  pragma solidity ^0.8.19;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/MetaTxLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
2:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/LegacyCollectLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/GovernanceLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/FollowLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/libraries/ActionLib.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```

https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol

```solidity
// lock to a single version removing the ^ for a safer code.
3:  pragma solidity ^0.8.15;
```