https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L88

1. Validation of `createProfileParams`: The code  does not validate the `createProfileParams` passed to the `createProfile` function. It's important to validate the input parameters to ensure they meet the required constraints and prevent any potential misuse or invalid data. Without proper validation, it could lead to unexpected behavior or security vulnerabilities.

2. Minting the profile NFT before creating the profile: In the `createProfile` function, the code mints a new profile NFT to the specified recipient address before calling `ProfileLib.createProfile` to actually create the profile. This order of operations may not be ideal as it creates the NFT before the profile is fully created. It's generally recommended to create the profile first and then mint the corresponding NFT to ensure consistency and avoid potential issues where the profile creation fails after the NFT has already been minted.

3. https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L149

https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/LensHub.sol#L165-L166

In the `LensHub` contract there are two functions:

`ChangeDelegatedExecutorConfig` and
`ChangeDelegatedExecutorConfig`

The first function allows for updating the delegated executors' configuration and potentially performing a configuration switch.

The second function allows for updating the delegated executors' configuration but does not perform a configuration switch.
Both functions call the same underlying `_changeDelegatedExecutorsConfig` function, which handles the actual logic of updating the delegated executors' configuration.

Given that the second function only updates the list of delegated executors and their corresponding approvals without changing any other configuration or performing a switch, it would indeed be more appropriate to rename it to something like `updateDelegatedExecutors` to better reflect its purpose and functionality. Renaming the function to `updateDelegatedExecutors` would provide clarity and make the code more readable and understandable for other developers.

4. https://github.com/code-423n4/2023-07-lens/blob/cdef6ebc6266c44c7068bc1c4c04e12bf0d67ead/contracts/base/LensBaseERC721.sol#L302-L303

`_transfer` is called before checking of the reciving contract is ERC721 compatible. 

check if the recipient contract can handle the token transfer and reverts the transaction if the check fails. Only if the check passes, the `_transfer` function is called to perform the actual transfer of the token.

This approach ensures that the transfer is conditional on the recipient contract's capabilities, reducing the risk of potential issues or unintended consequences