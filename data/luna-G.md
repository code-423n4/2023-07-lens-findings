
Gas Optimizations Report
========================

Table of contents
=================

* [Gas Findings](#gas-findings)
	* [[Gas-1] Do not cache msg.sender](#gas-1-do-not-cache-msgsender)
	* [[Gas-2] Use != 0 instead of > 0](#gas-2-use--0-instead-of--0)
	* [[Gas-3] Rearrange state variables](#gas-3-rearrange-state-variables)
	* [[Gas-4] Unused state variables can be removed to save gas](#gas-4-unused-state-variables-can-be-removed-to-save-gas)
	* [[Gas-5] Change transferFrom to transfer](#gas-5-change-transferFrom-to-transfer)
    * [[Gas-6] State variables that could be set immutable](#gas-6-state-variables-that-could-be-set-immutable)

# Gas Findings

## [Gas-1] Do not cache msg.sender
We recommend not to cache msg.sender since calling it is 2 gas while reading a variable is more.

        FollowNFTProxy.solL#14


## [Gas-2] Use != 0 instead of > 0
Using != 0 is slightly cheaper than > 0. We recommend to replace > with != in the following places:

        ModuleGlobals.solL#109
        BaseFeeCollectModule.solL#248
        LensHandles.solL#206
        LensHandles.solL#230


## [Gas-3] Rearrange state variables
You can change the order of the storage variables to decrease memory uses. This is because the new suggested new orders will use less storage slots.

        LensSeaDropCollection.sol: 
old number of slots: 4 
new number of slots: 3 
the new order of types is
        1. IModuleGlobals
        2. address
        3. uint16
        4. address


## [Gas-4] Unused state variables can be removed to save gas
Unused state variables are gas consuming at deployment (since they are located in storage) and are a bad code practice. Removing those variables will decrease deployment gas cost and improve code quality.

        LensSeaDropCollection.sol: ROYALTIES_BPS


## [Gas-5] Change transferFrom to transfer
transferFrom(address(this), *, **)' could be replaced by the following more gas efficient 'transfer(*, **)'. This replacement is more gas efficient and improves the code quality.

        ProfileCreationProxy.solL#58
        ProfileCreationProxy.solL#59


## [Gas-6] State variables that could be set immutable
You can set the following state variables to immutable and save gas:

        LensBaseERC721.sol: _totalSupply 
        LensHubStorage.sol: _lastInitializedRevision
        LensHubStorage.sol: _state
        LensHubStorage.sol: _profileCounter
        LensHubStorage.sol: _governance