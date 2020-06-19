---
fip: 9
title: Allow voting and proxying without a FIO Address
status: Accepted
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-06-03
updated: 2020-06-08
---

## Abstract
This FIP enables token holders to vote or proxy without requiring them to have a registered FIO Address.

Modified actions:
|Action|Endpoint|Description|
|---|---|---|
|voteproducer|vote_producer|FIO Address field can now be left blank.|
|voteproxy|proxy_vote|FIO Address field can now be left blank.|

## Motivation
In order to make voting/proxying "free" to users to encourage them to vote/proxy, the voteproducer and voteproxy actions require FIO Address, as FIO Address is needed to allow payment with bundled transactions instead of tokens. However, it introduced the **requirement** that the token holder have a FIO Address in order to vote/proxy. As recommended in [Issue 47](https://github.com/fioprotocol/fio/issues/47) it makes sense to also allow voting/proxying even when token holder does not have a FIO Address.

## Specification
#### Changes to existing actions
voteproducer and voteproxy actions will be modified to allow FIO Address parameter to be empty. This will have the same effect as if the user passed in the FIO Address and had no remaining bundled transactions, meaning the user will be charged the *vote_producer* or *proxy_vote* fee respectively.

## Rationale
It should have been built this way to beging with, as vote_producer and proxy_vote are the only calls which allow bundled transactions, but do not actually require a FIO Address to execute.

## Implementation
To be defined before FIP Acceptance

## Backwards Compatibility
No impact, as there is no change to call structure other than making a required field allowed to be blank.

## Future considerations
None
