---
iip: 35
title: ICON BTP Fee Gathering
author: Heonseung Lee (@leeheonseung)
discussions-to: https://github.com/icon-project/IIPs/issues/35
status: Draft
type: Standards Track
category: IRC
created: 2020-05-13
requires: 25
---

## Simple Summary

It standardizes the structure in which fees can be charged from BTP network users to cover maintaining costs and activation incentives the IIP-25 (BTP) network.

## Abstract

IIP-25 (BTP) Network fee is charged to users and standardize about operation policy.
Defines the operating entity that gathers a fee.
Defines Fee Aggregator (FA).
Defines interfaces related to FeeGathering BMC message.

## Motivation

In order to cover maintaining costs and activation incentives the IIP-25 (BTP) network, a structure is needed to charge fees from users who send BTP message.

The chain to which the entity operating the BTP Network belongs is defined as HUB.
The smart contract, which operates with fees from the BTP service, is defined as Fee Aggregator (FA).

The BTP network user pays a fee to the BSH that provides the BTP service of the chain to which he belongs.
The operating entity uses HUB's Fee Aggregator (FA) to collect the fees stored in each chain BSH and operates the BTP network.
Through HUB's BMC.sendFeeGatheringMessage, you can request a BTP message that sends a fee to each chain's BMC to FA.

## Specification

### Terminology

- HUB
  The chain to which the entity operating the BTP network belongs.
- Fee Aggregator (FA)
  Smart contract to to raise the necessary financial resources for BTP network operation by gathering fees collected from other chains connected.
- FeeGathering BMC message & Handle FeeGathering message
  Interface that processes the logic that transfers the fees stored in the BSH to the Fee Aggregator (FA).

### BTP Message Center - Extension

Ref. BTP message center

HUB's BMC sends FeeGathering BMC message to each link when certain conditions are satisfied, and calls handleFeeGathering from the service managed each BMC of that chains.

#### Message Struct

##### FeeGathering Message

Send to all of connected BMC on [BMC.sendFeeGatheringMessage](#sendfeegatheringmessage) on HUB

| Name  | Type           | Description                  |
| ----- | -------------- | ---------------------------- |
| _fa   | String         | BTP Address of FeeAggregator |
| _svcs | List of String | list of name of service      |

When BMC receive this message, find the BSH using 'svcs' and then calls [BSH.handleFeeGathering](#handlefeegathering) with '_fa'

### BTP Service Handler - Extension

Ref. BTP Service Handler

Interface that processes the logic that transfers the fees stored in the BSH to the Fee Aggregator (FA).

#### Interface

##### Writable methods

###### handleFeeGathering

```
@external
def handleFeeGathering(self, _fa: str):
```

- Description
  - Handle the FeeGathering Message.
  - Accept the error only from the BMC.
- Params
  - _fa: String ( BTP Address of Fee Aggregator )

## Implementation

TBD

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
