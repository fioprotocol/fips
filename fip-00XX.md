---
fip: XX
title: FIO Co-op
status: Draft
type: Functionality
author: David Gold <david@dapix.io>, Pawel Mastalerz <pawel@dapix.io>
created: 2020-10-27
updated: 2020-10-28
---

# Abstract
This FIP proposes **FIO Co-op**, an on-chain program for creating economic incentives for FIO end-users to adopt the protocol early.

# Motivation
The FIO Protocol is subject to the network effect, meaning the more participants adopt it, the more useful it is to all.

To accelerate the adoption, on-chain incetives for integrators in the form of [TPID rewards](https://kb.fioprotocol.io/fio-chain/fees#fee-distribution) and [New User Bounties](https://kb.fioprotocol.io/fio-token/token-distribution#new-user-bounties), as well as for block producers in the form of [rewards](https://kb.fioprotocol.io/fio-token/token-distribution#new-user-bounties) and [reserves](https://kb.fioprotocol.io/fio-token/token-distribution#block-producer-reserves) are built into the FIO Chain.

Currently, there are no incentives for end-users. When registered early, a FIO Address has limited use until more participants integrate it into their products and more end-users join the network. Yet early adopters add significant value to the network by helping it grow and therefore should be incetivized.

Key objective for FIO Co-op:
* Encourage early adoption of the FIO Protocol by end-users
* Cause users to seek FIO-integrated applications
* Entice users to spread the word about FIO to others
* Encourage users to hold FIO tokens for long periods of time
## User pitch
Register a FIO Address/Domain and you get a lifetime residual portion of all the future income to the FIO Protocol. The earlier you do so, the greater your share will be. If you register now, you'll get an X% greater allocation than a year from now. You can also choose to lock FIO tokens for a duration of time to receive an additional portion of the future income to the FIO Protocol.

# Specification
## Overview
FIO Co-op is an on-chain incentive program in which end-users who perform an Incentivized Action (pay for a FIO Address/Domain or lock FIO Tokens) are awarded FIO Points (FIOPs). The earlier the Incentivized Action is performed, the higher the amount of FIOPs awarded. After 250,000,000 blocks or approximatley 4 years, FIOPs will no longer be awarded.

Any time a fee is paid on FIO Chain, 25% is allocated to holders of FIOPs pro-rata. The payments continue indefinitely, even after new FIOPs are no longer being awarded.

## FIOP accrual
There are two types of FIOPs: Name Points and Lock Points.
### Name Points
Name Points are awarded when a FIO Address or FIO Domain is registered and are permanently attached to the Non-fungible Token (NFT) being registered, irrespective of which FIO Public Key is indicated as the owner of the NFT. However the account paying for the registration may choose not to award FIOPs to the NFT. This may be used by any entity which pays for FIO Addresses for its users and does not want to increase a risk of a sybil attack. Name Points are not awarded on renewals or bundle transaction purchases.

Name Points (N) are accrued according to this formula:

***N = (F/f)(1 - r)<sup>x</sup>***

where:
* ***f*** - Current register_fio_address fee
* ***F*** - Fee paid for FIO Address or FIO Domain Registration
* ***r*** - Percent decay per block
* ***x*** - Block number since program launch

#### Registrations before program launch
FIO Addresses and FIO Domains which were registered before program launch will have Name Points computed as if they were registered in block 1 since program launch. FIO Addresses and FIO Domains minted during genesis (registered before Mainnet launched) will have their Computed Name Points further increased by 25%.

Computed Name Points will not be awarded to the FIO Address or FIO Domain until it is renewed. Meaning a FIO Address registered before program launched will not be earning FIOP rewards until it is renewed. When it is renewed does not alter the Computed Name Points, provided it is renewed before it is burned.

### Lock Points
Lock Points are awarded when a user locks FIO Tokens and that user's account has at any time in the past executed [voteproducer](https://developers.fioprotocol.io/api/api-spec/reference/vote-producer/vote-producer-model) or [voteproxy](https://developers.fioprotocol.io/api/api-spec/reference/proxy-vote/proxy-vote-model) actions. The Lock tokens are awarded to the FIO Address specified during locking. The specified FIO Address does not have to belong to the account locking the tokens.

Lock Points (L) are accrued according to this formula: TBD

#### Locks before program launch
Tokens locked before program launch, will not earny any Lock Points.

### Program duration
FIOPs are only accrued in the first 250,000,000 blocks after program launch.

## FIOP lifecycle
* FIOPs never expire and are permanently attached to the NFT.
* When the FIO Address/Domain NFT is burned, attached FIOPs are permanently destroyed.
* FIOPs cannot be transferred. However, when the FIO Address/Domain NFT is transferred to a new owner, attached FIOPs get transferred as well.

## FIOP rewards

### Use cases

## New actions

# Rationale
When designing an end-user incetive program, the following was considered:
* Must be sybil attack proof
* Should provide greater economic benefits the early the user participates
* Payouts should be based on the future economic inputs to the FIO Protocol
* Should not add friction to the process of adopting the FIO Protocol
* Should enable permanent residual income as long as the user continues to participate

# Implementation
TBD

# Backwards Compatibility

# Future considerations

# Discussion link
https://fioprotocol.atlassian.net/wiki/spaces/WP/pages/16252931/FIO+Co-op
