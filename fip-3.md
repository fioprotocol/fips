---
fip: 3
title: Provide ability to cancel a request for funds
status:Proposed
type: Functionality
author: Ed Rotthoff <ed@dapix.io>
created: 2020-04-16
updated: 
---

## Abstract
This FIP implements the following:
* Adds new API end point and contract action for cancellation of a request for funds.


## Motivation
Presently the FIO API does not provide any way for a user to cancel a request for funds they have made:
* It is foreseeable that users will have a desire to cancel newly made requests for funds
* When a user enters errant data (for amounts or memos)
* When events in the users world conspire to render the request irrelevant.
* When a users account has been "hacked" and they wish to purge the account of fraudulent requests for funds.


## Specification
### Cancel Funds Request
#### New end point: *cancel_funds_request*
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_request_id|Yes|Positive Int|Existing Funds request id|
|max_fee|Yes|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address of TPID, See FIO Address validation rules|FIO Address of the wallet which generates this transaction. This FIO Address will be paid 10% of the fee.See FIO Protocol#TPIDs for details. Set to empty if not known.|
|actor|Yes|FIO account name|FIO account for the signer, the account owning this payee FIO Address.|
###### Example
```
{
	"fio_request_id": "27",
	"max_fee": 40000000000,
	"tpid": ""
        "Actor":"qwertfgewstd"
}
```
##### Processing
* Verify the request exists. Error if id is not present. 
* Check that no payment is yet received for this request. If payment received then error.
* Check that payee_fio_address is owned by actor.
* require auth of the actor
* verify that the fee for this does not exceed the max fee specified.
* charge appropriate fee (this will be a bundled fee transaction, fee will be same as reject)
* if there is not a status, create status to become cancelled.
* if there is a status, and status is cancelled, then error
* if there is a status, and the status, requested, rejected, then modify status to be cancelled.
* Return status json containing status (cancelled), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid Request ID|Request ID not found|400|"fio_request_id"|Value sent in not found|"Invalid FIO Request ID"|
|Request ID|Request cannot be cancelled because status is sent to blockchain (2)|400|"fio_request_id"|request with status sent to blockchain cannot be cancelled |
|Fee exceeds maximum|fee exceeded the specified max|400|"max_fee"|fee exceeds max_fee specified|"Fee exceeds maximum"|
|Invalid TPID|TPID is not empty or contains invalid FIO address|400|"tpid"|Value sent in is not empty and not a valid FIO Address format|"TPID must be empty or valid FIO address"|
|Invalid Actor|Actor does not match payee FIO Address owner of request|403|||"Invalid Actor"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|cancelled|
||fee_collected|String|fee amount collected SUFs|
###### Example
```
{
	
			"status": "cancelled",
			"fee_collected": "0"		
}
```

## Rationale
Custom end points were put in place to make interaction with FIO Protocol easier for developers by hiding the complexity of EOSIO tools. Enhancing the functionality is done for the same reason. Advanced tools such as *get_table* remains unchanged.

## Implementation
* Add new API end point for cancel_fio_request
  --modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code. Add new status for cancelled to fio.request.obt.hpp. Add new action to fio.request.obt.cpp and fio.request.obt.abi. dev test api endpoint and push action and resolve all issues (2 days)


## Backwards Compatibility
### New API end points
*/get_fio_domains* and */get_fio_address* are new end points and therefore do not impact existing users. */get_fio_names* end point remains unchanged.
