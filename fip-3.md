---
fip: 3
title: Provide ability to cancel a request for funds
status:Draft
type: Functionality
author: Ed Rotthoff <ed@dapix.io>
created: 2020-04-16
updated: 
---

## Abstract
This FIP implements the following:
* Adds new API end point and contract action for cancellation of a request for funds.
* Adds a new endpoint which will support paging that is called get_cancelled_requests.
* Adds new check to record obt action in the fio.request.obt contract to check that if the request id is specified, if there is a status of cancelled this is an error.


## Motivation
Presently the FIO API does not provide any way for a user to cancel a request for funds they have made:
* It is foreseeable that users will have a desire to cancel newly made requests for funds
* When a user enters errant data (for amounts or memos)
* When events in the users world conspire to render the request irrelevant.
* When a users account has been "hacked" and they wish to cancel fraudulent requests for funds.
* Once we have an ability to cancel, it will be useful to have an api endpoint called get_cancelled_requests.
* Once we have the ability to cancel, we want to check on record obt if the status of the request is cancelled, this is an error, whenever a request id is specified.


## Specification
### Cancel Funds Request
#### New end point: *cancel_funds_request*
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_request_id|Yes|Positive Int|Existing Funds request id|
|max_fee|Yes|Max fee SUFs|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address of TPID, See FIO Address validation rules|FIO Address of the wallet which generates this transaction. This FIO Address will be paid 10% of the fee.See FIO Protocol#TPIDs for details. Set to empty if not known.|
|pub_address|Yes|the FIO pub address of the canceller|FIO pub address relating to the account owning this payee FIO Address.|
###### Example
```
{
	"fio_request_id": "27",
	"max_fee": 40000000000,
	"tpid": ""
    "pub_address":"FIO5LfXWzK4bRbQSDvVeu2FoMxvNzR3HDsaWaa9oC1gEVfoyKNZ1s"
}
```
##### Processing
* Verify the request exists. Error if id is not present. 
* Check that no payment is yet received for this request. If status is sent to blockchain then error.
* Check that payee_fio_address is owned by actor.
* require auth of the actor
* verify that the fee for this does not exceed the max fee specified.
* charge appropriate fee (this will be a bundled fee transaction, fee will be same as reject)
* if there is not a status, create status to become cancelled.
* if there is a status, and status is cancelled, or rejected, then error
* if there is a status, then update the status to be cancelled.
* if there is not a status then create a new cancelled status.
* check that transaction does not exceed the max allowable tx size.
* bump the ram allowance by 512
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
## Fees
Add a new fee to the system.


## Rationale
Custom end points are used within the FIO Protocol easier for developers by hiding the complexity of EOSIO tools. Enhancing the functionality is done for the same reason. Advanced tools such as *get_table* remains unchanged.
We chose to permit the setting of a new status of cancelled whenever the status of the request is requested (in the present design this is when there is no entry in the fioreqstss table for this request)

## Implementation
* Add new API end point for cancel_fio_request
  --modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code. Add new status for cancelled to fio.request.obt.hpp. Add new action to fio.request.obt.cpp and fio.request.obt.abi. defined and add a new fee to the system, this fee will be the same cost as reject. dev test api endpoint and push action and resolve all issues (2 days)

