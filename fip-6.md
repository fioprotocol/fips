---
fip: 6
title: Provide locked token capability for use by foundation and FIO users
status: Draft
type: Functionality
author: Ed Rotthoff <ed@dapix.io>
created: 2020-04-16
updated: 
---

## Background
There is already a set of locks that have been created for FIO genesis, these locks were adequate for the main net launching process, and there is no need to change these locks. These locks and their behavior will remain entact. As we all know changing tables requires migration, and playback considerations and so we avoid this as possible going forward....The foundation has reqeusted the ability to perform the following operations. Transfer tokens to another account, and have these tokens be locked according to a set of unlocking periods. The locked tokens may be votable/proxy-able or not depending on the situation. Secondly it is seen as desirable to lock tokens (that are not presently locked) that are in the token owners account, these locked tokens should be votable or not depending on how the account owner wants these tokens to influence the vote,  and these tokens can have an unlocking schedule/milestone, also the token owner should be able to unlock any amount of the locked tokens necessary as my needs change. The design of these locks should provide a flexible way to permit the FIO genesis locks to function along with any number of locks of the new type in an account. The number of locks an account has should not be restricted other than the total of the locks cannot exceed the balance of the account. The locks schedule, votable attribute,  and unlockable attribute should be specified at lock creation to provide a flexible and configurable locking capability that does not impede any of the existing locks in the FIO protocol.  Each record in the lock table identifies the amount of the lock, locks can be added as much as the user wishes until the amount to be locked is more than the remaining balance. Locked tokens can be voted as long as the lock is made votable and the account in question has bundled tx remaining, or the amount of the fee in the unlocked balance. If the user votes for example, and the bundle vote count is expired, then if the balance of the account does not meet the required fees the user will not be able to vote or do other actions on the blockchain. The unlocked balance of the user account is equal to the amount of all the locks the user has of all kinds minus the balance in the account. 


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

Presently the FIO API provides a fixed set of locked token types which have been used at FIO genesis, the types of locks and associated characteristics of locks were implemented in code without the ability to configure the locks without re-coding the locking logic. We make a new data model for a lock which consists of a set of locking periods which are relative to the time of creation of the lock, and flags relating to the lock (votable, unlockable). These new locks do not relate to the "main net" locks used to provide tokens to FIO investors and employees at FIO genesis. New locks and old locks will all be processed with equal precedence according to the rules for the lock. :
* These new locks will be used by the FIO foundation to sell or grant FIO tokens.
* These can be used by users of the protocol to lock tokens within their own accounts (to protect from spending, and also to protect against hacking into accounts)
* These new locks can be used by FIO users to transfer locked tokens to other users.
* There should be maintenance calls provided to clean up state for the locks that are paid out.
* Multiple locks may be specified on a single account as long as the locks do not exceed the balance of the account. When a lock is added to an account the unlocked balance is checked to determine if the lock is to be permitted.
* The older type of locks used at main net genesis can co-exist with these new locks in accounts that already have a genesis token grant. both will be handled correctly.

## Specification
### Lock tokens
#### New end point: *lock_tokens* 
#### New action in new fio.lock contract locktokens
##### Request
When this request is processed, tokens will be transferred to the specified receiver public key from the actor account if these accounts are different, if they are the same, no transfer will take place, these tokens wil be locked, they can be voted if the lock specifies they are votable, they cannot be used for fees, or transferred until unlocked according to the specified schedule or unlocked (if the lock is unlockable). locked and unlocked funds may be intermixed within an accountthe unlocked funds will be usable as normal within the FIO protocol  (so fees can come from bundled transactions, and from these intermingled fees) . The accounting of all types locked tokens will be updated whenever transfer or voteproducer, or unlock are called by a user, so tokens will be unlocked whenever these calls are invoked. Users may have as many locks as they wish providing the sum of the locks does not exceed the balance of the account.
|Parameter|Required|Format|Definition|
|---|---|---|---|
|receiver_public_key|Yes|fio public key, see FIO public key validation rules.|FIO public key which will receive the locked tokens.|
|votable|Yes|0 is not votable, 1 is votable.|This indicates if the locked amount is votable.|
|unlockable|Yes|0 is not unlockable, 1 is unlockable.|This indicates if the lock is unlockable.|
|lock_periods|Yes|JSON Array of periods. See periods below. Min: 1, must total 100% |The locking periods for this token lock.|
|amount|Yes|amount SUFs| The amount of tokens to lock in SUFs|
|max_fee|Yes|max fee SUFs|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by /get_fee for correct value.|
|actor|Yes|FIO account name|FIO account for the signer|

###### Lock Periods
a set of locking periods is defined for each lock. the locking periods define the duration of the period, which is relative to the creation time of the lock, and a percent to unlock when the duration expires, durations are processed in the order specified with the earliest durations being first and the latest durations being last in the list of durations. Each percent is specified as a decimal percent.

###### Example
```
{
    "receiver_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
    "votable":0,
    "unlockable":0,
    "lock_periods": [ {
                        “duration”: 86400
                        “percent: 1.2
                      }, {
                        “duration”: 172800
                        “percent: 90.8
                      }, {
                        “duration”: 259200
                        “percent: 8.0
                      }
                    ],
    "actor": "aftyershcu22",
    "amount": 40000000000000
    "max_fee": 40000000000
}
```
##### Processing
* require auth of the actor
* Verify the receiver public key.
* Verify the votable value.
* Verify the lock periods
      Minimum of 1
      check that total is 100%. 
      check that duration is greater than 0
* Create account if account does not yet exist for the receiver public key
*  Verify the locking account has necessary balance.
* verify that the fee for this does not exceed the max fee specified.
* charge appropriate fee (this will be a bundled fee transaction, fee will be based on number of periods specified)
* check for locking in the same account, if not then check for account creation, and perform transfer.
* create entry in the locktokens table for these lock tokens.
* verify tx does not exceed max transaction size.
* increase RAM limit by 200+(60*number of lock periods) bytes
* Return status json containing status (ok), and fee charged.
##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid receiver public key|Invalid format|400|"receiver_public_key"|value of public key|"Invalid Public Key"|
|Invalid votable|Invalid format|400|"votable"|votable value|"Invalid votable value"|
|Lock Periods Invalid|less of equal to 0 duration specified on any lock period, or total percent does not sum to 100.0|400|"lock_periods"|Invalid Lock Periods |
|Invalid Actor|Actor does not exist on chain|403|||"Invalid Actor"|
|Insufficient balance| Actor does not have required amount of funds|400|"amount"|Value sent in, e.g. "100"|"Insufficient balance"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||fee_collected|String|fee amount collected SUFs|
|lock_id| string| the id of the lock that was created|
###### Example
```
{
  "lock_id": "5"
  "status": "OK",
  "fee_collected": 0		
}
```
## Fees
a new fee will be created for lock_tokens, 600000000 SUF per lock period, this is a mandatory fee.

## Specification
### UnLock tokens
#### New end point: *unlock_tokens* 
#### New action in new fio.lock contract unlocktokens
##### Request
When this request is processed, inputs will be verified,  the specified locks locking periods will be processed and the lock will be updated,
then the system will unlock the specified amount of tokens from the specified lock if the lock has the specified amount of tokens available in the remaining lock amount. 
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


### Clean lock table
#### New end point: *clean_lock_table* 
#### New action in new fio.lock contract cleanlocktbl
##### Request
This request can be used by BPs to perform the cleanup of locks in a system wide fashion. 
|Parameter|Required|Format|Definition|
|---|---|---|---|


##### Processing
* require auth fio system accounts.account == fioio::MSIGACCOUNT || account == fioio::WRAPACCOUNT || account == fioio::SYSTEMACCOUNT || account == fioio::ASSERTACCOUNT || account == fioio::REQOBTACCOUNT || account == fioio::FeeContract || account == fioio::AddressContract || account == fioio::TPIDContract || account == fioio::TokenContract || account == fioio::TREASURYACCOUNT || account == fioio::FIOSYSTEMACCOUNT || account == fioio::FIOACCOUNT
* read the locktokens table by remaining_locked_amount
* if first entry has remaining locked amount > 0 throw no work exception.
* traverse list until remaining locked amount > 0 or 50 entries removed.
* remove entry with remaining locked amound == 0
* return status ok and removed_count

##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No work|No items to remove.|400|||"No work"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||removed_count|int64 |number of items removed|
###### Example
```
{
   "status": "Ok",
   "removed_count": 25        
}
```

### Clean locks
#### New end point: *clean_locks* 
#### New action in new fio.lock contract cleanlocks
, 
##### Request
This is an call that will will clean the no longer useful (granted) locks for the signing actor, this allows users to clean up their exisitng locks for better performance in transfer and voting.
|Parameter|Required|Format|Definition|
|---|---|---|---|
|actor|Yes|FIO account name|FIO account for the signer|


##### Processing
* require auth actor.
* read the locktokens table by the specified account
* traverse all locks..
* if entry has remaining locked amount > 0 leave this lock.
* remove entry with remaining locked amount == 0
* remove up to but not exceeding 100 locks.
* throw no work error if no locks found to remove.
* allow this to be called once per day.
* return status ok and removed_count

##### Response
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No work|No items to remove.|400|||"No work"|
|time violation|time between calls exceeded.|400|||"Time violation"|

##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
||status|String|Ok|
||removed_count|int64|number of items removed|
###### Example
```
{
"status": "Ok",
"removed_count": 25        
}
```

### get locks
#### New end point: *get_locks* 
#### New action in new fio.lock contract getlocks
, 
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
  
  
     
     
     
     
     
     
    
    
  
