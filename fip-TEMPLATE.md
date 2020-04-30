---
fip: FIP number (leave as X until ready to merge to master)
title: FIP title
status: FIP status
type: FIP type
author: a list of the author’s or authors’ name(s) and/or username(s), or name(s) and email(s)
created: FIP create date
updated: FIP update date
---

## Abstract
A short (~200 word) description of the technical issue being addressed.
* List out specific relevant points 

Proposed new actions:
|Action|Endpoint|Description|
|---|---|---|
|newaction|new_endpoint|Example Description|
|newaction2|new_endpoint2|Example Description 2|
||new_endpoint3|Example Description 3|

## Terminology
* **terms** - define them here
* **more terms** - define them here

## Motivation
An explanation of why the existing protocol specification is inadequate to address the problem that the FIP solves.
* Include relevant points
  * And subpoints if needed

## Specification
Detailed definition of what is being changed, e.g. actions, API end-points, processing logic, exception handling, fees, etc.

Example:

### Define Core Concept Here

Descriptions, diagrams, etc.

#### Major Concept Example
More description about Major Concept
##### New action: *newaction*
##### New endpoint: /new_endpoint
##### New fee: priv_newaction, bundle-eligible (uses 1 bundled transaction)
##### Request

(examples below, replace with your own)

|Parameter|Required|Format|Definition|
|---|---|---|---|
|some_param|Yes|String|Human readable description here|
|another_param|Yes|FIO public key|More descriptions|

###### *content* format
The content element is a packed and encrypted version of the following data structure:
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|String|FIO Address of the party being added.|
|public_key|Yes|IO Public Key|FIO Public Key of the party being added.|
###### Example
```
{
	"fio_public_key_hash": "515184318471884685485465454464846864686484464694181384",
	"content": "...",
	"max_fee": 0,
	"tpid": "",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling. Explicitly allowed:
	* User does not need to have a FIO Address registered, as Friend List on a Public Key level
* Fee is collected
* Content is placed on chain
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Key in Friend List|Supplied FIO Publick key is already in Friend List|400|"fio_public_key_hash"|Value sent in, i.e. "1000000000"|"FIO public key already in Friend List"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if successful|
|fee_collected|Int|Amount of SUFs collected as fee|
###### Example
```
{
	"status": "OK",
	"fee_collected": 2000000000
}
```

## Rationale
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made.

## Implementation
Initially this section should include technical implementation strategy, such how this change made (contracts, core, etc.), how will the specification be accomplished in code, how will the code be tested/deployed. Before development beings, create a master ticket in the fio repo to track progress on development towards this FIP.

## Backwards Compatibility
All FIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The FIP must explain how the author proposes to deal with these incompatibilities.

## Future considerations
This section may include proposals for how the new functionality could be enhanced in the future.
