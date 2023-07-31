# [L-01] Unbounded input for Profile MetadataURI (and Publication ContentURI)

Via `LensHub`, a user is able to set their profile's metadata URI. They do so by calling `LensHub::setProfileMetadataURI` which in turn executes `ProfileLib::setProfileMetadataURI`. The issue raised here is that there is no check on input size for `string calldata metadataURI`.

In contrast, a profile `imageURI` is bounded to 6000. (See: `ProfileLib::_setProfileImageURI`)

```
uint16 constant MAX_PROFILE_IMAGE_URI_LENGTH = 6000;
```

```
if (bytes(createProfileParams.imageURI).length > MAX_PROFILE_IMAGE_URI_LENGTH) {
    revert Errors.ProfileImageURILengthInvalid();
}
```

The same issue exists for `Publications` - There is no byte size limit on `contentURI`.

## Why is this a problem?

A malicious attacker could input a huge string as their MetadataURI (or contentURI). 

Whilst this action is costly to execute, it can cause function calls that read from `StorageLib.getProfile` &/or `StorageLib.getPublication` to become orders of magnitude more expensive to execute. 

With this in mind, a malicious actor could:

1. Disincentivize `unfollow` through a large Profile `metadataURI`

2. Potentially affect validation costs, especially when they rely on loops, such as `ValidationLib::validateReferrersAndGetReferrersPubTypes`

Additionally, it's possible to grief transaction relay services (via metaTx) by draining their gas budgets in few but expensive transactions.

## Recommendation

I recommend including a reasonable size constraint on both `metadataURI` and `contentURI` input parameters.

## Note

When setting a large metadataURI on a profile and then testing for gas costs via retrieval of a different variable in the same struct, the gas costs were very high:
```
function testForGasWhenRetrievingUnrelatedVariable() public {
   hub.getProfile(profileWithLargeMetadataURI).pubCount;
}
```
```
[PASS] testForGasWhenRetrievingUnrelatedVariable() (gas: 398244988)
```
However, gas uptick was not as drastic when testing other interactions such as acting on a post via a comment referral with a very large `contentURI`.
