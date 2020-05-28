---
fip: 6
title: Transfer locked tokens
status: Accepted
type: Functionality
author: Ed Rotthoff <ed@dapix.io>, Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-16
updated: 2020-05-20
---
## Abstract
This FIP implements he ability to transfer tokens to a new account and lock those tokens on a pre-defined schedule.

Proposed new actions:
|Action|Endpoint|Description|
|---|---|---|
|trnsloctoks|transfer_locked_tokens|Transfer and locks tokens per provided schedule.|
||get_locks|Returns lock periods for account.|

Modified actions:
|Action|Endpoint|Description|
|---|---|---|
||get_fio_balance|Modified to add available token balance.|

## Motivation
The FIO Protocol includes [token locking functionality](https://kb.fioprotocol.io/fio-token/token-distribution#lockups-and-restrictions), which was built specifically to accommodate complex requirements for locking tokens [minted at Mainnet](https://kb.fioprotocol.io/fio-token/token-distribution#tokens-minted-at-mainnet). This functionality was not intended to be available post Mainnet.

The original use case was to **allow the Foundation to grant or sell tokens with attached locks**. During the design, the scope was extended to support the use case of allowing individuals to lock tokens for themselves as savings or for security reasons. However, supporting both use cases added unnecessary complexity and the FIP was scaled down to focus on the original use case. Individual users can still lock their own tokens using functionality developed for this FIP.

## Specification
### Overview
#### What is locked?
When the transfer of locked tokens is initiated, the target account (hashed from provided FIO Public Key) will be created, the specified amount will be transferred and a lock for that amount will be attached to the account. The account can function normally, funds may be transferred in and out of the account, except that the original amount of locked tokens remain locked until unlock periods are reached.
#### What variables define a lock?
To allow for most flexibility when tokens are locked the following variables define the type of lock:
* **Can vote** - when set to 1, 100% of tokens can be voted, even when locked.
* **Lock periods** - one or many time intervals at which specified percentage of tokens is unlocked.
#### What actions are disallowed for locked tokens in account?
* [trnsfiopubky](https://developers.fioprotocol.io/api/api-spec/reference/transfer-tokens-pub-key/transfer-tokens-pub-key-model) - when a transfer is initiated, only unlocked tokens in account can be transferred.
* [voteproducer](https://developers.fioprotocol.io/api/api-spec/reference/vote-producer/vote-producer-model)
	* If *can_vote* set to 0 only unlocked tokens are counted in vote.
	* If *can_vote* set to 1 all tokens in account are counted in vote, even when locked.
* Payment of fee. When fee is collected, only unlocked tokens may be used to pay for that fee. Bundled transactions can always be used, even when all tokens are locked.
#### How are tokens unlocked?
Funds in locked accounts are evaluated for unlocking and unlocked if eligible, whenever trnsfiopubky, voteproducer, or payment of fee are triggered.
### New actions
#### Transfer locked tokens
Transfer and locks tokens per provided schedule.
##### New end point: *transfer_locked_tokens* 
##### New action in new fio.system contract trnsloctoks
##### New fee: transfer_locked_tokens, 2000000000 per 90 days of longest duration.
###### Example
```
"unlock_periods": [
	{
		"duration": 86400,
		"percent": 1.2
	},
	{
		"duration": 172800,
		"percent": 90.8
	},
	{
		"duration": 259200,
		"percent": 8.0
	}
]
```
The fee will be: (1 90-day period in 259200 seconds * 300000000) = 600000000
##### RAM increase: 1,024 + (64 * number of lock periods) bytes
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|payee_public_key|Yes|Valid FIO Public Key|FIO public key of account where locked tokens will be sent.|
|can_vote|Yes|0 is not can_vote, 1 is can_vote.|This indicates if the locked amount can vote while locked.|
|unlock_periods|Yes|JSON Array of unlock periods. See unlock_periods below. See *Lock period verification* in *Processing* for requirements.|Schedule by which tokens become unlocked.|
|amount|Yes|Int|The amount of tokens to transfer and lock in SUFs|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### unlock_periods
|Parameter|Required|Format|Definition|
|---|---|---|---|
|duration|Yes|Int|Seconds from lock creation to when unlock of corresponding percentage occurs.|
|percent|Yes|Double|Percentage of locked tokens that unlock after corresponding duration lapsed.|
###### Example
```
{
	"payee_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
	"can_vote": 0,
	"unlock_periods": [
		{
			"duration": 86400,
			"percent": 50.0
		},
		{
			"duration": 172800,
			"percent": 25.0
		},
		{
			"duration": 259200,
			"percent": 25.0
		}
	],
	"amount": 40000000000000,
	"max_fee": 40000000000,
	"tpid": "rewards@wallet",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling:
	* Require auth of the actor.
	* Ensure payee public key does not hash down to existing account.
	* Lock period verification:
		* Minimum of 1 period. Maximum: 365 **(pending performance validation)**.
		* Sum of percentage in all periods is 100%.
		* Duration in each period is greater than 0.
	* Verify the locking account has necessary balance.
	* Verify that the fee for this does not exceed the max fee specified.
	* Verify transaction does not exceed max transaction size.
* Create account for the payee public key.
* Charge appropriate fee.
* Perform token transfer.
* Create entry in the locktokens table for these lock tokens.
* Increase RAM.
* Return response.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid payee public key|Specified public key is not valid FIO format.|400|"payee_public_key"|Value sent in, e.g. "notakey"|"Invalid Public Key."|
|Account already exist|Account hashed down from Public Key alreday exists.|400|"payee_public_key"|Value sent in|"Locked tokens can only be transferred to new account."|
|Invalid can_vote|Value sent in is not 0 or 1|400|"can_vote"|Value sent in, e.g. "-100"|"Invalid can_vote value."|
|Invalid unlock periods|See *Lock period verification* in *Processing*|400|"unlock_periods"|"Invalid unlock_periods."|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value."|
|Invalid amount|Amount is not numeric or not > 0|400|"amount"|Value sent in, e.g. "-1000000000"|"Invalid amount."|
|Insufficient balance|Account does not have enough funds for transfer and fee|400|"amount"|Value sent in, e.g. "1000000000"|"Insufficient balance."|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if successful|
|fee_collected|Int|Amount of SUFs collected as fee|
###### Example
```
{
	"status": "OK",
	"fee_collected": 0
}
```
#### Get locks
Returns lock periods for account.
##### New end point: *get_locks* 
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|Valid FIO Public Key|FIO Public Key of account, where locks exist.|
##### Processing
* Request is validated per Exception handling
	* Ensure hash and account match.
* Hash Public Key to account.
* Get the account from the fio account mapping.
* Read the locktokens table by the specified account.
* Traverse all lock periods.
* Add each period to the results.
* Return list of all lock periods.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No locked tokens in account.|No lock for this account.|404|||"No locked tokens in account."|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||lock_amount|Int|SUFs initial locked.|
||remaining_lock_amount|Int|SUFs still locked.|
||time_stamp|String|Time when lock started.|
||payouts_performed|Int|Number of unlock periods which were unlocked.|
||can_vote|Int|Indicates if the locked amount can vote while locked: 0 - can not vote, 1 - can vote.|
|unlock_periods|duration|Int|Seconds from lock creation to when unlock of coresponding percentage occurs.|
|unlock_periods|percent|Double|Percentage of locked tokens that unlock after coresponding duration lapsed.|
###### Example
```
{
	"lock_amount": 40000000000000,
	"remaining_lock_amount": 20000000000000,
	"time_stamp": "2020-03-11T18:30:56",
	"payouts_performed": 2,
	"can_vote": 1,
	"unlock_periods": [
		{
			"duration": 86400,
			"percent": 50.0
		},
		{
			"duration": 172800,
			"percent": 25.0
		},
		{
			"duration": 259200,
			"percent": 25.0
		}
	]
}
```
### Modifications to existing API calls
#### [/get_fio_balance](https://developers.fioprotocol.io/api/api-spec/reference/get-fio-balance/get-fio-balance)
get_fio_balance API call will be modified to add available token balance.
##### Request
Unchanged
###### Example
```
{
	"fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj"
}
```
##### Processing
* Available balance is added to response and includes only tokens which are not locked (by [Mainnet locks](https://kb.fioprotocol.io/fio-token/token-distribution#lockups-and-restrictions) or locks developed for this FIP).
##### Exception handling
Unchanged
##### Response
|Parameter|Format|Definition|
|---|---|---|
|balance|Int|Total SUF balance associated with supplied public key (includes locked tokens).|
|available|Int|Available SUF balance associated with supplied public key (does not include locked tokens).|
###### Example
```
{
	"balance": 100000000000,
	"available": 100000000000
}
```
### Modifications to plugins
#### History
Since this will be another action which creates a new account, History plug-in has to be modified to properly recognize those accounts.

## Rationale
### Other approaches considered
An approach was considered to lock tokens in smart contract instead of in account, but solution described in this FIP was deemed more appropriate as it's already used for [Mainnet locks](https://kb.fioprotocol.io/fio-token/token-distribution#lockups-and-restrictions) and offers more security than single contract account holding funds of multiple users.

### Changes made to FIP after accepting as Draft
1. Removed clean token actions. This is consistent with our strategy Pre-Mainnet and in FIP (e.g. [FIP-8](https://github.com/fioprotocol/fips/blob/master/fip-8.md#burning-expired-requests) of not removing things from state, except when required for functionality (e.g. burn expired domains). We need a comprehensive approach to what gets removed when and how across all contracts. This way we do not end up with different strategy for removing locks and different for removing requests. There is already FIP planned to address this.
1. Locking of existing accounts will not be allowed. Locks can only be applied to new account. The reason being it can change an account of a user without that user's knowledge or consent. For example, a User A may send 1 locked token to User B, without User B consenting to it. Now user B has locked tokens intermingled with unlocked tokens and that impacts their total and available token balance display, but they can't do anything about it.
1. Ability to lock user's own account will not be supported as the use case of allowing individuals to lock tokens for themselves as savings or for security reasons was descoped from this FIP.
1. Added TPID to lock_accounts. TPID should always be present on transactions expected to be exposed to users to encourage implementation by wallets.
1. Changed fee to be based on longest duration in unlock_periods.
1. Removed unlock_tokens action. This is handled automatically and not needed if only original use cases is being supported.
1. get_locks paging has been eliminated as there is only one lock per account and unlock periods are limited.
1. Added changes to get_fio_balance and History plug-in.

## Implementation
**May need to be revised.**
* Add new locking structures to the fio.system contract.
* Make a new table locktokens
	* int64 lockid   <primary index>     //this is the identifier of the lock
	* name owner_account;   <secondary index>  //this is the account that owns the lock
	* int64_t lock_amount = 0;        //this is the amount of the lock in FIO SUF
	* int32_t payouts_performed = 0;  //this is the number of payouts performed thus far.
	* int32 can_vote = 0;   //this is the flag indicating if the lock is votable/proxy-able
	* vector<lockperiods> periods;   // this is the locking periods for the lock
	* int64_t remaining_lock_amount = 0; <secondary index>  //this is the amount remaining in the lock in FIO SUF, get decremented as unlocking occurs.
	* uint32_t timestamp = 0;   //this is the time of creation of the lock, locking periods are relative to this time.
	* struct lockperiods     //this is the definition of locking periods.
	* int64_t duration;   //this is the duration of the lock period, relative to the timestamp on the lock.
	* double percent;   //this is the percent to unlock when the time period has passed.    
* Add tables and indexes to fio.system contract. (1 day)
* Integrate accounting logic and check for can transfer into token contract, and voting. Affected files (fio.token.hpp, fio.token.cpp, voting.cpp) (2 days).
* Add new API end point for lock_tokens - modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code.
* Add new action (locktokens) to fio.system.cpp. dev test api endpoint and push action and resolve all issues (1 days)
* Add new API end point for get_locks - modify chain_plugin cpp and hpp to add new params and code. dev test api endpoint and resolve all issues (1 days)
* Modify get_fio_balance to return balance and locked:numberlockedtokens. (4 hours)
* Modify History plugin for lock_tokens, to ensure tx gets into block explorer (1 days)

### Testing Considerations
* Create a lock verify the schedule is obeyed: 1 locking period, multiple locking periods.
* Make a votable lock, vote for producers.
* Make a votable lock, proxy this accounts vote to another, verify voting power.

## Backwards Compatibility
[/get_fio_balance](https://developers.fioprotocol.io/api/api-spec/reference/get-fio-balance/get-fio-balance) is the only existing API method being modified, but only a new response element is added, so wallets currently implementing this call should not be affected.

Token locks described in this FIP are different from [Mainnet locks](https://kb.fioprotocol.io/fio-token/token-distribution#lockups-and-restrictions) and a single account cannot be locked in both ways. The only overlap exists when computing available tokens in [/get_fio_balance](https://developers.fioprotocol.io/api/api-spec/reference/get-fio-balance/get-fio-balance).

## Future considerations
Support the use case of allowing individuals to lock tokens for themselves as savings or for security reasons should be considered.
