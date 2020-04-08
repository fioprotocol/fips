---
fip: 1
title: FIO Domain/FIO Address transfer and data purge
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-02
updated: 2020-04-03
---

## Abstract
This FIP implements the following:
* Adds ability to transfer FIO Domain to new owner using new action
* Adds ability to transfer FIO Address to new owner using new action
* Adds new API end points for FIO Domain transfer, FIO Address transfer
* Adds new fees for FIO Domain transfer and FIO Address transfer
* Modifies search logic for get_obt_data, get_pending_fio_requests, get_sent_fio_requests

## Motivation
Both FIO Domain and FIO Address are non-fungible tokens (NFTs) that are owned by a FIO Public Key. Ability to transfer ownership is a must, yet there is currently no support for it in FIO Protocol. It was always assumed that this will be one of the first improvements to the FIO Protocol.

## Specification 
### Transfer FIO Domain
#### New action: *xferdomain*
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
* Request is validated per Exception handling
* Fee is collected
* Account is derived from hashing new_owner_fio_public_key
* If new owner account does not exist, it gets created
* Owner of domain is changed from current account to account hashed from public key
	* Action explicitly not taken:
		* Domain expiration date is not updated
		* is_public flag is not updated
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Domain format|FIO Domain format is not valid|400|"fio_domain"|Value sent in, e.g. "alice"|"Invalid FIO domain"|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"new_owner_fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|FIO Domain expired|FIO Domain is expired|400|"fio_domain"|Value sent in, e.g. "alice"|"FIO Domain expired. Renew first."|
|FIO Domain not registered|FIO Domain is not registered|400|"fio_domain"|Value sent in, e.g. "alice"|"FIO Domain not registered"|
|Not owner of FIO Domain|The signer does not own the domain|403||||
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
#### New API end_point: */tranfer_fio_domain*
A new end point is added and maps to new action.
#### New fee
A new *tranfer_fio_domain* fee is added. Recommend initial amount same as transfer_tokens_pub_key (2 FIO as of 4/3/2020)
### Transfer FIO Address
#### New action: *xferaddress*
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|FIO Address|Valid and unexpired FIO Address|
|new_owner_fio_public_key|Yes|FIO public key|FIO Public Key of the new owner. If account for that key does not exist, it will be created as part of this call.|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### Example
```
{
	"fio_address": "purse@alice",
	"new_owner_fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj"
	"max_fee": 30000000000,
	"tpid": "rewards@wallet",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling
	* Explicitly allowed:
		* Transfer of FIO Address when domain of that address is expired
* Fee is collected
* Account is derived from hashing new_owner_fio_public_key
* If new owner account does not exist, it gets created
* Owner of address is changed from current account to account hashed from public key
	* Action explicitly not taken:
		* Past FIO Requests/Data is not updated.
		* Address expiration date is not updated
		* Address bundled transaction counter is not updated
* All existing Other Blockchain Public Addresses (OBPA) mappings for the FIO Address are purged.
* new_owner_fio_public_key is set as the chain_code:FIO, token_code:FIO of the transferred FIO Address 
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Address format|FIO Address format is not valid|400|"fio_domain"|Value sent in, e.g. "alice"|"Invalid FIO domain"|
|Invalid FIO Public Key|FIO Public Key format is not valid|400|"new_owner_fio_public_key"|Value sent in, e.g. "notakey"|"Invalid FIO Public Key"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|FIO Address expired|FIO Address is expired|400|"fio_address"|Value sent in, e.g. "purse@alice"|"FIO Address expired. Renew first."|
|FIO Address not registered|FIO Address is not registered|400|"fio_address"|Value sent in, e.g. "purse@alice"|"FIO Address not registered"|
|Not owner of FIO Address|The signer does not own the address|403||||
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
#### New API end_point: */tranfer_fio_address*
A new end point is added and maps to the new action.
#### New fee
A new *tranfer_fio_address* fee is added. Recommend initial amount same as transfer_tokens_pub_key (2 FIO as of 4/3/2020)

### Modification to existing queries
#### get_obt_data
get_obt_data is modified to only return OBT records which include the provided FIO Public Key. It currently returns OBT records which include FIO Addresses owned by the provided key at the time of query, which would cause new owner to see old requests, even though they could not decrypt the contents.
#### get_pending_fio_requests
get_pending_fio_requests is modified to only FIO Requests which include the provided FIO Public Key. It currently returns FIO Requests which include FIO Addresses owned by the provided key at the time of query, even though they could not decrypt the contents.
#### get_sent_fio_requests
get_sent_fio_requests is modified to only FIO Requests which include the provided FIO Public Key. It currently returns FIO Requests which include FIO Addresses owned by the provided key at the time of query, even though they could not decrypt the contents.

## Rationale
### Purging of data
Other Blockchain Public Addresses (OBPA) mappings should be purged on FIO Address transfer to avoid the risk of new owner not realizing there are mappings already attached to their address that are not theirs.

Aside from OBPA mappings, FIO Address is associated to FIO Requests and FIO Data. Initial approach was to purge this data, but that is not feasible since this data has a counter-party and they should be able to retrieve the data post transfer. You can also argue that the previous owner should be able to access their data, even though they have transferred ownership of the FIO Address.

Since the contents of FIO Request and FIO Data is encrypted using private/public keys at the time the request is sent, the new owner will not be able to decrypt it.

Wallets, at their discretion, may choose to check if FIO Address of displayed FIO Request has changed since original request and shos a notification to the user viewing an old request.

### Fees
Both FIO Domain and FIO Address transfers should each have a new fee type. The fee should be high enough to cover possible new account creation.

## Implementation
The following files will be affected during this implementation:
   * fio.address.cpp
   * chain_plugin.cpp/.hpp
   * chain_api_plugin.cpp
   * httpc.hpp ( clio )
   
The fio.address smart contact, Clio and the chain plugin are all required to be updated. The proposer will update to current fio version with the new clio and chain_plugin updates. They will then propose the msig for the fio.address contact to the active Block Producers.

Current Issue Page: https://github.com/fioprotocol/fio/issues/41
Current Branch: https://github.com/fioprotocol/fio/compare/develop...feature/transfer_domain
Current Pull Request: N/A

## Backwards Compatibility
### New actions
*xferdomain* and *xferaddress* are new actions and therefore do not impact existing users
### New API end points
*/tranfer_fio_domain* and */tranfer_fio_address* are new end points and therefore do not impact existing users
### New fees
*setfeevote* takes an array of fees and is not required for that action to have all possible fees. Adding new fees will therefore not break that action.
### Modification to existing queries
*get_obt_data*, *get_pending_fio_requests*, and *get_sent_fio_requests* will continue to only return relevant data, so modifications to the SDKs and wallet integrations are not required.

## Future considerations
An escrow functionality for domain transfer could be beneficial. A new domain transfer action could be created which would allow a domain owner to specify amount and optionally FIO Address of potential buyer. When executed the domain would be put in escrow until buyer makes a payment, using unique action, equal to specified amount. The domain would then be transferred to buyer and funds to seller.
