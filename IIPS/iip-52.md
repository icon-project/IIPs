---
iip: 52
title: ICON BTP Arbitrary Call Service Standard
author: Jaechang Namgoong (@sink772)
discussions-to: https://github.com/icon-project/IIPs/issues/52
status: Draft
type: Standards Track
category: IRC
created: 2022-07-13
requires: https://github.com/icon-project/IIPs/issues/25
---

## Simple Summary

A standard interface to make an arbitrary call service (a.k.a. `xcall`) between different blockchain networks via
ICON BTP protocols.

## Abstract

This document describes a new [BSH](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-25.md#btp-service-handler) interface
that can be used to invoke arbitrary cross-chain contract calls between different blockchains networks that support ICON BTP.
A new `xcall` service handler name will be used for the arbitrary call service.
The existing interfaces between BMC and BSH ([`sendMessage`](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-25.md#sendmessage)
and [`handleBTPMessage`](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-25.md#handlebtpmessage))
will be utilized for sending and receiving the requested arbitrary call payloads.

## Motivation

Adding a new service to an existing BTP infrastructure requires an audit of the code to ensure that the service
does not have any malicious behavior, because relays bear the gas cost of executing the service transaction.
To improve scalability and flexibility and to overcome the restriction mentioned above, we propose a new interface
for invoking arbitrary cross-chain contract calls that can appropriately charge the gas cost to users triggering
cross-chain transactions.

## Specification

### Encoding of the Calldata Payload

Encoding and decoding of the calldata are up to the DApps implementation.
`xcall` only passes the raw byte streams with the predefined method calls.
This is simple and easy to implement from the viewpoint of the `xcall`, and also the result can be sent back
to the caller using the same `sendCallMessage` interface.

### Basic Interfaces

#### sendCallMessage

DApps need to invoke the following method of `xcall` on the source chain to send a call message to the destination chain.

```java
/**
 * Sends a call message to the contract on the destination chain.
 *
 * @param _to The BTP address of the callee on the destination chain
 * @param _data The calldata specific to the target contract
 * @param _rollback (Optional) The data for restoring the caller state when an error occurred
 * @return The serial number of the request
 */
@Payable
@External
BigInteger sendCallMessage(String _to, byte[] _data, @Optional byte[] _rollback);
```

The `_to` parameter is the [BTP address](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-25.md#btp-address)
of the callee contract which receives the `_data` payload on the destination chain.

The `_data` parameter is an arbitrary payload defined by the DApp.

The `_rollback` parameter is for handling error cases, see [Error Handling](#error-handling) section below for details.

This method is payable, and DApps need to call this method with proper fees, see [Fees Handling](#fees-handling) section below for details.

When `xcall` on the source chain receives the call request message, it sends the `_data` to `_to` on the destination chain through BMC.

#### CallMessageSent

When `xcall` invokes [`sendMessage`](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-25.md#sendmessage) in BMC,
it returns `nsn` (network serial number) that identifies each BTP message from the source chain.
The following event is emitted after `xcall` receives the `nsn` from BMC, and can be used for BTP message tracking purpose.

```java
/**
 * Notifies that the requested call message has been sent.
 *
 * @param _from The chain-specific address of the caller
 * @param _to The BTP address of the callee on the destination chain
 * @param _sn The serial number of the request
 * @param _nsn The network serial number of the BTP message
 */
@EventLog(indexed=3)
void CallMessageSent(Address _from, String _to, BigInteger _sn, BigInteger _nsn);
```

#### CallMessage

When the `xcall` on the destination chain receives the call request through BMC, it emits the following event for notifying the user.

```java
/**
 * Notifies the user that a new call message has arrived.
 *
 * @param _from The BTP address of the caller on the source chain
 * @param _to A string representation of the callee address
 * @param _sn The serial number of the request from the source
 * @param _reqId The request id of the destination chain
 */
@EventLog(indexed=3)
void CallMessage(String _from, String _to, BigInteger _sn, BigInteger _reqId);
```

#### executeCall

The user on the destination chain recognizes the call request and invokes the following method on `xcall` with the given `_reqId`.

```java
/**
 * Executes the requested call message.
 *
 * @param _reqId The request Id
 */
@External
void executeCall(BigInteger _reqId);
```

#### handleCallMessage

When the user calls `executeCall` method, the `xcall` invokes the following predefined method in the target DApp
with the calldata associated in `_reqId`.

```java
/**
 * Handles the call message received from the source chain.
 * Only called from the Call Message Service.
 *
 * @param _from The BTP address of the caller on the source chain
 * @param _data The calldata delivered from the caller
 */
@External
void handleCallMessage(String _from, byte[] _data);
```

If the call request was a one-way message and DApp on the destination chain needs to send back the result (or error),
it may call the same method interface (i.e. `sendCallMessage`) to send the result message to the caller.
Then the user on the source chain would be notified via `CallMessage` event, and call `executeCall`,
then DApp on the source chain may process the result in the `handleCallMessage` method.

#### CallExecuted

To notify the execution result of DApp's `handleCallMessage` method, the following event is emitted after its execution.

```java
/**
 * Notifies that the call message has been executed.
 *
 * @param _reqId The request id for the call message
 * @param _code The execution result code
 *              (0: Success, -1: Unknown generic failure, >=1: User defined error code)
 * @param _msg The result message if any
 */
@EventLog(indexed=1)
void CallExecuted(BigInteger _reqId, int _code, String _msg);
```

### Error Handling

In success cases, DApp users only need to invoke two transactions (one for the source chain, and the other for
the destination chain).  However, there might be some error situations such as the execution of the call request
has failed on the destination chain. In this case, we need to notify the user on the source chain to rollback to the state
before the call request.

If a DApp needs to handle a rollback operation, it would fill some data in the last `_rollback` parameter of the `sendCallMessage`,
otherwise it would have a null value which indicates no rollback handling is required.

#### ResponseMessage

For all two-way messages (i.e., `_rollback` is non-null), the `xcall` on the source chain receives a response message
from the `xcall` on the destination chain and emits the following event regardless of its success or not.

```java
/**
 * Notifies that a response message has arrived for the `_sn` if the request was a two-way message.
 *
 * @param _sn The serial number of the previous request
 * @param _code The response code
 *              (0: Success, -1: Unknown generic failure, >=1: User defined error code)
 * @param _msg The result message if any
 */
@EventLog(indexed=1)
void ResponseMessage(BigInteger _sn, int _code, String _msg);
```

#### RollbackMessage

When an error occurred on the destination chain and the `_rollback` is non-null, `xcall` on the source chain emits the following event
for notifying the user that an additional rollback operation is required.

```java
/**
 * Notifies the user that a rollback operation is required for the request '_sn'.
 *
 * @param _sn The serial number of the previous request
 */
@EventLog(indexed=1)
void RollbackMessage(BigInteger _sn);
```

#### executeRollback

The user on the source chain recognizes the rollback situation and invokes the following method on `xcall` with the given `_sn`.
Note that the `executeRollback` can be called only when the original call request has responded with a failure.
It should be reverted when there is no failure response with the call request.

```java
/**
 * Rollbacks the caller state of the request '_sn'.
 *
 * @param _sn The serial number of the previous request
 */
@External
void executeRollback(BigInteger _sn);
```

Then the `xcall` invokes the `handleCallMessage` in the source DApp with the given `_rollback` data.
At this time, the `_from` would be the BTP address of `xcall`.

#### RollbackExecuted

As with the `CallExecuted` event above, the following event is emitted after the DApp's `handleCallMessage` execution
to notify its execution result.

```java
/**
 * Notifies that the rollback has been executed.
 *
 * @param _sn The serial number for the rollback
 * @param _code The execution result code
 *              (0: Success, -1: Unknown generic failure, >=1: User defined error code)
 * @param _msg The result message if any
 */
@EventLog(indexed=1)
void RollbackExecuted(BigInteger _sn, int _code, String _msg);
```

### Fees Handling

If a user wants to make a call from ICON to Target Network 1 (T1), he needs to pay X ICX,
and for Target Network 2 (T2), he needs to pay Y ICX.
That is, the fees depend on the destination network address.

The fees are divided into two types, one is for relays and the other is for protocol itself.
For example, for a destination network T1, the fees could be relayFee = 0.25 ICX and protocolFee = 0.01 ICX.
And relayFee goes to relays, protocolFee goes to BTP protocol (eventually, to the Fee Handler).
In this document, we don't address how to deal with these accrued fees for distribution,
but just define operational parts like how to get the proper fee amount before sending the call request, etc.

Here are getter and setter methods for the proper fees handling in `xcall`.
DApps that want to make a call to `sendCallMessage`, should query the total fee amount for the destination
network via `getFee` interface, and then enclose the appropriate fees in the method call.
Note that the protocol fee amount can be get/set via `xcall`, but the relay fee would be obtained from BMC
which manages BTP network connections.

```java
/**
 * Gets the fee for delivering a message to the _net.
 * If the sender is going to provide rollback data, the _rollback param should set as true.
 * The returned fee is the sum of the protocol fee and the relay fee.
 *
 * @param _net The network address
 * @param _rollback Indicates whether it provides rollback data
 * @return the sum of the protocol fee and the relay fee
 */
@External(readonly=true)
BigInteger getFee(String _net, boolean _rollback);

/**
 * Sets the protocol fee amount.
 *
 * @param _value The protocol fee amount in loop
 * @implNote Only the admin wallet can invoke this.
 */
@External
void setProtocolFee(BigInteger _value);

/**
 * Gets the current protocol fee amount.
 *
 * @return The protocol fee amount in loop
 */
@External(readonly=true)
BigInteger getProtocolFee();

/**
 * Sets the address of Fee Handler.
 * If _addr is null (default), it accrues protocol fees.
 * If _addr is a valid address, it transfers accrued fees to the address and
 * will also transfer the receiving fees hereafter.
 *
 * @param _addr The address of Fee Handler
 * @implNote Only the admin wallet can invoke this.
 */
@External
void setProtocolFeeHandler(@Optional Address _addr);

/**
 * Gets the current protocol fee handler address.
 *
 * @return The protocol fee handler address
 */
@External(readonly=true)
Address getProtocolFeeHandler();
```

## Implementation
* [The RI of the `xcall` between ICON to ICON](https://github.com/icon-project/btp/tree/iconloop-v2/javascore/xcall)

## References
* [ICON BTP Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-25.md)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
