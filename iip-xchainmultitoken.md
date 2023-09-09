---
iip: 
title: ICON XChain Multi Token Standard
author: Brandon WInkler (@bawinkl) or #bwnodnarb on discord, brandon@inanisinvictus.com
discussions-to: 
status: Draft
type: Standards Track
category: IRC
created: 9/8/2023
---

## Simple Summary
A standard interface for expanding the IRC-3 Non-Fungible Token & IRC-31 Multi Token standard for cross chain compatiability using XCall

## Abstract
This document describes an extension to the existing IRC-31 Multi Token standard that can be used to manage multiple token types in a contract across multiple blockchain networks.

## Motivation
While the existing token standards work great for local network transactions a solution is needed to allow the use of the IRC-31 (and similar) token formats in cross chain calls across XCall supported networks. 

This solution acts as an extension to the existing IRC-31 interface and aims to maintain those requirements while providing expanded functionality for cross chain communications using XCall.

## Specification
Using XCall, we should be able to have a single implementation of contracts that can interact across multiple blockchains. Gone should be the day of bridged and duplicated solutions for traking tokens across networks. By leveraging a cross chain compatible token format, a single contract can be used to deploy tokens on many chains and allow interactions from both local and supported external networks.

Existing methods have been modified to make use of a new NetworkAddress SDO (ScoreDataObject) for tracking and reference cross chain address references. Since the IRC-2 interface requires implementations using the Address class type, a new version of each method was created that is prefixed with "x_" that, instead, receives a string value in place of the Address. 

The new NetworkAddress type can accept local ICON network addresses, in addition to BTP [btp://][network id]/[address hash] and Network Address [network id]/[address hash] formats.

Additionally there is a new tracked variable "Network ID" that must be configured for the SCORE to indicate the network it is deployed on. This is required to accept local Address types (which are then converted into Network Address types) using this value. As an example, if this SCORE is deployed on the Berlin Test Net, the network address should be set as "0x7.icon", where as we would use "0x1.icon" for the ICON main net.

### Methods

#### getXCallContract
The SCORE will be required to track the XCall contract address on the ICON blockchain network to verify the source of cross chain communications passed to its handleCallMessage implementation.
```java
/**
 * Get the configured XCall Contract
 */
@External(readonly = true)
public Address getXCallContract()
```


#### setXCallContract
Similarly, the XCall contract needs to be managed. This should only be allowed to be updated by the SCORE owner.
```java
/**
* Sets the network ID for this score
* Required to support all local Network Address references
* Can only be set by the SCORE owner
* Must be a contract address
* 
* @param _value: the contract address
*/
@External
public void setXCallContract(Address _value)
```



#### getNetworkID
The contract will need to track and manage its own network ID configuration. This is the BTP network ID assigned to the network the SCORE is installed on. As an example the Berlin Test Network would be "0x7.icon", while the ICON main net would be "0x1.icon". This is required to support legacy IRC-31 interface methods and converting "Address" to the new "NetworkAddress" type.
```java
/**
* Get the configured network ID for this SCORE
*/
@External(readonly = true)
public String getNetworkID()
```



#### setNetworkID
```java
/**
* Sets the network ID for this score
* Required to support all local Network Address references
* Can only be set by the SCORE owner
* 
* @param _value: the network ID value
*/
@External
public void setNetworkID(String _value)
```

#### balanceOf
```java
/**
* This is the original IRC-31 implementation of balanceOf, which should convert the _owner address into the newly implemented NetworkAddress based on the configured network id and the _owner.
* Required to match the existing IRC-31 interface requirements
* @param _owner: the ICON address of the owner
* @param _id: the token ID
*/
@External(readonly = true)
public BigInteger balanceOf(Address _owner, BigInteger _id)
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
public BigInteger x_balanceOf(String _owner, BigInteger _id)
```


#### balanceOfBatch
```java
/**
* This is the original IRC-31 implementation of balanceOfBatch, which looks up the newly implemented NetworkAddress based on the configured Network ID and the _owners.
* Required to match the existing IRC-31 interface requirements
* @param _owner: and array of ICON addresses
* @param _id: an array of token IDs
*/
@External(readonly = true)
public BigInteger[] balanceOfBatch(Address[] _owners, BigInteger[] _ids)
```



#### x_balanceOfBatch
```java
/**
* A XCall compatible implementation of the balanceOfBatch function
* This implementation can accept any address, network address or btp address in string format
* Returns the balances of a set of owners and ids
* @param _owners: an array of addresses in string format, network address format ([NetworkID]/[Address]) or btp address format ([btp://][NetworkID]/[Address])
* @param _id: an array of token IDs
*/

@External(readonly = true)
public BigInteger[] x_balanceOfBatch(String[] _owners, BigInteger[] _ids)
```



#### transferFrom
```java
/**
* This is the original IRC-31 implementation of transferFrom, which looks up the newly implemented NetworkAddress based on the configured Network ID and the _from, _to and caller.
* Required to match the IRC-31 interface requirements
* @param _from: an ICON Address of the tokens to be transferred from
* @param _to: an ICON Address of the tokens to be transferred to
* @param _id: the token Id to transfer
* @param _value: the amount of tokens to transfer
* @param _data  Additional data that should be sent unaltered in call to {@code _to}
*/
@External
public void transferFrom(Address _from, Address _to, BigInteger _id, BigInteger _value, @Optional byte[] _data)
```



#### x_transferFrom
```java
/**
* A XCall compatible implementation of the transferFrom function
* This implementation can accept any address, network address or btp address in string format
* @param _from: an address in one of the following formats: an ICON address in string format, a network address ([NetworkID]/[Address]) or btp address ([btp://][NetworkID]/[Address])
* @param _to: an address in one of the following formats: an ICON address in string format, a network address ([NetworkID]/[Address]) or btp address ([btp://][NetworkID]/[Address])
* @param _id: the token id to transfer
* @param _value: the amount of tokens to transfer
* @param _data  additional data that should be sent unaltered in call to {@code _to}
*/
@External
public void x_transferFrom(String _from, String _to, BigInteger _id, BigInteger _value, @Optional byte[] _data)
```



#### transferFromBatch
```java
/**
 * This is the original IRC-31 implementation of transferFromBatch, which looks up the newly implemented NetworkAddress based on the configured Network ID and the _from, _to and caller.
 * Required to match the IRC-31 interface requirements
 * @param _from: an ICON Address of the tokens to be transferred from
 * @param _to: an ICON Address of the tokens to be transferred to
 * @param _ids: an array of token IDs to transfer
 * @param _value: an array of values to transfer (must match the length of _ids, each index in _ids corresponds to the values in this array)
 * @param _data  Additional data that should be sent unaltered in call to {@code _to}
 */
@External
public void transferFromBatch(Address _from, Address _to, BigInteger[] _ids, BigInteger[] _values, @Optional byte[] _data)
```


#### x_transferFromBatch
```java
/**
* A XCall compatible implementation of the transferFromBatch function
* This implementation can accept any address, network address or btp address in string format
* @param _from: an address in one of the following formats: an ICON address in string format, a network address ([NetworkID]/[Address]) or btp address ([btp://][NetworkID]/[Address])
* @param _to: an address in one of the following formats: an ICON address in string format, a network address ([NetworkID]/[Address]) or btp address ([btp://][NetworkID]/[Address])
* @param _ids: an array of token IDs to transfer
* @param _value: an array of values to transfer (must match the length of _ids, each index in _ids corresponds to the values in this array)
* @param _data  additional data that should be sent unaltered in call to {@code _to}
*/
@External
public void x_transferFromBatch(String _from, String _to, BigInteger[] _ids, BigInteger[] _values,
	@Optional byte[] _data)
```



#### setApprovalForAll
```java
/**
* This is the original IRC-31 implementation of setApprovalForAll, which looks up the newly implemented NetworkAddress based on the configured Network ID and the _from, _to and caller.
* Required to match the IRC-31 interface requirements
* @param _operator: an ICON Address of the intended operator
* @param _approved: a boolean value indicating approval or not
*/
@External
public void setApprovalForAll(Address _operator, boolean _approved)
```



#### x_setApprovalForAll
```java
/**
 A XCall compatible implementation of the setApprovalForAll function
 * This implementation can accept any address, network address or btp address in string format
 * @param _operator: an address in one of the following formats: an ICON address in string format, a network address ([NetworkID]/[Address]) or btp address ([btp://][NetworkID]/[Address])
 * @param _approved: a boolean value indicating approval or not
 */
@External
public void x_setApprovalForAll(String _operator, boolean _approved)
```



#### isApprovedForAll
```java
/**
 * This is the original IRC-31 implementation of isApprovedForAll, which looks up the newly implemented NetworkAddress based on the configured Network ID and the _from, _to and caller.
 * Required to match the IRC-31 interface requirements
 * @param _owner: an ICON Address of the owner
 * @param _operator: an ICON Address of the intended operator
 */
@External(readonly = true)
public boolean isApprovedForAll(Address _owner, Address _operator)
```


#### x_isApprovedForAll
```java
/**
 A XCall compatible implementation of the isApprovedForAll function
 * This implementation can accept any address, network address or btp address in string format
 * @param _owner: an address in one of the following formats: an ICON address in string format, a network address ([NetworkID]/[Address]) or btp address ([btp://][NetworkID]/[Address])
 * @param _operator: an address in one of the following formats: an ICON address in string format, a network address ([NetworkID]/[Address]) or btp address ([btp://][NetworkID]/[Address])
 */
@External(readonly = true)
public boolean x_isApprovedForAll(String _owner, String _operator)
```


### XCall handleCallMessage Implementation
```java
/*
     * Handles the call message received from the source chain
     * can only be called from the XCall service address
     * 
     * @param _from The exterenal address of the caller on the source chain
     * 
     * @param _data The calldata delivered from the caller in the following JSON
     * format (required values are based on the intended method):
     * {
     * method: "methodName", //required
     * data : {
     * _from: "", // A btp/network address string
     * _to: "", // A btp/network address string
     * _operator: "", // A btp/network address string
     * _owner: "", // A btp/network address string
     * _ids: [], // An array of BigInteger values representing tokenIDs
     * _values: [], //An array of BigInteger values representing a value (in case of
     * transfers or minting), the array length should match the _id length
     * _data: "", // an encoded byte array string
     * _approved: // 0x0 or 0x1 indicating true or false
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
* [XChainMultiToken](https://github.com/bawinkl/XChainMultitoken)
* [Berlin Test Net Contract] (https://tracker.berlin.icon.community/contract/cx55a873f1b2d35efbdf738181a3747060191657a5)

## References
* [IRC-3 Non-Fungible Token Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-3.md)
* [IRC-31 Multi Token Standar] (https://github.com/icon-project/IIPs/blob/master/IIPS/iip-31.md)
* [ERC-1155 Multi Token Standard](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1155.md)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
