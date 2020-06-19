---
fip: 3
title: Provide ability to cancel a request for funds
status: Accepted
type: Functionality
author: Ed Rotthoff <ed@dapix.io>
created: 2020-04-16
updated: 2020-04-30
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
#### New contract action cancelfndreq
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_request_id|Yes|Positive Int|Existing Funds request id|
|max_fee|Yes|Max fee SUFs|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address of TPID, See FIO Address validation rules|FIO Address of the wallet which generates this transaction. This FIO Address will be paid a percentage of the fee.See FIO Protocol Spec and Whitepaper for details of TPID. Set to empty if not known.|
|actor|Yes|the FIO account of the canceller|FIO pub account owning this payee FIO Address.|
###### Example
```
{
	"fio_request_id": 27,
	"max_fee": 40000000000,
	"tpid": ""
	"actor":"edrtfgtrthg"
}
```
##### Processing
* Verify the request exists. Error if id is not present. 
* Check that no payment is yet received for this request. If status is sent to blockchain then error.
* Check that payee_fio_address is owned by actor.
* verify that this payee key from the context table hashes to this actor.
* require auth of the actor
* verify that the fee for this does not exceed the max fee specified.
* charge appropriate fee (this will be a bundled fee transaction, fee will be same as reject)
* if there is not a status, create status to become cancelled.
* if there is a status, and status is cancelled, or rejected, or sent to blockchain then error
* if there is not a status then create a new cancelled status.
* check that transaction does not exceed the max allowable tx size.
* bump the ram allowance by 512
* Return status json containing status (cancelled), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Request not found|Request ID does not exist|400|"fio_request_id"|Value sent in, e.g. "10000000"|"No such FIO Request"|
|Invalid status or format|Request's status is not pending or inavlid format|400|"fio_request_id"|Value sent in, e.g. "blah"|"Only pending requests can be cancelled."|
|Fee exceeds maximum|fee exceeded the specified max|400|"max_fee"|fee exceeds max_fee specified|"Fee exceeds maximum"|
|Invalid TPID|TPID is not empty or contains invalid FIO address|400|"tpid"|Value sent in is not empty and not a valid FIO Address format|"TPID must be empty or valid FIO address"|
|Invalid actor|payee pub key hashed to account does not match actor FIO Address owner of request does not match actor|403|||"Invalid Actor"|
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|cancelled|
|fee_collected|Int|Fee amount collected SUFs|
###### Example
```
{	
   "status": "cancelled",
   "fee_collected": 0		
}
```
## Fees
Add a new fee to the system for cancel_fio_request: 600000000 SUF (uses 1 bundled transaction)

### Get Cancelled Requests
#### New end point: *get_cancelled_fio_requests*
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|See FIO Public Key validation rules|FIO public key of the requestee/payee.|
|limit|No|Positive Int|Number of domains to return. If omitted, all domains will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
|offset|No|Positive Int|First request from list to return. If omitted, 0 is assumed.|
###### Example
```
{
   "fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
   "limit": 100,
   "offset": 0
}
```
##### Processing
* Request is validated per Exception handling
* Return *limit* FIO requests starting at *offset* owned by *fio_public_key*.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid limit|limit format is not valid|400|"limit"|Value sent in, e.g. "-1"|"Invalid limit"|
|Invalid offset|offset format is not valid|400|"offset"|Value sent in, e.g. "-1"|"Invalid offset"|
|No Requests|No FIO Requests were found for provided FIO Public Key or that key does not hash to a known account|404|||"No FIO Requests"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|requests|fio_request_id|String|id of the cancelled request|
|requests|payer_fio_address|String|fio address of payer of the request|
|requests|payee_fio_address|String|fio address of payee of the request|
|requests|payer_fio_public_key|String|fio public key of payer of the request|
|requests|payee_fio_public_key|String|fio public key of payee of the request|
|requests|content|encrypted binary blob|encrypted content for the request|
|requests|timestamp|time|the timestamp when cancellation took place|
||more|Int|Number of remaining results|
###### Example
```
{
   "requests": [
   {
      "fio_request_id": "10",
      "payer_fio_address": "purse@alice",
      "payee_fio_address": "crypto@bob",
      "payer_fio_public_key": "FIO7167ErgCveJvuonvrEvVGhdWnkP4AEMfqvEd8s8raMkbbAXqhx",
      "payee_fio_public_key": "FIO7KGdMYj4ZMY2nUX9EaZu3G3GxZhTNXUq1tsNqC5rcP9rcmvWHq",
      "content": "...",
      "time_stamp": "2020-09-11T18:30:56"
   }
   ],
   "more": 0
}
```

## Rationale
Custom end points are used within the FIO Protocol easier for developers by hiding the complexity of EOSIO tools. Enhancing the functionality is done for the same reason. Advanced tools such as *get_table* remains unchanged.
We chose to permit the setting of a new status of cancelled whenever the status of the request is requested (in the present design this is when there is no entry in the fioreqstss table for this request)

## Implementation
* Add new API end point for cancel_fio_request
* Modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code. Add new status for cancelled to fio.request.obt.hpp. Add new action to fio.request.obt.cpp and fio.request.obt.abi. defined and add a new fee to the system, this fee will be the same cost as reject. dev test api endpoint and push action and resolve all issues (2 days)
* Add new API end point for get cancelled requests
* Modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code for the fetching of domains.  dev test and resolve all issues (1 day)
* Add new check for cancelled request and error message 
* Modify fio.request.obt.cpp sendobt action, add new error into processing of request id, dev test and resolve all issues. (4 hours)
