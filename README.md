# FIO Improvements Proposals (FIPs)
FIPs describe proposed changes to the FIO Protocol.

## FIPs
|FIP|Title|Status|
|---|---|---|
|[FIP-1](fip-0001.md)|FIO Domain/FIO Address transfer and data purge|Final|
|[FIP-2](fip-0002.md)|Improvements to paging via API|Final|
|[FIP-3](fip-0003.md)|Provide ability to cancel a request for funds|Final|
|[FIP-4](fip-0004.md)|Provide ability to remove pub address from the FIO protocol for a user|Final|
|[FIP-5](fip-0005.md)|Enhanced privacy|Draft|
|[FIP-6](fip-0006.md)|Transfer locked tokens|Final|
|[FIP-7](fip-0007.md)|Provide ability to burn FIO Address|Final|
|[FIP-8](fip-0008.md)|Public Address Request|Draft|
|[FIP-9](fip-0009.md)|Allow voting and proxying without a FIO Address|Final|
|[FIP-10](fip-0010.md)|Redesign Fee Computations|Final|
|[FIP-11](fip-0011.md)|Add ability to purchase bundled transactions|Final|
|[FIP-12](fip-0012.md)|Move action whitelisting into state|Final|
|[FIP-13](fip-0013.md)|Ability to retrive all public addresses for a FIO Address|Final|
|[FIP-14](fip-0014.md)|Ensuring API response data integrity|Draft|
|[FIP-15](fip-0015.md)|Chain and token code standard|Final|
|[FIP-16](fip-0016.md)|CLIO Enhancements|Final|
|[FIP-17a](fip-0017a.md)|FIO Token Wrapping|Accepted|
|[FIP-17b](fip-0017b.md)|FIO Domain Wrapping|Accepted|
|[FIP-18](fip-0018.md)|Chain-level public address|Final|
|[FIP-19](fip-0019.md)|Add ability to retrieve all received FIO Requests|Final|
|[FIP-20](fip-0020.md)|FIO Co-op|Deferred|
|[FIP-21](fip-0021.md)|FIO Staking|Accepted|
|[FIP-22](fip-0022.md)|Retire FIO Tokens|Accepted|
|[FIP-23](fip-0023.md)|Temporary adjustment of Block Producers Reserves|Final|
|[FIP-24](fip-0024.md)|Secure messsage standard|Draft|
|[FIP-25](fip-0025.md)|Return bundle transaction count in get_fio_names|Final|
|[FIP-26](fip-0026.md)|FIO Domain Marketplace|Accepted|
|[FIP-27](fip-0027.md)|NFT Signatures|Final|
|[FIP-28](fip-0028.md)|Delay FIO Address burning after expiration|Final|
|[FIP-29](fip-0029.md)|Temporarily disable Transfer of locked tokens|Deferred|
|[FIP-30](fip-0030.md)|pNetwork Support|Accepted|
|[FIP-31](fip-0031.md)|Eliminate FIO Address Expiration|Final|
|[FIP-32](fip-0032.md)|Allow unlimited size of content parameter in New Funds Request|Accepted|
|[FIP-33](fip-0033.md)|Allow $ in token codes|Accepted|

## Contributing
### Review FIPs
Everyone is encouraged to review existing FIPs and provide feedback.
### New FIP Process
* If you're not yet sure if your idea should a FIP, start by opening a Github issue and create your FIP draft there first to ask for feedback from the community.
* To propose a FIP, fork this repository and create a pull request for your FIP with status *Draft*. The FIP number can remain undetermined at this stage. Use the [fip-TEMPLATE.md](fip-TEMPLATE.md) file as your starting point and see [Successful FIP includes](fips#successful-fip-includes) for what a FIP should contain. Also, use existing FIPs as examples.
* Repo custodians will review the FIP PR and comment. Once comments have been addressed, the FIP will be merged to Master with the status *Draft*.
* Reach out to the community and solicit feedback on your FIP.
* Continue to update your FIP based on community feedback and submit pull requests as needed. Once you feel the community has contributed and the FIP is ready, creare a pull request to update the status to *Accepted*.
* Once repo custodians accept your FIP, you can code your solution and submit pull requests to the relevant repo.
* After the code has been merged and deployed to Mainnet, the status will be changed to *Final*.
### Discussing a FIP
Create an issue describing the FIP or section of a FIP you would like to open up for discussion. Assign it to the author of that FIP.

## FIP Type
* Core - improvements requiring a consensus fork
* Functionality - adds or modifies functionality without need for consensus fork
* Standard - documents standards used in FIO Protocol

## FIP Status
* Draft - a FIP that is open for consideration.
* Accepted - a FIP that is planned for immediate adoption. Changes may still be made as required by development.
* Final - a FIP that has been adopted, coded, merged, and deployed by the block producers on FIO mainnet.
* Deferred - a FIP that is not being considered for immediate adoption. May be reconsidered in the future.

## Successful FIP includes
### Preamble
Each FIP must begin with an RFC 822 style header preamble, preceded and followed by three hyphens (---). This header is also termed “front matter” by Jekyll. The headers must appear in the following order. All other headers are required.
```
---
fip: FIP number
title: FIP title
status: FIP status
type: FIP type
author: a list of the author’s or authors’ name(s) and/or username(s), or name(s) and email(s)
created: FIP create date
updated: FIP update date
---
```
### Abstract
A short (~200 word) description of the technical issue being addressed.

### Motivation
An explanation of why the existing protocol specification is inadequate to address the problem that the FIP solves.

### Specification
Detailed definition of what is being changed, e.g. actions, API end-points, processing logic, exception handling, fees, etc.

### Rationale
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made.

### Implementation
Initially this section should include technical implementation strategy, such how this change made (contracts, core, etc.), how will the specification be accomplished in code, how will the code be tested/deployed. Before development beings, create a master ticket in the fio repo to track progress on development towards this FIP.

### Backwards Compatibility
All FIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The FIP must explain how the author proposes to deal with these incompatibilities.

### Future considerations
This section may include proposals for how the new functionality could be enhanced in the future.

### Discussion link
Provide a link(s) to where this FIP is being discussed.
