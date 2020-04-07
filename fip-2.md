---
fip: 2
title: Improvements to paging via API
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-06
updated: 2020-04-07
---

## Abstract

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
|limit|No|Positive Int|Number of request to return. If omitted, all requests will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
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
* *limit* FIO Domains starting at *offset* owned by provided FIO Public Key are returned.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|No FIO Domains|No FIO Domains were found for provided FIO Public Key or that key does not hash to a known account|404|||"No FIO Domains"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|fio_domains|fio_domain|String|FIO Domain for requested public key.|
|fio_domains|expiration|String|Expiration date.|
|fio_domains|is_public|Int|0 - domain is not public, 1 - domain is public|
||more|Int|	Number of remaining results|
###### Example
```
{
	"fio_domains": [
		{
			"fio_domain": "alice",
			"expiration": "2020-09-11T18:30:56",
			"is_public": 0
		}
	]
}
```

### Get FIO Addresses
#### New end point: *get_fio_addresses*
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|FIO Public Key|Valid FIO Public Key|
|limit|No|Positive Int|Number of request to return. If omitted, all requests will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
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
* *limit* FIO Addresses starting at *offset* owned by provided FIO Public Key are returned.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|No FIO Addresses|No FIO Addresses were found for provided FIO Public Key or that key does not hash to a known account|404|||"No FIO Addresses"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|fio_addresses|fio_address|String|FIO Address for requested public key.|
|fio_addresses|expiration|String|Expiration date.|
||more|Int|	Number of remaining results|
###### Example
```
{
	"fio_addresses": [
		{
			"fio_address": "purse@alice",
			"expiration": "2020-09-11T18:30:56"
		}
	]
}
```

## Rationale


## Implementation


## Backwards Compatibility


## Future considerations
