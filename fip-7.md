---
fip: 7
title: Provide ability to deactivate FIO Addresses and Domains
status: Draft
type: Functionality
author: Casey Gardiner <casey@dapix.io>
created: 2020-04-22
updated: 2020-04-30
---

## Abstract
This FIP implements the following:
* Adds the ability for FIO Address owners the ability to burn an owned address.
* Adds the ability for FIO Domain owners the ability to set their domain as expired.
* Adds new API end points for FIO Domain deactivation and FIO Address burning.
* Adds new fees for FIO Domain deactivation and FIO Address burning.

## Motivation
Presently the FIO Blockchain only burns and removes old addresses and domains once they reach their expiration date. This feature proposal will allow address owners the ability to burn their addresses at any time and will allow domain owners that are looking to deactivate the domain to have their expiration set to the current time. Setting this expiration allows for the continued use of the domain during a grace period ( currently set to 90 days ). All FIO Addresses linked to that domain will be burned after their expirations but will not be able to be utilized. The domain will follow normal expired domain processes after the grace period. Once burned, a new domain owner may register the domain. If domain owner wishes to activate their domain after deactiviation, owners may call `/renew_fio_domain` and continue normal operation. 

## Specification
### Burn Addresses
#### New end point: *burn_fio_address* 
#### New action in new fio.address contract burnaddress
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|FIO Address, see FIO Address validation rules.|FIO Address that is being requested to burn.|
|actor|Yes|FIO account name|FIO account for the signer, the account owning this FIO Address.|
|max_fee|Yes|max fee SUFs|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address of TPID, See FIO Address validation rules|FIO Address of the wallet which generates this transaction. This FIO Address will be paid 10% of the fee.See FIO Protocol#TPIDs for details. Set to empty if not known.|

###### Example
```
{
    "fio_address": "purse@alice",
    "actor": "aftyershcu22",
    "max_fee": 400000000,
    "tpid": "rewards@wallet"
}
```
##### Processing
* Require auth of the actor
* Request is validated per Exception handling.
* Removes the fio_address from the fionames table.
* Removes the TPID entry inside the tpid index table. 
* Return status json containing status (ok), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Address format|FIO Address format is not valid|400|"fio_address"|Value sent in, e.g. "purse@alice"|"Invalid FIO Address"|
|Not owner of FIO Address|The signer does not own the address|403||||
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "400000000"|"Insufficient funds to cover fee"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "400000000"|"Fee exceeds supplied maximum"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|FIO Address not registered|FIO Address is not registered|400|"fio_address"|Value sent in, e.g. "purse@alice"|"FIO Address not registered"|
|FIO Address is active producer|Supplied FIO Address is registered as producer and is_active = 1|400|"fio_address"|Value sent in, e.g. "purse@alice"|"FIO Address is active producer. Unregister first."|
|FIO Address is proxy|Supplied FIO Address is registered as proxy|400|"fio_address"|Value sent in, e.g. "purse@alice"|"FIO Address is proxy. Unregister first."|

##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|Ok|
|fee_collected|String|fee amount collected SUFs|
###### Example
```
{
    "status": "Ok",
    "fee_collected": "400000000"		
}
```
###### New fee
A new fee will be created for `burn_fio_address`. This fee will not be bundle eligible and should cost ~400000000 SUF.

## Specification
### Dactivate Domain
#### New end point: *deactivate_fio_domain* 
#### New action in new fio.address contract setdomainexp
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_domain|Yes|FIO Domain, see FIO Domain validation rules.|FIO domain that is being requested to burn.|
|actor|Yes|FIO account name|FIO account for the signer, the account owning this FIO Domain.|
|max_fee|Yes|max fee SUFs|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address of TPID, See FIO Address validation rules|FIO Address of the wallet which generates this transaction. This FIO Address will be paid 10% of the fee.See FIO Protocol#TPIDs for details. Set to empty if not known.|

###### Example
```
{
    "fio_domain": "alice",
    "actor": "aftyershcu22",
    "max_fee": 800000000,
    "tpid": "rewards@wallet"
}
```
##### Processing
* Require auth of the actor.
* Request is validated per Exception handling.
* Sets fio_domain expiration date inside the domains table to now().
* Return status json containing status (ok), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Domain format|FIO Domain format is not valid|400|"fio_domain"|Value sent in, e.g. "alice"|"Invalid FIO domain"|
|FIO Domain not registered|FIO Domain is not registered|400|"fio_domain"|Value sent in, e.g. "alice"|"FIO Domain not registered"|
|Not owner of FIO Domain|The signer does not own the domain|403||||
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "800000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "800000000"|"Fee exceeds supplied maximum"|
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|Ok|
|fee_collected|String|fee amount collected SUFs|
###### Example
```
{
    "status": "Ok",
    "fee_collected": "800000000"        
}
```
## Fees
A new fee will be created for `deactivate_fio_domain`. This fee will not be bundle eligible and should cost ~800000000 SUF.

## Rationale
Adding the functionality to remove these addresses and setting domains to expired at any given time gives the owners flexibility. There are many reason why someone would want to burn their FIO Address/Domain:
   * Business/Personal requirements
   * Cost effectivness 
   * Spam prevention

## Implementation
The following files will be affected during this implementation:
   * fio.address.cpp
   * chain_plugin.cpp/.hpp
   * chain_api_plugin.cpp
   * httpc.hpp ( clio )
   
The fio.address smart contact, Clio, and the chain plugin are all required to be updated. The proposer will update to current fio version with the new clio and chain_plugin updates. They will then propose the msig for the fio.address contact to the active Block Producers. It is suggested that the top 21 Block Producers vote for the fees of the two new endpoints before the execution of the msig.

On execution [RAM of signer will be increased](https://developers.fioprotocol.io/fio-protocol/resource-management#ram-limits) by 512 bytes.
