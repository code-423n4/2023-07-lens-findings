## Comments
I spent a lot of time on this codebase, I think perhaps the most intensive and complete audit I have done to date. I have indicated 24 hours, but I think it may be more.
This codebase is very well written. The developers have put in a lot of effort to ensure safe and sound code practices.

## Approach taken
- Many hours of manual reading of the codebase
- Checking access control
- Checking logic
- Checking array length checks
- Checking variable setting and usage
- Checking difference between proxy and implementation initialize.
- Checking if statements and require statements for correct usage of  "||" and  "&&" operators
- Looking up suspected issues in Solodit to verify if they were indeed issues.
- Comparing V2 changes with the V1 github codebase
- Found one or two changes in V1 repo that were not in the repo that was audited in the previous V1 contest
- Reading all documentation mentioned 
- Reading of known-findings
- Reading of previous audit report
- Ran mainnet and testnet fork tests for migration of profiles and tried a few changes to test
- Ran other tests and created log outputs to check values were correct
- Compared V1 LensHub to current LensHub using a local diff
- Compared storage slots of LensHub with LensHub V1
- Compared storage slots of the Profile contracts

## Architecture
From what I could see all architecture choices have been made for valid reasons and are well thought out.

## Codebase Quality
Outstanding!

## Centralization risk
As mentioned in the previous audit comments and in the contest documentation, Centralization risk is there but not to such a degree as to cause any concern.

## Systemic Risk
I think the biggest risk right now is the upgrade. Although it has been taken into account, there still may be unforeseen issues, as I have submitted in one report I think the abitltity to rollback quickly is highly recommended so as to restore any functionality as soon as possible.

### Time spent:
24 hours