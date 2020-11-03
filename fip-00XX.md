---
fip: XX
title: FIO Co-op
status: Draft
type: Functionality
author: David Gold <david@dapix.io>, Pawel Mastalerz <pawel@dapix.io>
created: 2020-10-27
updated: 2020-11-03
---

# Abstract
This FIP proposes **FIO Co-op**, an on-chain program for creating economic incentives for FIO end-users to adopt the protocol early.

Fee distribution modifications:
|Recipient|Before FIP|After FIP|
|---|---|---|
|Block Producers|85%|75%|
|FIOP Holders|0%|10%|

Token supply modifications:
|Token group|Before FIP|After FIP|
|---|---|---|
|FIOP Holders Bounty|0|10,000,000|
|BP Reserves|10,000,000|30,000,000|
|FIO Address Giveaways|125,000,000|95,000,000|

**Token supply increase/decerase: 0**

Modified actions:
|Contract|Action|Endpoint|Description|
|---|---|---|---|
|fio.address|regaddress|register_fio_address|Added logic to compute and grant FIOPs.|
|fio.address|regdomain|register_fio_domain|Added logic to compute and grant FIOPs.|
|fio.address|renewaddress|renew_fio_address|Added logic to grant FIOPs to FIO Addresses registered before program launch.|
|fio.address|burnexpired|burn_expired|Added logic to update *Total FIOPs in Circulation* when NFT is burned.|
|\*|Any action which collects a fee||Modify fee collection logic to allocate FIOP Rewards to pool|

New actions:
|Contract|Action|Endpoint|Description|
|---|---|---|---|
|fio.address|regaddressnf|register_fio_address_nofiops|Registers FIO Address, but does not grant FIOPs.|
|fio.treasury|fiopclaim|pay_fiop_rewards|Computes FIOP rewards and processes FIOP payments.|

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
FIO Co-op is an on-chain incentive program in which end-users who register FIO Address or FIO Domain are granted FIO Points (FIOPs). The earlier the registration is performed, the higher the amount of FIOPs granted. Any time a fee is paid on FIO Chain, a percentage is allocated to holders of FIOPs pro-rata.

## FIOP grants
FIOPs are granted when a FIO Address or FIO Domain is registered and are permanently attached to the Non-fungible Token (NFT) being created, irrespective of which FIO Public Key is indicated as the owner of the NFT. **FIOPs are not granted on renewals or bundle transaction purchases or any other activity associated with that NFT.**

Optionally, the account paying for the FIO Address registration may choose not to grant FIOPs to the NFT being created. This may be used by any entity which pays for FIO Addresses for its users and does not want to increase a risk of a sybil attack.

Amount of FIOPs (P) granted is determined by this formula:

***P = (F/f)(1 - r)<sup>x</sup>***

where:
* ***f*** - Current register_fio_address fee
* ***F*** - Fee paid for FIO Address or FIO Domain Registration
* ***r*** - *Percent decay per second*: 0.000002%
* ***x*** - Seconds since program launch

#### Registrations before program launch
FIO Addresses and FIO Domains which were registered before the program launch have their FIOPs computed as if they were registered zero seconds since program launch. The FIOPs are further increased by:
|When registered|Increase|
|---|---|
|At Mainnet|25%|
|After Mainnet but before program start|10%|

FIO Address registered before program launch do not have their FIOPs granted until the FIO Address is renewed after the program starts. Meaning a FIO Address registered before program launched does not earn FIOP rewards until after it has been renewed.

FIO Domains registered before program launch have their FIOPs granted immediately after the program starts.

### Program duration
FIOPs are only granted in the first 3 years after program launch. After that time FIOP Holders continue to receive the FIOP Rewards, but no new FIOPs are granted.

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

### Distribution period
FIOP Rewards accumulate in the FIOP Rewards pool for a period of 30 days before they become eligible for distribution. If a distribution to a single NFT is less that register_fio_address fee, the amount is not transferred, but rather attached to the NFT and is added to the next distribution.

### FIOP Holders Bounty
To further incentivize end users, FIOP Reward pool is increased before distribution by minting new tokens akin to [New User Bounties](https://kb.fioprotocol.io/fio-token/token-distribution#new-user-bounties). New tokens minted are capped at 10,000,000 and increase the FIOP Rewards based on the following schedule:
|Time since program launch|Increase in FIOP Rewards|
|---|---|
|30 days|100%|
|60 days|95%|
|90 days|90%|
|120 days|85%|
|150 days|80%|
|180 days|75%|
|210 days|70%|
|240 days|65%|
|270 days|60%|
|300 days|55%|
|330 days|50%|
|360 days|45%|
|390 days|40%|
|420 days|35%|
|450 days|30%|
|480 days|25%|
|510 days|20%|
|540 days|15%|
|570 days|10%|
|600 days|5%|

In order to keep the cap on tokens minted at 1,000,000,000, the Foundation is reducing the [FIO Address Giveaways](https://kb.fioprotocol.io/fio-token/token-distribution#tokens-minted-over-time) pool from 125,000,000 to 115,000,000 FIO Tokens.

## Changes to BP Reserves
In order to mitigate the impact of reduced Block Producer rewards and to extend the time when those rewards are guaranteed, the [Block Producer Reserves pool](https://kb.fioprotocol.io/fio-token/token-distribution#tokens-minted-over-time) is increased from 10,000,000 to 30,000,000 FIO Tokens.

In order to keep the cap on tokens minted at 1,000,000,000, the Foundation is further reducing the [FIO Address Giveaways](https://kb.fioprotocol.io/fio-token/token-distribution#tokens-minted-over-time) pool from 115,000,000 to 95,000,000 FIO Tokens.

## Modifications to existing actions
### [regaddress](https://developers.fioprotocol.io/api/api-spec/reference/register-fio-address/register-fio-address-model)
Added logic to compute and grant FIOPs.
#### Request
No changes
#### Processing
* If program [sill active](#program-duration), FIOPs are computed based on [FIO grants](#fiop-grants) and are granted to the NFT and *Total FIOPs in Circulation* is updated.
#### Exception handling
No changes
#### Response
No changes

### [regdomain](https://developers.fioprotocol.io/api/api-spec/reference/register-fio-domain/register-fio-domain-model)
Added logic to compute and grant FIOPs.
#### Request
No changes
#### Processing
* If program [sill active](#program-duration), FIOPs are computed based on [FIO grants](#fiop-grants) and are granted to the NFT and *Total FIOPs in Circulation* is updated.
#### Exception handling
No changes
#### Response
No changes

### [renewaddress](https://developers.fioprotocol.io/api/api-spec/reference/renew-fio-address/renew-fio-address-model)
Added logic to grant FIOPs to FIO Addresses registered before program launch.
#### Request
No changes
#### Processing
* If NFT has no granted FIOPs, program is [sill active](#program-duration) and FIOPs were computed, computed FIOPs are granted to the NFT.
#### Exception handling
No changes
#### Response
No changes

### [burnexpired](https://developers.fioprotocol.io/api/api-spec/reference/burn-expired/burn-expired-model)
Added logic to update *Total FIOPs in Circulation* when NFT is burned.
#### Request
No changes
#### Processing
* Reduce *Total FIOPs in Circulation* by amount attached to burned NFTs.
#### Exception handling
No changes
#### Response
No changes

### Any action which collects a fee
Modify fee collection logic to allocate FIOP Rewards to pool:
* Determine if pool is current, meaning time from pool creation is less than *Distribution period*
  * If pool is current, add *FIOP Holders*% of fee being collected to the pool. Entire fee continues to be transferred to treasury.
  * If pool is not current:
    * Attach *Total FIOP Circulating Supply* to the pool
    * Create a new pool

## New actions
### Register FIO Address without FIOPs
Registers FIO Address, but does not grant FIOPs.
#### Contract: fio.address
#### New action: *regaddressnf*
#### New end point: /register_fio_address_nofiops
#### New fee: register_fio_address_nofiops, not bundle-eligible
#### RAM increase: 2,560 bytes
#### Request body
Same as [regaddress](https://developers.fioprotocol.io/api/api-spec/reference/register-fio-address/register-fio-address-model).
#### Processing
Same as [regaddress](https://developers.fioprotocol.io/api/api-spec/reference/register-fio-address/register-fio-address-model) except that FIOPs are not granted to the NFT.
#### Exception handling
Same as [regaddress](https://developers.fioprotocol.io/api/api-spec/reference/register-fio-address/register-fio-address-model).
#### Response body
Same as [regaddress](https://developers.fioprotocol.io/api/api-spec/reference/register-fio-address/register-fio-address-model).

### Payout FIOP rewards
Computes FIOP rewards and processes FIOP payments.
#### Contract: fio.treasury
#### New action: *fiopclaim*
#### New end point: /pay_fiop_rewards
#### New fee: None
#### RAM increase: To be determined during implementation
#### Request body
None
#### Processing
* Identify first unpaid FIOP Reward pool.
  * If none return 400:No work to perform.
* Mint new [FIOP Holders Bounty](#fiop-holders-bounty) tokens and increase the *Pool FIO amount at close*.
* Identify first NFT (FIO Address/Domain) which has not been paid for that pool. NFT is eligible to be paid for that pool if it exists (even if after expiration and before burning) was created before that reward pool was started.
* Compute FIOP Reward.
  * FIOP Reward is: *NFT's FIOP balance* / *circulating FIOPs at pool close* * *Pool FIO amount at close*.
  * If *FIOP carry-over* is present, add it to FIOP Reward.
  * Track *FIO paid from pool*.
* If FIOP Reward >= register_fio_address amount, transfer FIO tokens to account which owns the NFT.
  * Annotate NFT as paid for current pool.
* If FIOP Reward < register_fio_address amount, incerement *FIOP carry-over* and do not transfer FIO tokens.
* Continue processing until as many NFTs as can safely be processed in single transaction.
* Once all NFTs for a specific pool have been processed and *FIO paid from pool* is > 0 (this can happen when NFTs get burned), add that amount to the current open pool.
#### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|
|No work to perform|No work to perform|400|||"No work to perform"|
#### Response body
|Parameter|Format|Definition|
|---|---|---|
|status|String|OK if successful|
|nfts_paid|Int|Number of NFTs which were paid|
##### Example
```
{
  "status": "OK",
  "nfts_paid": 100
}
```

# Rationale
When designing an end-user incentive program, the following was considered:
* Must be sybil attack proof
* Should provide greater economic benefits the early the user participates
* Payouts should be based on the future economic inputs to the FIO Protocol
* Should not add friction to the process of adopting the FIO Protocol
* Should enable permanent residual income as long as the user continues to participate

The following features were considered, but were not made part of this FIP as it was determined that they would add too much complexity at this stage:
* Continue to award FIOPs after the initial grant based on on-chain activity
* Award FIOPs for locking tokens.

# Implementation
* The following variables should be defined in contract:
  * *Percent decay per second*
  * *Program duration*
  * *Distribution period*
  * *FIOP Holders Bounty cap*
* Deployment dependency:
  * Foundation will transfer 25,000,000 FIO Tokens from [FIO Address Giveaways](https://kb.fioprotocol.io/fio-token/token-distribution#tokens-minted-over-time) to eosio for the purpose of retiring (burning).
* Deployment considerations:
  * 25,000,000 FIO Tokens held by eosio are retired.
  * Allocate FIOPs to addresses [registered before program launch](#registrations-before-program-launch).
    * For FIO Addresses, FIOPs are [computed and recorded but not granted](#registrations-before-program-launch). Those FIOPs are reflected in *Total FIOPs in Circulation*.
  * Create first pool as part of deployment.

# Backwards Compatibility
No impact on existing functionality.

# Future considerations
May be considered in the future:
* Continue to award FIOPs after the initial grant based on on-chain activity
* Award FIOPs for locking tokens.

# Discussion link
https://fioprotocol.atlassian.net/wiki/spaces/WP/pages/16252931/FIO+Co-op
