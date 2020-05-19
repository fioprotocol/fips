---
fip: 5
title: Enhanced privacy
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-09
updated: 2020-05-19
---

## Terminology
* **Native Blockchain Public Address (NBPA)** - this is the public address on a native blockchain that is needed to send funds and is associated to the FIO Address using [/add_pub_address](https://developers.fioprotocol.io/api/api-spec/reference/add-pub-address/add-pub-address-model)
* **Payee** - is the user receiving funds. In the Send scenario, this is the user who places NBPA on the FIO Chain and allows Payer to see it so that the Payer can send funds using this NBPA. In Request scenario, this is the user sending a FIO Request.
* **Payer** - is the user sending funds using FIO Address. In the Send scenario, Payer will type a FIO Address in wallet, that wallet will look up the corresponding NBPA on native blockchain and transaction will be executed. In Request scenario, Payer will repons to a FIO Request sent by Payee.
* **Sender** - is the user sending a transaction on the FIO Chain to another user (Receiver)
* **Receiver** - is the user receiving a transaction on the FIO Chain from another user (Sender)

## Abstract
This FIP implements new features to enhance privacy of the FIO Protocol. It extends the existing FIO Data encryption scheme to include NBPAa and in addition introduces a new way to exchange FIO Requests, FIO Data, and NBPAs in way, which obfuscates the fact that the users are interacting with each other.

Proposed new actions:

|Action|Endpoint|Description|
|---|---|---|
|privaddadr|priv_add_pub_address|Add encrypted NBPA to FIO Chain|
|privganbpa|priv_grant_pub_address_access|Grants friend access to see NBPA|
||priv_get_pub_address|Allows friend to retrieve NBPA they are authorized to see|
|privfundsreq|priv_new_funds_request|Allows for creation of FIO Requests in privacy mode|
|privrecsend|priv_record_send_action|Allows for recording of OBT Data or rejecting a FIO Request in privacy mode|
||priv_get_received_fio_requests|Returns received FIO Request in privacy mode|
||priv_get_sent_fio_requests|Returns sent FIO Requests in privacy mode|
||priv_get_received_actions|Returns OBT Data or FIO Request rejections in privacy mode|
|setaddpriv|set_fio_address_privacy|Sets privacy mode for FIO Address|
||privacy_check|Checks privacy mode of FIO Address|

## Motivation
Even though contents of FIO Requests and OBT Records is already encrypted, there are two other areas where privacy of the FIO Protocol can be improved:
* Anytime a FIO Request or OBT Record is stored on chain, the FIO Addresses of both parties are stored unencrypted. This allows blockchain observers to:
  * Associate FIO Addresses transacting with each other, even though the details of those transactions are private
* NBPAs mapped to FIO Addresses are stored on-chain unencrypted. This allows blockchain observers to:
  * Associate NBPAs to identifiable FIO Addresses
  * Associate NBPAs on different blockchains belonging to a single individual/entity through a common FIO Address

## Specification
### Search Index
Today when data is exchanged on the FIO Chain (FIO Requests, FIO Data) FIO Addresses and FIO Public Keys of both parties are attached to each record. This is done because:
* **Sender** - account data (e.g. FIO Public Address) of the signer has to be visible for the blockchain to validate the transaction.
* **Receiver** - there needs to be a way for the receiver to know that there is data on the blockchain for them to receive. Today the Receiver uses their FIO Public Key to query for data, therefore the Sendeer has to include this in the transaction they expect the Receiver to receive.

In order to obfuscate that Sender and Receiver are interacting, only one party's information has to be encrypted. Obfuscating Sender's data is difficult without changing the architecture of the blockchain to a more complex privacy chain architecture. Obfuscating the Receiver's data is easy, but it makes it challanging for the Receiver to identify transactions on the chain that contain informationn for them.

To solve this challenge, this FIP introduces the concept of **Search Index**. This index uses [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) and can be derived by both Sender and Receiver and requires their own private key and the other party's public key.

#### Sender derives Receiver's Search Index to include on sent transactions
* Sender wants to derive Search Index for Receiver and types their FIO Address into their wallet
* Sender's wallet fetches FIO Public Key associated with FIO Address entered
* Sender's wallet uses Receiver's FIO Private Key and Receiver's FIO Public Key to derive a shared secret (Secret) using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange).
* Receiver's FIO Public Key is hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector to create Receiver's Search Index.

![](images/Diagram-Deriving-Search-Index.PNG)

#### Receiver derives their own Search Index to look for transactions to them
* Receiver wants to derive their own Search Index and types Sender's FIO Address into their wallet
* Receiver's wallet fetches FIO Public Key associated with FIO Address entered
* Receiver's wallet uses Receiver's FIO Private Key and Sender's FIO Public Key to derive a shared secret (Secret) using [Diffie-Hellman Key Exchange scheme](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange).
* Receiver's FIO Public Key is hashed using [Hash Function A](https://github.com/fioprotocol/fiojs/blob/3b3604bb148043dfb7e7c2982f4146a59d43afbe/src/tests/encryption-fio.test.ts#L65) and Secret as initialization vector to create Receiver's Search Index.
 
It is important to note that Friend List will exist between FIO Public Keys, not FIO Addresses, as past transactions should not be readable by the new owner of the FIO Address. This has the following consequences:
* Once user transfers their FIO Address to another public key (even if they control it), they will lose their whitelist. That is already a moot point, since they will not be able to decrypt the data anyway.
* Once whitelist is established it exists between all FIO Addresses owned by either party.

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
#### Add NBPA.
Adds NBPA to chain.
##### New action: *privaddadr*
##### New endpoint: /priv_add_pub_address
##### New fee: priv_add_pub_address, bundle-eligible (uses 1 bundled transaction)
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
|FIO Address expired|Supplied FIO Address has expired|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Address expired."|
|FIO Domain expired|Domain of supplied FIO Address has expired more than 30 days ago|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Domain expired."|
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
#### Grant NBPA access
Grants NBPA access to specific friend by placing a decrypt secret for them.
##### New action: *privganbpa*
##### New endpoint: /priv_grant_pub_address_access
##### New fee: priv_grant_pub_address_access, bundle-eligible (uses 1 bundled transaction)
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
#### Get NBPA
Looks up NBPA for supplied FIO Address and chain code.
##### New endpoint: /priv_get_pub_address
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
New Funds Request when utilizing Friend List.
#### New funds request
##### New action: *privfundsreq*
##### New endpoint: /priv_new_funds_request 
##### New fee: priv_new_funds_request, bundle-eligible (uses 2 bundled transaction)
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
#### Records send action.
This action is made to record information about a send transaction. Because content is encrypted, two calls used in unencrypted mode ([/record_obt_data](https://developers.fioprotocol.io/api/api-spec/reference/record-obt-data/record-obt-data-model) and [/reject_funds_request](https://developers.fioprotocol.io/api/api-spec/reference/reject-funds-request/reject-funds-request-model)) are combined into a single action.
##### New action: *privrecsend*
##### New endpoint: /priv_record_send_action 
##### New fee: priv_record_send_action, bundle-eligible (uses 2 bundled transaction)
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
#### Get received FIO Requests
Requests call polls for any requests sent to a receiver by a specified sender. Because status is now encrypted, it's no longer possible to return only pending FIO Requests, like it is in current [/get_pending_fio_requests](https://developers.fioprotocol.io/api/api-spec/reference/get-pending-fio-requests/get-pending-fio-requests). It will now be up to the wallet to decrypt each request and obtain status.
##### New endpoint: /priv_get_received_fio_requests 
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
#### Get sent FIO Requests
Sent requests call polls for any requests sent by provided FIO public key.
##### New endpoint: /priv_get_sent_fio_requests 
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
#### Get information about actions received.
Requests call polls for any actions taken by payer on the payee. Because status is now encrypted, OBT Data and status is now combined in same call. It will now be up to the wallet to decrypt each action and obtain status.
##### New endpoint: /priv_get_received_actions 
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

### FIO Address privacy
#### Set FIO Address privacy options
Sets your FIO Address privacy options.
##### New action: *setaddpriv*
##### New endpoint: /set_fio_address_privacy
##### New fee: set_fio_address_privacy, bundle-eligible (uses 1 bundled transaction)
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|String|FIO Address to set private/public.|
|privacy|Yes|Int|0 - FIO Address is public; 1 - FIO Address is private and FIO Request to public FIO Addresses are allowed; 2 - FIO Address is private and FIO Request to public FIO Addresses are not allowed;|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
###### Example
```
{
	"fio_address": "purse@alice",
	"privacy": 1,
	"max_fee": 2000000000,
	"tpid": "rewards@wallet",
	"actor": "aftyershcu22"
}
```
##### Processing
* Request is validated per Exception handling.
* Fee is collected
* privacy option is set
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|FIO Address expired|Supplied FIO Address has expired|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Address expired."|
|FIO Domain expired|Domain of supplied FIO Address has expired more than 30 days ago|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"FIO Domain expired."|
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
#### Check FIO Address privacy
Checks FIO Address privacy options.
##### New endpoint: /privacy_check
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|fio_address|Yes|String|FIO Address to check.|
###### Example
```
{
	"fio_address": "purse@alice"
}
```
##### Processing
* Request is validated per Exception handling.
* Privacy option is returned
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|FIO Address not found|Supplied FIO Address cannot be found|404||||
##### Response
|Parameter|Format|Definition|
|---|---|---|
|privacy|Int|Privacy option is returned.|
###### Example
```
{
	"privacy": 1
}
```

### Modification to existing queries
#### get_fio_names, get_fio_addresses
Should now return privacy for all FIO Addresses
```
{
	"fio_addresses": [
		{
			"fio_address": "purse@alice",
			"expiration": "2020-09-11T18:30:56",
			"privacy": 1
		}
	]
}
```
#### new_funds_request
##### new_funds_request to *private* FIO Address
An attemp to send new_funds_request to *private* FIO Address (payer) should result in:
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|FIO Address private|Payer's FIO Addreess is private|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"Payer's FIO Addreess is private."|
##### new_funds_request from *private* to *public* FIO Address
An attemp to send new_funds_request from *private* to *public* FIO Address (payer) when privacy option is set to 2 should result in:
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|Restricted by privacy setting|Payee FIO Address is private and payer is public|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"Payer's FIO Addreess is public and request not allowed."|

#### record_obt_data
An attemp to send record_obt_data to *private* FIO Address (payee) should result error (unless in response to FIO Request):
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|FIO Address private|Payee's FIO Addreess is private|400|"payer_fio_address"|Value sent in, i.e. "alice@purse"|"Payee's FIO Addreess is private."|

## Rationale
### Complexity tradeoffs
Privacy on a public blockchain is a complex problem. The Dapix team has spent several weeks in 2019 brainstorming solutions with number of experts. Without a doubt, the solution contemplated in this FIP is adding significant complexity to the blockchain and for the integrators, even though it is planned that a lot of the complexity will be eliminated in the SDKs.

Complexity could be reduced if certain privacy requirements are relaxed. Specifically:
1. Accept the fact that it will always be public that two FIO Addresses are interacting with each other and that it is sufficient to simply encrypt the content of the interaction. This would: 1) Eliminate the need for fio_public_key_hash, as one could simply use the FIO Address; 2) Allow retention of existing FIO Request workflow; 3) Allow Request for friending to be put on chain. In current form, user has to initiate adding someone to Friend List from within their wallet, i.e. by typing in a FIO Address.
1. If #1 is accepted, FIO Request status can additionally be made public, which would push request filtering, such as "show only pending requests" to the blockchain API, instead of having it done insode wallet/SDK.
1. Eliminate the flexibility of allowing users multi-level encryption of NBPAs. Only address-level encryption would be supported, which would basically mean that once someone is your friend, they will get access to all keys you have shared for any FIO Address you grant them access to.

### Other approaches considered
* FIO Private Messaging - akin to [Onion Routing](https://en.wikipedia.org/wiki/Onion_routing) a new message protocol would be created and BPs would act as message relays. This would be a way to exchange a Request for Friend without any blockchain record. Why discounted:
  * Makes looking up requests harder
  * Require new calls to be built, e.g. *Fetch BPs*
  * Requires messaging functionality to be built
  * Creates complexity of disconnected calls
    * What if BP suddenly goes offline
    * What if original transaction gets forked out, but notification doesn't or vv.
    * Will need retries and error messages, etc.
* Make calls using non-linked keys - in current implementation the new_funds_request call and record_send calls have to be made by the corresponding payee or payer specified in the call. This means that the data is disclosed and the owner signature makes it easy to trace the signer's FIO Address. With this approach privacy-minded users can choose to use different private keys to sign these calls instead of keys which can trace back to their FIO Address. Why discounted:
  * Does not offer privacy by design
  * Makes looking up requests harder
  * Using a separate key may be cumbersome for some
  * May require wallet implementation
* Randomize FIO Addresses - in this approach, a record is placed by the known party on the blockchain (request placed by payee, reject by payer, record send by payer), but the counter party is disclosed together with X number of other randomly selected FIO Addresses. Why discounted:
  * Depending on the activity of the other members in the random group, the recipient may still have to inspect a large number of requests. For example, what is Alice is paired in a group with Amazon, which has million requests.

### Why do we encrypt a NBPA with three secret keys?
Assuming we want to offer the flexibility to users to decide which user can see which NBPA, this was meant to be an alternative to encrypting NBPA for each user separately. For example, if a user has placed 20 NBPAs on the FIO Chain for Payer 1 and they now wanted to give the same access to Payer 2, they would need to place 20 new entries on the FIO Chain. With secret keys, they will place the 20 NBPAs once and then just give one secret key to Payer 1 and 1 secret key to Payer 2, reducing the amount of data that needs to be stored on chain.

We've considered using a SLIP-44 derivation path for those keys, but it does not seem like a good approach, because there would still need to be some index that matched the keys to records on the FIO blockchain.

## Implementation
Will be provided in a later stage of the FIP.

## Backwards compatibility
It is anticipated that both public (existing) and private FIO Addresses will coexist simultaneously indefinitely. It's also likely that some wallets would not implement the enhanced privacy features.

Every FIO Address will elect a privacy option as follows:
* 0 - FIO Address is public
* 1 - FIO Address is private and FIO Requests to public FIO Addresses are allowed
* 2 - FIO Address is private and FIO Requests to public FIO Addresses are not allowed
All existing FIO Addresses will be set to 0 and all existing *public* functionality will continue to work as before. 

The following is an interaction matrix of public user interactions:
|Interaction|Public -> Public|Public -> Private|Private 1 -> Public|Private 2 -> Public|Private -> Private|
|---|---|---|---|---|---|
|Send to FIO Address|As is|As is if NBPA published|As is|As is|Friend workflow|
|Request funds|As is|Not allowed|As is|Not allowed|Friend workflow|
|Record OBT|As is|Not allowed|As is|Not allowed|Friend workflow|

All existing calls will function as is. All new calls for privacy users are designated with the priv_ prefix.

This model should allow a user with privacy option set to 1 or 2 to restores seed phrases to a wallet which does not yet support privacy, and not break, although some features will not work.

## Future considerations
Isn't this enough?
