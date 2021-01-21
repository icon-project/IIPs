---
iip: 31
title: ICON Multi Token Standard
author: Jaechang Namgoong (@sink772)
discussions-to: https://github.com/icon-project/IIPs/issues/31
status: Draft
type: Standards Track
category: IRC
created: 2020-12-07
---

## Simple Summary
A standard interface for managing multiple token types (fungible or non-fungible) on ICON network.

## Abstract
This document describes an interface that can be used to manage multiple token types in a contract.
While the existing standards only support single type of token per contract, either fungible or non-fungible,
this standard allows for each token ID to represent a new configurable token type,
which may have its own metadata, supply and other attributes.

## Motivation
Existing token standards like IRC-2 and IRC-3 only can handle a single type of token,
thus users need to deploy multiple token contracts separately to manage different types of tokens.
In some applications, notably games that may be creating thousands of traceable items and exchanging them between users,
this limitation is something that is hard to endure and costs much.

Inspired by the [ERC-1155 Multi Token Standard](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md),
we propose new functionality that is capable of transferring multiple token types at once and saving the transaction costs.

## Specification

### Methods

#### balanceOf
```python
@external(readonly=True)
def balanceOf(self, _owner: Address, _id: int) -> int:
    """
    Returns the balance of the owner's tokens.

    :param _owner: the address of the token holder
    :param _id: ID of the token
    :return: the _owner's balance of the token type requested
    """
```

#### balanceOfBatch
```python
@external(readonly=True)
def balanceOfBatch(self, _owners: List[Address], _ids: List[int]) -> List[int]:
    """
    Returns the balance of multiple owner/id pairs.

    :param _owners: the addresses of the token holders
    :param _ids: IDs of the tokens
    :return: the list of balance (i.e. balance for each owner/id pair)
    """
```

#### transferFrom
```python
@external
def transferFrom(self, _from: Address, _to: Address, _id: int, _value: int, _data: bytes = None):
    """
    Transfers `_value` amount of an token `_id` from one address to another address,
    and must emit `TransferSingle` event to reflect the balance change.
    When the transfer is complete, this method must invoke `onIRC31Received(Address,Address,int,int,bytes)` in `_to`,
    if `_to` is a contract. If the `onIRC31Received` method is not implemented in `_to` (receiver contract),
    then the transaction must fail and the transfer of tokens should not occur.
    If `_to` is an externally owned address, then the transaction must be sent without trying to execute
    `onIRC31Received` in `_to`.
    Additional `_data` can be attached to this token transaction, and it should be sent unaltered in call
    to `onIRC31Received` in `_to`. `_data` can be empty.
    Throws unless the caller is the current token holder or the approved address for the token ID.
    Throws if `_from` does not have enough amount to transfer for the token ID.
    Throws if `_to` is the zero address.

    :param _from: source address
    :param _to: target address
    :param _id: ID of the token
    :param _value: the amount of transfer
    :param _data: additional data that should be sent unaltered in call to `_to`
    """
```

#### transferFromBatch
```python
@external
def transferFromBatch(self, _from: Address, _to: Address, _ids: List[int], _values: List[int], _data: bytes = None):
    """
    Transfers `_values` amount(s) of token(s) `_ids` from one address to another address,
    and must emit `TransferSingle` or `TransferBatch` event(s) to reflect all the balance changes.
    When all the transfers are complete, this method must invoke `onIRC31Received(Address,Address,int,int,bytes)` or
    `onIRC31BatchReceived(Address,Address,int[],int[],bytes)` in `_to`,
    if `_to` is a contract. If the `onIRC31Received` method is not implemented in `_to` (receiver contract),
    then the transaction must fail and the transfers of tokens should not occur.
    If `_to` is an externally owned address, then the transaction must be sent without trying to execute
    `onIRC31Received` in `_to`.
    Additional `_data` can be attached to this token transaction, and it should be sent unaltered in call
    to `onIRC31Received` in `_to`. `_data` can be empty.
    Throws unless the caller is the current token holder or the approved address for the token IDs.
    Throws if length of `_ids` is not the same as length of `_values`.
    Throws if `_from` does not have enough amount to transfer for any of the token IDs.
    Throws if `_to` is the zero address.

    :param _from: source address
    :param _to: target address
    :param _ids: IDs of the tokens (order and length must match `_values` list)
    :param _values: transfer amounts per token (order and length must match `_ids` list)
    :param _data: additional data that should be sent unaltered in call to `_to`
    """
```

#### setApprovalForAll
```python
@external
def setApprovalForAll(self, _operator: Address, _approved: bool):
    """
    Enables or disables approval for a third party ("operator") to manage all of the caller's tokens,
    and must emit `ApprovalForAll` event on success.

    :param _operator: address to add to the set of authorized operators
    :param _approved: true if the operator is approved, false to revoke approval
    """
```

#### isApprovedForAll
```python
@external(readonly=True)
def isApprovedForAll(self, _owner: Address, _operator: Address) -> bool:
    """
    Returns the approval status of an operator for a given owner.

    :param _owner: the owner of the tokens
    :param _operator: the address of authorized operator
    :return: true if the operator is approved, false otherwise
    """
```

### Events

#### TransferSingle
```python
@eventlog(indexed=3)
def TransferSingle(self, _operator: Address, _from: Address, _to: Address, _id: int, _value: int):
    """
    Must trigger on any successful token transfers, including zero value transfers as well as minting or burning.
    When minting/creating tokens, the `_from` must be set to zero address.
    When burning/destroying tokens, the `_to` must be set to zero address.

    :param _operator: the address of an account/contract that is approved to make the transfer
    :param _from: the address of the token holder whose balance is decreased
    :param _to: the address of the recipient whose balance is increased
    :param _id: ID of the token
    :param _value: the amount of transfer
    """
```

#### TransferBatch
```python
@eventlog(indexed=3)
def TransferBatch(self, _operator: Address, _from: Address, _to: Address, _ids: bytes, _values: bytes):
    """
    Must trigger on any successful token transfers, including zero value transfers as well as minting or burning.
    When minting/creating tokens, the `_from` must be set to zero address.
    When burning/destroying tokens, the `_to` must be set to zero address.

    :param _operator: the address of an account/contract that is approved to make the transfer
    :param _from: the address of the token holder whose balance is decreased
    :param _to: the address of the recipient whose balance is increased
    :param _ids: serialized bytes of list for token IDs (order and length must match `_values`)
    :param _values: serialized bytes of list for transfer amounts per token (order and length must match `_ids`)

    NOTE: RLP (Recursive Length Prefix) would be used for the serialized bytes to represent list type.
    """
```

#### ApprovalForAll
```python
@eventlog(indexed=2)
def ApprovalForAll(self, _owner: Address, _operator: Address, _approved: bool):
    """
    Must trigger on any successful approval (either enabled or disabled) for a third party/operator address
    to manage all tokens for the `_owner` address.

    :param _owner: the address of the token holder
    :param _operator: the address of authorized operator
    :param _approved: true if the operator is approved, false to revoke approval
    """
```

#### URI
```python
@eventlog(indexed=1)
def URI(self, _id: int, _value: str):
    """
    Must trigger on any successful URI updates for a token ID.
    URIs are defined in RFC 3986.
    The URI must point to a JSON file that conforms to the "ERC-1155 Metadata URI JSON Schema".

    :param _id: ID of the token
    :param _value: the updated URI string
    """
```

### Token Receivers

Smart contracts that want to receive tokens from IRC31-compatible token contracts must implement
all of the following receiver methods to accept transfers.

#### onIRC31Received
```python
@external
def onIRC31Received(self, _operator: Address, _from: Address, _id: int, _value: int, _data: bytes):
    """
    A method for handling a single token type transfer, which is called from the multi token contract.
    It works by analogy with the fallback method of the normal transactions and returns nothing.
    Throws if it rejects the transfer.

    :param _operator: The address which initiated the transfer
    :param _from: the address which previously owned the token
    :param _id: the ID of the token being transferred
    :param _value: the amount of tokens being transferred
    :param _data: additional data with no specified format
    """
```

#### onIRC31BatchReceived
```python
@external
def onIRC31BatchReceived(self, _operator: Address, _from: Address, _ids: List[int], _values: List[int], _data: bytes):
    """
    A method for handling multiple token type transfers, which is called from the multi token contract.
    It works by analogy with the fallback method of the normal transactions and returns nothing.
    Throws if it rejects the transfer.

    :param _operator: The address which initiated the transfer
    :param _from: the address which previously owned the token
    :param _ids: the list of IDs of each token being transferred (order and length must match `_values` list)
    :param _values: the list of amounts of each token being transferred (order and length must match `_ids` list)
    :param _data: additional data with no specified format
    """
```

### Metadata Extensions

To provide rich set of asset metadata for a given token, smarts contracts need to implement the following method
to return an URI that points to a JSON file.

```python
@external(readonly=True)
def tokenURI(self, _id: int) -> str:
    """
    Returns an URI for a given token ID.

    :param _id: ID of the token
    :return: the URI string
    """
```

## Implementation
TBD

## References
* [IRC-2 Token Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md)
* [IRC-3 Non-Fungible Token Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-3.md)
* [ERC-1155 Multi Token Standard](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
