---
fip: 10
title: Redesign Fee Computations
status: Draft
type: Functionality
author: Ed Rotthoff <ed@dapix.io>; Pawel Mastalerz <pawel@dapix.io>
created: 2020-06-07
updated: 2020-06-11
---

## Abstract
This FIP implements a redesign of the fee voting and fee computation in the FIO Protocol. Specifically:
* Fee computation will be removed from onblock and from [setfeevote](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-ratios/submit-fee-ratios-model) and [setfeemult](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-multiplier/submit-fee-multiplier-model) and attached to a dedicated action/endpoint.
* Any block producer will be permitted to vote for fees.
* A fee will be collected whenever a [setfeevote](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-ratios/submit-fee-ratios-model) and [setfeemult](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-multiplier/submit-fee-multiplier-model) is executed.
* Only votes of top 21 block producers will be used to compute the fees. 

Proposed new actions:
|Action|Endpoint|Description|
|---|---|---|
|computefees|compute_fees|Computes fee votes and sets fees in the FIO Protocol.|

Modified actions:
|Action|Endpoint|Description|
|---|---|---|
|setfeevote|set_fee_vote|Can now be called by any BP. Fee is added. Fee computation removed from this action.|
|setfeemult|set_fee_multiplier|Can now be called by any BP. Fee is added. Fee computation removed from this action.|

## Motivation
The FIO Protocol was designed to enable top 21 producers to vote on protocol fees via [setfeevote](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-ratios/submit-fee-ratios-model) and [setfeemult](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-multiplier/submit-fee-multiplier-model).

The [computation of fees](https://developers.fioprotocol.io/fio-chain/bp#setting-fees) occurs after every [setfeevote](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-ratios/submit-fee-ratios-model) or [setfeemult](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-multiplier/submit-fee-multiplier-model). The fees are also recomputed in the onblock every 126 blocks.

It has been found that the processing required to compute fees is too large to be handled in onblock or attached to action of setting fees, especially if number of block producers allowed to vote is increased beyond top 21, which has been discussed in [Issue 147](https://github.com/fioprotocol/fio/issues/147) and now part of this FIP.

To address this issue, a new endpoint dedicated to fee computation should be implemented. This will follow the same approach as other areas of the FIO Protocol, such as burning of expired addresses, and claiming of rewards.

## Specification
### New actions
#### Compute fees
Computes and sets fees in the FIO Protocol.
##### New end point: *compute_fees* 
##### New action in new fio.system contract computefees
##### RAM increase: none, this does not increase state
##### New fee: this action will not have a fee, as it will only process if work is needed and return exception otherwise
##### Request
Empty
##### Processing
* Request is validated per Exception handling.
  * Throw no work exception if no work is left to do.
* Compute the number of fees to process.
* Assemble the next NFP fees to process.
* Process the fees (set the votesPending false on update).
* Return response.
* Any account on the protocol can call this, but BPs should call as a service. Note it throws no work exception when its done all necessary work and so is protected from polluting the block log.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No work|No work to perform|400|"computefees"|"computefees"|"No work."|
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if successful|
|fees_processed|Int|Number of fees processed|
###### Example
```
{
	"status": "OK",
	"fees_processed": 7  
}
```

### Modifications to existing actions
#### [setfeevote](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-ratios/submit-fee-ratios-model)
Modified to use new table, be callable by any BP, and charge a fee.
##### New fee: add submit_fee_ratios fee: 1000000000, not eligible for bundled transactions
##### Request
|Group|Parameter|Required|Format|Definition|
|---|---|---|---|---|
|fee_ratios||Yes|JSON Array|Array of fees and corresponding ratios.|
|fee_ratios|end_point|Yes|String|Name of endpoint for which fee is being set.|
|fee_ratios|value|Yes|Int|Fee in SUFs which will be multiplied by multiplier.|
||max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
||actor|Yes|12 character string|Valid actor of signer.|
###### Example
```
{
  "fee_ratios": [
    {
      "end_point": "register_fio_domain",
      "value": 40000000000
    },
    {
      "end_point": "register_fio_address",
      "value": 2000000000
    },
    {
      "end_point": "renew_fio_domain",
      "value": 40000000000
    },
    {
      "end_point": "renew_fio_address",
      "value": 2000000000
    },
    {
      "end_point": "add_pub_address",
      "value": 30000000
    },
    {
      "end_point": "transfer_tokens_pub_key",
      "value": 100000000
    },
    {
      "end_point": "new_funds_request",
      "value": 60000000
    },
    {
      "end_point": "reject_funds_request",
      "value": 30000000
    },
    {
      "end_point": "record_obt_data",
      "value": 60000000
    },
    {
      "end_point": "set_fio_domain_public",
      "value": 30000000
    },
    {
      "end_point": "register_producer",
      "value": 10000000000
    },
    {
      "end_point": "register_proxy",
      "value": 1000000000
    },
    {
      "end_point": "unregister_proxy",
      "value": 20000000
    },
    {
      "end_point": "unregister_producer",
      "value": 20000000
    },
    {
      "end_point": "proxy_vote",
      "value": 30000000
    },
    {
      "end_point": "vote_producer",
      "value": 30000000
    },
    {
      "end_point": "auth_delete",
      "value": 20000000
    },
    {
      "end_point": "auth_link",
      "value": 20000000
    },
    {
      "end_point": "auth_update",
      "value": 50000000
    },
    {
      "end_point": "msig_propose",
      "value": 50000000
    },
    {
      "end_point": "msig_approve",
      "value": 20000000
    },
    {
      "end_point": "msig_unapprove",
      "value": 20000000
    },
    {
      "end_point": "msig_cancel",
      "value": 20000000
    },
    {
      "end_point": "msig_exec",
      "value": 20000000
    },
    {
      "end_point": "msig_invalidate",
      "value": 20000000
    }
  ],
  "max_fee": 0,
  "actor": "aftyershcu22"
}
```
##### Processing
Modify the existing action to:
* Store submitted value, but not calculate fees.
* Use the new fees table and set the votesPending to 1.
* Allow any block producer to call it.
* Charge a fee.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Not a BP|Actor is not an active (meaning has is_active = 1) BP.|400|"actor"|Value sent in, e.g. "aftyershcu22"|"Not an active BP."|
|Not positive|Fee value is not positive|400|"value"|Value sent in, e.g. "-1"|"Value has to be positive."|
|Invalid end_point|End point is not valid or does not exist.|400|"end_point"|Value sent in, e.g. "xxx"|"Invalid end_point."|
|Too soon since last call|Call was made less than 3600 seconds since last call.|400|||"Too soon since last call."|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|Signer’s FIO Public Key does not match actor|Signer’s FIO Public Key does not match actor|403|||Type: invalid_signature|
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
#### [setfeemult](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-multiplier/submit-fee-multiplier-model)
Modified to use new table, be callable by any BP, and charge a fee.
##### New fee: add submit_fee_multiplier fee: 400000000, not eligible for bundled transactions
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|multiplier|Yes|Double|Multiplier|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|actor|Yes|12 character string|Valid actor of signer|
##### Processing
Modify the action to:
* Use the new fees table and set the votesPending to 1.
* Allow any block producer to call it.
* Charge a fee.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Not a BP|Actor is not an active (meaning has is_active = 1) BP.|400|"actor"|Value sent in, e.g. "aftyershcu22"|"Not an active BP."|
|Not positive|Multiplier value is not positive|400|"value"|Value sent in, e.g. "-1"|"Value has to be positive."|
|Too soon since last call|Call was made less than 120 seconds since last call.|400|||"Too soon since last call."|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|Signer’s FIO Public Key does not match actor|Signer’s FIO Public Key does not match actor|403|||Type: invalid_signature|
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

## Implementation
### Endpoint updates
#### Affected fee based endpoints
The following need to be migrated to the new table:
* fio.token
* fio.address
* fio.system
* fio.fee
* fio.request.obt

#### Affected chain plugin endpoints reading fees
The following need to be migrated to the new table:
* chain_plugin

### Modifications to plugins
#### History plugin
Anything relating to fees needs to use the new fees table after the setcode block number.

### Processing limits 
The number of fees processed per call must be adaptable as the number of producers registered increases and as the number of fees increases. Present testing shows that the protocol can comfortably handle 32 fees voted on by 21 producers, so we round this down to become an initial estimated processing limit of 600 Producer Fees Voted (PFV) where producer fees voted equals number of fees times number of voters.

We will limit the amount of work performed by each call to compute fees so that it does not perform more work than 600 PFV. We will compute a number of work iterations to be (nw) using number of producers (nP) and number of fees (nF) such that nW = (nP * nF) / PFV and this will become the number of work iterations necessary. To determine the number of fees to process (NFP), we will divide the number of fees by this number (nW) and this will become the number of fees to process during each call to the compute fees. A semaphore will be used to track the processing of fees. This semaphore (which is a flag) will be introduced into the fees table, this flag will be called "votesPending". When a fee is voted on, this flag will be set to 1 for the associated fee in the fee table (votes are pending). When fees are computed and set in the protocol this flag will be set to 0 (no votes are pending). This flag will be used as a secondary index in the fees table and will be used to search for next set of fees (set size equals NFP) to be processed. After determining the fees to process the system will process these and update them into the fees table with votesPending set 0. 

## Backwards Compatibility
### Table migrations
To avoid table migrations we will make a new fees table, called fiofees1 and we will migrate contracts to use this new table. In this way migrations will not become necessary on chain. The list of impacted contracts is included above.
### Impact to existing calls
All entities using existing [setfeevote](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-ratios/submit-fee-ratios-model) and [setfeemult](https://developers.fioprotocol.io/api/api-spec/reference/submit-fee-multiplier/submit-fee-multiplier-model) actions will have to migrate to new actions, which now inlcude max_fee. However, since this call only impacts block producers, impact is minimized.

## Future considerations
None
