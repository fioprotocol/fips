# FIO Improvements Proposals (FIPs)
FIPs describe proposed changes to the FIO Protocol.

## Contributing
### Review FIPs
Everyone is encouraged to review existing FIPs and provide feedback.
### Proposing a FIP
Fork the repository and add your FIP to it with status *Draft*. See *Successful FIP includes* below for what a FIP should include. Reach out to the community and solicit feedback. Once you are ready submit a pull request to add your FIP to the main fips repo. Once repo custodians accepts your FIP, you can code the solution and submit pull request to relevant repo. Once the code has been pushed, the status will be changed to *Adopted*.

## FIP Type
* Core - improvements requiring a consensus fork
* Functionality - adds or modifies functionality without need for consensus fork
* RFC - Describes a design issue, or provides general guidelines or information to the FIO community, but does not propose a new feature.

## FIP Status
* Draft - a FIP that is open for consideration.
* Accepted - a FIP that is planned for immediate adoption.
* Final - a FIP that has been adopted.
* Deferred - a FIP that is not being considered for immediate adoption. May be reconsidered in the future.

## Successful FIP includes
### Preamble
Each EIP must begin with an RFC 822 style header preamble, preceded and followed by three hyphens (---). This header is also termed “front matter” by Jekyll. The headers must appear in the following order. All other headers are required.
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
Initially this section should include technical implementation strategy, such how this change made (contracts, core, etc.), how will the specification be accomplished in code, how will the code be tested/deployed. Eventually it should reference a specific pull-request.

### Backwards Compatibility
All FIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The FIP must explain how the author proposes to deal with these incompatibilities.

### Future considerations
This section may include proposals for how the new functionality could be enhanced in the future.
