## Any comments for the judge to contextualize your findings
Some bugs were discovered in the follows migration logic.

## Approach taken in evaluating the codebase
- I checked lenster.xyz to gain a high-level understanding of the project.
- I conducted a manual review of the code within the scope of the audit.
- I studied the Lens v1 code to better understand the implementation.
- I asked questions to the project sponsor to clarify any uncertainties.
- I wrote proof-of-concepts (POCs) for the bugs I discovered during the review.
- I ensured that the bugs identified in the previous audit did not resurface.

## Architecture recommendations
- If UserA blocks UserB, the system should automatically block UserA by UserB as well, preventing UserA from following UserB.
- Referrers should not be an input controlled by the actors because they could enter profiles they control as referrers to earn referral fees.
- Burned profiles should not hold any followNFTs

## Codebase quality analysis
Despite being a unique project with a focus on solving on-chain social network challenges, the codebase is well-organized and easy to understand. Additionally, it includes well-written Foundry tests. Moreover, the codebase is not vulnerable to common attack vectors.

## Centralization risks
- Whitelisted profile creators can create many unused profiles 

## Mechanism review
-

## Systemic risks
Contracts that were Out Of Scope in this audit should be audited before launch.

### Time spent:
35 hours