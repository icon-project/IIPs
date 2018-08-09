---
iip: 2
title: ICON Token Standard
author: Jaechang Namgoong (@sink772)
discussions-to: https://github.com/icon-project/IIPs/issues/2
status: Draft
type: Standards Track
category: IRC
created: 2018-08-07
---

## Simple Summary
A standard interface for tokens on ICON network.

## Abstract
This draft IRC describes a token standard interface to provide basic functionality to transfer tokens.
We adopted the token fallback mechanism inspired by ERC223, that a token contract can implement to prevent accidental transfers of tokens to contracts and make token transactions behave like other ICX transactions.

## Motivation
A token standard interface allows any tokens on ICON to be re-used by other third parties, from wallets to decentralized exchanges.

## Specification

### Methods

#### name
Returns the name of the token. e.g. `MySampleToken`.
```python
@external(readonly=True)
def name(self) -> str:
```

#### symbol
Returns the symbol of the token. e.g. `MST`.
```python
@external(readonly=True)
def symbol(self) -> str:
```

#### decimals
Returns the number of decimals the token uses. e.g. `18`.
```python
@external(readonly=True)
def decimals(self) -> int:
```

#### totalSupply
Returns the total token supply.
```python
@external(readonly=True)
def totalSupply(self) -> int:
```

#### balanceOf
Returns the account balance of another account with address `_owner`.
```python
@external(readonly=True)
def balanceOf(self, _owner: Address) -> int:
```

#### transfer
Transfers `_value` amount of tokens to address `_to`, and MUST fire the `Transfer` event. This function SHOULD throw if the `self.msg.sender` account balance does not have enough tokens to spend. If `_to` is a contract, this function MUST invoke the function `tokenFallback(Address, int, bytes)` in `_to`. If the `tokenFallback` function is not implemented in `_to` (receiver contract), then the transaction must fail and the transfer of tokens should not occur. If `_to` is an externally owned address, then the transaction must be sent without trying to execute `tokenFallback` in `_to`.  `_data` can be attached to this token transaction. `_data` can be empty.
```python
@external
def transfer(self, _to: Address, _value: int, _data: bytes=None):
```

### Eventlogs

#### Transfer
Must trigger on any successful token transfers.
```python
@eventlog(indexed=3)
def Transfer(self, _from: Address, _to: Address, _value: int, _data: bytes):
    pass
```

### Token Fallback

A function for handling token transfers, which is called from the token contract, when a token holder sends tokens. `_from` is the address of the sender of the token, `_value` is the amount of incoming tokens, and `_data` is arbitrary attached data. It works by analogy with the fallback function of the normal transactions and returns nothing.
```python
@external
def tokenFallback(self, _from: Address, _value: int, _data: bytes):
```

## Implementation
TBD

## References
* [https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)
* [https://github.com/ethereum/EIPs/issues/223](https://github.com/ethereum/EIPs/issues/223)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
