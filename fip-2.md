---
fip: 2
title: Improvements to paging via API
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-06
updated: 2020-04-08
---

## Abstract
This FIP implements the following:
* Adds new API end points for fetching FIO Domains with support for paging
* Adds new API end points for fetching FIO Addresses with support for paging
* Adds ability to return records created after specified time for */get_sent_fio_requests*, */get_pending_fio_requests*, and */get_obt_data*

## Motivation
There are currently deficiencies in paging for certain API calls:
* /get_fio_names has no paging at all. If an account has more FIO Domains or FIO Addresses than can be returned before table read time out, only a partial results are returned without warning to the user or ability to retrieve the rest.
* /get_obt_data, /get_pending_fio_requests and /get_sent_fio_requests has limit and offset already, but implementing wallets have expressed desire to cache the data locally and requested ability to query by providing a time stamp and only receiving requests since that time stamp.

## Specification
### Get FIO Domains
#### New end point: *get_fio_domains*
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|FIO Public Key|Valid FIO Public Key|
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
* Return *limit* FIO Domains starting at *offset* owned by *fio_public_key*.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid limit|limit format is not valid|400|"limit"|Value sent in, e.g. "-1"|"Invalid limit"|
|Invalid offset|offset format is not valid|400|"offset"|Value sent in, e.g. "-1"|"Invalid offset"|
|No FIO Domains|No FIO Domains were found for provided FIO Public Key or that key does not hash to a known account|404|||"No FIO Domains"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|fio_domains|fio_domain|String|FIO Domain for requested public key.|
|fio_domains|expiration|String|Expiration date.|
|fio_domains|is_public|Int|0 - domain is not public, 1 - domain is public|
||more|Int|Number of remaining results|
###### Example
```
{
	"fio_domains": [
		{
			"fio_domain": "alice",
			"expiration": "2020-09-11T18:30:56",
			"is_public": 0
		}
	],
	"more": 0
}
```

### Get FIO Addresses
#### New end point: *get_fio_addresses*
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|FIO Public Key|Valid FIO Public Key|
|limit|No|Positive Int|Number of addresses to return. If omitted, all addresses will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
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
* Return *limit* FIO Addresses starting at *offset* owned by *fio_public_key*.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid limit|limit format is not valid|400|"limit"|Value sent in, e.g. "-1"|"Invalid limit"|
|Invalid offset|offset format is not valid|400|"offset"|Value sent in, e.g. "-1"|"Invalid offset"|
|No FIO Addresses|No FIO Addresses were found for provided FIO Public Key or that key does not hash to a known account|404|||"No FIO Addresses"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|fio_addresses|fio_address|String|FIO Address for requested public key.|
|fio_addresses|expiration|String|Expiration date.|
||more|Int|Number of remaining results|
###### Example
```
{
	"fio_addresses": [
		{
			"fio_address": "purse@alice",
			"expiration": "2020-09-11T18:30:56"
		}
	],
	"more": 0
}
```

### Update /get_sent_fio_requests
The existing API end point will be updated to add optional *min_create_time* parameter.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|FIO Public Key|Valid FIO Public Key|
|limit|No|Positive Int|Number of request to return. If omitted, all requests will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
|offset|No|Positive Int|First request from list to return. If omitted, 0 is assumed.|
|min_create_time|No|Epoch time|Requests created after this time will be returned|
###### Example
```
{
	"fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
	"limit": 100,
	"offset": 0,
	"min_create_time": 1554746730
}
```
##### Processing
* Request is validated per Exception handling
* Return *limit* FIO Requests starting at *offset* where *payee_fio_public_key* is equal to *fio_public_key* and FIO Request created after *min_create_time*
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid limit|limit format is not valid|400|"limit"|Value sent in, e.g. "-1"|"Invalid limit"|
|Invalid offset|offset format is not valid|400|"offset"|Value sent in, e.g. "-1"|"Invalid offset"|
|Invalid time|min_create_time format is not valid|400|"min_create_time"|Value sent in, e.g. "-1"|"Invalid min_create_time"|
|No FIO Requests|No FIO Requests were found for provided FIO Public Key or that key does not hash to a known account|404|||"No FIO Requests"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|requests|fio_request_id|Int|Id of that funds request.|
|requests|payer_fio_address|String|FIO Address of the payer. This address initiated payment.|
|requests|payee_fio_address|String|FIO Address of the payee. This address is receiving payment.|
|requests|payer_fio_public_key|String|FIO public key of the payer.|
|requests|payee_fio_public_key|String|FIO public key of the payee.|
|requests|content|String|See new_funds_request|
|requests|time_stamp|Int|Timestamp of request|
|requests|status|String|Status of request.|
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
			"time_stamp": "2020-09-11T18:30:56",
			"status": "rejected"
		}
	],
	"more": 0
}
```

### Update /get_pending_fio_requests
The existing API end point will be updated to add optional *min_create_time* parameter.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|FIO Public Key|Valid FIO Public Key|
|limit|No|Positive Int|Number of request to return. If omitted, all requests will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
|offset|No|Positive Int|First request from list to return. If omitted, 0 is assumed.|
|min_create_time|No|Epoch time|Requests created after this time will be returned|
###### Example
```
{
	"fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
	"limit": 100,
	"offset": 0,
	"min_create_time": 1554746730
}
```
##### Processing
* Request is validated per Exception handling
* Return *limit* FIO Requests starting at *offset* where *payer_fio_public_key* is equal to *fio_public_key* and FIO Request created after *min_create_time*
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid limit|limit format is not valid|400|"limit"|Value sent in, e.g. "-1"|"Invalid limit"|
|Invalid offset|offset format is not valid|400|"offset"|Value sent in, e.g. "-1"|"Invalid offset"|
|Invalid time|min_create_time format is not valid|400|"min_create_time"|Value sent in, e.g. "-1"|"Invalid min_create_time"|
|No FIO Requests|No FIO Requests were found for provided FIO Public Key or that key does not hash to a known account|404|||"No pending FIO Requests"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|requests|fio_request_id|Int|Id of that funds request.|
|requests|payer_fio_address|String|FIO Address of the payer. This address initiated payment.|
|requests|payee_fio_address|String|FIO Address of the payee. This address is receiving payment.|
|requests|payer_fio_public_key|String|FIO public key of the payer.|
|requests|payee_fio_public_key|String|FIO public key of the payee.|
|requests|content|String|See new_funds_request|
|requests|time_stamp|Int|Timestamp of request|
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

### Update /get_obt_data
The existing API end point will be updated to add optional *min_create_time* parameter.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|FIO Public Key|Valid FIO Public Key|
|limit|No|Positive Int|Number of request to return. If omitted, all requests will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
|offset|No|Positive Int|First request from list to return. If omitted, 0 is assumed.|
|min_create_time|No|Epoch time|Records created after this time will be returned|
###### Example
```
{
	"fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
	"limit": 100,
	"offset": 0,
	"min_create_time": 1554746730
}
```
##### Processing
* Request is validated per Exception handling
* Return *limit* OBT data records starting at *offset* where *payer_fio_public_key* or *payee_fio_public_key* is equal to *fio_public_key* and OBT data record created after *min_create_time*.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid limit|limit format is not valid|400|"limit"|Value sent in, e.g. "-1"|"Invalid limit"|
|Invalid offset|offset format is not valid|400|"offset"|Value sent in, e.g. "-1"|"Invalid offset"|
|Invalid time|min_create_time format is not valid|400|"min_create_time"|Value sent in, e.g. "-1"|"Invalid min_create_time"|
|No OBT data records|No OBT data records were found for provided FIO Public Key or that key does not hash to a known account|404|||"No OBT data records"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|obt_data_records|fio_request_id|Int|Id of that funds request.|
|obt_data_records|payer_fio_address|String|FIO Address of the payer. This address initiated payment.|
|obt_data_records|payee_fio_address|String|FIO Address of the payee. This address is receiving payment.|
|obt_data_records|payer_fio_public_key|String|FIO public key of the payer.|
|obt_data_records|payee_fio_public_key|String|FIO public key of the payee.|
|obt_data_records|content|String|See record_obt_data|
|obt_data_records|fio_request_id|Int|Id of funds request, if present|
|obt_data_records|status|String|Status of OBT record|
|obt_data_records|time_stamp|Int|Timestamp of request|
||more|Int|Number of remaining results|
###### Example
```
{
	"obt_data_records": [
		{
			"payer_fio_address": "purse@alice",
			"payee_fio_address": "crypto@bob",
			"payer_fio_public_key": "FIO7167ErgCveJvuonvrEvVGhdWnkP4AEMfqvEd8s8raMkbbAXqhx",
			"payee_fio_public_key": "FIO7KGdMYj4ZMY2nUX9EaZu3G3GxZhTNXUq1tsNqC5rcP9rcmvWHq",
			"content": "...",
			"fio_request_id": 10,
			"status": "sent_to_blockchain",
			"time_stamp": "2020-09-11T18:30:56"
		}
	],
	"more": 0
}
```

## Rationale
Custom end points were put in place to make interaction with FIO Protocol easier for developers by hidding the complexity of EOSIO tools. Enhancing the functionality is done for the same reason. Advanced tools such as *get_table* remains unchanged.

## Implementation
Pending

## Backwards Compatibility
### New API end points
*/get_fio_domains* and */get_fio_address* are new end points and therefore do not impact existing users. */get_fio_names* end point remains unchanged.
### Modification to existing end points
New optional parameter is added to *get_sent_fio_requests*, *get_pending_fio_requests*, and *get_obt_data*, but since it can be omitted, there is no impact to existing users. Returned data remains unchanged.
