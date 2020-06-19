---
iip: 16
title: ICON Security Token Standard
author: Patrick Park (@cordiallys)
discussions-to: https://github.com/icon-project/IIPs/issues/16
status: Draft
type: Standards Track
category: IRC
created: 2019-01-21
---

## Simple Summary

This document represents the standard protocol specification for the security token of ICON network.

## Abstract

The security token standard describes how to represent full ownership or split ownership of a particular asset.
This standard is compatible with standard IRC2 token [IIP-2](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md) and inspired by several security token standards such as [EIP-1400](https://github.com/SecurityTokenStandard/EIP-Spec/blob/master/eip/eip-1400.md), and [EIP-1410](https://github.com/SecurityTokenStandard/EIP-Spec/blob/master/eip/eip-1410.md).
Methods for Partially Fungible Tokens (PFT) was additionally written for grouping tokens into partition sets, and each partition is represented by a key-value format consisting of a unique key and balance.
Through this standard, ownership of certain assets can be divided and represented, tracked, viewed, privately owned and transferred. It may also be entrusted to a third party provider and may be controlled under strict authorization.

## Motivation

The objective is to securitize and liquidate assets on the ICON blockchain, by issuing and managing security tokens.
The standard supports legal document asserting the rights of tokenized assets on a blockchain, partition management with partially fungible tokens in tranche, and interfaces for managing operator privileges.

## Rationale

#### Partition

A Partially-Fungible Token allows attachment of metadata for a partial balance of a token holder. These partial balances are called partitions, which have an unique name and aa partial balance of the token.

#### Document Management

Since security tokens entail rights and oblications either from investor or the issuer, the ability to connect legal documents with the relevant contracts or partitions is importannt.

#### Check for Security Token Transfer

The transfer of security token may fail for a variety of reasons. The reasons can be application specific, including failures such as KYC/AML invalidity, maximum number of investors reached, or partition state being non-transferable. An interface is needed to check whether the security tokens are transferable before the transfer and to return the reason for failures.

#### Token Control by Operator

Security tokens may transfer or retrieve tokens and partitions from a regulatory authority or operator with regulatory consent even if they are not holders of the token. For example, if an investor has falsified the KYC/AML, an holder lost a key, or a fraudulent transaction that needs to be restored, operator permission to control the tokens or partitions of other holders is required.

## Specification

### Security Token Interface

```python
class TokenStandard(ABC):
    # ======================================================================
    # IRC 2
    # ======================================================================
    @abstractmethod
    def name(self) -> str:
        pass

    @abstractmethod
    def symbol(self) -> str:
        pass

    @abstractmethod
    def decimals(self) -> int:
        pass

    @abstractmethod
    def totalSupply(self) -> int:
        pass

    @abstractmethod
    def balanceOf(self, _owner: Address) -> int:
        pass

    # ======================================================================
    # Token Information
    # ======================================================================
    @abstractmethod
    def balanceOfByPartition(self, _partition: str, _owner: Address) -> int:
        pass

    @abstractmethod
    def partitionsOf(self, _owner: Address) -> dict:
        pass

    # ======================================================================
    # Document Management
    # ======================================================================
    @abstractmethod
    def getDocument(self, _name: str) -> dict:
        pass

    @abstractmethod
    def setDocument(self, _name: str, _uri: str, _document_hash: str) -> None:
        pass

    # ======================================================================
    # Partition Token Transfer
    # ======================================================================
    @abstractmethod
    def transferByPartition(self, _partition: str, _to: Address, _amount: int, _data: bytes = None) -> None:
        pass

    @abstractmethod
    def operatorTransferByPartition(self, _partition: str, _from: Address, _to: Address, _amount: int, _data: bytes = None) -> None:
        pass

    # ======================================================================
    # Operator Management
    # ======================================================================
    @abstractmethod
    def authorizeOperator(self, _operator: Address) -> None:
        pass

    @abstractmethod
    def revokeOperator(self, _operator: Address) -> None:
        pass

    @abstractmethod
    def authorizeOperatorForPartition(self, _partition: str, _operator: Address) -> None:
        pass

    @abstractmethod
    def revokeOperatorForPartition(self, _partition: str, _operator: Address) -> None:
        pass

    # ======================================================================
    # Operator Information
    # ======================================================================
    @abstractmethod
    def isOperator(self, _operator: Address, _owner: Address) -> bool:
        pass

    @abstractmethod
    def isOperatorForPartition(self, _partition: str, _operator: Address, _owner: Address) -> bool:
        pass

    # ======================================================================
    # Token Issuance
    # ======================================================================
    @abstractmethod
    def issueByPartition(self, _partition: str, _to: Address, _amount: int, _data: bytes) -> None:
        pass

    # ======================================================================
    # Token Redemption
    # ======================================================================
    @abstractmethod
    def redeemByPartition(self, _partition: str, _from: Address, _amount: int, _data: bytes) -> None:
        pass

    # ======================================================================
    # Transfer Validity
    # ======================================================================
    @abstractmethod
    def canTransferByPartition(self, _partition: str, _to: Address, _amount: int, _data: bytes = None) -> str:
        pass
```

### Methods

#### name

Returns the name of the token.

```python
@external(readonly=True)
def name(self) -> str:
```

#### symbol

Returns the symbol of the token.

```python
@external(readonly=True)
def symbol(self) -> str:
```

#### decimals

Returns the number of decimals to the token uses

```python
@external(readonly=True)
def decimals(self) -> int:
```

#### totalSupply

Returns the total token supply

```python
@external(readonly=True)
def totalSupply(self) -> int:
```

#### balanceOf

Retruns the account balance of all partitions of the account address `_owner`.

```python
@external(readonly=True)
def balanceOf(self, _owner: Address) -> int:
```

#### balanceOfPartition

Returns the balance of the partition `_partition` of the account address `_owner`.

```python
@external(readonly=True)
def balanceOfByPartition(self, _partition: str, _owner: Address) -> int:
```

#### partitionsOf

Returns the partition information of the account address `_owner`. The information of partition consists of the name for the partition and the associated token amount.

```python
@external(readonly=True)
def partitionsOf(self, _owner: Address) -> dict:
```

#### getDocument

Returns information of the document `_name`.

```python
@external(readonly=True)
def getDocument(self, _name: str) -> dict:
```

#### setDocument

Sets information of document by the unique `_name`. Raw documentation is organized offchain. `_uri` and `_document_hash` are stored on the blockchain to prevent forgery.  
Examples of documents may include legal rights, transfer provisions, sales term, etc.

```python
@external
def setDocument(self, _name: str, _uri: str, _document_hash: str) -> None:
```

#### transferByPartition

Transfers `_partition` to a specific account with `_to`. You should call `self.msg.sender` to initiate transfer when the `_amount` of the `_partition` is sufficient. The `@eventlog` `TransferByPartition` should be called on `transferByPartition` calls.

```python
@external
def transferByPartition(self, _partition: str, _to: Address, _amount: int, _data: bytes = None) -> None:
```

#### operatorTransferByPartition

Transfers `_partition` from the account`_from` to the account `_to`. The `@eventlog` `TransferByPartition` should be called on `transferByPartition` calls. Must throw if message sender is not an authorized operator.

```python
@external
def operatorTransferByPartition(self, _partition: str, _from: Address, _to: Address, _amount: int, _data: bytes = None) -> None:
```

#### authorizeOperator

Grants `_operator` control to the issued token. The `@eventlog AuthorizeOperator` must be triggered when granting access control.

```python
def authorizeOperator(self, _operator: Address) -> None:
```

#### revokeOperator

Revokes the control granted to the `_operator`. The `@eventlog RevokeOperator` must be triggered when revoking access control.

```python
def revokeOperator(self, _operator: Address) -> None:
```

#### authorizeOperatorForPartition

Grants `_operator` the control to the partition `_partition` of the account `_owner`. The `@eventlog AuthorizeOperatorForPartition` must be triggered when granting control.

```python
def authorizeOperatorForPartition(self, _partition: str, _operator: Address) -> None:
```

#### revokeOperatorForPartition

Revokes the control of `_partition` from `_owner` granted to `_operator`. The `@eventlog RevokeOperatorForPartition` must be triggered when revoking access control.

```python
def revokeOperatorForPartition(self, _partition: str, _operator: Address) -> None:
```

#### isOperator

Returns whether `_operator` has access control to token.

```python
@external(readonly=True)
def isOperator(self, _operator: Address, _owner: Address) -> bool:
```

#### isOperatorForPartition

Returns whether `_operator` has access control to `_partition` of `\_owner.

```python
@external(readonly=True)
 def isOperatorForPartition(self, _partition: str, _operator: Address, _owner: Address) -> bool:
```

#### issueByPartition

Issues tokens of `_partition` to the account `_to` in the amount of `_amount`. Specific metadata can be attached in `_data`. The `_partition` and balance quantity of the account `_to` must be changed together. The `issueByPartition` method must be controlled by the contract owner.

The `@eventlog IssueByPartition` for this method should be triggered when calling the `IssueByPartition` method.

```python
 def issueByPartition(self, _partition: str, _to: Address, _amount: int, _data: bytes) -> None:
```

#### redeemByPartition

Redeems the tokens from `_partition` of account `_from` by `_amount`. At the time of redemption, `_from`’s total balance should also be changed. The `redeem` method must be controlled and by the contract owner. When invoking the `redeemByPartition` method, `@eventlog RedeemByPartition` for this method should be triggered.

```python
def redeemByPartition(self, _partition: str, _from: Address, _amount: int, _data: bytes) -> None:
```

#### canTransferByPartition

The standard provides an on-chain function to determine whether a transfer will succeed, and return details indicating the reason if the transfer is not valid.

```python
@external
def canTransferByPartition(self, _partition: str, _to: Address, _amount: int, _data: bytes = None) -> str:
```

### Eventlogs

#### TransferByPartition

```python
@eventlog(indexed=2)
def TransferByPartition(self, _partition: str, _operator: Address, _from: Address, _to: Address, _amount: int, _data: bytes):
    pass
```

#### IssueByPartition

```python
@eventlog(indexed=3)
def IssueByPartition(self, _partition: str, _to: Address, _amount: int, _data: bytes):
    pass
```

#### RedeemByPartition

```python
@eventlog(indexed=3)
def RedeemByPartition(self, _partition: str, _operator: Address, _owner: Address, _amount: int, _data: bytes):
    pass
```

#### AuthorizeOperator

```python
@eventlog(indexed=2)
def AuthorizeOperator(self, _operator: Address, _sender: Address):
    pass
```

#### RevokeOperator

```python
@eventlog(indexed=2)
def RevokeOperator(self, _operator: Address, _sender: Address):
    pass
```

#### AuthorizeOperatorForPartition

```python
@eventlog(indexed=3)
def AuthorizeOperatorForPartition(self, _owner: Address, _partition: str, _operator: Address):
    pass
```

#### RevokeOperatorForPartition

```python
@eventlog(indexed=3)
def RevokeOperatorForPartition(self, _owner: Address, _partition: str, _operator: Address):
    pass
```

#### SetDocument

```python
@eventlog(indexed=3)
def SetDocument(self, _name: str, _uri: str, _document_hash: str):
    pass
```

####

## Implementaion

- [ICON IRC16 Security Token Standard RI](https://github.com/icon2infiniti/Samples/blob/master/IRC16/)

## References

- [https://github.com/ethereum/EIPs/issues/1411](https://github.com/ethereum/EIPs/issues/1411)

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
