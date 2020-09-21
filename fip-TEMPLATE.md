---
fip: FIP number (leave as X until ready to merge to master)
title: FIP title
status: FIP status
type: FIP type
author: a list of the author’s or authors’ name(s) and/or username(s), or name(s) and email(s)
created: FIP create date
updated: FIP update date
---

# Abstract
A short (~200 word) description of the technical issue being addressed.
* List out specific relevant points 

Proposed new actions:
|Action|Endpoint|Description|
|---|---|---|
|newaction|new_endpoint|Example Description|
|newaction2|new_endpoint2|Example Description 2|
||new_endpoint3|Example Description 3|

# Terminology
* **terms** - define them here
* **more terms** - define them here

# Motivation
An explanation of why the existing protocol specification is inadequate to address the problem that the FIP solves.
* Include relevant points
  * And subpoints if needed

# Specification
Detailed definition of what is being changed, e.g. actions, API end-points, processing logic, exception handling, fees, etc.
## Define Core Concept Here
Overview, descriptions, diagrams, etc.
## New actions
### Example action
Define propossed new action.
#### New action: *newaction*
#### New endpoint: /new_endpoint
#### New fee: new_endpoint, bundle-eligible (uses 1 bundled transaction)
#### RAM increase: define if this actrrion needs to increase users RAM and if so by how much. See [Resource Management](https://developers.fioprotocol.io/fio-protocol/resource-management) for more details.
#### Request
Describe what parameters are passed in. Example:
|Parameter|Required|Format|Definition|
|---|---|---|---|
|some_param|Yes|String|Human readable description here|
|another_param|Yes|FIO public key|More descriptions|
|complex_parameter|Yes|Encrypted blob|See complex_parameter below|
##### *complex_parameter* format
Use this convention for describing parameters which can have multiple nested parameters, e.g. a JSON array, or encrypted blob.
|Parameter|Required|Format|Definition|
|---|---|---|---|
|some_nested_param|Yes|String|Human readable description here|
|another_nested_param|Yes|FIO public key|More descriptions|
##### Example
```
{
	"some_param": "515184318471884685485465454464846864686484464694181384",
	"another_param": 0,
	"complex_parameter": "JhTnxX9ntI9n1eucNuJzHS1/JXeLj+GYmPD1uXG/5PBixQeHg40d4p4yHCm6fxfn7eKzcY"
}
```
#### Processing
Describe what exactly is happening once request is being processed. Example:
* Request is validated per Exception handling.
* Fee is collected
* Content is placed on chain
#### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Name the error condition, so it can be referenced|Describe what triggers the error|What is the HTTP error code which should be returned. Follow [400](https://developers.fioprotocol.io/api/api-spec/models/error-400), [403](https://developers.fioprotocol.io/api/api-spec/models/error-403), [404](https://developers.fioprotocol.io/api/api-spec/models/error-404) conventions.|For 400 type only, specify which field triggered this error.|For 400 type only, specify which value triggered this error.|Provide descriptive error message.|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
#### Response
Describe what parameters are returned. Example:
|Parameter|Format|Definition|
|---|---|---|
|name of parameter|Format of parameter|Description of what the parameter means.|
|status|String|OK if successful|
|fee_collected|Int|Amount of SUFs collected as fee|
##### Example
```
{
	"status": "OK",
	"fee_collected": 2000000000
}
```
### Changes to existing actions
If there are any changes to existing actions or endpoints define them here in the same format as new actrions.

# Rationale
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made.

# Implementation
Initially this section should include technical implementation strategy, such how this change made (contracts, core, etc.), how will the specification be accomplished in code, how will the code be tested/deployed. Before development beings, create a master ticket in the fio repo to track progress on development towards this FIP.

# Backwards Compatibility
All FIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The FIP must explain how the author proposes to deal with these incompatibilities.

# Future considerations
This section may include proposals for how the new functionality could be enhanced in the future.

# Discussion link
Provide a link(s) to where this FIP is being discussed.
