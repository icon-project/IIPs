---
iip: 25
title: ICON BTP Standard
author: MoonKyu Song (@mksong-icon)
discussions-to: https://github.com/icon-project/IIPs/issues/25
status: Draft
type: Standards Track
category: IRC
created: 2020-03-04
---

## Simple Summary

Standard interfaces for contracts used in Blockchain Transmission Protocol (BTP)

## Abstract
<!--
Blockchain Transmission Protocol is used to deliver messages between blockchains. There're two kinds of applications are used in the protocol. The relay is used to deliver messages between blockchains. The relay delivers the messages and the smart contracts handle the delivered messages. Those smart contracts verify and decode the delivered messages, then it processes the decoded messages according to the services. Complex and heavy functions of smart contracts can be shared if they are implemented in standardized interfaces. And also they can be extended more easily.
-->
Blockchain Transmission Protocol is used to deliver messages between blockchains.
A Relay delivers BTP messages between blockchains and smart contracts will verify and decode the delivered messages according to the services.
Complex and heavy functions of smart contracts can be shared if they are implemented according to standardized interfaces. Smart contracts are also more extensible staying compliant to the standards.


## Motivation

BTP Messages from multiple services are delivered to multiple blockchains.
BTP smart contracts to support multiple services and blockchains.
BTP Message Center (BMC) is the center of smart contracts.

For each blockchain, BTP Message Verifier (BMV) verifies the relay message and decodes it into standardized messages (BTP Messages).

For each service, BTP Message Handler (BSH) handles received messages of the service and sends messages through the BTP Message Center (BMC).


## Specification

### Terminology

* [Network Address](#network-address)

  A string to identify blockchain network

* [BTP Address](#btp-address)

  A string of URL for locating an account of the blockchain network

* [Relay Message](#relay-message)

  A message that a relay sends to the blockchain.

* [BTP Message](#btp-message)

  Standardized messages delivered between different blockchains

* [BTP Message Center (BMC)](#btp-message-center)

  BMC accepts messages from a relay (Relay Messages).
  A Relay Message contains standardized messages (BTP Messages) and proof of existence for these messages.
  Corresponding BMV will verify and decode the Relay Message, then the BMC will process the BTP Messages.

  If the destination of the message isn't current BMC, then it's sent to the next BMC to reach its destination.
  If current BMC is the destination, then it's dispatched to the corresponding BSH.
  If the message cannot be processed, then it sends an error back to the source.


* [BTP Message Verifier (BMV)](#btp-message-verifier)

  BMV verifies a Relay Message and decodes it into BTP Messages.


* [BTP Service Handler (BSH)](#btp-service-handler)

  BSH handles BTP Messages of the service. It also sends messages according to different service scenarios.

### Network Address

A string to identify blockchain network

```
<NID>.<Network System>
```

**Network System**:
Short name of the blockchain network system.

| Name     | Description           |
|:---------|:----------------------|
| icon     | ICON MainNet (goloop) | 
| moonbeam | Moonbeam              |

**NID**:
ID of the network in the blockchain network system.

> Example

| Network Address | Description                                               |
|:----------------|:----------------------------------------------------------|
| `0x1.icon`      | ICON network with nid="0x1" (main net)                    |
| `0x2.icon`      | ICON sub network with nid="0x2" (sub network on main net) |
| `0x5.moonbeam`  | Moonbeam network with nid="0x5"                           |

### BTP Address

A string of URL for locating an account of the blockchain network

> Example
```
btp://<Network Address>/<Account Identifier>
```
**Account Identifier**:
Identifier of the account including smart contract.
It should be composed of URL safe characters except "."(dot).

> Example
```
btp://0x1.icon/hxc0007b426f8880f9afbab72fd8c7817f0d3fd5c0
btp://0x2.icon/hx7ab72fd8c7812680f9afbab72fd8c7817f0d3fd5
btp://0x5.moonbeam/0x5425F5d4ba2B7dcb277C369cCbCb5f0E7185FB41
```

It could be expanded to other resources.

### BTP Message

A message delivered across blockchains.

| Name | Type     | Description                                          |
|:-----|:---------|:-----------------------------------------------------|
| src  | String   | Network Address of source network                    |
| dst  | String   | Network Address of destination network               |
| svc  | String   | name of the service                                  |
| sn   | Integer  | serial number of the message                         |
| msg  | Bytes    | serialized bytes of Service Message or Error Message |
| nsn  | Integer  | network serial number of the message                 |
| fee  | RelayFee | relay fee information                                |

If **sn** is negative, **msg** should be Error Message.

If **nsn** is negative, then it's a responding message to the message from **dst** with **-nsn**.
Positive **nsn** and **src** or negative **nsn** and **dst** can be used to make ID for tracking message delivery. They are used for generating [BTP Event](#btpevent)

If **fee** is null, then [BMC](#btp-message-center) does nothing. Otherwise, it processes fee information and updates it after processing.

It would be serialized in [RLP serialization](#rlp-serialization) as structure.

### Relay Fee

Relay fee information for the message

| Name   | Type      | Description                        |
|:-------|:----------|:-----------------------------------|
| net    | String    | Network Address of relay fee       |
| values | []Integer | Relay fee for relaying the message |

If **fee** is not empty, then BMC removes the first entry from the list and it will accrue the fee. for claim in later (using [claimReward](#claimreward) )


### Error Message

A message for delivering error information.

| Name | Type    | Description   |
|:-----|:--------|:--------------|
| code | Integer | error code    |
| msg  | String  | error message |

It would be serialized in [RLP serialization](#rlp-serialization).

Defined codes are followings.

| Code | Description                   |
|:-----|:------------------------------|
| 0    | Success                       |
| 1    | Unknown                       |
| 2    | No route to the destination   |
| 3    | No registered service handler |
| 4    | Reverted by service handler   |

### RLP serialization

For encoding [BTP Message](#btp-message) and [Error Message](#error-message), it uses Recursive Length Prefix (RLP).
RLP supports bytes and list naturally.
Here are some descriptions about other types.

#### String

It uses UTF-8 encoded bytes of the string.
There is no termination bytes.

#### Integer

It uses the shortest form of two's complemented bytes representations.
If it's negative, the highest bit of the first byte should be 1.

> Example

| Value | Encoded bytes |
|:------|:--------------|
| 0     | 0x00          |
| -1    | 0xff          |
| -128  | 0x80          |

#### Structure

It encodes the structure as a list of field values in the order. When it decodes bytes to the structure, it doesn't have enough values for the fields, then those fields are reset as null values. It has more values than fields, then those values are dropped and silently ignored.

### Relay Message

It's used to deliver BTP Messages along with other required contents. Normally, it contains the following.

* BTP Messages along with their proof of existence
* Trust information updates along with their proof of consensus

The relay gathers the information through APIs of a source blockchain system and its internal database. The actual content of the message is decided according to the blockchain system and BMV implementation.


### BTP Message Center

#### Introduction

BTP Message Center is a smart contract that builds BTP Message and sends it to a relay and handles Relay Message from the other relay. It stores the following information.

* Routing information
  * Reachable networks and corresponding BMCs
  * Next BMCs for reachable networks
  * Directly connected networks
  * Current network address
* Smart contracts
  * BMV address for the directly connected network
  * BSH address for the service

#### Setup

1. Registers [BSH](#btp-service-handler)s for the services.
   (BSH should be deployed before the registration)
2. Registers [BMV](#btp-message-verifier)s for the directly connected blockchains.
   (BMV should be deployed before the registration)
3. Adds links, BMCs of directly connected blockchains
4. Adds routes to other BMCs of in-directly connected blockchains

#### Send a message

BSH sends a message through [BMC.sendMessage](#sendmessage). It accepts only requests from the registered BTP Service Handler (BSH). If the service name of those requests is not in the service names of the BSH, then it will be rejected.

Then it builds a BTP Message from the request.
1. Decides destination BMC from the given Network Address
2. Fills in other information from given parameters.
3. Serializes them for sending.

Then it tries to send the BTP Message.
1. Decide next BMC from the destination referring routing information.
2. Get sequence number corresponding to the next.
3. Emit the event, [Message](#message) including the information.

The event will be monitored by the Relay, it will build Relay Message
for next BMC.

#### Receive a message

It receives the Relay Message, then it tries to decode it with registered
BMV. It may contain multiple BTP Messages.
It dispatches received BTP Messages one-by-one in sequence.

If it is the destination, then it tries to find the BSH for the
service, and then calls [BSH.handleBTPMessage](#handlebtpmessage).
It calls [BSH.handleBTPError](#handlebtperror) if it's an error.

If it's not the destination, then it sends the message to
the next BMC reaching the destination.

If it fails, then it replies an error.
BTP Message with error reply is composed of the following,
* sn : negated serial number of the message.
* dst : BTP Address of the source.
* src : BTP Address of the BMC.
* msg : Error Message including error code and message.

#### BMC service message

BMC service message is a BTP message dispatched by BMC.
In that case service name would be `bmc`.
And the message is RLP encoded list of following values.

| Name    | Type   | Description                                            |
| :------ | :----- | :----------------------------------------------------- |
| type    | String | type of BMC Service Message (Init, Link, Unlink, Sack) |
| payload | Bytes  | serialized bytes of Message                            |

Payload of message is also a RLP encoded list of the fields in the following
message types.

##### Init Message

send to given _link on [BMC.addLink](#addlink)

| Name  | Type           | Description                          |
| :---- | :------------- | :----------------------------------- |
| links | List of String | list of BTP Address of connected BMC |

BMC could update status of connected BMC to use to resolve route.

##### Link Message

send to all of connected BMC except given _link on [BMC.addLink](#addlink)

| Name | Type   | Description                  |
| :--- | :----- | :--------------------------- |
| link | String | BTP Address of connected BMC |

BMC could update status of connected BMC to use to resolve route.

##### Unlink Message

send to all of connected BMC except given _link on [BMC.removeLink](#removelink)

| Name | Type   | Description                  |
| :--- | :----- | :--------------------------- |
| link | String | BTP Address of connected BMC |

BMC could update status of connected BMC to use to resolve route.

#### Interface

##### Writable methods

###### handleRelayMessage
```python
@external
def handleRelayMessage(self, _prev: str, _msg: str):
```
* Params
  - _prev: String ( BTP Address of the previous BMC )
  - _msg: String ( base64 encoded string of serialized bytes of Relay Message )
* Description:
  - It verifies and decodes the Relay Message with BMV and dispatches BTP
    Messages to registered BSHs.
  - It's allowed to be called by the BMC.

###### sendMessage
```python
@external
@payable
def sendMessage(self, _to: str, _svc: str, _sn: int, _msg: bytes) -> int:
```
* Params
  - _to: String ( Network Address of destination network )
  - _svc: String ( name of the service )
  - _sn: Integer ( serial number of the message )
  - _msg: Bytes ( serialized bytes of Service Message )
* Description:
  - Sends the message to a specific network.
  - If _sn is postive, then it assumes that the message is two-way message. It expects a reponse or a delivery failure.
  - If _sn is zero, then it assumes that the message is one-way message. It could be dropped by an error. 
  - If _sn is negative, then it sends the response for the message from _to with same _svc and negated _sn.
    If there is no message related with _svc, _to and _sn, then it reverts.
    It uses negated value of nsn of the message for nsn of the responding BTP message.
  - Only allowed to be called by registered BSHs.
* Returns
  - Network serial number
  - It would zero on sending response [TBD]

###### addService
```python
@external
def addService(self, _svc: str, _addr: Address):
```
* Params
  - _svc: String (the name of the service)
  - _addr: Address (the address of the smart contract handling the service)
* Description:
  - Registers the smart contract for the service.
  - Called by the operator to manage the BTP network.

###### removeService
```python
@external
def removeService(self, _svc: str):
```
* Params
  - _svc: String (the name of the service)
* Description:
  - De-registers the smart contract for the service.
  - Called by the operator to manage the BTP network.

###### addVerifier
```python
@external
def addVerifier(self, _net: str, _addr: Address):
```
* Params
  - _net: String (Network Address of the blockchain )
  - _addr: Address (the address of BMV)
* Description
  - Registers BMV for the network.
  - Called by the operator to manage the BTP network.

###### removeVerifier
```python
@external
def removeVerifier(self, _net: str):
```
* Params
  - _net: String (Network Address of the blockchain )
* Description
  - De-registers BMV for the network.
  - May fail if it's referred by the link.
  - Called by the operator to manage the BTP network.

###### addLink
```python
@external
def addLink(self, _link: str):
```
* Params
  - _link: String (BTP Address of connected BMC)
* Description
  - If it generates the event related to the link, the relay shall
    handle the event to deliver BTP Message to the BMC.
  - If the link is already registered, or its network is already
    registered then it fails.
  - If there is no verifier related with the network of the link,
    then it fails.
  - Initializes status information for the link.
  - Called by the operator to manage the BTP network.
  - It sends [BMC.Init](#init-message) to added BMC.
  - It sends [BMC.Link](#link-message) to already registered BMCs.

###### removeLink
```python
@external
def removeLink(self, _link: str):
```
* Params
  - link: String (BTP Address of connected BMC)
* Description
  - Removes the link and status information.
  - Called by the operator to manage the BTP network.
  - It sends [BMC.Unlink](#unlink-message) to other remaining BMCs.

###### addRoute
```python
@external
def addRoute(self, _dst: str, _link: str):
```
* Params
  - _dst: String ( Network Address of the destination network )
  - _link: String ( BTP Address of the next BMC for the destination )
* Description:
  - Add route to the network.
  - May fail if there is already registered route to the network.
  - Called by the operator to manage the BTP network.

###### removeRoute
```python
@external
def removeRoute(self, _dst: str):
```
* Params
  - dst: String ( Network Address of the destination network )
* Description:
  - Remove route to the network.
  - Called by the operator to manage the BTP network.

##### Read-only methods

###### getServices
```python
@external(readonly=True)
def getServices(self) -> dict:
```
* Description
  - Get registered services.
* Returns
  - A dictionary with the name of the service as key and address of the BSH
    related to the service as value.
    ```json
    {
      "token": "cx72eaed466599ca5ea377637c6fa2c5c0978537da"
    }
    ```

###### getVerifiers
```python
@external(readonly=True)
def getVerifiers(self) -> dict:
```
* Description
  - Get registered verifiers.
* Returns
  - A dictionary with the Network Address as a key and smart contract
    address of the BMV as a value.
    ```json
    {
        "0x1.icon": "cx72eaed466599ca5ea377637c6fa2c5c0978537da"
    }
    ```

###### getLinks
```python
@external(readonly=True)
def getLinks(self) -> list:
```
* Description
  - Get registered links.
* Returns
  -  A list of links ( BTP Addresses of the BMCs )
  ```json
  [ "btp://0x1.icon/cx9f8a75111fd611710702e76440ba9adaffef8656" ]
  ```

###### getRoutes
```python
@external(readonly=True)
def getRoutes(self) -> dict:
```
* Description:
  - Get routing information.
* Return
  - A dictionary with the Network Address of the destination network as key and
    the BTP Address of the next(link) as a value.
    ```json
    {
      "0x2.icon": "btp://0x1.icon/cx9f8a75111fd611710702e76440ba9adaffef8656"
    }
    ```

###### getStatus
```python
@external(readonly=True)
def getStatus(self, _link: str) -> dict:
```
* Params
  - _link: String ( BTP Address of the connected BMC )
* Description:
  - Get status of BMC.
  - Used by the relay to resolve next BTP Message to send.
  - If target is not registered, it will fail.
* Return
  - The object contains followings fields.

    | Field    | Type    | Description                                      |
    |:---------|:--------|:-------------------------------------------------|
    | tx_seq   | Integer | next sequence number of the next sending message |
    | rx_seq   | Integer | next sequence number of the message to receive   |
    | verifier | Object  | status information of the BMV                    |

  - verifier is object returned by [BMV.getStatus](#bmvgetstatus)


##### Events

###### Message
```python
@eventlog(indexed=1)
def Message(self, _next: str, _seq: int, _msg: bytes):
```
* Indexed: 1
* Params
  - _next: String ( BTP Address of the BMC to handle the message )
  - _seq: Integer ( sequence number of the message from current BMC to the next )
  - _msg: Bytes ( serialized bytes of BTP Message )
* Description
  - Sends the message to the next BMC.
  - The relay monitors this event.

###### BTPEvent
```python
@eventlog(indexed=2)
def BTPEvent(self, _src: str, _nsn: int, _next: str, _event: str):
```
* Indexed: 2
* Params
  - _src: String ( Network Address of the source network of the message )
  - _nsn: Integer ( Network Serial Number of the message in the source network )
  - _next: String ( Network Address of the next network for route )
  - _event: String ( Event name )

    | Event name | Description                                                    |
    |:-----------|:---------------------------------------------------------------|
    | SEND       | The message from _src with _nsn is sent to _next               |
    | ROUTE      | The message from _src with _nsn is routed to _next             |
    | DROP       | The message from _src with _nsn is dropped                     |
    | RECEIVE    | The message from _src with _nsn is received                    |
    | REPLY      | The reply for the message from _src with _nsn is sent to _next |
    | ERROR      | The error for the message from _src with _nsn is sent to _next |

* Description
  - It's generated while the message is processed in the BMC.
  - _src and _nsn can be used as ID for tracking message delivery.

* Example
  A message delivery from A to B through H
  ```
  A.BMC : BTPEvent(A,<nsn>,H,SEND)
  H.BMC : BTPEvent(A,<nsn>,B,ROUTE)
  B.BMC : BTPEvent(A,<nsn>,null,RECEIVE)
  ```
  If it's responded
  ```
  B.BMC : BTPEvent(A,<nsn>,H,REPLY)
  H.BMC : BTPEvent(A,<nsn>,A,ROUTE)
  A.BMC : BTPEvent(A,<nsn>,null,RECEIVE)
  ```

##### Error codes

Error codes will be used on [error message](#error-message) by BMC.
Followings are defined error codes.

| Code | Description                                                 |
|:-----|:------------------------------------------------------------|
| 0    | <a id="error-code-success"></a>Success                      |
| 1    | Unknown                                                     |
| 2    | No route to the destination                                 |
| 3    | No registered service handler                               |
| 4    | <a id="error-code-reverted"></a>Reverted by service handler |

#### Relay Fee

To sustain BTP network, [Relay Message](#relay-message)s need to be delivered continuously.
To make relays deliver them, it charges fees for delivery and distributes them to relays as much as they did.
For this, we have invented to **Relay Fee** feature.

[BMC](#btp-message-center) do following works.
* Knows fee information for the delivery.
* Charges [BSH](#btp-service-handler) the sum of the fees for the delivery.
* Inject fee information on sending a BTP message.
* Collect fees for each relays on compiling BTP messages.
* Handle claim requests from the relays.

##### Added Behavior

###### Know fee information

BMC needs to know fees for each relay message deliveries for a delivery.
If there is no routing blockchain, it requires two fees for a delivery.
If A is the source, and B is the destination, then it needs to know two fees.

* `Fee(A,B)`, fee from A to B
* `Fee(B,A)`, fee from B to A

If there are N mediating blockchains (`C1` to `Cn`), then it needs to know forward direction fees as the following.

`Fee(C1,C2)`, `Fee(C2, C3)`, ... , `Fee(Cn-1,Cn)`

And also it needs to know backward direction fees for response as the following.

`Fee(C2,C1)`, `Fee(C3, C2)`, ... , `Fee(Cn,Cn-1)`

So, it needs to know `2(N-1)` fees.

These values would be used for **fee** field of [BTP Message](#btp-message).
Please refer [Inject fee information](#inject-fee-information)

The implementor need to make own implementation to manage those values.
But those values needs to be ready before sending any messages.

###### Charge fee

Whenver [BSH](#btp-service-handler) sends a new message to the destination
with [sendMessage](#sendmessage), it should also pay a proper fee.
[getFee](#getfee) should return proper fee amount to pay.

When it sends the response for the message, it doesn't need to pay a fee,
because it's already payed by original sender of the message.

###### Inject fee information

Whenever it sends a message with [sendMessage](#sendmessage), then it may fill **fee** field of [BTP Message](#btp-message) with proper information

If it sends a new message (not response), then it sets **fee** of [BTP Message](#btp-message) as [Relay fee](#relay-fee) with the following values.

* src : Network Address of current blockchain
* fee : A list of fees to pay for each relay message delivery
  If the message is two-way message, having positive *sn*, then it includes forward and backward fees in the order.
  If the message is one-way message, having zero *sn*, then it includes forward fees.

Otherwise, it uses stored information which is stored in the step
([Dispatch BTP Message](#dispatch-btp-message))

###### Compile BTP Mesasge

Whenever it receives the BTP message from the relay, it checkes **fee** field of [Relay Fee](#relay-fee). If it's not available or empty, then it does nothing. Otherwise, it fetches the first entry of it accrues the fee as the reward for the sender. Transaction sender is a relay.

**Example**

BTP Message is sent to `0x2.icon` at `0x1.icon`.
BMC of `0x2.icon` receives the message like following.

```json
{
  "src": "0x1.icon",
  "dst": "0x2.icon",
  "sn" : 10,
  "nsn" : 376,
  "fee": {
    "network": "0x1.icon",
    "fee": [ 10, 8 ]
  }
}
```

It accrues the fee as the reward before dispatching it.

```json
{
  "src": "0x1.icon",
  "dst": "0x2.icon",
  "sn" : 10,
  "nsn" : 376,
  "fee": {
    "network": "0x1.icon",
    "fee": [ 8 ]
  }
}
```

Then it follows [receives a message](#receive-a-message).

###### Dispatch BTP message

When the message arrives at the destination, it stores **fee** field of [Relay Fee](#relay-fee) for the response if the serial number is positive (two-way message). And this information should be used on sending response for the message.

Abandoned fees will be accrued as a reward for BMC, itself.

##### Writable methods

###### claimReward
```python
@external
@payable
def claimReward(self, _network: str, _receiver: str):
```
* Params
  - _network: String ( Network Address of the reward )
  - _receiver: String ( Reward receiver in the target network )
* Description
  - Claim accrued reward for relaying messages by the **sender**.
  - If network is same as current network, then it directly transfer reward to it.
  - Otherwise, it sends a [Claim Message](#claim-message) to the BMC of the target network.

* Event
  - [ClaimReward](#claimreward-event) on sending claim request to the other network.
  - [ClaimRewardResult](#claimrewardresult-event) on receiving the result of the claim request

##### Read-only methods

###### getFee
```python
@external(readonly=True)
def getFee(self, _network: str, _response: bool) -> int:
```
* Params
  - network: String ( Network Address of the destination network )
  - response: Bool ( Whether it requires response )
* Description
  - Returns relay fee to send a message to the network.
  - If it wants to get a response or delivery failure for the message, then _response must be true and positive value is used for _sn on [BMC.sendMessage](#sendmessage)
* Returns
  - Relay fee for the delivery

###### getReward
```python
@external(readonly=True)
def getReward(self, _network: str, _addr: Address) -> int:
```
* Params
  - network: String ( Network Address of the reward )
  - addr: Address ( Address of the relay )
* Description
  - Get accrued reward amount of the network for the relay
* Returns
  - Amount of the reward for the relay in the unit of target network

##### Events

###### ClaimReward event
```python
@eventlog(indexed=2)
ClaimReward(self, _sender: Address, _network: str, _amount: int, _nsn: int):
```
* Params
  - sender: Address ( Claiming requestor )
  - network: String ( Network Address for the reward )
  - nsn: Integer ( Network Serial Number for the request )
* Description
  - It's generated when it succeeds to send claiming request to the 
* Event

###### ClaimRewardResult event
```python
@eventlog(indexed=2)
ClaimRewardResult(self, _sender: Address, _network: str, _nsn: int, _result: int):
```
* Params
  - sender: Address ( Claiming requestor )
  - network: String ( Network Address for the reward )
  - nsn: Integer ( Network Serial Number for the request )
  - result: Integer ( Result of the request )
* Description
  - It's generated when it receives the result of the claiming request.

##### BMC service message for Relay Fee

Its extension to [BMC service message](#bmc-service-message)

###### Claim Message

Type of message is `Claim`.

| Name     | Type    | Description                    |
|:---------|:--------|:-------------------------------|
| amount   | Integer | Amount of to be claimed        |
| receiver | String  | Receiver of the claimed reward |

On success, it sends [error message](#error-message) with zero code.
BMC may use `nsn` of [BTP Message](#btp-message) to handle the result.

### BTP Message Verifier

#### Introduction

BTP Message Verifier verifies and decodes Relay Messages to
[BTP Message](#btp-message)s.
A Relay Message is composed of both BTP Messages and with proof of
existence for these BTP Messages.

For easy verification, it may update trust information for the
followed events. Most of the implementations may track the hashes of block
headers.

If the blockchain system provides proof of absence of the
BTP Messages, then it's enough for the verifier to sustain the last state only.
It updates the hash only if it sees proof of absence of further
BTP Messages in the block.

Most blockchain systems don't provide proof of absence for their data, therefore, it is mandatory to
provide methods to verify historical hashes.

Merkle Accumulator can be used for verifying old hashes.
BMV sustains roots of Merkle Tree Accumulator, and relay
will sustain all elements of Merkle Tree Accumulator. The relay
may make the proof of any one of old hashes.
So, even if byzantine relay updated the trust information with the
proof of new block, normal relay can send BTP Messages in
the past block with the proof.

#### Interface

##### Writable methods

###### BMV.handleRelayMessage
```python
@external
def handleRelayMessage(self, _bmc: str, _prev: str, _seq: int, _msg: bytes) -> list:
```
* Description
  - Decodes Relay Messages and process BTP Messages
  - If there is an error, then it sends a BTP Message containing the
    Error Message
  - BTP Messages with old sequence numbers are ignored. A BTP Message contains future sequence number will fail.
* Params
  - _bmc: String ( BTP Address of the BMC handling the message )
  - _prev: String ( BTP Address of the previous BMC )
  - _seq: Integer ( next sequence number to get a message )
  - _msg: Bytes ( serialized bytes of Relay Message )
* Returns
  - List of serialized bytes of a [BTP Message](#btp-message)

##### Read-only methods

###### BMV.getStatus
```python
@external(readonly=True)
def getStatus(self) -> dict:
```
* Description
  - Get status of the BMV
* Returns
  - A dictionary with following required keys. Additional keys are allowed for own purpose

    | Name   | Type    | Description                |
    |:-------|:--------|:---------------------------|
    | height | Integer | Height of finalized blocks |
    | extra  | Bytes   | Encoded extra data         |

  - `extra` field is used for chain specific status information.

### BTP Service Handler

#### Introduction

BSH can send messages through BTP Message Center(BMC) from any user
request, the request can also come from other smart contracts.
BSHs are also responsible for handling message from other BSHs.

BSH can communicate with other BSHs with the same service name.
If there is already a service using the same name, then it should choose
a different name for the service when registering a new service.
If the intention is to become a part of the service, then it should
use same name. BSH follows the protocol of the service.

Before a BSH is registered to the BMC, it's unable to send messages, and unable to handle messages from others.
To become a BSH, following criteria must be met,

1. Implements the interface
2. Registered to the BMC through [BMC.addService](#addservice)

After registration, it can send messages through
[BMC.sendMessage](#sendmessage).
If there is an error while delivering message, BSH will
return error information though [handleBTPError](#handlebtperror).
If messages are successfully delivered, BMC will call
[handleBTPMessage](#handlebtpmessage) of the target BSH.
While processing the message, it can reply though
[BMC.sendMessage](#sendmessage).

#### Security

BSH should not handle messages or errors from other contract
except BMC.
BMC also accepts only the service messages from registered BSH.
BSH can have other APIs, but APIs related with BMC are
only called by BMC.

#### Interface

##### Writable methods

###### handleBTPMessage
```python
@external
def handleBTPMessage(self, _from: str, _svc: str, _sn: int, _msg: bytes):
```
* Description
  - Handles BTP Messages from other blockchains.
  - Accepts messages only from BMC.
  - If it fails, then BMC will generate a BTP Message that includes
    error information, then delivered to the source.
* Params
  - _from: String ( Network Address of source network )
  - _svc: String ( name of the service )
  - _sn: Integer ( serial number of the message )
  - _msg: Bytes ( serialized bytes of ServiceMessage )

###### handleBTPError
```python
@external
def handleBTPError(self, _src: str, _svc: str, _sn: int, _code: int, _msg: str):
```
* Description
  - Handle the error on delivering the message.
  - Accept the error only from the BMC.
* Params
  - _src: String ( Network Address of BMC that generated the error )
  - _svc: String ( name of the service )
  - _sn: Integer ( serial number of the original message )
  - _code: Integer ( code of the error )
  - _msg: String ( message of the error )


### Message delivery flow

![Components](../assets/iip-draft_btp/btp_components.svg)

1. BSH sends a Service Message through BMC.

   * BSH calls [BMC.sendMessage](#sendmessage) with followings.

     | Name    | Type    | Description                                   |
     |:--------|:--------|:----------------------------------------------|
     | _to     | String  | Network Address of the destination blockchain |
     | _svc    | String  | Name of the service.                          |
     | _sn     | Integer | Serial number of the message.                 |
     | _msg    | Bytes   | Service message to be delivered.              |

   * BMC lookup the destination BMC belonging to *_to*.
     If there is no known BMC to the network, then it will fail.

   * BMC builds a BTP Message.

     | Name | Type    | Description                                   |
     |:-----|:--------|:----------------------------------------------|
     | src  | String  | Network address of the current blockchain     |
     | dst  | String  | Network address of the destination blockchain |
     | svc  | String  | Given service name                            |
     | sn   | Integer | Given serial number                           |
     | msg  | Bytes   | Given service message                         |
     | nsn  | Integer | Generated network serial number               |
     | fee  | Fee     | Fee information for the target                |

   * BMC decide the next BMC according to the destination.
     If there is no route to the destination BMC.

   * BMC sends BTP Message using the method that the blockchain supports.
     It may send an message through the event containing the following
     information.

     | Name  | Type    | Description                                |
     |:------|:--------|:-------------------------------------------|
     | _next | String  | BTP Address of the next BMC                |
     | _seq  | Integer | Sequence number of the msg to the next BMC |
     | _msg  | Bytes   | Serialized BTP Message                     |

2. The BTP Message Relay(BMR) detects events.
   * Relay detects [BMC.Message](#message) through various ways.
   * Relay can confirm that it occurs and it's finalized.

3. BMR gathers proofs
   * Relay gathers proofs of the event(POE)s
     - Proof for the new block
     - Proof for the event in the block
   * Relay builds Relay Message including the following,
     - Proof of new events
     - New events including the BTP Message.
   * Relay calls [BMC.handleRelayMessage](#handlerelaymessage)
     with built Relay Message.

   | Name  | Type   | Description                                     |
   |:------|:-------|:------------------------------------------------|
   | _prev | String | BTP Address of the previous BMC                 |
   | _msg  | Bytes  | serialized Relay Message including BTP Messages |

4. BMC handles Relay Message

   * It finds BMV for the network address of the previous BMC.
   * It gets the sequence number of the next message from the source network.
   * BMC calls [BMV.handleRelayMessage](#bmvhandlerelaymessage)
     to decode Relay Message and gets a list of BTP Messages.

     | Name  | Type    | Description                                               |
     |:------|:--------|:----------------------------------------------------------|
     | _bmc  | String  | BTP Address of the current BMC                            |
     | _prev | String  | BTP Address of given previous BMC                         |
     | _seq  | Integer | Next sequence number of the BTP Message from previous BMC |
     | _msg  | Bytes   | The Relay Message                                         |

5. BMV decodes Relay Message

   * Verifies and decodes Relay Messages, then returns a list of
     BTP Messages.
   * If verification of the message is unsuccessful, it fails.
   * The events from the previous BMC to the current BMC will be processed.
   * The events should have proper sequence number, otherwise it fails.

6. BSH handles Service Messages

   * BMC dispatches BTP Messages.
   * If the destination isn't current one, then it locates
     the next BMC and routes the message.
   * If the destination is the current one, then it locates BSH
     for the service of the BTP Message.
   * If *_sn* is positive and it stores *fee* and *nsn* for the response
     to *_from* for the message with  *_svc* and *_sn*.
   * Calls [BSH.handleBTPMessage](#handlebtpmessage) if
     the message has a non-negative value as *_sn*.

     | Name  | Type    | Description                           |
     |:------|:--------|:--------------------------------------|
     | _from | String  | Network Address of the source network |
     | _svc  | String  | Given service name                    |
     | _sn   | Integer | Given serial number                   |
     | _msg  | Bytes   | Given service message                 |

   * Otherwise, it calls [BSH.handleBTPError](#handlebtperror).

     | Name  | Type    | Description                                    |
     |:------|:--------|:-----------------------------------------------|
     | _src  | String  | Network Address of BMC generateed the message  |
     | _svc  | String  | Service name of the message                    |
     | _sn   | Integer | Serial number of the message                   |
     | _code | Integer | Given error code                               |
     | _msg  | String  | Given error message                            |

7. BSH sends a response

    * It should send a response if *_sn* is positive.

     | Name  | Type    | Description                         |
     |:------|:--------|:------------------------------------|
     | _to   | String  | _from of the message                |
     | _svc  | String  | _svc of the message                 |
     | _sn   | Integer | Negated value of _sn of the message |
     | _msg  | Bytes   | Response to the message             |

8. BMC sends a response

    * It gets stored response information ( *fee* and *nsn* )

    * It builds BTP message for response

     | Name | Type    | Description                                   |
     |:-----|:--------|:----------------------------------------------|
     | src  | String  | Network address of the current blockchain     |
     | dst  | String  | Network address of the destination blockchain |
     | svc  | String  | Service name                                  |
     | sn   | Integer | 0                                             |
     | msg  | Bytes   | Response to the message                       |
     | nsn  | Integer | Negated value of nsn of the BTP message       |
     | fee  | Fee     | Fee information of the BTP message            |

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->


## Implementation
<!--The implementations must be completed before any IIP is given status "Final", but it need not be completed before the IIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
