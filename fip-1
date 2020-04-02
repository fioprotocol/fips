---
fip: 1
title: FIO Domain/FIO Address transfer and data purge
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-02
updated:
---

## Abstract

## Motivation
Both FIO Domain and FIO Address are non-fungible tokens (NFTs) that are owned by a FIO public key. There is currently no support in FIO Protocol for transferring of ownership of either NFT. It was always assumed that this will be one of the first improvements to the FIO Protocol. FIO Address can be associated to FIO Requests, FIO Data, address mappings, and owner private key is required to decrypt some of this data. It is therefore prudent that when a FIO Address is transferred, the associated data is first purged.

## Specification 
### Transfer FIO Domain
#### New action: *domtransfer*
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_domain|Yes|FIO Domain|Valid and unexpired FIO Domain|
|new_owner_fio_public_key|Yes|FIO public key|FIO Public Key of the new owner. If account for that key does not exist, it will be created as part of this call.|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### Example
```
{
	"fio_domain": "alice",
	"new_owner_fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj"
	"max_fee": 30000000000,
	"tpid": "rewards@wallet",
	"actor": "aftyershcu22"
}
```
##### Processing
* Fee is collected
* If account is derived from hashing new_owner_fio_public_key
* If account does not exist, it gets created
* Owner of domain is changed from current account to account hashed from public key
##### Exception handling
|Error condition|Condition description|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Domain format|Domain is not valid format|400|"fio_domain"|Value sent in, e.g. "alice"|"Invalid FIO domain"|
|Invalid FIO Public Key|FIO public key is not valid format|400|"new_owner_fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid forma is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|Not owner of domain|The signer does not own the domain|403||||
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if succesful|
|fee_collected|Int|Amount of SUFs collected as fee|
###### Example
```
{
	"status": "OK",
	"fee_collected": 2000000000
}
```
