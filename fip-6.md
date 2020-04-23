---
fip: #
title: Provide ability to burn FIO Addresses and Domains
status: Draft
type: Functionality
author: Casey Gardiner <casey@dapix.io>
created: 2020-04-22
updated: 
---

## Abstract
This FIP implements the following:
* Allows FIO Address owners the ability to burn stale or unused addresses.
* Allows FIO Domain owners the ability to set their domain as expired. This allows for the continued use of the domain until the new owner registers during the grace period. If the domain does not receive a new owner in time, all FIO Addresses linked to that domain will be burned after their expirations.
## Motivation
Presently the FIO Blockchain only burns and removes old addresses and domains once they reach their expiration date. Added functionality to remove these addresses and domains at any given time gives the owners flexibility.

## Specification
### Burn Addresses
#### New end point: *deactivate_fio_address* 
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
* Request is validated per Exception handling
* Removes the fio_address from the fionames table
* Return status json containing status (ok), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Address|Address does not match format|400|"fio_address"||"Invalid FIO Address"|
|Invalid Actor|Actor does not exist on chain|403|||"Invalid Actor"|
|Insufficient Funds|Actor does not have required funds|403|||"Insufficient funds to cover fee"|
|Fee exceeds maximum|fee exceeded the specified max|400|"max_fee"|fee exceeds max_fee specified|"Fee exceeds maximum"|
|Invalid TPID|TPID is not empty or contains invalid FIO address|400|"tpid"|Value sent in is not empty and not a valid FIO Address format|"TPID must be empty or valid FIO address"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||fee_collected|String|fee amount collected SUFs|
###### Example
```
{
    "status": "Ok",
    "fee_collected": "400000000"		
}
```
## Fees
A new fee will be created for `deactivate_fio_address` . This fee will be of type 0 and should cost 400000000 SUF

## Specification
### Burn Domain
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
* Require auth of the actor
* Request is validated per Exception handling
* Sets fio_domain expiration date inside the domains table to now()
* Return status json containing status (ok), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Domain|Domain does not match format|400|"fio_domain"||"Invalid FIO domain"|
|Invalid Actor|Actor does not exist on chain|403|||"Invalid Actor"|
|Insufficient Funds|Actor does not have required funds|403|||"Insufficient funds to cover fee"|
|Fee exceeds maximum|fee exceeded the specified max|400|"max_fee"|fee exceeds max_fee specified|"Fee exceeds maximum"|
|Invalid TPID|TPID is not empty or contains invalid FIO address|400|"tpid"|Value sent in is not empty and not a valid FIO Address format|"TPID must be empty or valid FIO address"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||fee_collected|String|fee amount collected SUFs|
###### Example
```
{
    "status": "Ok",
    "fee_collected": "800000000"        
}
```
## Fees
A new fee will be created for `deactivate_fio_domain` . This fee will be of type 0 and should cost 800000000 SUF

## Rationale

## Implementation
