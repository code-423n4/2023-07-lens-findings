
### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] NFT contract redefines mint()/safeMint(), but not both | [NFT contract redefines mint()/safeMint(), but not both](#nft-contract-redefines-mintsafemint-but-not-both) | 1 |
|[L-2] Reentrancy vulnerabilities | [Reentrancy vulnerabilities](#reentrancy-vulnerabilities) | 2 |
|[L-3] Use of transferFrom() rather than safeTransferFrom() for NFTs | [Use of transferFrom() rather than safeTransferFrom() for NFTs](#use-of-transferfrom-rather-than-safetransferfrom-for-nfts) | 2 |

Total: 3 issues

### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-1] Unused return | [Unused return](#unused-return) | 2 |
|[N-2] Interfaces Should Be Defined in Separate Files From Their Usage | [Interfaces Should Be Defined in Separate Files From Their Usage](#interfaces-should-be-defined-in-separate-files-from-their-usage) | 1 |
|[N-3] Redundant Inheritance | [Redundant Inheritance](#redundant-inheritance) | 1 |
|[N-4] Event Emission Preceding External Calls: A Best Practice | [Event Emission Preceding External Calls: A Best Practice](#event-emission-preceding-external-calls-a-best-practice) | 1 |
|[N-5] Setter No Initial Value Check Detection | [Setter No Initial Value Check Detection](#setter-no-initial-value-check-detection) | 1 |
|[N-6] Variable names too similar | [Variable names too similar](#variable-names-too-similar) | 12 |
|[N-7] Missing Underscore Prefix on Large Number Literals | [Missing Underscore Prefix on Large Number Literals](#missing-underscore-prefix-on-large-number-literals) | 2 |
Total: 7 issues

#


## NFT contract redefines mint()/safeMint(), but not both
- Severity: Low
- Confidence: High

### Description
ERC721 contracts should have both _mint() and _safeMint() consistently redefined for spec compatibility. The _mint() variant is supposed to skip onERC721Received() checks, whereas _safeMint() does not. If one of these functions is re-implemented or has new arguments, the other should as well. 

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/base/LensBaseERC721.sol
```
 
Line: 30          abstract contract LensBaseERC721 is ERC165, ILensERC721 
```
Redefine _mint() without redefine _safeMint().
[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L30-L511](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L30-L511)


</details>

# 


## Use of transferFrom() rather than safeTransferFrom() for NFTs
- Severity: Low
- Confidence: High

### Description

Use of `transferFrom()` rather than `safeTransferFrom()` for NFTs in will lead to the loss of NFTs
The EIP-721 standard says the following about `transferFrom()`:
```solidity
    /// @notice Transfer ownership of an NFT -- THE CALLER IS RESPONSIBLE
    ///  TO CONFIRM THAT `_to` IS CAPABLE OF RECEIVING NFTS OR ELSE
    ///  THEY MAY BE PERMANENTLY LOST
    /// @dev Throws unless `msg.sender` is the current owner, an authorized
    ///  operator, or the approved address for this NFT. Throws if `_from` is
    ///  not the current owner. Throws if `_to` is the zero address. Throws if
    ///  `_tokenId` is not a valid NFT.
    /// @param _from The current owner of the NFT
    /// @param _to The new owner
    /// @param _tokenId The NFT to transfer
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
```
https://github.com/ethereum/EIPs/blob/78e2c297611f5e92b6a5112819ab71f74041ff25/EIPS/eip-721.md?plain=1#L103-L113
Code must use the `safeTransferFrom()` flavor if it hasn't otherwise verified that the receiving address can handle it

    

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/misc/ProfileCreationProxy.sol
```
 
Line: 58          LENS_HANDLES.transferFrom(address(this), destination, handleId)
```

[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L58)


- File: contracts/misc/ProfileCreationProxy.sol
```
 
Line: 59          ILensHub(LENS_HUB).transferFrom(address(this), destination, profileId)
```

[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L59](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/ProfileCreationProxy.sol#L59)


</details>

# 


## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Only report reentrancy that acts as a double call (see `reentrancy-eth`, `reentrancy-no-eth`).

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- Reentrancy in File: contracts/FollowNFT.sol
```
 
Line: 368          function _replaceFollower(
        uint256 currentFollowerProfileId,
        uint256 newFollowerProfileId,
        uint256 followTokenId
    ) internal 
```
:
	External calls:
	- File: contracts/FollowNFT.sol
```
 
Line: 376          ILensHub(HUB).emitUnfollowedEvent(currentFollowerProfileId, _followedProfileId)
```

	State variables written after the call(s):
	- File: contracts/FollowNFT.sol
```
 
Line: 383          _baseFollow({followerProfileId: newFollowerProfileId, followTokenId: followTokenId, isOriginalFollow: false})
```

		- File: contracts/FollowNFT.sol
```
 
Line: 392          _followDataByFollowTokenId[followTokenId].followerProfileId = uint160(followerProfileId)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 393          _followDataByFollowTokenId[followTokenId].followTimestamp = uint48(block.timestamp)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 394          delete _followDataByFollowTokenId[followTokenId].profileIdAllowedToRecover
```

		- File: contracts/FollowNFT.sol
```
 
Line: 396          _followDataByFollowTokenId[followTokenId].originalFollowTimestamp = uint48(block.timestamp)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 403          _followDataByFollowTokenId[followTokenId].originalFollowTimestamp = mintTimestamp
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L368-L384](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L368-L384)


- Reentrancy in File: contracts/FollowNFT.sol
```
 
Line: 255          function burn(uint256 followTokenId) public override 
```
:
	External calls:
	- File: contracts/FollowNFT.sol
```
 
Line: 256          _unfollowIfHasFollower(followTokenId)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 412          ILensHub(HUB).emitUnfollowedEvent(followerProfileId, _followedProfileId)
```

	State variables written after the call(s):
	- File: contracts/FollowNFT.sol
```
 
Line: 257          super.burn(followTokenId)
```

		- File: contracts/base/LensBaseERC721.sol
```
 
Line: 389          --_balances[owner]
```

	- File: contracts/FollowNFT.sol
```
 
Line: 257          super.burn(followTokenId)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 429          _followApprovalByFollowTokenId[followTokenId] = approvedProfileId
```

	- File: contracts/FollowNFT.sol
```
 
Line: 257          super.burn(followTokenId)
```

		- File: contracts/base/LensBaseERC721.sol
```
 
Line: 440          _tokenApprovals[tokenId] = to
```

	- File: contracts/FollowNFT.sol
```
 
Line: 257          super.burn(followTokenId)
```

		- File: contracts/base/LensBaseERC721.sol
```
 
Line: 392          delete _tokenData[tokenId]
```

	- File: contracts/FollowNFT.sol
```
 
Line: 257          super.burn(followTokenId)
```

		- File: contracts/base/LensBaseERC721.sol
```
 
Line: 390          --_totalSupply
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L255-L258](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L255-L258)


</details>

# 


## Interfaces Should Be Defined in Separate Files From Their Usage
- Severity: Non-Critical
- Confidence: High

### Description
Interfaces should be defined in separate files, so that it's easier for future projects to import them, and to avoid duplication later on if they need to be used elsewhere in the project.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/misc/access/Governance.sol
```
 
Line: 8          interface ILensHub_V1 
```

[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/Governance.sol#L8-L10](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/Governance.sol#L8-L10)


</details>

# 




## Unused return
- Severity: Non-Critical
- Confidence: Medium

### Description
The return value of an external call is not stored in a local or state variable.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/base/LensBaseERC721.sol
```
 
Line: 476          try IERC721Receiver(to).onERC721Received(msg.sender, from, tokenId, _data) returns (bytes4 retval) {
                return retval == IERC721Receiver.onERC721Received.selector;
            } catch (bytes memory reason) {
                if (reason.length == 0) {
                    revert Errors.NonERC721ReceiverImplementer();
                } else {
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }
            }
```
 ignores return value 

[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L476-L486](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/LensBaseERC721.sol#L476-L486)


- File: contracts/base/upgradeability/FollowNFTProxy.sol
```
 
Line: 15          ILensHub(msg.sender).getFollowNFTImpl().functionDelegateCall(data)
```
 ignores return value 

[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/upgradeability/FollowNFTProxy.sol#L15](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/base/upgradeability/FollowNFTProxy.sol#L15)


</details>

# 


## Missing Underscore Prefix on Large Number Literals
- Severity: Non-Critical
- Confidence: High

### Description
In Solidity, it is common to work with large number literals to represent values such as timestamps, amounts, or other numerical data. However, when dealing with large numbers, readability can be improved by explicitly indicating the magnitude of the value.

By prefixing large number literals with an underscore, it provides a visual indication to developers and auditors that the value is intentionally large and not a typographical error. It helps prevent misinterpretation or confusion regarding the intended value and promotes code comprehension.

For example, instead of writing uint256 balance = 1000000000000000000;, the convention suggests writing uint256 balance = 1_000_000_000_000_000_000;. The underscores act as separators and improve readability without affecting the actual value.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/FollowNFT.sol
```
 
Line: 55          _setRoyalty(1000)
```
 use `_` for 1000

[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L55](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L55)



- File: contracts/misc/LegacyCollectNFT.sol
```
 
Line: 48          _setRoyalty(1000)
```
 use `_` for 1000

[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol#L48](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/LegacyCollectNFT.sol#L48)



</details>

# 

## Redundant Inheritance
- Severity: Non-Critical
- Confidence: High

### Description
Detects when a contract inherits from another contract that it already indirectly inherits from.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/interfaces/ILensERC721.sol
```
 
Line: 11          interface ILensERC721 is IERC721, IERC721Timestamped, IERC721Burnable, IERC721MetaTx, IERC721Metadata 
```
 The contract `ILensERC721` has redundant inheritance: `IERC721` is already inherited indirectly.

[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensERC721.sol#L11](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensERC721.sol#L11)


</details>

# 


## Setter No Initial Value Check Detection
- Severity: Non-Critical
- Confidence: Medium

### Description
This detector flags setter functions that lack an initial value check. Lack of such a check can lead to assigning wrong values to the variable, causing unexpected contract behavior.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/misc/access/ControllableByContract.sol
```
 
Line: 30          function setControllerContract(address newControllerContract) external onlyOwner 
```
 Setter lacks an initial value check
[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L30-L33](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/misc/access/ControllableByContract.sol#L30-L33)


</details>

# 


## Variable names too similar
- Severity: Non-Critical
- Confidence: Medium

### Description
Detect variables with names that are too similar.

<details>

<summary>
There are 12 instances of this issue:

</summary>

###
- Variable File: contracts/LensHub.sol
```
 
Line: 492          uint256 followedProfileId
```
 is too similar to File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 348          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492)


- Variable File: contracts/LensHub.sol
```
 
Line: 492          uint256 followedProfileId
```
 is too similar to File: contracts/LensHub.sol
```
 
Line: 344          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492)


- Variable File: contracts/LensHub.sol
```
 
Line: 492          uint256 followedProfileId
```
 is too similar to File: contracts/LensHub.sol
```
 
Line: 492          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492)


- Variable File: contracts/LensHub.sol
```
 
Line: 492          uint256 followedProfileId
```
 is too similar to File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 241          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492)


- Variable File: contracts/LensHub.sol
```
 
Line: 492          uint256 followedProfileId
```
 is too similar to File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 231          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492)


- Variable File: contracts/LensHub.sol
```
 
Line: 492          uint256 followedProfileId
```
 is too similar to File: contracts/LensHub.sol
```
 
Line: 321          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/LensHub.sol#L492)


- Variable File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 348          uint256 followedProfileId
```
 is too similar to File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 348          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348)


- Variable File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 348          uint256 followedProfileId
```
 is too similar to File: contracts/LensHub.sol
```
 
Line: 344          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348)


- Variable File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 348          uint256 followedProfileId
```
 is too similar to File: contracts/LensHub.sol
```
 
Line: 492          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348)


- Variable File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 348          uint256 followedProfileId
```
 is too similar to File: contracts/LensHub.sol
```
 
Line: 321          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348)


- Variable File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 348          uint256 followedProfileId
```
 is too similar to File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 241          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348)


- Variable File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 348          uint256 followedProfileId
```
 is too similar to File: contracts/interfaces/ILensProtocol.sol
```
 
Line: 231          uint256 followerProfileId
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/interfaces/ILensProtocol.sol#L348)


</details>

# 


## Event Emission Preceding External Calls: A Best Practice
- Severity: Non-Critical
- Confidence: Medium

### Description

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls


<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- Reentrancy in File: contracts/FollowNFT.sol
```
 
Line: 255          function burn(uint256 followTokenId) public override 
```
:
	External calls:
	- File: contracts/FollowNFT.sol
```
 
Line: 256          _unfollowIfHasFollower(followTokenId)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 412          ILensHub(HUB).emitUnfollowedEvent(followerProfileId, _followedProfileId)
```

	Event emitted after the call(s):
	- File: contracts/base/LensBaseERC721.sol
```
 
Line: 441          emit Approval(ownerOf(tokenId), to, tokenId)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 257          super.burn(followTokenId)
```

	- File: contracts/FollowNFT.sol
```
 
Line: 430          emit FollowApproval(approvedProfileId, followTokenId)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 257          super.burn(followTokenId)
```

	- File: contracts/base/LensBaseERC721.sol
```
 
Line: 394          emit Transfer(owner, address(0), tokenId)
```

		- File: contracts/FollowNFT.sol
```
 
Line: 257          super.burn(followTokenId)
```


[https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L255-L258](https://github.com/code-423n4/2023-07-lens/blob/main/contracts/FollowNFT.sol#L255-L258)


</details>

# 
