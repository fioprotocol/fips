---
fip: XX
title: NFT Signatures
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@fioprotocol.io>
created: 2021-05-05
updated: 2021-05-05
---

# Abstract
The objective of this FIP is to extend FIO Protocol functionality to enable NFTs to be mapped to a FIO Address. It will allow anyone to see which FIO Address has mapped ("signed") a particulart NFT and if it is a FIO Address they trust, they can also trust the NFT.

Example use case:
* Artist publishes the FIO Domain they own, e.g. artist, on their website.
* They register a new FIO Address for every art NFT, e.g. masterpiece1@artist
* They map new NFT (chain, contract address, token ID, image url) to masterpiece1@artist
* Now, irrespective of who owns the NFT at anytime, it's easy to prove if an NFT (based on chain, contract address, token ID or image url) is mapped to a FIO Address on the artist's FIO Domain and by therefore know if it was created by the artist.

## New actions
|Contract|Action|Endpoint|Description|
|---|---|---|---|

## Modified actions
|Contract|Action|Endpoint|Description|
|---|---|---|---|

# Motivation
Extending the FIO Protocol to allow NFT mappings makes sense as FIO Addresses were always envisioned to replace complicated addresses such as public addresses or NFT contract addresses and NFT token IDs.

# Specification
## Key concepts

## High-level

## New actions
### Map NFT
Maps a specific NFT to a FIO Address.
#### Contract: fio.address
#### New action: *addnft*
#### New end point: /add_nft
#### New fee: add_nft, bundle-eligible (1 bundled transaction) fee amount will be determined during development and updated here
#### RAM increase: To be determined during implementation
#### Request body
|Group|Parameter|Required|Format|Definition|
|---|---|---|---|---|
||fio_address|Yes|String|FIO Address to map to.|
|nfts|Yes|Array|List of NFTs to map, max 5|
|nfts|chain_code|Yes|String|Chain code, see [FIP-15|https://github.com/fioprotocol/fips/blob/master/fip-0015.md]|
|nfts|contract_address|Yes|String|Public address on NFT contract. Max. 128 characters.|
|nfts|token_id|Yes|String|Token ID of NFT. May be left blank if not applicable. Max 64 characters.|
|nfts|url|Yes|String|URL of NFT asset, e.g. image url. May be left blank if not applicable. Max 64 characters.
|nfts|hash|Yes|String|SHA-256 hash of NFT asset, e.g. image url. May be left blank if not applicable. Max 64 characters.|
|nfts|metdata|Yes|String|Future use.|
|max_fee|Yes|Positive Int|Maximum amount of SUFs the user is willing to pay for fee. Should be preceded by [/get_fee](https://developers.fioprotocol.io/api/api-spec/reference/get-fee/get-fee) for correct value.|
|tpid|Yes|FIO Address|FIO Address of the entity which generates this transaction. TPID rewards will be paid to this address. Set to empty if not known.|
|actor|Yes|12 character string|Valid actor of signer|
##### Example
```
{
  "fio_address": "purse@alice",
  "nfts": [
    {
      "chain_code": "ETH",
      "contract_address": "0x63c0691d05f441f42915ca6ca0a6f60d8ce148cd",
      "token_id": "100010001",
      "url": "ipfs://ipfs/QmZ15eQX8FPjfrtdX3QYbrhZxJpbLpvDpsgb2p3VEH8Bqq",
      "hash": "f83b5702557b1ee76d966c6bf92ae0d038cd176aaf36f86a18e2ab59e6aefa4b"
    },
    {
      "chain_code": "EOS",
      "contract_address": "atomicassets",
      "token_id": "2199023271139",
      "url": "",
      "hash": ""
    }
  ],
  "max_fee": 0,
  "tpid": "rewards@wallet",
  "actor": "aftyershcu22"
}
```
#### Processing

#### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|

|Invalid amount value|amount format is not valid|400|"amount"|Value sent in, e.g. "-100"|"Invalid amount value"|
|Invalid fee value|max_fee format is not valid|400|"max_fee"|Value sent in, e.g. "-100"|"Invalid fee value"|
|Fee exceeds maximum|Actual fee is greater than supplied max_fee|400|"max_fee"|Value sent in, e.g. "1000000000"|"Fee exceeds supplied maximum"|
|Insufficient balance|Available (unlocked and unstaked) balance in Staker's account is less than chain fee + *amount*|400|"max_fee"|Value sent in, e.g. "100000000000"|"Insufficient balance"|
|Invalid TPID|tpid format is not valid|400|"tpid"|Value sent in, e.g. "notvalidfioaddress"|"TPID must be empty or valid FIO address"|
|Signer not actor|Signer not actor|403|||Type: invalid_signature|
#### Response body
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if successful|
|fee_collected|Int|Amount of SUFs collected as fee|
##### Example
```
{
  "status": "OK",
  "fee_collected": 2000000000
}
```

# Rationale
TBD

# Implementation
TBD

# Backwards Compatibility
TBD

# Future considerations
None

# Discussion link
https://fioprotocol.atlassian.net/wiki/spaces/FC/pages/113606966/NFT+Digital+Signature

