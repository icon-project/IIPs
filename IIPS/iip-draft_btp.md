---
iip: <to be assigned>
title: ICON BTP Standard
author: MoonKyu Song (@mksong-icon)
discussions-to: MoonKyu Song (@mksong-icon)
status: Draft
type: Standard Track
category: IRC
created: 2020-03-04
---

## Simple Summary

Standard interfaces for contracts used in Blockchain Transmission Protocol(BTP)

## Abstract

Blockchain Transmission Protocol is used to deliver messages between blockchains.
There're two kinds of applications are used in the protocol.
The relay is used to deliver messages between blockchains.
The relay delivers the messages and the smart contracts handle the delivered messages.
Those smart contracts verify and decode the delivered messages, then it processes the decoded messages according to the services.
Complex and heavy functions of smart contracts can be shared if they are implemented in standardized interfaces. And also they can be extended more easily.


## Motivation

BTP Messages of multiple services are delivered between multiple blockchains.
BTP smart contracts support multiple services and blockchains.
BTP Message Center(BMC) is a center of smart contracts.

For each blockchain, BTP Message Verifier(BMV) verifies the relay message and decodes it into standardized messages (BTP Messages).

For each service, BTP Message Handler(BSH) handles received messages of the service and sends messages through the BMC.


## Specification

### Terminology
  
* [Network Address](#network-address)

  A string for addressing blockchain network
  
* [BTP Address](#btp-address)

  URI for addressing a resource in a blockchain
  
* [Relay Message](#relay-message)

  A message that the relay sends to the blockchain.
  
* [BTP Message](#btp-message)

  Standardized message delivered between blockchains

* [BTP Message Center(BMC)](#btp-message-center)

  It accepts a message from a relay (Relay Message).
  A Relay Message contains standardized messages(BTP Messages) and proof of existence of them.
  Corresponding BMV is used for verifying and decoding the Relay Message.
  Then the BMC processes BTP Messages.

  If the destination of the message isn't current BMC, then it's sent to the next BMC reaching the destination.
  If current BMC is the destination, then it's dispatched to corresponding BSH.
  If there is no way to handle the message, then it sends an error back to the source.


* [BTP Message Verifier(BMV)](#btp-message-verifier)

  It verifies a Relay Message and decodes it into BTP Messages.


* [BTP Service Handler(BSH)](#btp-service-handler)

  It handles BTP Messages of the service. It also sends messages according to the service scenario.

### Network Address

A string to identify blockchain network

```
<NID>.<Network System>
```

**Network System**:
Short name of the blockchain network system.

| Name   | Description          |
|:-------|:---------------------|
| icon   | Loopchain for ICON   |
| iconee | Loopchain Enterprise |

**NID**:
ID of the network in the blockchain network system.

> Example

| Network Address | Description                                  |
|:----------------|:---------------------------------------------|
| `0x1.icon`      | ICON Network with nid="0x1" <- ICON Main-Net |
| `0x1.iconee`    | Loopchain Enterprise network with nid="0x1"  |

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
btp://0x1.iconee/cx429731644462ebcfd22185df38727273f16f9b87
```

It could be expanded to other resources.

### BTP Message

A message delivered across blockchains.

| Name | Type    | Description                                          |
|:-----|:--------|:-----------------------------------------------------|
| src  | String  | BTP Address of source BMC                            |
| dst  | String  | BTP Address of destination BMC                       |
| svc  | String  | name of the service                                  |
| sn   | Integer | serial number of the message                         |
| msg  | Bytes   | serialized bytes of Service Message or Error Message |

if **sn** is negative, **msg** should be Error Message.
It would be serialized in [RLP serialization](#rlp-serialization).


### Error Message

A message for delivering error information.

| Name | Type    | Description   |
|:-----|:--------|:--------------|
| code | Integer | error code    |
| msg  | String  | error message |

It would be serialized in [RLP serialization](#rlp-serialization).


### RLP serialization

For encoding [BTP Message](#btp-message) and [Error Message](#error-message), it uses RLP.
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

### Relay Message

It's used to deliver BTP Messages along with other required contents. Normally, it contains the following.

* BTP Messages along with proof of the existence of them
* Trust information updates along with proof of consensus on them

The relay gathers the information through APIs of a source blockchain system and the internal database. The actual content of the message is decided according to the blockchain system and BMV implementation.


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
3. Add links, BMCs of directly connected blockchains
4. Add routes to other BMCs of in-directly connected blockchains

#### Send a message

BSH sends a message through [BMC.sendMessage](#sendmessage). It accepts only requests from the registered BTP Service Handler(BSH). Of course, if the service name of those requests is not in the service names of the BSH, then they will be rejected.

Then it builds a BTP Message from the request.
1. Decide destination BMC from given Network Address
2. Fill in other information from parameters.
3. Serialize them for sending.

Then it tries to send the BTP Message.
1. Decide next BMC from the destination referring routing information.
2. Get sequence number corresponding to the next.
3. Emit the event, [Message](#message) including the information.

The event will be monitored by the Relay, it will build Relay Message
for next BMC.

#### Receive a message

It receives the Relay Message, then it tries to decode it with registered
BMV. It may contain multiple BTP Messages.
It dispatches received BTP Messages one-by-one in the sequence.

If it is the destination, then it tries to find the BSH for the
service, and then calls [BSH.handleBTPMessage](#handlebtpmessage).
It calls [BSH.handleBTPError](#handlebtperror) if it's an error.

If it's not the destination, then it sends the message to
the next BMC reaching the destination.

If it fails, then it replies an error.
BTP Message for error reply is composed of followings.
* sn : negated serial number of the message.
* dst : BTP Address of the source.
* src : BTP Address of the BMC.
* msg : Error Message including error code and message.

#### Interface

##### Writable methods

###### handleRelayMessage
```python
@external
def handleRelayMessage(self, _from: str, _msg: str):
```
* Params
  - from: String ( BTP Address of the BMC generates the message )
  - msg: String ( base64 encoded string of serialized bytes of Relay Message )
* Description:
  - It verifies and decodes the Relay Message with BMV and dispatches BTP
    Messages to registered BSHs.
  - It's allowed to be called by the BMC.

###### sendMessage
```python
@external
def sendMessage(self, dst: str, svc: str, sn: int, msg: bytes):
```
* Params
  - dst: String ( Network Address of destination network )
  - svc: String ( name of the service )
  - sn: Integer ( serial number of the message, it should be positive )
  - msg: Bytes ( serialized bytes of Service Message )
* Description:
  - It sends the message to specific network.
  - It's allowed to be called by registered BSHs.
  
###### addService
```python
@external
def addService(self, name: str, addr: Address):
```
* Params
  - name: String (the name of the service)
  - addr: Address (the address of the smart contract handling the service)
* Description:
  - It registers the smart contract for the service.
  - It's called by the operator to manage the BTP network.
  
###### removeService
```python
@external
def removeService(self, name: str):
```
* Params
  - name: String (the name of the service)
* Description:
  - It de-registers the smart contract for the service.
  - It's called by the operator to manage the BTP network.
  
###### addVerifier
```python
@external
def addVerifier(self, net_addr: str, addr: Address):
```
* Params
  - net_addr: String (Network Address of the blockchain )
  - addr: Address (the address of BMV)
* Description
  - Registers BMV for the network.
  - It's called by the operator to manage the BTP network.
  
###### removeVerifier
```python
@external
def removeVerifier(self, net_addr: str):
```
* Params
  - net_addr: String (Network Address of the blockchain )
* Description
  - De-registers BMV for the network.
  - It may fail if it's referred by the link.
  - It's called by the operator to manage the BTP network.
  
###### addLink
```python
@external
def addLink(self, link: str):
```
* Params
  - link: String (BTP Address of connected BMC)
* Description
  - If it generates the event related with the link, the relay shall
    handle the event to deliver BTP Message to the BMC.
  - If the link is already registered, or its network is already
    registered then it fails.
  - If there is no verifier related with the network of the link,
    then it fails.
  - It initializes status information for the link.
  - It's called by the operator to manage the BTP network.
    
###### removeLink
```python
@external
def removeLink(self, link: str):
```
* Params
  - link: String (BTP Address of connected BMC)
* Description
  - It removes the link and status information.
  - It's called by the operator to manage the BTP network.
  
###### addRoute
```python
@external
def addRoute(self, dst: str, link: str):
```
* Params
  - dst: String ( BTP Address of the destination BMC )
  - link: String ( BTP Address of the next BMC for the destination )
* Description:
  - Add route to the BMC.
  - It may fail if there more than one BMC for the network.
  - It's called by the operator to manage the BTP network.
  
###### removeRoute
```python
@external
def removeRoute(self, dst: str):
```
* Params
  - dst: String ( BTP Address of the destination BMC )
* Description:
  - Remove route to the BMC.
  - It's called by the operator to manage the BTP network.
  
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
    related with the service as value.
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
        "0x1.iconee": "cx72eaed466599ca5ea377637c6fa2c5c0978537da"
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
  [ "btp://0x1.iconee/cx9f8a75111fd611710702e76440ba9adaffef8656" ]
  ```
  
###### getRoutes
```python
@external(readonly=True)
def getRoutes(self) -> dict:
```
* Description:
  - Get routing information.
* Return
  - A dictionary with the BTP Address of the destination BMC as key and
    the BTP Address of the next as value.
    ```json
    {
      "btp://0x2.iconee/cx1d6e4decae8160386f4ecbfc7e97a1bc5f74d35b": "btp://0x1.iconee/cx9f8a75111fd611710702e76440ba9adaffef8656"
    }
    ```
    
###### getStatus
```python
@external(readonly=True)
def getStatus(self, link: str) -> dict:
```
* Params
  - link: String ( BTP Address of the connected BMC )
* Description:
  - Get status of BMC.
  - It's used by the relay to resolve next BTP Message to send.
  - If target is not registered, it will fail.
* Return
  - The object contains followings fields.
  
    | Field    | Type    | Description                                      |
    |:---------|:--------|:-------------------------------------------------|
    | tx_seq   | Integer | next sequence number of the next sending message |
    | rx_seq   | Integer | next sequence number of the message to receive   |
    | verifier | Object  | status information of the BMV                    |
  
  
##### Events

###### Message
```python
@eventlog(indexed=1)
def Message(self, next: str, seq: int, msg: bytes):
```
* Indexed: 1
* Params
  - next: String ( BTP Address of the BMC to handle the message )
  - seq: Integer ( sequence number of the message from current BMC to the next )
  - msg: Bytes ( serialized bytes of BTP Message )
* Description
  - It sends the message to the next BMC.
  - The relay monitors this event.

### BTP Message Verifier

#### Introduction

BTP Message Verifier verify and decode Relay Message to
[BTP Message](#btp-message)s.
Relay Message is composed of both BTP Messages and with proof of
existence of BTP Messages.

For easy verification, it may update trust information for the
followed events. Most of the implementations may track the hashes of block
headers.
 
If the blockchain system provides proof of the absence of the
BTP Messages, then it's enough that the verifier sustains only
the last state.
It updates the hash only if it sees proof of the absence of further
BTP Messages in the block.

But most blockchain system doesn't provide the proof of absence of data.
So, they need to provide the method to verify any of old hashes.

Merkle Accumulator can be used for verifying old hashes.
BMV sustains roots of Merkle Tree Accumulator, and the relay
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
def handleRelayMessage(self, _bmc: str, _from: str, _seq: int, _msg: bytes) -> list:
```
* Description
  - Decodes Relay Message and process BTP Messages
  - If there is an error, then it sends a BTP Message containing
    Error Message
  - It ignores BTP Messages with old sequence numbers. But if it
    sees a BTP Message with future sequence number, it should fail.
* Params
  - bmc: String ( BTPAddress of the BMC handles the message )
  - from: String ( BTPAddress of the BMC generates the message )
  - seq: Integer ( next sequence number to get a message )
  - msg: Bytes ( serialized bytes of Relay Message )
* Returns
  - List of serialized bytes of a [BTP Message](#btp-message)

### BTP Service Handler

#### Introduction

It may send messages through BTP Message Center(BMC) from any user
request. Of course, the request may come from other smart contracts.
It also have responsibility to handle the message from other BSHs.

BSH can communicate other BSHs with same service name.
If there is already the service using same name, then it should choose
other name for the service when it registers a new service.
Of course, if it wants to be a part of the service, then it should
use same name. And also it follows the protocol of the service.

Before it's registered to the BMC, it can't send an message, and also
it won't handle the message from others.
To be BSH, followings are required.

1. Implements the interface
2. Registered to the BMC through [BMC.addService](#addservice)

After the registration, it may send messages through
[BMC.sendMessage](#sendmessage).
If there is an error while it delivers the message, then it will
return error information though [handleBTPError](#handlebtperror).
If it's successfully delivered, then BMC will call
[handleBTPMessage](#handlebtpmessage) of the target BSH.
While it processes the message, it may reply though
[BMC.sendMessage](#sendmessage).

#### Security

It should not handle messages or errors from other contract
except the BMC.
BMC also accepts only the service messages from registered BSH.
Of course, BSH may have other APIs, but APIs related with BMC are
only called by BMC.

#### Interface

##### Writable methods

###### handleBTPMessage
```python
@external
def handleBTPMessage(self, _from: str, _svc: str, _sn: int, _msg: bytes):
```
* Description
  - Handle BTP Message from other blockchain.
  - Accept the message only from the BMC.
  - If it fails, then BMC will generate BTP Message including
    error information, then it would be delivered to the source.
* Params
  - from: String ( address of source network )
  - svc: String ( name of the service )
  - sn: Integer ( serial number of the message )
  - msg: Bytes ( serialized bytes of ServiceMessage )

###### handleBTPError
```python
@external
def handleBTPError(self, _from: str, _svc: str, _sn: int, _code: int, _msg: str):
```
* Description
  - Handle the error on delivering the message.
  - Accept the error only from the BMC.
* Params
  - from: String ( BTP Address of BMC generates the error )
  - svc: String ( name of the service )
  - sn: Integer ( serial number of the original message )
  - code: Integer ( code of the error )
  - msg: String ( message of the error )


### Message delivery flow

![Components](../assets/iip-draft_btp/btp_components.svg)

1. BSH sends an Service Message through BMC.

   * BSH calls [BMC.sendMessage](#sendmessage) with followings.

     | Name    | Type    | Description                                   |
     |:--------|:--------|:----------------------------------------------|
     | dst_net | String  | Network Address of the destination blockchain |
     | svc     | String  | Name of the service.                          |
     | sn      | Integer | Serial number of the message.                 |
     | msg     | Bytes   | Service message to be delivered.              |

   * BMC lookup the destination BMC belonging to *dst_net*.
     If there is no known BMC to the network, then it fails.

   * BMC builds an BTP Message.

     | Name | Type    | Description                                   |
     |:-----|:--------|:----------------------------------------------|
     | src  | String  | BTP Address of current BMC                    |
     | dst  | String  | BTP Address of destination BMC in the network |
     | svc  | String  | Given service name                            |
     | sn   | Integer | Given serial number                           |
     | msg  | Bytes   | Given service message                         |

   * BMC decide the next BMC according to the destination.
     If there is no route to the destination BMC.

   * BMC generates an event with BTP Message.
   
     | Name | Type    | Description                                |
     |:-----|:--------|:-------------------------------------------|
     | next | String  | BTP Address of the next BMC                |
     | seq  | Integer | Sequence number of the msg to the next BMC |
     | msg  | Bytes   | Serialized BTP Message                     |

2. The BTP Message Relay(BMR) detects events.
   * The relay detects [BMC.Message](#message) through various ways.
   * The relay can confirm that it occurs and it's finalized.
   
3. BMR gathers proofs
   * Relay gathers proofs of the event(POE)s
     - Proof for the new block
     - Proof for the event in the block
   * Relay builds Relay Message including followings.
     - Proof of the new events
     - New events including the BTP Message.
   * Relay calls [BMC.handleRelayMessage](#handlerelaymessage)
     with built Relay Message.
     
   | Name | Type   | Description                                     |
   |:-----|:-------|:------------------------------------------------|
   | prev | String | BTP Address of the previous BMC                 |
   | msg  | Bytes  | serialized Relay Message including BTP Messages |
     
4. BMC handles Relay Message

   * It finds BMV for the network address of the previous BMC.
   * It gets the sequence number of the next message from the source network.
   * BMC calls [BMV.handleRelayMessage](#bmvhandlerelaymessage)
     to decode Relay Message and get a list of BTP Messages.
     
     | Name | Type    | Description                                               |
     |:-----|:--------|:----------------------------------------------------------|
     | bmc  | String  | BTP Address of current BMC                                |
     | prev | String  | BTP Address of given previous BMC                         |
     | seq  | Integer | Next sequence number of the BTP Message from previous BMC |
     | msg  | Bytes   | The Relay Message                                         |

5. BMV decodes Relay Message

   * It verifies and decodes Relay Message, then returns a list of
     BTP Messages.
   * If it fails to verify the message, then it fails.
   * The events from the previous BMC to the current BMC will be processed.
   * The events should have proper sequence number, otherwise it fails.

6. BSH handles Service Messages

   * BMC dispatches BTP Messages.
   * If the destination BMC isn't current one, then it locates
     the next BMC and generates the event.
   * If the destination BMC is current one, then it locates BSH
     for the service of the BTP Message.
   * It calls [BSH.handleBTPMessage](#handlebtpmessage) if
     the message have positive value as *sn*.

     | Name | Type    | Description                           |
     |:-----|:--------|:--------------------------------------|
     | from | String  | Network Address of the source network |
     | svc  | String  | Given service name                    |
     | sn   | Integer | Given serial number                   |
     | msg  | Bytes   | Given service message                 |
     
   * Otherwise, it calls [BSH.handleBTPError](#handlebtperror).
   
     | Name | Type    | Description                                    |
     |:-----|:--------|:-----------------------------------------------|
     | from | String  | BTP Address of the BMC who generates the error |
     | svc  | String  | Given service name                             |
     | sn   | Integer | Given serial number                            |
     | code | Integer | Given error code                               |
     | msg  | String  | Given error message                            |


## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->


## Implementation
<!--The implementations must be completed before any IIP is given status "Final", but it need not be completed before the IIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
