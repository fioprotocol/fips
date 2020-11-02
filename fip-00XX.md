---
fip: XX
title: FIO Co-op
status: Draft
type: Functionality
author: David Gold <david@dapix.io>, Pawel Mastalerz <pawel@dapix.io>
created: 2020-10-27
updated: 2020-11-02
---

# Abstract
This FIP proposes **FIO Co-op**, an on-chain program for creating economic incentives for FIO end-users to adopt the protocol early.

# Motivation
The FIO Protocol is subject to the network effect, meaning the more participants adopt it, the more useful it is to all.

To accelerate the adoption, on-chain incentives for integrators in the form of [TPID rewards](https://kb.fioprotocol.io/fio-chain/fees#fee-distribution) and [New User Bounties](https://kb.fioprotocol.io/fio-token/token-distribution#new-user-bounties), as well as for block producers in the form of [rewards](https://kb.fioprotocol.io/fio-token/token-distribution#new-user-bounties) and [reserves](https://kb.fioprotocol.io/fio-token/token-distribution#block-producer-reserves) are built into the FIO Chain.

Currently, there are no incentives for end-users. When registered early, a FIO Address has limited use until more participants integrate it into their products and more end-users join the network. Yet early adopters add significant value to the network by helping it grow and therefore should be incentivized.

Key objective for FIO Co-op:
* Encourage early adoption of the FIO Protocol by end-users
* Cause users to seek FIO-integrated applications
* Entice users to spread the word about FIO to others

## User pitch
Register a FIO Address/Domain and you get a lifetime residual portion of all the future income to the FIO Protocol. The earlier you do so, the greater your share will be. If you register now, you'll get an X% greater allocation than a year from now.

# Specification
## Overview
FIO Co-op is an on-chain incentive program in which end-users who register FIO Address or FIO Domain are awarded FIO Points (FIOPs). The earlier the registration is performed, the higher the amount of FIOPs awarded. After 3 years, FIOPs will no longer be awarded.

Any time a fee is paid on FIO Chain, 20% is allocated to holders of FIOPs pro-rata. The payments continue indefinitely, even after new FIOPs are no longer being awarded.

## FIOP accrual
FIOPs are awarded when a FIO Address or FIO Domain is registered and are permanently attached to the Non-fungible Token (NFT) being registered, irrespective of which FIO Public Key is indicated as the owner of the NFT. However the account paying for the registration may choose not to award FIOPs to the NFT. This may be used by any entity which pays for FIO Addresses for its users and does not want to increase a risk of a sybil attack. FIOPs are not awarded on renewals or bundle transaction purchases.

FIOPs (P) are accrued according to this formula:

***P = (F/f)(1 - r)<sup>x</sup>***

where:
* ***f*** - Current register_fio_address fee
* ***F*** - Fee paid for FIO Address or FIO Domain Registration
* ***r*** - Percent decay per second
* ***x*** - Seconds since program launch

#### Registrations before program launch
FIO Addresses and FIO Domains which were registered before the program launch have their FIOPs computed as if they were registered zero seconds since program launch. The FIOPs are further increased by:
|When registered|Increase|
|---|---|
|At Mainnet|25%|
|After Mainnet but before program start|10%|

FIO Address registered before program launch do not have their FIOPs attached until the FIO Address is renewed after the program starts. Meaning a FIO Address registered before program launched does not earn FIOP rewards until after it has been renewed.

FIO Domains registered before program launch have their FIOPs attached immediatley after the program starts.

### Program duration
FIOPs are only accrued in the first 3 years after program launch.

## FIOP lifecycle
* FIOPs never expire and are permanently attached to the NFT.
* When the FIO Address/Domain NFT is burned, attached FIOPs are permanently destroyed.
* FIOPs cannot be transferred. However, when the FIO Address/Domain NFT is transferred to a new owner, attached FIOPs get transferred as well.

## FIOP Rewards
### Fee distribution
[On-chain fee distribution](https://kb.fioprotocol.io/fio-chain/fees#fee-distribution) is modified as follows:
|Recipient|Share before FIP|Share after FIP|
|---|---|---|
|Block Producers|85%|75%|
|Entity facilitating transaction (TPID) or, if not provided, Block Producers.|10%|10%|
|Foundation|5%|5%|
|FIOP Holders|0%|10%|

## Changes to BP Reserves
In order to mitigate the impact of reduced Block Producer rewards and to extend the time when those rewards are guaranteed, the [Block Producer Reserves pool](https://kb.fioprotocol.io/fio-token/token-distribution#tokens-minted-over-time) is increased from 10,000,000 to 30,000,000 FIO Tokens.

At the same time, in order to keep the cap on tokens minted at 1,000,000,000, the Foundation is reducing the FIO Address Giveaways pool from 125,000,000 to 105,000,000 FIO Tokens.

## New actions

# Rationale
When designing an end-user incentive program, the following was considered:
* Must be sybil attack proof
* Should provide greater economic benefits the early the user participates
* Payouts should be based on the future economic inputs to the FIO Protocol
* Should not add friction to the process of adopting the FIO Protocol
* Should enable permanent residual income as long as the user continues to participate

The following features were considered, but were not made part of this FIP as it was determiend that they would add too much complexity at this stage:
* Continue to award FIOPs after the initial grant based on on-chain activity
* Award FIOPs for locking tokens.

# Implementation
TBD

# Backwards Compatibility

# Future considerations

# Discussion link
https://fioprotocol.atlassian.net/wiki/spaces/WP/pages/16252931/FIO+Co-op
