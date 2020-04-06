---
fip: 2
title: Improvements to paging via API
status: Draft
type: Functionality
author: Pawel Mastalerz <pawel@dapix.io>
created: 2020-04-06
updated: 2020-04-06
---

## Abstract

## Motivation
There are currently deficiencies in paging for certain API calls:
* /get_fio_names has no paging at all. If an account has more FIO Domains or FIO Addresses than can be returned before table read time out, only a partial results are returned without warning to the user or ability to retrieve the rest.
* /get_obt_data, /get_pending_fio_requests and /get_sent_fio_requests has limit and offset already, but implementing wallets have expressed desire to cache the data locally and requested ability to query by providing a time stamp and only receiving requests since that time stamp.

## Specification 


## Rationale


## Implementation


## Backwards Compatibility


## Future considerations
