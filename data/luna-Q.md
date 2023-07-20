
QA Report
=========

Table of content
================

* [QA Findings](#qa-findings)
	* [Multiplication instead division in compares](#multiplication-instead-division-in-compares)
	* [open todos](#open-todos)
	* [Missing two steps verification process](#missing-two-steps-verification-process)

# QA Findings

## Multiplication instead division in compares
Division causes ceiling and therefore loss of precision. In the other hand multiplication does not. In the following situations you can rearange the equation to use multiplication instead of division.

- ModuleGlobals.solL#109


## open todos
Open TODOs can point to architecture or programming issues that still need to be resolved.

- LensSeaDropCollection.solL#21
- LensHandles.solL#166
- LensHandles.solL#175


## Missing two steps verification process
The process of transferring ownership is dangerous since typing the wrong address can lead to severe implications. It is better to have to steps verification process with set and claim functions to decrease the chances of human error.

- Governance.sol
- ProxyAdmin.sol

