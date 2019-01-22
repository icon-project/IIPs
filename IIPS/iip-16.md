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
This standard is compatible with standard IRC2 token [IIP-2](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md) and inspired by several security token standards such as [EIP-1400](https://github.com/SecurityTokenStandard/EIP-Spec/blob/master/eip/eip-1400.md), [EIP-1410](https://github.com/SecurityTokenStandard/EIP-Spec/blob/master/eip/eip-1410.md). 
A standard for the Partially Fungible Token (PFT) was additionally written for grouping tokens into partition sets, and each partition is represented by a key-value format consisting of a unique key and balance.
Through this standard, ownership of certain assets can be divided and represented, tracked, viewed, privately owned and transferred. It may also be entrusted to a third party provider and may be controlled under strict authorization.

## Motivation

The objective is to securitize and liquidate assets on the ICON block chain, by issuing and managing security tokens.
The standard supports legal document asserting the rights of tokenized assets on a block chain, partition management with partially fungible tokens in tranche, and interfaces for managing operator privileges.

## Specification

### 1. Partially Fungible Token Interface

```python
class PartiallyFungibleTokenStandard(ABC):

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

    @abstractmethod
    def balanceOfPartition(self, _partition: str, _owner: Address) -> int:
        pass

    @abstractmethod
    def partitionsOf(self, _owner: Address) -> dict:
        pass

    @abstractmethod
    def transfer(self, _partition: str, _to: Address, _value: int, _data: bytes) -> None:
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

Retruns the account balance of an account with address `_owner`.

```python
@external(readonly=True)
def balanceOf(self, _owner: Address) -> int:
```

#### balanceOfPartition

Returns the balance of the partition with `_partition`. 

```python
@external(readonly=True)
def balanceOfPartition(self, _partition: str, _owner: Address) -> int:
```

#### partitionsOf

Returns the partition information of an account with address `_owner`. The information of partition consists of a unique key with `_name` and token amount in partition with `_amount`   

```python
@external(readonly=True)
def partitionsOf(self, _owner: Address) -> dict:
```

#### transfer

Transfers or splits `_partition` to a specific account with `_to`. You should call `self.msg.sender` when the `_amount` of the `_partition` is sufficient. The `@eventlog` should be called on `transfer` call. ( See implementation as below). The transfer method can be managed by the operator specified in the *Security Token Standard* below.

```python
@external
def transfer(self, _partition: str, _to: Address, _amount: int, _data: bytes = None) -> None:
```


------


### 2. Security Token Standard Interface

```python
class SecurityTokenStandard(PartiallyFungibleTokenStandard):

    @abstractmethod
    def setDocument(self, _name: str, _uri: str, _document_hash: str) -> None:
        pass

    @abstractmethod
    def getDocument(self, _name: str) -> dict:
        pass

    @abstractmethod
    def isControllable(self) -> bool:
        pass

    @abstractmethod
    def isIssuable(self) -> bool:
        pass

    @abstractmethod
    def issue(self, _partition: str, _owner: Address, _investor: int, _data: bytes) -> None:
        pass

    @abstractmethod
    def redeem(self, _partition: str, _investor: Address, _amount: int, _data: bytes) -> None:
        pass

    @abstractmethod
    def authorizeOperator(self, _operator: Address) -> None:
        pass

    @abstractmethod
    def authorizeOperatorForPartition(self, _owner: Address, _partition: str, _operator: Address) -> None:
        pass

    @abstractmethod
    def revokeOperator(self, _operator: Address) -> None:
        pass

    @abstractmethod
    def revokeOperatorForPartition(self, _owner: Address, _partition: str, _operator: Address) -> None:
        pass

    @abstractmethod
    def isOperator(self, _operator: Address) -> bool:
        pass

    @abstractmethod
    def isOperatorForPartition(self, _owner: Address, _partition: str, _operator: Address) -> bool:
        pass
```



### Methods

#### setDocument

Sets information of document identified by the unique with `_name`. Raw documentation is organized in offchain. `_uri` and `_document_hash` are stored in blockchain to prevent forgery and falsification.

```python
@external
def setDocument(self, _name: str, _uri: str, _document_hash: str) -> None:
```

#### getDocument

Returns information of document with `_name`. 

```python
@external(readonly=True)
def getDocument(self, _name: str) -> dict:
```

#### isControllable

Returns whether token is controllable.

```python
@external(readonly=True)
def isControllable(self) -> bool:
```

#### isIssuable

Returns whether token is issuable.

```python
@external(readonly=True)
def isIssuable(self) -> bool:
```

#### issue

Distributes the issued token to _investors through `_partition` as `_amount`. Specific metadata can be treated with `_data`. `_partition` should be issued as `_amount` within the `totalSupply` quantity, The `_partition` and balance quantity of the `_investor` must be changed together. The `issue` method must be controlled by the Operator as below.
```
if self.isIssuable() and self.isOperator(self.msg.sender):
#issue
```

The `@eventlog` for this method should be triggered when calling the `issue` method.

```python
def issue(self, _partition: str, _investor: Address, _amount: int, _data: bytes) -> None:
```

#### redeem

Redeems the `_partition` of a specific account by `_amount`. At the time of redemption, `_investor`’s __partitions should be reimbursed by `_amount` and `_investor`’s total balance should also be changed. The `redeem` method must be controlled and issued by the operator. When invoking the `redeem` method, `@eventlog` for this method should be triggered.

```python
def redeem(self, _partition: str, _investor: Address, _amount: int, _data: bytes) -> None:
```

#### authorizeOperator

Grants `_operator` access control to issued token. The `_operator` and `self.owner` can control the token. The `@eventlog` must be triggered when granting access control.

```python
def authorizeOperator(self, _operator: Address) -> None:
```

#### authorizeOperatorForPartition

Grants `_operator` the `_partition` access control of `_owner`. The `@eventlog` must be triggered when granting access control.

```python
def authorizeOperatorForPartition(self, _owner: Address, _partition: str, _operator: Address) -> None:
```

#### revokeOperator

Revokes the access granted from `_operator`. Only `self.owner` and `_operator` can call this method. The `@eventlog` must be triggered when revoking access control.

```python
def revokeOperator(self, _operator: Address) -> None:
```

#### revokeOperatorForPartition

Revokes the `_partition` access control of `_owner` from `_operator`. Only `self.owner` and `_operator` can call this method. The `@eventlog` must be triggered when revoking access control.

```python
def revokeOperatorForPartition(self, _owner: Address, _partition: str, _operator: Address) -> None:
```

#### isOperator

Returns whether `_operator` has access control to token.

```python
@external(readonly=True)
def isOperator(self, _operator: Address) -> bool:
```

#### isOperatorForPartition

Returns whether `_operator` has access control to `_partition` of `_owner.

```python
@external(readonly=True)
def isOperatorForPartition(self, _owner: Address, _partition: str, _operator: Address) -> bool:
```

####  

## Implementaion
After completing audit process, it will be released officially.  
https://repo.theloop.co.kr/sto/icon-sto-standard/tree/master/score



## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
