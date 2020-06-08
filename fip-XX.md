---
fip: XX
title: Redesign Fee Computations From Fee Votes
status: Draft
type: Functionality
author: Ed Rotthoff <ed@dapix.io>
created: 2020-06-07
updated: 2020-06-07
---
## Abstract
This FIP implements a redesign of the fee voting and fee computations in the FIO protocol.

Proposed new actions:
|Action|Endpoint|Description|
|---|---|---|
|updatefees|update_fees|Processes fee votes and updates fees in the FIO protocol.|

Affected Actions:
|Action|Endpoint|Description|
|---|---|---|
|setfeevote|set_fee_vote|Permits voter to specify the voting ratios to use in fee computations.|
|setfeemult|set_fee_multiplier|Permits voter to specify the voting multiplier to use in fee computations.|



## Motivation
The FIO Protocol was initially desgined to provide voting of the top 21 producers.  The update of fees on chain was intended to be performed automatically whenever fee voting takes place, the fees were also intended to be updated in the onblock on a 126 block boundary according to this design. It has been found that the additional processing required to process th fees, especially when more voters are permitted to participate, causes enough overhead that both voting for fees, and processing of fees on the block boundary time out when the processing attempts to complete. It will be better for the prototocol to make a new endpoint which is designed in the same manner as other areas of the FIO protocol (such as burning of expired addresses, and claiming of rewards) to perform the computation of fees in a way that is sustainable from an overhead perspective.


## Specification
### Overview
### Any registered producer will be permitted to vote for the fees to be collected within the FIO protocol. A fee will be collected whenever a fee vote is set (if the voter is not in the top 21 BPs). The fee votes of the top 21 producers will be used to contribute to the fees computed within the protocol. 

### Assumptions 
### The number of fees processed per call must be adaptable as the number of producers registered increases and as the number of fees increases. Present testing shows that the protocol can comfortably handle  (32 fees voted on by 21 producers ) so we round this down to become an initial estiamted processing limit of 600 producer fees voted where prioducer fees voted equals number of fees times number of voters. In our calculations we will refer to this as PFV. We will limit the amount of work performed by each call to update fees so that it does not perform more work than this.  We will cmpute a number of work iterations to be (nw)  using number of producers (nP) and number of fees (nF) such that nW =  (nP * nF) / PFV and this will become the number of work iterations necessary. To determine the number of fees to process (NFP), we will divide the number of fees by this number (nW) and this will become the number of fees to process during each call to the update fees. A semaphore will be used to track the processing of fees. This semaphore (which is a flag) will be introduced into the fees table, this flag will be called "votesPending". When a fee is voted on this flag will be set to 1 for the associated fee in the fee table (votes are pending). When fees are computer ans set in the protocol this flag will be set to 0 (no votes are pending). This flag will be used as a secondary index in the fees table and will be used to search for  next set of fees (set size equals NFP) to be processed. After determining the fees to process the system will process these and update them into the fees table with votesPending set 0. 

### Table migrations. 
### To avoid table migrations we will make a new fees table,  called fiofees1 and we will migrate contracts to use this new table. In this way migrations will not become necessary on chain. The list of impacted contracts is included below.


### New actions
#### Update Fees

##### New end point: *update_fees* 
##### New action in new fio.system contract updfees
##### RAM increase: none, this does not increase state
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|actor|Yes|12 character string|Valid actor of signer|
###### Example
```
{
	"actor": "aftyershcu22"
}
```
##### Processing
*  Note -- any account on the protocol can call this, but BPs should call as a service. Note it throws no work exception when its done all necessary work and so is protected from polluting the block log.
	* Require auth of the actor.
	* Compute the number of fees to process (see above)
        * Assemble the next NFP fees to process.
        * Process the fees (set the votesPending false on update)
        * Throw no work exception if no work is left to do.
* Return response.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No work| No work to perform|403|" "|" "|"No Work"|
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if successful|
|fee_collected|Int|Amount of SUFs collected as fee|
###### Example
```
{
	"status": "OK",
	"fees_processed": 7  
}
```
#### Submit Fee Ratio
Modify the action to use the new fees table and set the votesPending to 1.

#### Submit Fee multiplier
Modify the action to use the new fees table and set the votesPending to 1.

#### All fee based endpoints are affected, to migrate to the new table.
fio.token
fio.address
fio.system
fio.fee
fio.request.obt

#### All chain plugin endpoints reading fees are affected. To migrate to the new table.
chain_plugin


### Modifications to plugins
#### History plugin
Anything relating to fees needs to use the new fees table after the setcode block number.




## Implementation


## Backwards Compatibility


## Future considerations

