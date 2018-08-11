---
iip: 6
title: ICON Name Service Standard
author: Phyrex Tsai <phyrex@portal.network>, Portal Network Team
discussions-to: https://github.com/icon-project/IIPs/issues/6
status: Draft
type: Standards Track
category : IRC
created: 2018-08-06
---

## Abstract

This draft proposal details the ICON Name Service inspired by the Ethererum Name Service, which originated from the EIP137. This protocol and related interface definition is to provide extendable resoultion of easy-to-remeber, human-recognizable names to services and resources identifers from both centralized and decentralized systems, and only updated the names whenever the underlying resource (public wallet address, smart contract, content-addressed data, etc) changes.

It's common nowadays that using URI with certain convention mapping to the specific resources. With this experience, we utilize the advantage of blockchain to provide blockchain domain names as short, stable, and human-readable identifiers used to specify network resources. In this way, users can enter a memorable string, such as `loopchain.icon`, just like how we normally do on browser, and be directed to the appropriate resource. The mapping between names and resources may change in any scenarios, so a website may change the corresponding IPFS hashes, projects may update their smart contract, a user may change wallets, or a IPFS document may be updated to a new version, without the domain name changing. Further, a domain need not specify a single resource; different record types allow the same domain to reference different resources. For instance, the browser may resolve `loopchain.icon` to the IPFS hash to redirect to IPFS content, even resolve the same address to a smart contract.

## Motivation

Hash is the fundamental of blockchain and used to represent an account of an identity, but the hash address is not easily remember for the normal user, by integrate the ICON Name Service will bring three following advantages:

- Human-readable identity, users can easily catch up with the name like `loopchain.icon` instead of hash address.
- Multiple services under a single name, such as a DApp hosted in IPFS, a contract address, and even a mail server.
- Increaing the security by tamper proof of the accessing smart contract for the mapping address.

These three features will lower the gap of blockchain technologies and improvement the user adoption.

## Specification

The INS system has thsee main parts:

### Registry
INS registry is a single contract that provides a mapping from any registered name to the resolver responsible for it, and permits the owner of a name to set the resolver address, and to create subdomains, potentially with different owners to the parent domain.

### Registrars
INS registrar is a smart contract that owns a Top-level domain, and allocates subdomains of it according to some set of rules defined in the smart contract code. For any available domain name, users will be able to purchase. The rules for the INS registrar smart contract will be interatly updated according to meet the benefit of whole ICON community.

### Resolvers
INS resolvers are responsible for performing resources from a domain name, a resources can be a contract address, wallet address and IPFS hash. Utilizing the resolver smart contract enables users and developers using the decentralized resources easily bring more ideas and values to the ecosystem.

### Reverse Resolvers
INS reverse resolver is a smart contract records that mapping from addresses to names.

### Registry Smart Contract
The INS registry contract exposes the following functions:

Returns the owner of the specified node.
```python
@external(readonly=True)
def owner(node) -> Address: 
```

Returns the resolver for the specified node.
```python
@external(readonly=True)
def resolver(node) -> Address: 
```

Returns the time-to-live (TTL) of the `node`; that is, the maximum duration for which a `node`'s information may be cached. 
```python
@external(readonly=True)
def ttl(node) -> int: 
```

Transfers ownership of a `node` to another registrar. And this function may only be called by the current owner of `node`.
```python
@external
def setOwner(node, owner: Address) : 
```

Create a subdomain and the sets its owner to `owner`
```python
@external
def setSubnodeOwner(node, label, owner: Address) : 
```

Sets the resolver address for `node`. And this function may only be called by the owner of `node`.
```python
@external
def setResolver(node, resolver: Address) : 
```

### Registrar Smart Contract
The INS registrar contract exposes the following functions:

Register a name, or change the `owner` of an existing registration.
```python
@external
def register(node, owner: Address) : 
```

### Resolver Smart Contract
The INS resolver contract exposes the following functions:

Sets the hash address for `node`. And this function may only called by the owner of `node`.
```python
@external
def setAddr(node, addr: Address) : 
```

Returns the hash address for the specified node.
```python
@external(readonly=True)
def addr(node) -> Address : 
```

Sets the multihash associated with an INS node.
```python
@external
def setMultihash(node, hash) : 
```

Returns the multihash associated with an INS node.
```python
@external(readonly=True)
def multihash(node) -> str : 
```

Sets the text data associated with an INS node and key.
```python
@external
def setText(node, key, value) : 
```

Returns the text data associated with an INS node and key.
```python
@external(readonly=True)
def text(node, key) : 
```

### Reverse Resolver Smart Contract
The INS reverse resolver contract exposes the following functions:

Sets the `name` record for the reverse INS record associated with the calling account.
```python
@external
def setName(name) : 
```

Returns the hash for a given address's reverse records.
```python
@external(readonly=True)
def node(addr: Address) -> str : 
```

## Implementation

[Draft Version]  
The ICON Name Service has impelement as below.

### INS Registry Implementation

The INS Registry is a root storage of the ICON Name Service.

```python
class INSRegistry(IconScoreBase):

    __OWNER = 'owner'
    __RESOLVER = 'resolver'
    __TTL = 'ttl'

    @eventlog(indexed=2)
    def Transfer(self, node: bytes, owner: Address) : pass

    @eventlog(indexed=3)
    def NewOwner(self, node: bytes, label: bytes, owner: Address) : pass

    @eventlog(indexed=2)
    def NewResolver(self, node: bytes, resolver: Address) : pass

    @eventlog(indexed=2)
    def NewTTL(self, node: bytes, ttl: int) : pass

    def __init__(self, db: IconScoreDatabase, addr_owner: Address) -> None:
        super().__init__(db, addr_owner) 
        # Use DictDB to store value as map
        self.__owner = DictDB(self.__OWNER, db, value_type=Address)
        self.__resolver = DictDB(self.__RESOLVER, db, value_type=Address)
        self.__ttl = DictDB(self.__TTL, db, value_type=int)

    def on_install(self) -> None:
        super().on_install()
        # initial all the node owner is the contract owner
        self.__owner['0x0'] = self.msg.sender

    def on_update(self) -> None:
        super().on_update()

    @external(readonly=True)
    def owner(node: bytes) -> Address:
        return self.__owner[node]

    @external(readonly=True)
    def resolver(node: bytes) -> Address:
        return self.__resolver[node]

    @external(readonly=True)
    def ttl(node: bytes) -> int:
        return self.__ttl[node]

    @external
    def setOwner(node: bytes, addr: Address) -> None:
        if self.__owner[node] == self.msg.sender:
            self.Transfer(node, addr)
            self.__owner[node] = addr

    @external
    def setSubnodeOwner(node: bytes, label: bytes, addr: Address) -> None:
        if self.__owner[node] == self.msg.sender:
            # TODO check keccak hash
            subnode = concate(node, label)
            self.NewOwner(node, label, addr)
            self.__owner[subnode] = addr

    @external
    def setResolver(node: bytes, resolver: Address) -> None:
        if self.__owner[node] == self.msg.sender:
            self.NewResolver(node, addr)
            self.__resolver[node] = resolver

    @external
    def setTTL(node: bytes, ttl: int) -> None:
        if self.__owner[node] == self.msg.sender:
            self.NewTTL(node, ttl)
            self.__ttl[node] = ttl

    # Concate node and label
    def concate(node: bytes, label: bytes) -> bytes: 
        keccak256 = keccak.new(digest_bits=256)
        keccak256.update(node + label)
        hexdigest = keccak256.hexdigest()
        return hexdigest

```

### INS Resolver Implementation

The INS Resolver have to determine the INS Registry `owner` by allowed to update resources.

```python
class INSRegistryInterface(InterfaceScore):
    @interface
    def owner(node: bytes) -> Address: pass


class INSResolver(IconScoreBase):

    __ADDR_INS_REGISTRY_SCORE = 'addr_ins_registry_score'
    __ADDRESS = 'address'
    __MULTIHASH = 'multihash'
    __TEXT = 'text'
    __NAME = 'name'

    @eventlog(indexed=2)
    def AddressChanged(node: bytes, addr: Address) : pass

    @eventlog(indexed=3)
    def MultihashChanged(node: bytes, name: str, multihash: bytes) : pass

    @eventlog(indexed=3)
    def TextChanged(node: bytes, key: str, value: str) : pass

    @eventlog(indexed=2)
    def NameChanged(node: bytes, name: str) : pass

    def __init__(self, db: IconScoreDatabase, owner: Address) -> None:
        super().__init__(db, owner)

        self.__addr_ins_registry_score = VarDB(self.__ADDR_TOKEN_SCORE, db, value_type=Address)
        self.__address = DictDB(self.__ADDRESS, db, value_type=Address)
        self.__multihash = DictDB(self.__MULTIHASH, db, value_type=bytes, depth=2)
        self.__text = DictDB(self.__TEXT, db, value_type=str, depth=2)
        self.__name = DictDB(self.__NAME, db, value_type=str)

        self.__ins_registry_score = self.create_interface_score(self.__addr_ins_registry_score.get(), INSRegistryInterface)

    # TODO ins_address should be defined
    def on_install(self, ins_address: str = '') -> None:
        super().on_install()

        ins_registry_address = Address.from_string(ins_address)
        self.__addr_ins_registry_score.set(ins_registry_address)

        self.__ins_registry_score = self.create_interface_score(self.__addr_ins_registry_score.get(), INSRegistryInterface)

    def on_update(self) -> None:
        super().on_update()

    @external
    def setAddr(node: bytes, addr: Address) -> None:
        if (self.__ins_registry_score.owner(node) == self.msg.sender):
            self.AddressChanged(node, addr)
            self.__address[node] = addr

    @external
    def setMultihash(node: bytes, name: str, multihash: bytes) -> None:
        if (self.__ins_registry_score.owner(node) == self.msg.sender):
            self.MultihashChanged(node, multihash)
            self.__multihash[node][name] = multihash

    @external
    def setTest(node: bytes, key: str, value: str) -> None:
        if (self.__ins_registry_score.owner(node) == self.msg.sender):
            self.TextChanged(node, key, value)
            self.__text[node][key] = value

    @external
    def setName(node: bytes, name: str) -> None:
        if (self.__ins_registry_score.owner(node) == self.msg.sender):
            self.NameChanged(node, name)
            self.__name[node] = name

    @external(readonly=True)
    def addr(node: bytes) -> Address:
        return self.__address[node]

    @external(readonly=True)
    def multihash(node: bytes, name: str) -> bytes:
        return self.__multihash[node][name]

    @external(readonly=True)
    def text(node: bytes, key: str) -> str:
        return self.__text[node][key]

    @external(readonly=True)
    def name(node: bytes) -> str:
        return self.__name[node]
    
```

### INS Registrar Implementation

INS Registrar allows users to register names without any cost.

```python
class INSRegistryInterface(InterfaceScore):

    @interface
    def owner(node: bytes) -> Address: pass

    @interface
    def setSubnodeOwner(node: bytes, label: bytes, owner: Address) : pass


class INSRegistrar(IconScoreBase):

    __ADDR_INS_REGISTRY_SCORE = 'addr_ins_registry_score'
    __ROOT_NODE = 'root_node'

    def __init__(self, db: IconScoreDatabase, owner: Address) -> None:
        super().__init__(db, owner)

        self.__addr_ins_registry_score = VarDB(self.__ADDR_TOKEN_SCORE, db, value_type=Address)
        self.__root_node = VarDB(self.__ROOT_NODE, db, value_type=bytes)

        self.__ins_registry_score = self.create_interface_score(self.__addr_ins_registry_score.get(), INSRegistryInterface)

    # TODO ins_address should be defined
    def on_install(self, ins_address: str = '') -> None:
        super().on_install()

        one_icx = 1 * 10 ** 18

        ins_registry_address = Address.from_string(ins_address)
        self.__addr_ins_registry_score.set(ins_registry_address)

        self.__ins_registry_score = self.create_interface_score(self.__addr_ins_registry_score.get(), INSRegistryInterface)

    def on_update(self) -> None:
        super().on_update()

    # TODO This function is only define register an INS, not include @payable algorithm
    @external
    def register(subnode: bytes, owner: Address): None:
        if (self.__ins_registry_score.owner(subnode) == "" or self.__ins_registry_score.owner(subnode) == self.msg.sender):
            self.__ins_registry_score.setSubnodeOwner(self.__root_node, subnode, owner)

```

## References
- [https://github.com/ethereum/EIPs/blob/master/EIPS/eip-137.md](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-137.md)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
