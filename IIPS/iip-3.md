---
iip: 3
title: ICON Non-Fungible Token Standard
author: Jaechang Namgoong (@sink772)
discussions-to: https://github.com/icon-project/IIPs/issues/3
status: Last Call
review-period-end: 2020-07-13
type: Standards Track
category: IRC
created: 2018-08-27
---

## Simple Summary
A standard interface for non-fungible tokens on ICON network.

## Abstract
This draft IRC describes non-fungible tokens (NFTs) standard interface to provide basic functionality to track and transfer NFTs.
NFTs can represent ownership over digital or physical assets, which can be owned and transferred by individuals as well as consignment to third party operators.

## Motivation
A standard interface allows wallet/broker/auction applications to work with any NFT on ICON network.

This standard is inspired by the ERC721 token standard. Unlike [IRC2](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md) token standard, where we adopted the token fallback mechanism, we preserved the approve/transferFrom methods in this NFTs standard, because there can not be multiple withdrawals with NFTs as in ERC20 (see [ERC20 API: An Attack Vector on Approve/TransferFrom Methods](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit)).

## Specification

### Methods

#### name
Returns the name of the token. e.g. `CryptoBears`.
```python
@external(readonly=True)
def name(self) -> str:
```

#### symbol
Returns the symbol of the token. e.g. `CBT`.
```python
@external(readonly=True)
def symbol(self) -> str:
```

#### balanceOf
Returns the number of NFTs owned by `_owner`. NFTs assigned to the zero address are considered invalid, so this function SHOULD throw for queries about the zero address.
```python
@external(readonly=True)
def balanceOf(self, _owner: Address) -> int:
```

#### ownerOf
Returns the owner of an NFT.  Throws if `_tokenId` is not a valid NFT.
```python
@external(readonly=True)
def ownerOf(self, _tokenId: int) -> Address:
```

#### getApproved
Returns the approved address for a single NFT. If there is none, returns the zero address. Throws if `_tokenId` is not a valid NFT.
```python
@external(readonly=True)
def getApproved(self, _tokenId: int) -> Address:
```

#### approve
Allows `_to` to change the ownership of `_tokenId` from your account. The zero address indicates there is no approved address.  Throws unless `self.msg.sender` is the current NFT owner.
```python
@external
def approve(self, _to: Address, _tokenId: int):
```

#### transfer
Transfers the ownership of your NFT to another address, and MUST fire the `Transfer` event. Throws unless `self.msg.sender` is the current owner.  Throws if `_to` is the zero address.  Throws if `_tokenId` is not a valid NFT.
```python
@external
def transfer(self, _to: Address, _tokenId: int):
```

#### transferFrom
Transfers the ownership of an NFT from one address to another address, and MUST fire the `Transfer` event.  Throws unless `self.msg.sender` is the current owner or the approved address for the NFT.  Throws if `_from` is not the current owner. Throws if `_to` is the zero address. Throws if `_tokenId` is not a valid NFT.
```python
@external
def transferFrom(self, _from: Address, _to: Address, _tokenId: int):
```

### Eventlogs

#### Transfer
Must trigger on any successful token transfers.
```python
@eventlog(indexed=3)
def Transfer(self, _from: Address, _to: Address, _tokenId: int):
    pass
```

#### Approval
Must trigger on any successful call to `approve(Address,int)`.
```python
@eventlog(indexed=3)
def Approval(self, _owner: Address, _approved: Address, _tokenId: int):
    pass
```

## Implementation
* [ICON IRC3 NFT Token Standard RI](https://github.com/icon2infiniti/Samples/tree/master/IRC3)

## References
* [https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
