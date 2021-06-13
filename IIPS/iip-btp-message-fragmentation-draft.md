---
iip: TBD
title: BTP message fragmentation 
author: heonseung lee (@leeheonseung)
discussions-to: https://github.com/icon-project/IIPs/issues/40
status: Draft
type: Standards Track
category IRC
created: 2020-06-14
requires IIP-25
---

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the IIP.-->
If the size of the relay message containing the block proof of the SRC chain exceeds the transaction size allowed by the target chain, define and add logic to fragment the BTP message from the BMC to handle this.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
If the size of Relay Message containing Block Proof larger than Transaction size of target chain, a problem occurs. To solve this, add the logic to fragment and assemble the Relay Message.

## Motivation
<!--The motivation is critical for IIPs that want to change the ICON protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the IIP solves. IIP submissions without sufficient motivation may be rejected outright.-->
Block proof of the ICON chain can be verified in the BMV of the chain using only the parts related to BTP.
However, in certain chains, it is sometimes necessary to receive and verify the entire Block Proof.

At this time, the ICON Relay Sender fragments the RelayMessage and delivers it to the ICON BMC, 
and it is necessary to assemble and verify it.

For this reason, there is no issue for ICON Block Proof verification on target chain,
so BTP 1.1 implements it only on the ICON BMC.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current ICON platforms.-->

### Relay

#### Interface

##### segment

```go
func (s *sender) Segment(rm *module.RelayMessage, height int64) ([]*module.Segment, error)
```

- Description:
  - Fragment Relay Message as much as Transaction Limit size.

##### relay

```go
func (s *sender) Relay(segment *module.Segment) (module.GetResultParam, error)
```

- Description:
  - If the RelayMessage size containing Block Proof exceeds the transaction to be transmitted, the RelayMessage is fragmented and delivered according to the size. 
  - If not exceeded, it is transmitted as a normal RelayMessage.
- Definition of index
  - Indicates the sequence of fragment messages.
  - The first index is total message count * -1
  - The last index is 0

##### sendFragment

```go
func (s *sender) sendFragment(rmp *BMCRelayMethodParams, idx int) (module.GetResultParam, error)
```

- Description:
  - It indexes the fragmented message separated through the segment and transmits it to the targetchain.



### BMC

#### Interface

handleFragment

```java
void handleFragment(String _prev, String _msg, int _idx);
```

- Params
  - _prev: String (BTP Address of the previous BMC)
  - _msg: String (Fragmented base64 encoded string of serialized bytes of Relay Message)
  - _idx: Integer (Index of fragment) 
- Description:
  - Concat the BTP messages that come with _idx in the order of _idx.
  - If _idx < 0, it is determined as the first message, and if _idx == 0, it is determined as the last fragmentaion message.
    - Eg)
      - -3, 1, 2, 0
      - -1, 0
  - Instead of verifying each time each fragmentation message is processed, 
    it is assembled and verified when the last fragmentation message arrives.
  - Assemble fragments of the Relay Message and call by BMC::handleRelayMessage.
  - It's allowed to be called by registered Relay.

## Implementation
<!--The implementations must be completed before any IIP is given status "Final", but it need not be completed before the IIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->
TBD

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
