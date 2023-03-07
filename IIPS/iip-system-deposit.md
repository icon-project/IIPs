---
iip: 59
title: ICON System Deposit
author: MoonKyu Song (@mksong-icon)
discussions-to: https://github.com/icon-project/IIPs/discussions/65
status: Draft
type: Standards Track
category: Core
created: 2022-12-16
---

## Simple Summary

System Deposit is used for system to pay fee for transactions supporting the network.

## Abstract

Message Verifier contracts in [BTP](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-25.md) do lots of work to verify messages.  And it consumes lots of steps. If message delivery rate is high enough to pay fees, then it's not a problem.

And also if the blockchain provides way to prevent delivering blocks without messages, then it would not be a problem but, most blockchains don't provide any ways to prevent it.  And number of those blocks can be huge under low rate message delivery and periodical block generation.

It's required to reduce fee of relays on delivering blocks without any messages to make them have usable fee consumption.

## Motivation

It's hard to estimate amount fee used for verifying messages, because it's depends on the network activities and number of integrated networks and so on. So, we can't use normal reward issuing mechanism.

Instead of it, let the system doesn't charge fee for that work. System Deposit is defined. There is no limit in amount of fee to pay. It's used only for paying fee. System Deposit is granted to limited contracts that the network consider as system components.

Previously, Fee Sharing and Virtual Steps are supported only for the most outer frame, but to enable a deposit for [BTP Message Verifier(BMV)](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-25.md#btp-message-verifier), it's required to support deposit for all frame.

## Specification

To use System Deposit, then it's firstly enabled for the contract. And the contract needs to set Fee Sharing Portion while it executes transactions. Payer of the fee would be the system.

### Enable System Deposit

Enabling System Deposit needs to be agreed by PReps. For this Governance needs to support new proposals for enabling and disabling System Deposit of the contract. And also the system needs to support the feature.
For this, the following APIs needs to be added to the Chain SCORE(`cx0000000000000000000000000000000000000000`).


### setUseSystemDeposit

```python
@external()
def setUseSystemDeposit(self, address: Address, yn: bool):
```
**Parameters**
* **address** : Address of contract
* ***yn*** : Whether it enables System Deposit of it or not


### getUseSystemDeposit

```python
@external(readonly=True)
def getUseSystemDeposit(self, address: Address) -> bool:
```
**Parameters**
* **address** : Address of contract

**Returns**
* Whether System Deposit is enabled for the contract

### Use System Deposit

To set fee proportion, use following APIs in the environments.

> For Python SCORE
```python
def set_fee_sharing_proportion(self, proportion: int):
```

> For Java SCORE
```java
public static void setFeeSharingProportion(int proportion);
```

A value between 0 and 100 for percent may be used for proportion.

If it sets the fee proportion, it pays the steps used by itself and callee excluding already paid.

For example, there are contract A and contract B and contract A calls contract B. If contract A uses 100 steps and the contract B uses 100 steps, then contract B pays nothing and contract A sets the fee sharing proportion 50%, then it pays 50% of 200 steps and the other 50% is paid by the sender of the transaction. If contract B also sets the free sharing proportion 50%, then contract B pays 50 steps and contract A pays 75 steps and the sender pays 75 steps.

### Check System Deposit usage

Fee for steps paid by System Deposit isn't deposited to treasury. So, it doesn't influence total circulation.
But you may find the result in `stepUsedDetails` field of the receipt.

```json
{
  "stepUsedDetails": {
    "cx0000000000000000000000000000000000000000": "0x27b3"
  }
}
```

## Backwards Compatibility

Fee sharing feature is limited to the direct call from EoA , but after application of System Deposit, it also applies to inter-called cases.

## Test Cases

* [FeeSharingTest](https://github.com/icon-project/goloop/blob/master/testsuite/java/foundation/icon/test/cases/FeeSharingTest.java) in GOLOOP

## Implementation

* [FeePayerInfo](https://github.com/icon-project/goloop/blob/master/service/contract/feepayerinfo.go) in GOLOOP

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
