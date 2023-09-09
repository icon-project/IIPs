---
iip:
title: ICON XChain Fungible Token
author: Brandon WInkler (@bawinkl) or #bwnodnarb on discord, brandon@inanisinvictus.com
discussions-to: 
status: Draft
type: Standards Track
category: IRC
created: 9/8/2023
---

## Summary
A standard interface for expanding the IRC-2 Token standard for cross chain compatiability using XCall

## Abstract
This document describes an interface that can be used to manage fungible tokens in a contract with extended functionality allowing
cross chain address referencing for XCall enabled transactions. While the existing IRC-2 standard only support local network transactions on
the ICON network (largely because their address referencing is tightly coupled to the "Address" type which may not be compatible with addressing schemes on other blockchains), the XChain Fungible token extends the existing IRC-2 standard by implementing a "Network Address" type that allows referencing using a Network ID and address in unision to track balances. Additionally there is an implementation of XCall's "handleCallMessage" allowing this contract to be called through XCall, from other implemented networks.

## Motivation
Using XCall, we should be able to have single-sourced token contracts that can interact across multiple blockchains. Gone should be the day of bridged and duplicated solutions for traking tokens. By leveraging a cross chain compatible token format, a single contract can be used to deploy tokens on many chains.

This project is part of the XCall Incentivized test net submissions https://iconfoundation.notion.site/Build-a-cross-chain-game-12c95e1aa02f4a4e9772b38abf31a2ce

and is an extension of the existing IRC-2 Token standard : https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md

## Specification

As an extension of the IRC-2 token standard, the ICON XChain Fungible Token includes all existing required methods to fulfill the IRC-2 interface requirements. Existing methods have been modified to make use of a new NetworkAddress SDO (ScoreDataObject) for tracking and reference cross chain address references. Since the IRC-2 interface requires implementations using the Address class type, a new version of each method was created that is prefixed with "x_" that, instead, receives a string value in place of the Address. 

The new NetworkAddress type can accept local ICON network addresses, in addition to BTP [btp://][network id]/[address hash] and Network Address [network id]/[address hash] formats.

Additionally there is a new tracked variable "Network ID" that must be configured for the SCORE to indicate the network it is deployed on. This is required to accept local Address types (which are then converted into Network Address types) using this value. As an example, if this SCORE is deployed on the Berlin Test Net, the network address should be set as "0x7.icon", where as we would use "0x1.icon" for the ICON main net.

### Methods
Below are included the methods and relevant changes applied against the existing IRC-2 token standard.

#### balanceOf
```java
/**
 * This is the original IRC-2 implementation of balanceOf, which should convert Address into a NetworkAddress and store the balance of a given address as a string value in the Network Address format "[network id]/[address hash]"
 * Required to match the existing IRC-2 interface requirements
 * @param _owner: the ICON address of the owner
 */
@External(readonly = true)
public BigInteger balanceOf(Address _owner)
```

#### x_balanceOf
```java
/**
 * A XCall compatible implementation of the balanceOf function
 * Returns the balance of a given address
 * This implementation can accept any address, network address or btp address in string format
 * @param _owner: the targeted address in one of the following formats: icon address, network address ([NetworkID]/[Address]) or btp address ([btp://][NetworkID]/[Address])
 * @param _id: the token ID
 */
@External(readonly = true)
public BigInteger x_balanceOf(String _owner)
```

#### transfer
The standard IRC-2 implementation of the transfer method with the exception that it should 
Transfers `_value` amount of tokens to address `_to`, and MUST fire the `Transfer` event. This function SHOULD throw if the `self.msg.sender` account balance does not have enough tokens to spend. If `_to` is a contract, this function MUST invoke the function `tokenFallback(Address, int, bytes)` in `_to`. If the `tokenFallback` function is not implemented in `_to` (receiver contract), then the transaction must fail and the transfer of tokens should not occur. If `_to` is an externally owned address, then the transaction must be sent without trying to execute `tokenFallback` in `_to`.  `_data` can be attached to this token transaction. `_data` can be empty.
```java
/**
* The standard IRC-2 implementation of transfer
* Transfers `_value` amount of tokens to address `_to`, and MUST fire the `Transfer` event. This function SHOULD throw if the `self.msg.sender` account balance does not have enough tokens to spend.
* @param _to: an ICON address type to transfer value to
* @param _value: the amount of tokens in LOOP value to transfer
* @param _data: an optional byte encoded array that will be passed in tokenfallback if the address is a local network contract 
*/
@External(readonly = true)
public void transfer(Address _to, BigInteger _value, @Optional byte[] _data)
```

#### x_transfer
The XCall compatible implementation of the transfer function that accepts a string in the local network format, Network Address format or BTP format. All other requirements and implementations of the original transfer method should remain intact.
```java
/**
* A XCall compatible implementation of the transfer method
* This implementation can accept any address, network address or btp address in string format for _to
* Transfers `_value` amount of tokens to address `_to`, and MUST fire the `Transfer` event. This function SHOULD throw if the `self.msg.sender` account balance does not have enough tokens to spend.
* Returns the balances of a set of owners and ids
* @param _to: an address in string format, network address format ([NetworkID]/[Address]) or btp address format ([btp://][NetworkID]/[Address])
* @param _value: the amount of tokens in LOOP value to transfer
* @param _data: an optional byte encoded array that will be passed in tokenfallback if the address is a local network contract 
*/
@External
public void x_transfer(String _to, BigInteger _value, @Optional byte[] _data)
```



### XCall handleCallMessage method
The handleCallMessage method must be implemented to allow external networks to affect state changes using XCall to this contract.

```java
/*
 * Handles the call message received from the source chain
 * should only be called from the XCall service address and should handle "transfer"
 * requests. 
 * Recommended to use a json format outlined below, however implementation is flexible.
 * This method should parse the json from _data and also verify that it only receives transactions
 * from the XCall service contract.
 * 
 * @param _from The exterenal address of the caller on the source chain
 * @param _data The calldata delivered from the caller in the following JSON
 * format (required values are based on the intended method):
 * {
 * method: "transfer", //required
 * data : {
 * _from: "", // A btp/network address string
 * _to: "", // A btp/network address string
 * _value: "0x1" // BigInteger in Loop value representation
 * _data: "", // an encoded byte array string
 * }
 * }
 */
@External
public void handleCallMessage(String _from, byte[] _data)
```

### Network Address Implementation

The network address format is critical to the implementation of cross chain token referencing since it stores and references addresses using the btp and/or network address format instead of relying just on "Address" in the standard. This is an SDO class included as part of the implementation and is stored as a string value when stored in the State DB in the contract. 

```java
@ScoreDataObject
public class NetworkAddress {

    protected String networkID;
    protected String address;

    public NetworkAddress()
    {
        networkID = "";
        address = ""; 
    }

    /**
     * @param address The local network address by itself (ex: hxc5e0b88cb9092bbd8b004a517996139334752f62), a BTP address in the format of [btp://][networkID]/[address] or a network address without the leading btp component [networkID]/[address]
     * @param networkID
     */
    public NetworkAddress(String addr, String networkID)
    {
       parseNetworkAddress(addr, networkID);
    }

    /**
     * @param address The local network address by itself (ex: hxc5e0b88cb9092bbd8b004a517996139334752f62), a BTP address in the format of [btp://][networkID]/[address] or a network address without the leading btp component [networkID]/[address]
     * @param networkID
     */
    public NetworkAddress(Address addr, String networkID)
    {
       parseNetworkAddress(addr.toString(), networkID);
    }

    /**
     * Parses a network address from a BTP network address string [btp://][networkID]/[address] with the [btp://] componenet being optional.
     * will revert if address is null or blank
     * will revert if address does not meet one of the expectedc formats
     * @param addr Required; The local network address by itself, a BTP address in the format of [btp://][networkID]/[address] or a network address without the leading btp component [networkID]/[address]
     * @param networkID Optional; A network ID value, required only if the addr is in the local network format or a blank string
     */
    private void parseNetworkAddress(String addr, String netID)
    {
        //Force our address to a lower case variant, in case it wasn't
        addr = addr.toLowerCase();

        //Revert if the address is blank or null
        if(addr == null || addr.length() == 0)
            Context.revert("address cannot be null or blank when parsing a network address");

        //Remove the btp component if it exists
        if(addr.startsWith("btp://"))
        {
            addr = addr.substring(6);    
        }

        //Revert if the string does not contian the expected seperator ("/") or contains more than one
        if(addr.indexOf("/") < 0 && (netID == null || netID.length() == 0))
            Context.revert("address does not appear to be in the expected network address format ([btp://][networkID]/[address]) and a network ID was not provided.");
        else if (addr.indexOf("/") > -1 && addr.indexOf("/") != addr.lastIndexOf("/"))
             Context.revert("address does not appear to be in the expected btp or network address format ([btp://][networkID]/[address]).");

        //Parse the network and address components from the string or set them to the assigned values
        if(addr.indexOf("/") > -1)
        {
            networkID = addr.substring(0, addr.indexOf("/"));
            address = addr.substring(addr.indexOf("/") + 1);
        }
        else
        {
            networkID = netID;
            address = addr;
        }
    }

    public String getNetworkID()
    {
        return this.networkID;
    }

    public String setNetworkID(String value)
    {
        return this.networkID = value;
    }

    public String getAddress()
    {
        return this.address;
    }

    public String setAddress(String value)
    {
        return this.address = value;
    }

    /**
     * Writes the newtork address to an object using ObjectWriter
     * @param w the object writer
     * @param na the network address
     */
    public static void writeObject(ObjectWriter w, NetworkAddress na) {         
        
        w.beginList(1);
        w.write(na.networkID);
        w.write(na.address);
        w.end();
    }

    /**
     * Reads the network address object from an object reader
     * @param r the object reader
     */
    public static NetworkAddress readObject(ObjectReader r) {

        String networkID = "";
        String address = "";   

        r.beginList();
        if(r.hasNext())
        networkID = r.readString();
        if(r.hasNext())
        address = r.readString();
        r.end();

        return new NetworkAddress(networkID, address);
    }

    /**
     * Returns a string representation of a network address in [networkID]/[address] format
     */
    public String toString() {
        return networkID + "/" + address;
    }

    /**
     * Returns a map representation of the network address containing networkID and address keys
     */
    public Map<String, String> toMap() {
        Map<String, String> map = Map.of(   
                "networkID", this.networkID,
                "address", this.address);
            return map;
    }

     /**
      * Overriding equals() to compare two network address objects
      */
     @Override
     public boolean equals(Object o) {
  
         // If the object is compared with itself then return true 
         if (o == this) {
             return true;
         }
  
         /* Check if o is an instance of Complex or not
           "null instanceof [type]" also returns false */
         if (!(o instanceof NetworkAddress)) {
             return false;
         }
          
         // typecast o to Complex so that we can compare data members
         NetworkAddress c = (NetworkAddress)o;
          
         // Compare the data members and return accordingly
         return this.address.equals(c.address)
                 && this.networkID.equals(c.networkID);
     }
}

```

## Implementation
* [XChainFungibleToken] (https://github.com/bawinkl/XChainFungibleToken)
* [Berlin Test Net Contract] (https://tracker.berlin.icon.community/contract/cxf0c6f569797e3ddb43ad29b6b2fdb0060f7f467e)

## References
* [ICON XCall documentation] (https://www.xcall.dev/what-is-xcall)
* [IRC-2 Token Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md)
* [IRC-3 Non-Fungible Token Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-3.md)
* [ERC-1155 Multi Token Standard](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
