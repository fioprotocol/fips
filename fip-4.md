---
fip: 4
title: Provide ability to remove pub address from the FIO protocol for a user.
status: Accepted
type: Functionality
author: Ed Rotthoff <ed@dapix.io>
created: 2020-04-16
updated: 
---

## Abstract
This FIP implements the following:
* Adds new API end point and contract action for remove pub address.


## Motivation
Presently the FIO API does not provide any way for a user to remove public address mappings for a user:
* It is foreseeable that users will have a need to remove pub address mappings
* When a certain token is no longer supported
* Whenever a user might desire to remove a certain mapping from state for privacy reasons.


## Specification
### Remove pub address
#### New end point: *remove_pub_address* 
#### New action in fio address contract remaddress
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|fio address, see FIO address validation rules.|FIO Address which will have public addresses removed.|
|public_addresses|Yes|JSON Array. See public_addresses below. Min: 1 |The public address to be removed from the address mapping.|
|max_fee|Yes|max fee SUFs|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address of TPID, See FIO Address validation rules|FIO Address of the wallet which generates this transaction. This FIO Address will be paid 10% of the fee.See FIO Protocol#TPIDs for details. Set to empty if not known.|
|actor|Yes|FIO account name|FIO account for the signer, the account owning this payee FIO Address.|
###### Example
```
{
    "fio_address": "purse@alice",
    "public_addresses": [{
                          "chain_code": "BTC",
                          "token_code": "BTC",
                          "public_address": "1PMycacnJaSqwwJqjawXBErnLsZ7RkXUAs"
                         },{
                           "chain_code": "ETH",
                           "token_code": "ETH",
                           "public_address": "0xab5801a7d398351b8be11c439e05c5b3259aec9b"
                         }
                        ]
    "max_fee": 0,
    "tpid": "rewards@wallet",
    "actor": "aftyershcu22"
}
```
##### Processing
* Verify the fio address format.
* Verify the fio address exists. 
* Verify the number of public addresses specified. Minimum of 1
* Verify the public addresses specified exist in the mappings on the chain
* Verify that all information (chain code, token code and public address match the on chain mapping), error if failure
* if any one of the public addresses does not exist on chain then error.
* Check that fio_address is owned by actor.
* require auth of the actor
* verify that the fee for this does not exceed the max fee specified.
* charge appropriate fee (this will be a bundled fee transaction, fee will be same as reject)
* verify tx does not exceed max transaction size.
* since this reduces state NO RAM BUMP.
* remove all specified public addresses.
* Return status json containing status (ok), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO address|Invalid format, or fio_address not found|400|"fio_address"|fio address invalid or not found|"Invalid FIO Address"|
|Public addresses invalid|Public addresses contains information that does not exist, or does not match the on chain mappings|400|"public_addresses"|Invalid Public Addresses |
|Fee exceeds maximum|fee exceeded the specified max|400|"max_fee"|fee exceeds max_fee specified|"Fee exceeds maximum"|
|Invalid TPID|TPID is not empty or contains invalid FIO address|400|"tpid"|Value sent in is not empty and not a valid FIO Address format|"TPID must be empty or valid FIO address"|
|Invalid Actor|Actor does not match payee FIO Address owner of request|403|||"Invalid Actor"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||fee_collected|String|fee amount collected SUFs|
###### Example
```
{
	
    "status": "Ok",
    "fee_collected": "0"		
}
```
## Fees
a new fee will be created remove_pub_address, type 1, 600000000 SUF, this is bundled, bundle counter will be decremented 1 for each call.

### Remove ALL pub address
#### New end point: *remove_all_pub_addresses* 
#### New action in fio address contract remalladdr
##### Request
this request will remove all fio addresses, with the exception of the FIO pub address.

|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|fio address, see FIO address validation rules.|FIO Address which will have public addresses removed.|
|max_fee|Yes|max fee SUFs|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|tpid|Yes|FIO Address of TPID, See FIO Address validation rules|FIO Address of the wallet which generates this transaction. This FIO Address will be paid 10% of the fee.See FIO Protocol#TPIDs for details. Set to empty if not known.|
|actor|Yes|FIO account name|FIO account for the signer, the account owning this payee FIO Address.|
###### Example
```
{
   "fio_address": "purse@alice",
   "max_fee": 0,
   "tpid": "rewards@wallet",
   "actor": "aftyershcu22"
}
```
##### Processing
* Verify the fio address format.
* Verify the fio address exists. 
* Verify the public addresses exist on the chain for this address, if not error
* Check that fio_address is owned by actor.
* require auth of the actor
* verify that the fee for this does not exceed the max fee specified.
* charge appropriate fee (this will be a bundled fee transaction, fee will be same as reject)
* verify tx does not exceed max transaction size.
* since this reduces state NO RAM BUMP.
* remove all public addresses.
* Return status json containing status (ok), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO address|Invalid format, or fio_address not found|400|"fio_address"|fio address invalid or not found|"Invalid FIO Address"|
|Fee exceeds maximum|fee exceeded the specified max|400|"max_fee"|fee exceeds max_fee specified|"Fee exceeds maximum"|
|Invalid TPID|TPID is not empty or contains invalid FIO address|400|"tpid"|Value sent in is not empty and not a valid FIO Address format|"TPID must be empty or valid FIO address"|
|Invalid Actor|Actor does not match payee FIO Address owner of request|403|||"Invalid Actor"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||fee_collected|String|fee amount collected SUFs|
###### Example
```
{
   "status": "Ok",
   "fee_collected": "0"        
}
```
## Fees
a new fee will be created remove_pub_addresses, type 1, 600000000 SUF, this is bundled, bundle counter will be decremented 1 for each call.


## Implementation
* Add new API end point for remove pub address
  --modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code. Add new action (remaddress) to fio.address.cpp and fio.address.abi. dev test api endpoint and push action and resolve all issues (1 days)
  * Add new API end point for remove all pub addresses
  --modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code. Add new action (remaddress) to fio.address.cpp and fio.address.abi. dev test api endpoint and push action and resolve all issues (1 days)
