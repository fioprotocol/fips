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
* **Native Blockchain Public Address (NBPA)** - this is the public address on a native blockchain that is needed to send funds and is associated to the FIO Address using [/add_pub_address](https://developers.fioprotocol.io/api/api-spec/reference/add-pub-address/add-pub-address-model)
* **Payee** - is the user receiving funds. In the Send scenario, this is the user who places NBPA on the FIO Chain and allows Payer to see it so that the Payer can send funds using this NBPA. In Request scenario, this is the user sending a FIO Request.
* **Payer** - is the user sending funds using FIO Address. In the Send scenario, Payer will type a FIO Address in wallet, that wallet will look up the corresponding NBPA on native blockchain and transaction will be executed. In Request scenario, Payer will repons to a FIO Request sent by Payee.

## Motivation
[FIP-5 Enhanced privacy via friending](fip-5.md) proposes to improve FIO Protocol privacy in two areas:
1. NBPAs mapped to FIO Addresses are stored on-chain unencrypted.
1. Anytime a FIO Request or OBT Record is stored on chain, the FIO Addresses of both parties are stored unencrypted.

The solution proposed in FIP-5 is [arguably complex](fip-5.md#complexity-tradeoffs), both in terms of blockchain code, but, more importantly, in terms of wallet integration. This FIP proposes that issue #1 is the primary privacy concern that should be addressed and that issue #2 should not be addressed at this time to substantially reduce complexity.

Issue #1 could be solved by simply moving NBPAs from being publicly accessible to being exchange encrypted between two users, the same way FIO Request is.

In addition, Request for Public Address may have other useful applications. It may allow the Payee to withhold the release of the public address, preventing receipt of funds, until certain information is received. For example, a crypto exchange may not allow a deposit to occur until they have received information about the counter party, in order to comply with the [Travel Rule](https://www.fatf-gafi.org/documents/documents/virtual-currency-definitions-aml-cft-risk.html). 

## Specification

## Implementation

## Backwards Compatibility

## Future considerations
