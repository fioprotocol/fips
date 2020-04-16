---
fip: 3
title: Enhanced privacy via friending
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-09
updated: 2020-04-16
---

## Abstract

## Terminology
* **Native Blockchain Public Address (NBPA)** - this is the public address on a native blockchain that is needed to send funds and is associated to the FIO Address using [/add_pub_address](https://developers.fioprotocol.io/api/api-spec/reference/add-pub-address/add-pub-address-model)
* **Payee** - is the user receiving funds. In the Send scenario, this is the user who places NBPA on the FIO Chain and allows Payer to see it so that the Payer can send funds using this NBPA. In Request scenario, this is the user sending a FIO Request.
* **Payer** - is the user sending funds using FIO Address. In the Send scenario, Payer will type a FIO Address in wallet, that wallet will look up the corresponding NBPA on native blockchain and transaction will be executed. In Request scenario, Payer will repons to a FIO Request sent by Payee.
* **Sender** - is the user sending a transaction on the FIO Chain to another user (Receiver)
* **Receiver** - is the user receiving a transaction on the FIO Chain from another user (Sender)

## Motivation
Even though contents of FIO Requests and OBT Records is already encrypted, there are two other areas where privacy of the FIO Protocol can be improved:
* NBPAs mapped to FIO Addresses are stored on-chain unencrypted. This allows blockchain observers to:
  * Associate NBPAs to identifiable FIO Addresses
  * Associate NBPAs on different blockchains belonging to a single individual/entity through a common FIO Address
* Anytime a FIO Request or OBT Record is stored on chain, the FIO Addresses of both parties are stored unencrypted. This allows blockchain observers to:
  * Associate FIO Addresses transacting with each other, even though the details of those transactions are private

## Specification
### Friend List
The core concept which accomplishes above objectives is a Friend List. When two users are friends, one can encrypt information (NBPA mappings, FIO Request, FIO Data, etc.) and put them on the chain in a structure which enables only the other party to find it and only the two parties to decrypt it.

This concept will allow the following:
* Place FIO Request, OBT Record on the blockchain with only one party visible by an observer.
* Place NBPAs mappings on the blockchain with only one party visible by an observer.

It is important to note that Friend List will exist between FIO Public Keys, not FIO Addresses, as past transactions should not be readable by the new owner of the FIO Address. This has the following consequences:
* Once user transfers their FIO Address to another public key (even if they control it), they will lose their whitelist. That is already a moot point, since they will not be able to decrypt the data anyway.
* Once whitelist is established it exists between all FIO Addresses owned by either party.

#### Adding a friend to a Friend List
* Receiver wants to allow Sender to send transactions to Receiver and types FIO Address into their wallet
* Receiver's wallet fetches FIO Public Key associated with FIO Address entered
* Receiver's wallet uses Receiver's FIO Private Key and Sender's FIO Public Key to derive a shared secret (Secret) using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange).
* Receiver's wallet records on the blockchain:
  * Sender's FIO Address and FIO Public Key encrypted symmetrically with Receiver's FIO Private Key. This will be used by Receiver to be able to restore their Friend List from wallet seed phrases without relying on local storage.
  * Sender's FIO Public Key hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector. This will be used as a **Look-up Index** by Sender to check if they are on whitelist or to identify a transaction that is intended for them.
 
![](images/Diagram-Adding-to-Friend-List.PNG)

#### Checking a Friend List
The Sender can check if they have been added to the Receiver's Friend List by hashing their own FIO Public Key using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector.
 
![](images/Diagram-Checking-Friend-List.PNG)

### Friend List actions
The following is a list of all new contract actions and endpoint added.
#### Add to Friend List. New action: *addfriend*; New endpoint: /priv_add_friend
Adds user to Friend List.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key_hash|Yes|String|FIO Public Key of party being added hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector. This will be used as a look-up index by Sender to check if they are on Friend List.|
|content|Yes|FIO public key|Encrypted blob. See below. This will be used by the owner of Friend List to be able to restore their Friend List from wallet seed phrases without relying on local storage.|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### *content* format
The content element is a packed and encrypted version of the following data structure:
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|String|FIO Address of the party being added.|
|public_key|Yes|IO Public Key|FIO Public Key of the party being added.|
###### Example
```
{
	"fio_public_key_hash": "515184318471884685485465454464846864686484464694181384",
	"content": "...",
	"max_fee": 0,
	"tpid": "",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling. Explicitly allowed:
	* User does not need to have a FIO Address registered, as Friend List on a Public Key level
* Fee is collected
* Content is placed on chain
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Key in Friend List|Supplied FIO Publick key is already in Friend List|400|"fio_public_key_hash"|Value sent in, i.e. "1000000000"|"FIO public key already in Friend List"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
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
	"fee_collected": 2000000000
}
```
#### Remove from Friend List. New action: *removefriend*; New endpoint: /priv_remove_friend
Removes user from Friend List.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key_hash|Yes|String|FIO Public Key of party being removed hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector.|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### Example
```
{
	"fio_public_key_hash": "515184318471884685485465454464846864686484464694181384",
	"max_fee": 0,
	"tpid": "",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling
* Fee is collected
* Record is removed from chain
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Key not in Friend List|Supplied FIO Publick key is not in Friend List|400|"fio_public_key_hash"|Value sent in, i.e. "1000000000"|"FIO public key not in Friend List"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
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
	"fee_collected": 2000000000
}
```
#### Get Friend List. New endpoint: /priv_get_friend_list
Returns Friend List.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|FIO public|FIO public key of owner.|
|limit|No|Positive Int|Number of records to return. If omitted, all records will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
|offset|No|Positive Int|First record from list to return. If omitted, 0 is assumed.|
###### Example
```
{
	"fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
	"limit": 100,
	"offset": 0
}
```
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Empty list|No items in friend list|404||"No items in friend list"|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|friends|fio_public_key_hash|String|See Add to Friend List|
|friends|content|String|See Add to Friend List|
||more|Int|Number of remaining results|
###### Example
```
{
    "friends": [
        {
            "fio_public_key_hash": "515184318471884685485465454464846864686484464694181384",
            "content": "...",
        }
    ],
    "more": 0
}
```
#### Check Friend List. New endpoint: /priv_check_friend_list
Checks if a user is in a Friend's list. Can be called by either party.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key_hash|Yes|FIO public|See Add to Friend List|
###### Example
```
{
	"fio_public_key_hash": "515184318471884685485465454464846864686484464694181384"
}
```
##### Response
|Parameter|Format|Definition|
|---|---|---|
|is_friend|Int|1 - in Friend's List; 0 - not in Friend's List|
###### Example
```
{
	"is_friend": 1
}
```

### NBPA mappings
In existing implementation, NBPAs are placed on FIO Chain unencrypted. Friend List functionality can be leveraged to store NBPAs privately.

In order to offer the most flexibility to users and to reduce the amount of content stored on the FIO Chain, each NBPA is encrypted symmetrically three separate times using different random secret key each time:
* FIO Address level secret key – used to encrypt all NBPAs associated with that FIO Address.
* Chain level secret key – used to encrypt all NBPAs for a particular chain (i.e. Ethereum).
* NBPA level secret key – only used to encrypt one NBPA.

NBPA level secret key can decrypt just one NBPA. Chain level secret key can decrypt current public addresses for specific chain and all future NBPAs for that chain published by the owner. FIO Address level secret key can decrypt all NBPAs associated with that FIO Address. The FIO Address owner can then decide which of these decrypt keys to make available to which friend by placing them on the FIO Chain encrypted.

#### Placing NBPA on chain
Each NBPA will be encrypted using three unique secret keys. Method of deriving secret keys (where + is string concatenation):
```
HMAC(hashName, secret, hash) == result
HMAC('sha256', WalletSecret, 'FIO Address level secret key') == NBPA level secret key
HMAC('sha256', WalletSecret, 'Blockchain level secret key') == Blockchain level secret key
HMAC('sha256', WalletSecret, ''NBPA secret key' + AddressPublicKey) == NBPA level secret key
```
The NBA will then be placed on the FIO Chain using new action as three distinct encrypted blobs. The data includes:
* Chain code
* NBPA encrypted with FIO Address level secret key - this secret will be used to encrypt all NBPAs placed by that FIO Address
* NBPA encrypted with Chain level secret key - this secret will be used to encrypt all NBPAs of a particular chain, e.g. Ethereum.
* NBPA encrypted with NBPA level secret key - this secret is different for every NBPA

![](images/Diagram-Placing-NBPA.PNG)

#### Placing secret decrypt key on chain
A secret key is placed on the FIO Chain for specific Payer using new action. The data includes:
* Look-up Index - Payee derives the same index as used for Adding a friend to a Friend List. It allows the Payer to get the key intended for them.
* Chain code (if secret key is chain level)
* Key ID (if secret key is for specific key)
* Token code
* Secret Key encrypted asymmetrically using the friend’s public key to derive shared secret using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange).

#### Looking up NBPA
Payer looks for NBPA using:
* Look-up index - Payer derives the same index as used for Adding a friend to a Friend List.
* FIO Address
* Chain code
* Token code

The blockchain will look for secret key with provided Look-up index and chain and token code and return correct level encrypted NBPA and corresponding Encrypted Secret Key if found.

##### Example
* Payee places NBPA as follows:
  * NBPA ID: 1
  * Blockchain code: BTC
  * Token code: BTC
  * NBPA encrypted with FIO Address level secret key (Secret 1): ABC
  * NBPA encrypted with Chain level secret key (Secret 2): DEF
  * NBPA encrypted with NBPA level secret key (Secret 3): GHI
* Payee places secret for Payer 1 to decrypt all BTC addresses:
  * Look-up index: index_for_payer_1
  * Blockchain code: BTC
  * Key ID: 1
  * Encrypted Secret Key: XXX
* Payee places secret for Payer 2 to decrypt just one BTC address:
  * Look-up index: index_for_payer_2
  * Blockchain code: BTC
  * Key ID: 1
  * Encrypted Secret Key: YYY
* Payer 1 sends index_for_payer_1 and BTC/BTC and receives:
  * XXX - Payer 1 decrypts Secret 2 with their private key
  * DEF - Payer 1 decrypts NBPA with Secret 2
* Payer 2 sends index_for_payer_2 and BTC/BTC and receives:
  * YYY - Payer 2 decrypts Secret 3 with their private key
  * GHI - Payer 2 decrypts NBPA with Secret 3

### NBPA mappings actions
The following is a list of all new contract actions and endpoint added.
#### Add NBPA. New action: *privaddadr*; New endpoint: /priv_add_pub_address
Adds NBPA to chain.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|String|FIO Address for which NBPA is being added.|
|public_addresses|Yes|JSON Array. See public_addresses below Min: 1 item Max: 5 items|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### *public_addresses* format
|Parameter|Required|Format|Definition|
|---|---|---|---|
|chain_code|Yes|String|Chain code of NBPA being added, not encrypted.|
|token_code|Yes|String|Token code of NBPA being added, not encrypted.|
|public_key_address_level|Yes|String|NBPA encrypted with FIO Address-level secret.|
|public_key_chain_level|Yes|String|NBPA encrypted with chain-level secret.|
|public_key_nbpa_level|Yes|String|NBPA encrypted with NBPAs-level secret.|
###### Example
```
{
	"fio_address": "purse@alice",
	"public_addresses": [
		{
			"chain_code": "BTC",
			"token_code": "BTC",
			"public_key_address_level": "jK4P6yxXwoZjCz4xodEVMY63PZQEvs1GVRUSusohSpFzO8mBegxGE1Zl7yYqxwqHIWYrxihOxJya51QMVzyJVv8z2S2fnWUB",
			"public_key_chain_level": "5zsyqCjsZ9bAGfQybQKIGZL36UQldukmoJMq410SQzIVeSus1dozqGZu4X8Fh0qzHioAL4dwIkORz7cG8XAtQ1ZOG9vaK9su",
			"public_key_nbpa_level": "uE98xys0iDlmJyPdrTof09iZbqxIJEDXZC6KlyRdtgdgrt2E1XWUr04h9JE7CvtGCHDan2G58x0BzzXi7NnsTyCaSi05ird1"
		},
		{
			"chain_code": "ETH",
			"token_code": "USDC",
			"public_key_address_level": "cXg5EEs5i0wqR5K0eskMo02Q3Tsqpt0VCeU9MPAkvlPC3g5cJd690aeEmBlkHsvOGTEI7KiQZ7W4kWfwaOCTq8rDja3p6kRw",
			"public_key_chain_level": "GUzY0kzw61nFChs9EWVear5oHNRLQRci2qbv4kQQNztg8JY6Q3atEnKYrcdfDWr4HzmXx23hDfZfWbw4LjwyFSlZyOqc6z3G",
			"public_key_nbpa_level": "ddegiZfBqENdtVWGsgzeDsSJHlQGwmlTtUxpzEcI4hkkbVhHbR96ow9tPD1EALLnqigl9hF4scxt9qPfCh1bavPWLN9X1jie"
		}
	],
	"max_fee": 0,
	"tpid": "rewards@wallet",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling.
* Fee is collected
* Content is placed on chain
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid chain format|Supplied chain code is not a valid format.|400|"chain_code"|Value sent in, i.e. "BTC!@#$%^&\*()"|"Invalid Chain Code"|
|Invalid token format|Supplied token code is not a valid format.|400|"token_code"|Value sent in, i.e. "BTC!@#$%^&\*()"|"Invalid Token Code"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|Not owner of FIO Address|The signer does not own the FIO Address|403||||
|FIO Address not found|Supplied FIO Address cannot be found|404||||
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if successful|
|key_id|Int|ID of key|
|fee_collected|Int|Amount of SUFs collected as fee|
###### Example
```
{
	"status": "OK",
	"key_id: 1000000,
	"fee_collected": 2000000000
}
```
#### Grant NBPA access. New action: *privganbpa*; New endpoint: /priv_grant_pub_address_access
Grants NBPA access to specific friend by placing a decrypt secret for them.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key_hash|Yes|String|FIO Public Key of friend being granted access hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector.|
|decrypt_keys|Yes|JSON Array. See decrypt_keys below Min: 1 item Max: 5 items|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### *decrypt_keys* format
|Parameter|Required|Format|Definition|
|---|---|---|---|
|level|Yes|String|Level of decryption key: address, chain, nbpa|
|level_index|Yes|String|Level index: for address-level: FIO Address, for chain-level: chain code, for nbpa: key ID|
|decrypt_key|Yes|String|Encrypted key|
###### Example
```
{
	"fio_public_key_hash": "515184318471884685485465454464846864686484464694181384",
	"decrypt_keys": [
		{
			"level": "chain",
			"level_index": "BTC",
			"decrypt_key": "5zsyqCjsZ9bAGfQybQKIGZL36UQldukmoJMq410SQzIVeSus1dozqGZu4X8Fh0qzHioAL4dwIkORz7cG8XAtQ1ZOG9vaK9su"
		},
		{
			"level": "nbpa",
			"level_index": "1000",
			"decrypt_key": "GUzY0kzw61nFChs9EWVear5oHNRLQRci2qbv4kQQNztg8JY6Q3atEnKYrcdfDWr4HzmXx23hDfZfWbw4LjwyFSlZyOqc6z3G"
		}
	],
	"max_fee": 0,
	"tpid": "rewards@wallet",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling.
* Fee is collected
* Content is placed on chain
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Key not in Friend List|Supplied FIO Public key is not in Friend List|400|"fio_public_key_hash"|Value sent in, i.e. "1000000000"|"FIO public key not in Friend List"|
|Invalid level format|Supplied level not a valid format.|400|"level"|Value sent in, i.e. "something|"Invalid level."|
|Invalid chain format|Supplied chain code is not a valid format.|400|"level_index"|Value sent in, i.e. "BTC!@#$%^&\*()"|"Invalid Chain Code"|
|Invalid nbpa id format|Supplied nbpa id is not valid or key does not belong to signer.|400|"level_index"|Value sent in, i.e. "BTC!@#$%^&\*()"|"Invalid Chain Code"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|Not owner of FIO Address|The signer does not own the FIO Address|403||||
|FIO Address not found|Supplied FIO Address cannot be found|404||||
##### Response
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if successful|
|fee_collected|Int|Amount of SUFs collected as fee|
###### Example
```
{
	"status": "OK",
	"fee_collected": 2000000000
}
```
#### Get NBPA. New endpoint: /priv_get_pub_address
Looks up NBPA for supplied FIO Address and chain code.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key_hash|Yes|String|FIO Public Key of caller hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector.|
|chain_code|Yes|Chain code for requested NBPA.|
|token_code|Yes|Token code for requested NBPA.|
###### Example
```
{
	"fio_public_key_hash": "515184318471884685485465454464846864686484464694181384",
	"fio_address": "purse@alice",
	"chain_code": "FIO",
	"token_code": "FIO"
}
```
##### Processing
* Request is validated per Exception handling.
* NBPA is looked up using:
  * Provided FIO Address
  * Chain code
  * Token code
* Decrypt key is looked up using fio_public_key_hash and appropriate decrypt key is returned:
  * if nbpa-level decrypt key exists, it is returned
  * if chain-level decrypt key exists, it is returned
  * if address-level decrypt key exists, it is returned
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Public Address not found|There was no matching NBPA and/or matching decrypt key|404|||"Public address not found"|
##### Response
|Parameter|Format|Definition|
|---|---|---|
|public_address|String|Encrypted public key.|
|decrypt_key|String|Encrypted decrypt key.|
###### Example
```
{
	"public_address": "GUzY0kzw61nFChs9EWVear5oHNRLQRci2qbv4kQQNztg8JY6Q3atEnKYrcdfDWr4HzmXx23hDfZfWbw4LjwyFSlZyOqc6z3G",
	"decrypt_key": "ddegiZfBqENdtVWGsgzeDsSJHlQGwmlTtUxpzEcI4hkkbVhHbR96ow9tPD1EALLnqigl9hF4scxt9qPfCh1bavPWLN9X1jie"
}
```

### FIO Request and FIO Data
#### New Private Funds Request
* Payee initiate a new private funds request to Payer (types FIO Address into wallet)
* Payee's wallet checks to see if Payee is added to Payer's whitelist:
  * Fetches FIO Public Key associated to Payer FIO Address
  * Uses Payee's FIO Private Key and Payer's FIO Public Key to derive a shared secret (Secret) using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange).
  * Computes look-up index
    * Payee's FIO Public Key hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector.
  * Check blockchain if look-up index is associated with Payer's FIO Address
* Payee's wallet submits new private funds request with:
  * Unencrypted:
    * Payee FIO Public Key. This will allow Payee to locate their own requests.
    * Payee's FIO Address. This will be used to deduct bundled transactions.
    * Auto-generated FIO Request ID same as now.
  * Hashed:
    * Payer's FIO Public Key hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payer's FIO Public Key as initialization vector. This will be used by Payer to find requests that are for them.
  * Encrypted with Secret + random IV
    * Payer FIO Address
    * Request details (chain, amount, memo)
* Payee should also add Payer to their own whitelist at the same time

![](images/Diagram-New-Funds-Request.PNG)

#### Fetching New Funds Request
* Payer's wallet will compute look-up index by hashing Payer's FIO Public Key using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payer's FIO Public Key as initialization vector.

![](images/Diagram-Fetching-Funds-Request.PNG)

#### Rejecting Funds Request
* Payer's wallet records new action with:
  * Unencrypted:
    * Payer's FIO Public Key. This will allow Payer to locate their own responses.
    * Payer's FIO Address. This will be used to deduct bundled transactions.
  * Hashed:
    * Payee's FIO Public Key hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payee's FIO Public Key as initialization vector. This will be used by Payee to find responses.
  * Encrypted with Secret + IV
    * FIO request ID
    * Status = rejected
    
![](images/Diagram-Rejecting-Funds-Request.PNG)

#### Recording OBT Data aka accepting funds request
* Payer's wallet records new action with:
  * Unencrypted:
    * Payer's FIO Public Key. This will allow Payer to locate their own responses.
    * Payer's FIO Address. This will be used to deduct bundled transactions
  * Hashed:
    * Payee's FIO Public Key hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payee's FIO Public Key as initialization vector. This will be used by Payee to find responses.
  * Encrypted with Secret + IV:
    * FIO request ID (if in response to request)
    * Status = sent_to_blockchain
    * Payer FIO Address
    * Payee FIO Address
    * Send details (chain, amount, memo)
    
#### Fetching sent FIO Requests
* Payee sends their FIO public key, which is unencrypted on chain

#### Fetching send actions
* This is a new construct which returns all send actions, e.g. request rejections or recorded OBT content. Since the details are now encrypted, it's not possible to get the status together with Funds Requests.
* Payee's wallet will compute look-up index by hashing Payer's FIO Public Key using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payer's FIO Public Key as initialization vector use it to fetch:
  * Reject messages
  * Record OBT messages
* The Payee's wallet is responsible for putting together all requests from get_sent_fio_requests with their associated statuses

### FIO Request and FIO Data actions
#### New funds request. New action: *privfundsreq*; New endpoint: /priv_new_funds_request 
New Funds Request when utilizing Friend List.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|payee_fio_address|Yes|FIO Address|FIO Address of the payee.|
|payee_fio_public_key|Yes|FIO Public Key|FIO Public Key of the payee.|
|lookup_index_for_payer|Yes|String|Derived as follows: Payee's FIO Private Key and Payer's FIO Public Key to derive a shared secret (Secret) using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). Payer's FIO Public Key is then hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payer's FIO Public Key as initialization vector. This will be used by Payer to find requests that are for them.|
|content|Yes|FIO public key|Encrypted blob. Same as content element of [/new_funds_request](https://developers.fioprotocol.io/api/api-spec/reference/new-funds-request/new-funds-request-model)|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### Example
```
{
	"payee_fio_address": "crypto@bob",
	"payee_fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
	"lookup_index_for_payer": "515184318471884685485465454464846864686484464694181384",
	"content": "...",
	"max_fee": 0,
	"tpid": "",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling
* Fee is collected
* Content is placed on chain
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Address|Format of FIO Address not valid or FIO Address does not exist.|400|"payee_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Address invalid or does not exist."|
|FIO Address expired|Supplied FIO Address has expired|400|"payee_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Address expired."|
|FIO Domain expired|Domain of supplied FIO Address has expired more than 30 days ago|400|"payee_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Domain expired."|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|Not owner of FIO Address|The signer does not own the FIO Address|403||||
##### Response
|Parameter|Format|Definition|
|---|---|---|
|fio_request_id|Int|ID of FIO Request created.|
|status|String|OK if successful|
|fee_collected|Int|Amount of SUFs collected as fee|
###### Example
```
{
	"fio_request_id": "10",
	"status": "OK",
	"fee_collected": 0
}
```
#### Records send action. New action: *privrecsend*; New endpoint: /priv_record_send_action 
This action is made to record information about a send transaction. Because content is encrypted, two calls used in unencrypted mode ([/record_obt_data](https://developers.fioprotocol.io/api/api-spec/reference/record-obt-data/record-obt-data-model) and [/reject_funds_request](https://developers.fioprotocol.io/api/api-spec/reference/reject-funds-request/reject-funds-request-model)) are combined into a single action.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|payer_fio_address|Yes|FIO Address|FIO Address of the payer.|
|payer_fio_public_key|Yes|FIO Public Key|FIO Public Key of the payer.|
|lookup_index_for_payee|Yes|String|Derived as follows: Payer's FIO Private Key and Payee's FIO Public Key to derive a shared secret (Secret) using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). Payee's FIO Public Key is then hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payee's FIO Public Key as initialization vector. This will be used by Payee to find send actions that are for them.|
|content|Yes|FIO public key|Encrypted blob. Same as content element of [/record_obt_data](https://developers.fioprotocol.io/api/api-spec/reference/record-obt-data/record-obt-data-model)|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### Example
```
{
	"payer_fio_address": "purse@alice",
	"payer_fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
	"lookup_index_for_payee": "515184318471884685485465454464846864686484464694181384",
	"content": "...",
	"max_fee": 0,
	"tpid": "",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling
* Fee is collected
* Content is placed on chain
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Invalid FIO Address|Format of FIO Address not valid or FIO Address does not exist.|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Address invalid or does not exist."|
|FIO Address expired|Supplied FIO Address has expired|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Address expired."|
|FIO Domain expired|Domain of supplied FIO Address has expired more than 30 days ago|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Domain expired."|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Insufficient funds to cover fee|Account does not have enough funds to cover fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Insufficient funds to cover fee"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|Not owner of FIO Address|The signer does not own the FIO Address|403||||
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
#### Get received FIO Requests. New endpoint: /priv_get_received_fio_requests 
Requests call polls for any requests sent to a receiver by a specified sender. Because status is now encrypted, it's no longer possible to return only pending FIO Requests, like it is in current [/get_pending_fio_requests](https://developers.fioprotocol.io/api/api-spec/reference/get-pending-fio-requests/get-pending-fio-requests). It will now be up to the wallet to decrypt each request and obtain status.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|lookup_indexes_for_payer|Yes|JSON Array|Array of key hashes. Derived as follows: Payer's FIO Private Key and Payee's FIO Public Key to derive a shared secret (Secret) using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). Payer's FIO Public Key is then hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payer's FIO Public Key as initialization vector.|
|limit|No|Positive Int|Number of requests to return. If omitted, all requests will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
|offset|No|Positive Int|First request from list to return. If omitted, 0 is assumed.|
###### Example
```
{
	"lookup_indexes_for_payer": [
		"515184318471884685485465454464846864686484464694181384",
		"515184318471884685485465454464846864686484464694181385",
		"515184318471884685485465454464846864686484464694181386",
		"515184318471884685485465454464846864686484464694181387"
	],
	"limit": 100,
	"offset": 0
}
```
##### Processing
* Request is validated per Exception handling
* Return *limit* FIO Requests starting at *offset* where payee in *lookup_indexes_for_payer*.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No FIO Requests|No requests were found|404|||"No FIO Requests."|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|requests|payee_fio_address|String|FIO Address of the payee.|
|requests|payee_fio_public_key|String|FIO Public Key of the payee.|
|requests|lookup_index_for_payer|String|See priv_new_funds_request|
|requests|content|String|Encrypted blob. See priv_new_funds_request|
||more|Int|Number of results remaining|
###### Example
```
{
	"requests": [
		{
			"payee_fio_address": "crypto@bob",
			"payee_fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
			"lookup_index_for_payer": "515184318471884685485465454464846864686484464694181384",
			"content": "..."
		}
	],
	"more": 0
}
```
#### Get sent FIO Requests. New endpoint: /priv_get_sent_fio_requests 
Sent requests call polls for any requests sent by provided FIO public key.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_public_key|Yes|String|FIO public key of Payee.|
|limit|No|Positive Int|Number of requests to return. If omitted, all requests will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
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
* Return *limit* FIO Requests starting at *offset* where payee's FIO Public Key is *fio_public_key*.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No FIO Requests|No requests were found|404|||"No FIO Requests."|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|requests|payee_fio_address|String|FIO Address of the payee.|
|requests|payee_fio_public_key|String|FIO Public Key of the payee.|
|requests|lookup_index_for_payer|String|See priv_new_funds_request|
|requests|content|String|Encrypted blob. See priv_new_funds_request|
||more|Int|Number of results remaining|
###### Example
```
{
	"requests": [
		{
			"payee_fio_address": "crypto@bob",
			"payee_fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
			"lookup_index_for_payer": "515184318471884685485465454464846864686484464694181384",
			"content": "..."
		}
	],
	"more": 0
}
```
#### Get information about actions received. New endpoint: /priv_get_received_actions 
Requests call polls for any actions taken by payer on the payee. Because status is now encrypted, OBT Data and status is now combined in same call. It will now be up to the wallet to decrypt each action and obtain status.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|lookup_indexes_for_payer|Yes|JSON Array|Array of key hashes. Derived as follows: Payee's FIO Private Key and Payer's FIO Public Key to derive a shared secret (Secret) using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). Payer's FIO Public Key is then hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret + Payer's FIO Public Key as initialization vector.|
|limit|No|Positive Int|Number of actions to return. If omitted, all actions will be returned. Due to table read timeout, a value of less than 1,000 is recommended.|
|offset|No|Positive Int|First action from list to return. If omitted, 0 is assumed.|
###### Example
```
{
	"lookup_indexes_for_payer": [
		"515184318471884685485465454464846864686484464694181384",
		"515184318471884685485465454464846864686484464694181385",
		"515184318471884685485465454464846864686484464694181386",
		"515184318471884685485465454464846864686484464694181387"
	],
	"limit": 100,
	"offset": 0
}
```
##### Processing
* Request is validated per Exception handling
* Return *limit* actions starting at *offset* where payer in *lookup_indexes_for_payer*.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No Received Actions|No requests were found|404|||"No Received Actions."|
##### Response
|Group|Parameter|Format|Definition|
|---|---|---|---|
|actions|payer_fio_address|String|FIO Address of the payer.|
|actions|payer_fio_public_key|String|FIO Public Key of the payer.|
|actions|lookup_index_for_payer|String|See priv_record_send_action|
|actions|content|String|Encrypted blob. See priv_record_send_action|
||more|Int|Number of results remaining|
###### Example
```
{
	"actions": [
		{
			"payer_fio_address": "crypto@bob",
			"payer_fio_public_key": "FIO8PRe4WRZJj5mkem6qVGKyvNFgPsNnjNN6kPhh6EaCpzCVin5Jj",
			"lookup_index_for_payer": "515184318471884685485465454464846864686484464694181384",
			"content": "..."
		}
	],
	"more": 0
}
```

## Rationale
## Other approaches considered
FIO Private Messaging
Make calls using non-linked keys
Randomize FIO Addresses

### Why do we encrypt a NBPA with three secret keys?
This was meant to be an alternative to encrypting NBPA for each user separately. For example, if a user has placed 20 NBPAs on the FIO Chain for Payer 1 and they now wanted to give the same access to Payer 2, they would need to place 20 new entries on the FIO Chain. With secret keys, they will place the 20 NBPAs once and then just give one secret key to Payer 1 and 1 secret key to Payer 2, reducing the amount of data that needs to be stored on chain.

### Deriving secret keys
We've considered using a SLIP-44 derivation path for those keys, but it does not seem like a good approach, because there would still need to be some index that matched the keys to records on the FIO blockchain.

## Implementation
## Backwards compatibility
To support backwards compatibility, every FIO Address will elect to be *private* or *public*. If *public* is elected, the FIO Address will continue to function as is. If *private* is elected, it will only support new private actions and all public action will respond with advisory that this address only supports private mode. To support existing users, all new FIO Addresses will be set to public by default.

## Future considerations
