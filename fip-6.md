---
fip: 6
title: Token locking
status: Draft
type: Functionality
author: Ed Rotthoff <ed@dapix.io>, Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-16
updated: 2020-05-07
---
## Abstract
This FIP implements the following:
* Adds configurable locking mechanism for FIO tokens which is usable by all users of FIO 
* Locked tokens will have any number of lock periods, each period will have a duration and a percent to unlock after the duration. Durations will be processed relative to the locks creation time.
* Users will be able to lock tokens within an account that they own (transfer is not required to lock tokens)
* Users will be able to unlock an amount of tokens that are locked (when the lock is specified to be unlockable) this permits the lock to be used as a kind of secure account, adding a layer of protection for funds.
* Users will be able to vote using locked tokens (when the lock is specified to be votable)
* Adds a new endpoint which will support creation of locks that is called lock_tokens.
* Adds a new endpoint which will support cleaning of the system by BPs and users that is called clean_lock_table.
* Adds a new endpoint which will support cleaning of an accounts locks that is called clean_locks
* Adds an endpoint which will list the locks associated with a FIO public key called get_locks
* Modify the get_fio_balance endpoint to return the amount of locked tokens along with the balance.
* Modify the transfer of tokens and voting to perform the update of the locks for an account based on the specified periods.

## Motivation
The FIO Protocol includes [token locking functionality](https://kb.fioprotocol.io/fio-token/token-distribution#lockups-and-restrictions), which was built specifically to accommodate complex requirements for locking tokens [minted at Mainnet](https://kb.fioprotocol.io/fio-token/token-distribution#tokens-minted-at-mainnet). This functionality was not intended to be available post Mainnet.

However, there is a need to allow token locking post Mainnet for two primary use cases:
* The Foundation desires the ability to lock tokens it grants or sells to others.
* FIO token holders may be interested in locking tokens for security or other purposes.

## Specification
### Overview
#### What is locked?
When the lock action is executed, the target account (hashed from provided FIO Public Key) is subject to the lock, but only for the specified amount, meaning an account can have both locked and unlocked tokens. If lock action is targetting a different account then the signing actor, that account must not exist and will be created and funds will be transferred to that account.
#### What variables defne a lock?
To allow for most flexibility when tokens are locked the following variables define the type of lock:
* **Votable** - when set to 1, 100% of tokens can be voted, even when locked.
* **Unlockable** - when set to 1, the tokens can be unlocked by the account owner on demand.
* **Lock periods** - one or many time intervals at which specified percentage of tokens is unlocked.
#### What actions are disallowed for locked tokens in account?
* [trnsfiopubky](https://developers.fioprotocol.io/api/api-spec/reference/transfer-tokens-pub-key/transfer-tokens-pub-key-model) - when a transfer is initiated, only unlocked tokens in account can be transferred.
* [voteproducer](https://developers.fioprotocol.io/api/api-spec/reference/vote-producer/vote-producer-model)
	* If *votable* set to 0 only unlocked tokens are counted in vote.
	* If *votable* set to 1 all tokens in account are counted in vote, even when locked.
* Payment of fee. When fee is collected, only unlocked tokens may be used to pay for that fee. Bundled transactions can always be used, even when all tokens are locked.
#### How are tokens unlocked?
The accounting of all types locked tokens will be updated whenever transfer or voteproducer, or unlock are called by a user, so tokens will be unlocked whenever these calls are invoked. Users may have as many locks as they wish providing the sum of the locks does not exceed the balance of the account.
### New actions
#### Lock tokens
Locks tokens in designated account per provided lock schedule.
##### New end point: *lock_tokens* 
##### New action in new fio.lock contract locktokens
##### New fee: lock_tokens, 300000000 per unlock period + 300000000 per 90 days of longest duration.
##### RAM increase: 200 + (60 * number of lock periods) bytes
Example:
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
The fee will be: (3 unlock_periods * 300000000) + (1 90-day period in 259200 seconds * 300000000) = 600000000
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|lock_public_key|Yes|Valid FIO Public Key|FIO public key for account to be locked.|
|votable|Yes|0 is not votable, 1 is votable.|This indicates if the locked amount is votable while locked.|
|unlockable|Yes|0 is not unlockable, 1 is unlockable.|This indicates if the lock is unlockable.|
|unlock_periods|Yes|JSON Array of lock periods. See unlock_periods below. Min: 1, sum of percentage in all periods is 100%, duration in each period is greater than 0|Schedule by which tokens become unlocked.|
|amount|Yes|Int|The amount of tokens to lock in SUFs|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### unlock_periods
|Parameter|Required|Format|Definition|
|---|---|---|---|
|duration|Yes|Int|Seconds from lock creation to when unlock of coresponding percentage occurs.|
|percent|Yes|Percentage float|Percentage of locked tokens that unlock after coresponding duration lapsed.|
###### Example
```
{
	"lock_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
	"votable": 0,
	"unlockable": 0,
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
	],
	"amount": 40000000000000,
	"max_fee": 40000000000,
	"tpid": "rewards@wallet",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling
	* Require auth of the actor
	* Lock period verifictaion
		* Minimum of 1 period
		* Sum of percentage in all periods is 100%
		* Duration in each period is greater than 0
	* Verify the locking account has necessary balance.
	* Verify that the fee for this does not exceed the max fee specified.
	* Verify tx does not exceed max transaction size.
* Create account if account does not yet exist for the lock public key
* Charge appropriate fee or deduct bundled transaction
* Perform transfer, if not same account.
* Create entry in the locktokens table for these lock tokens.
* Increase RAM
* Return response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid lock public key|Specified public key is not valid FIO format.|400|"lock_public_key"|Value sent in, e.g. "notakey"|"Invalid Public Key."|
|Account already exist|Account hashed down from Public Key alreday exists AND is not the signing actor.|400|"lock_public_key"|Value sent in, e.g. "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj"|"Tokens can only be locked on actor's or new account."|
|Invalid votable|Value sent in is not 0 or 1|400|"votable"|Value sent in, e.g. "-100"|"Invalid votable value."|
|Invalid unlockable|Value sent in is not 0 or 1|400|"unlockable"|Value sent in, e.g. "-100"|"Invalid unlockable value."|
|Invalid unlock periods|See *Lock period verifictaion* in *Processing*|400|"unlock_periods"|"Invalid unlock_periods."|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
##### Response
|Parameter|Format|Definition|
|---|---|---|
|lock_id|Int|ID of lock created.|
|status|String|OK if successful|
|fee_collected|Int|Amount of SUFs collected as fee|
###### Example
```
{
	"lock_id": 5,
	"status": "OK",
	"fee_collected": 0
}
```

#### Unlock tokens
Unlocks tokens per lock schedule.
##### New end point: *unlock_tokens* 
##### New action in new fio.lock contract unlocktokens
##### New fee: unlock_tokens: 300000000
##### RAM increase: none
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|lock_amounts|Yes| string json string of lockids and amounts .|JSON data specifying what locks and amounts.|
|max_fee|Yes|max fee SUFs|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|actor|Yes|FIO account name|FIO account for the signer|
###### Lock amounts
this is a list of id, amount pairs, specifying the id of each lock and the amount to unlock for the lock matching the id.
###### Example
```
{
"lock_amounts": [ {
                    “id”: "5",
                    “amount: 1000000000000
                  }
                ],
  "actor": "aftyershcu22",
  "max_fee": 40000000000
}
```
##### Processing

When this request is processed, inputs will be verified,  the specified locks locking periods will be processed and the lock will be updated,
then the system will unlock the specified amount of tokens from the specified lock if the lock has the specified amount of tokens available in the remaining lock amount. 

* require auth of the actor
* Verify number of items in lock amounts, 1 or more.
* Verify format and values for lock_id, amount, max_fee, actor. invalid input error if incorrect.
* Verify the lock id exists in the locktokens table, error if none exist.
* Verify the owner of the lock matches the actor. signature error if no match
* perform the locking schedule for the lock (updating record of unlocked tokens)
* Verify the lock in question has a remaining_locked_amount >= amount else (insufficient locked funds for unlocking error)
* update the remaining locked amount -= amount
* verify tx does not exceed max transaction size.
* increase RAM limit by 200+(60*number of lock periods) bytes
* Return status json containing status (ok), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid lock id|Invalid format|400|"lock_id"|lock id invalid|"Invalid Lock ID"|
|Invalid Actor|Actor does not exist on chain|403|||"Invalid Actor"|
|Insufficient lock balance| Lockdoes not have required amount of funds|400|"amount"|Value sent in, e.g. "100"|"Insufficient lock balance"|
|Signature Error| owner of lock does not match actor specified|500|Signature Error|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||fee_collected|String|fee amount collected SUFs|
###### Example
```
{
  "status": "Ok",
  "fee_collected": 0        
}
```
## Fees
a new fee will be created for lock_tokens, 600000000 SUF per lock period, this is a mandatory fee.

#### get locks
##### New end point: *get_locks* 
##### New action in new fio.lock contract getlocks
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|fio public key, see FIO public key validation rules.|FIO public key to search locks.|
|offset|no|number greater or equal 0.|offset at which to begin results|
|limit|no|number greater than 0|number of results to display|
##### Processing
* hash the pub key.
* get the account from the fio account mapping
* ensure hash and account match.
* read the locktokens table by the specified account
* traverse all locks..
* add each lock to the results.
* provide paging for results.
* return list of locks.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No results found|No items found.|400|fio_pub_key|value of pub key|"No results"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||locks|String|json formatted list of locks|
###### Example
```
{
  "locks": 
          [
           {
            "owner": "edrgrdftrgyt"
            "lock_amount": 1000000000000
            "payouts_performed": 2
            "lock_id": 1
            "unlockable": 1
            "votable":1
            "periods": "lock_periods": [ {
                                       “duration”: 86400
                                       “percent: 33.3
                                       },
                                       {
                                       “duration”: 172800
                                       “percent: 33.3
                                       }, 
                                       {
                                       “duration”: 86400
                                       “percent: 33.3
                                       }
                                    ]
           "remaining_lock_amount" : 333333333333
           "timestamp":12344555
          }
         ],
  "more": 0
}
```

## Rationale
it was decided to provide locked funds to the receiving account because of the following reasons.
1) This places the security and ownership of the locked funds in the hands of the receiver instead of a FIO system account.
2) Since we have already learned how to effectively integrate the necessary logic to manage locked funds within transfer and voting within FIO, we can easily and reliably add this logic for these locks.


### Changes made to FIP after accepting as Draft
1. Removed clean token actions. This is consistant with our strategy Pre-Mainnet and in FIP (e.g. [FIP-8](https://github.com/fioprotocol/fips/blob/master/fip-8.md#burning-expired-requests) of not removing things from state, except when required for functionality (e.g. burn expired domains). I think we need a comprehensive approach to what gets removed when and how across all contracts. This way we do not end up with different strategy for removing locks and different for removing requests. There is alreday FIP planned to address this.
1. Locking of existing accounts not owned by signer will not be allowed, even if it's only for the amount being sent. Locks can only be applied to new account or account owned by signer. The reason being it can change an account of a user without that user's knowledge or consent. For example a User A may send 1 locked token to User B, without User B consenting to it. Now user B has locked tokens intermingled with unlocked tokens and that impacts their total and available token balance display, but they can't do anything about it.
1. Added TPID to lock_accounts. TPID should always be present on transactions expected to be exposed to users to encourgae implementation by wallets.
1. Added variable fee based on both number of locks and the longest duration.

#### Pending
1. Change get_balance
1. What's the purpose of *Unlockable*, if there is no waiting period? If a key is compromised, the attacker just has one extra action to run. Or was the assumption that the user would place unlock permission under the conrol of another account/key? Maybe better to start periods on first unlock.
1. accounting of all types locked tokens will be updated whenever transfer or voteproducer. Why should we only do it on unlock?
1. Do we need a fee on unlock?
1. Do we need a RAM bump on unlock?
1. Why does unlock require lock ID and amount. Shouldn't it jsut unlock all locks?
1. Should we even allow multiple locks, should there be a max?

## Implementation
* make a new system contract called fio.lock.
    make a new table locktokens
        int64 lockid   <primary index>     //this is the identifier of the lock
        name owner_account;   <secondary index>  //this is the account that owns the lock
        int64_t lock_amount = 0;        //this is the amount of the lock in FIO SUF
        int32_t payouts_performed = 0;  //this is the number of payouts performed thus far.
        int32 unlockable=0;   //this is the flag indicating if the lock is unlockable
        int32 votable = 0;   //this is the flag indicating if the lock is votable/proxy-able
        vector<lockperiods> periods;   // this is the locking periods for the lock
        int64_t remaining_lock_amount = 0; <secondary index>  //this is the amount remaining in the lock in FIO SUF,     get decremented as unlocking occurs.
        uint32_t timestamp = 0;   //this is the time of creation of the lock, locking periods are relative to this time.
        struct lockperiods     //this is the definition of locking periods.
        int64_t duration;   //this is the duration of the lock period, relative to the timestamp on the lock.
        double percent;   //this is the percent to unlock when the time period has passed.
        
* create new contract and new account, integrate accounting into the system 
    create contract, add tables and indexes to contract, and add to code base, add to dev launch, generate pub key for this account, add to system accounts in the FIO system account checks throughout the system, integrate accounting logic and check for can transfer into token contract, and voting. Affected files (fio.token.hpp, fio.token.cpp, voting.cpp) (2 days).
* Add new API end point for lock_tokens
  --modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code. Add new action (locktokens) to fio.lock.cpp and fio.lock.abi. dev test api endpoint and push action and resolve all issues (1 days)
  * Add new API end point for unlock_tokens
  --modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code. Add new action (locktokens) to fio.lock.cpp and fio.lock.abi. dev test api endpoint and push action and resolve all issues (1 days)
  * Add new API end point for clean_locks, clean_locks_table
  --modify chain_api_plugin to add new endpoint, modify chain_plugin.cpp and hpp to add new params and code. Add new action (locktokens) to fio.lock.cpp and fio.lock.abi. dev test api endpoint and push action and resolve all issues (1 days)
  Add new API end point for get_locks
  --modify chain_plugin cpp and hpp to add new params and code. dev test api endpoint and resolve all issues (1 days)
  --Modify get_fio_balance to return balance and locked:numberlockedtokens. (4 hours)
  Modify History plugin for lock_tokens, to ensure tx gets into block explorer (1 days)

#### Testing Considerations
* Create a lock verify the schedule is obeyed :  1 locking period, multiple locking periods.
* Add a new lock to an account that already has a genesis token grant.
* Make a votable lock, vote for producers.
* Make a votable lock, proxy this accounts vote to another, verify voting power.
* Make an unlockable lock, lock and unlock funds in this account and verify spend.
* Make multiple locks on one account. Verify locks are processed correctly.
