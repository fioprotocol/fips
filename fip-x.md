---
fip: X
title: Request for Public Address
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-27
updated: 2020-04-27
---

## Abstract

## Terminology
* **Native Blockchain Public Address (NBPA)** - this is the Public Address on a native blockchain that is needed to send funds and is associated to the FIO Address using [/add_pub_address](https://developers.fioprotocol.io/api/api-spec/reference/add-pub-address/add-pub-address-model)
* **Payee** - is the user receiving funds. In the Send scenario, this is the user who places NBPA on the FIO Chain and allows Payer to see it so that the Payer can send funds using this NBPA. In Request scenario, this is the user sending a FIO Request.
* **Payer** - is the user sending funds using FIO Address. In the Send scenario, Payer will type a FIO Address in wallet, that wallet will look up the corresponding NBPA on native blockchain and transaction will be executed. In Request scenario, Payer will respond to a FIO Request sent by Payee.

## Motivation
[FIP-5 Enhanced privacy via friending](fip-5.md) proposes to improve FIO Protocol privacy in two areas:
1. NBPAs mapped to FIO Addresses are stored on-chain unencrypted.
1. Anytime a FIO Request or OBT Record is stored on chain, the FIO Addresses of both parties are stored unencrypted.

The solution proposed in FIP-5 is [arguably complex](fip-5.md#complexity-tradeoffs), both in terms of blockchain code, but, more importantly, in terms of wallet integration. This FIP proposes that issue #1 is the primary privacy concern that should be addressed and that issue #2 should not be addressed at this time to substantially reduce complexity.

Issue #1 could be solved by allowing Payer to first request an NBPA from Payee via an on-chain Request for Public Address with optional encrypted metadata, and then allowing the Payee to place the NBPA via an on-chain Grant Public Address, where the NBPA is encrypted for the Payer. There is no need for a Friend's List or complex hashing to derive search indexes.

In addition, Request for Public Address may have other useful applications. It may allow the Payee to withhold the release of the public address, preventing receipt of funds, until certain information is received. For example, a crypto exchange may not allow a deposit to occur until they have received information about the counter party, in order to comply with the [Travel Rule](https://www.fatf-gafi.org/documents/documents/virtual-currency-definitions-aml-cft-risk.html). 

## Specification
### Overview
Conceptually Request for Public Address is very similar to [New Funds Request](https://developers.fioprotocol.io/api/api-spec/reference/new-funds-request/new-funds-request-model) except for the contents. Similarly, Granting Access to Public Key is similar to responding to New Funds Request using [Record OBT Data](https://developers.fioprotocol.io/api/api-spec/reference/record-obt-data/record-obt-data-model).

![](images/Diagram-Request-for-Public-Address.PNG)

When Bob grants access to the NBPA, he would also specify how long that NBPA is valid, so that, in the future, Alice may send Bob crypto without the need to request the public address again.

Typical sequence of events:
1. Alice wants to send Bob 1 BTC and types his FIO Address: bob@hodl in the send area of her wallet.
1. The wallet checks the blockchain for any mappings for chain_code: BTC, token_code: BTC for bob@hodl using existing [/get_pub_address](https://developers.fioprotocol.io/api/api-spec/reference/get-pub-address/get-pub-address).
1. Since Bob does not make any NBPA public, the blockchain returns a "Public address not found".
1. Alice's wallet automatically recommends she requests a Public Address from Bob. She accepts the suggestion and the wallet submits Request for Public Address for chain_code: BTC, token_code: BTC to bob@hodl.
1. Bob receives the request in the existing FIO Requests section of the wallet and with a tap grants access to the BTC address to Alice for a period of 1 month.
1. Alice can see all her existing Public Address requests in her wallet together with the status and public address (until expiration). The wallet may also monitor specific requests and show a notification once NBPA access is granted.
1. Alice selects the FIO Address of Payee and executes a transaction.
1. Until the NBPA expires, she can simply type in bob@hodl in send area of BTC and the existing [/get_pub_address](https://developers.fioprotocol.io/api/api-spec/reference/get-pub-address/get-pub-address) will return the NBPA which is encrypted for her alleviating the need for another Request for Public Address.

## Rationale
Encrypting blockchain code.

## Implementation

## Backwards Compatibility

## Future considerations
