---
iip: <to be assigned>
title: ICONex Connect for Mobile
author: Jeonghwan Ahn (@a1ahn)
discussions-to: https://github.com/icon-project/IIPs/issues/14
status: Draft
type: Standard Track
category (*only required for Standard Track): IRC
created: 2019-01-02
---

## Simple Summary

A simple protocol for 3rd party Applications that send transactions, ICX coins or IRC tokens via ICONex.

## Abstract

This protocol enables the 3rd party Applications to send ICX coins, messages, and IRC Tokens via ICONex wallets.
Requests and responses are provided in JSON format.

## Motivation

A service offers easy wallet management tools for ICON dApps and helps them to work more easily with ICON network via ICONex.

## Specification

### Types
| VALUE type | Description | Example |
|:----------|:----|:----|
| T_ADDR_EOA | "hx" + 40 digit HEX string | hxbe258ceb872e08851f1f59694dac2558708ece11 |
| T_ADDR_SCORE | "cx" + 40 digit HEX string | cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32 |
| T_HASH | "0x" + 64 digit HEX string | 0xc71303ef8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238 |
| T_INT | "0x" + lowercase HEX string | 0xa |
| T_BIN_DATA | "0x" + lowercase HEX string. Length must be even. | 0x34b2 |
| T_SIG | base64 encoded string | VAia7YZ2Ji6igKWzjR2YsGa2m53nKPrfK7uXYW78QLE+ATehAVZPC40szvAiA6NEU5gCYB4c4qaQzqDh2ugcHgA= |
| T_DATA_TYPE | Type of data | call, deploy, or message |

### Basic Transmission
All requests and responses follow the Basic API Convention JSON format as below.

ICONex Connect uses base64 encoded strings to Inter-App Communications.

#### Base JSON Format
* Request

| Key | Type | Description |
| --- | ---- | ----------- |
| data | String | Base64 encoded string of JSON Object |
| caller | String | A caller to receive the response from ICONex. |

* Response

| Key | Type | Description |
| --- | ---- | ----------- |
| data | String | Base64 encoded strings of JSON Object. |

### Basic API Convention

```
// Request
{
    "id": $INT1,
    "method": "$STRING1"
    "params": {
        "$KEY1": "$VALUE1",
        "$KEY2": "$VALUE2",
        ...
    }
}

// Response
{
    "id": $INT1,
    "code": $INT2,    // if code == 1 success, else fail
    "result": "$VALUE1"
}
```

* Request

| Key | Type | Description | Required |
| --- | ---- | ----------- | -------- |
| id | INT | Request Identifier | Required |
| method | String | Method for current request | Required |
| params | T_DATA_TYPE | Parameters for current request | Optional |

* Response

| Key | Type | Description | Required |
| --- | ---- | ----------- | -------- |
| id | INT | Request Identifier. Exactly same as requested. | Required |
| code | INT | Status code for response. if code == 1 success else fail | Required |
| result | String | Contains message for current response | Optional |

### Methods
* List of methods

| Method | Description | Required Parameters |
| ------ | ----------- | ------------------- |
| bind | Request wallet address | - |
| sign | Request sign for transaction | version, from, value, stepLimit, timestamp, dataType(optional), data(optional) |
| sendICX | Request send ICX | from, to, value |
| sendToken | Request send IRC token | from, to, value, contractAddress |

#### bind
* Return selected wallet's address.

##### Parameters
NULL

##### Returns
Selected wallet's address. (T_ADDR_EOA)

#### sign
* Returns the signature of the transaction hash.

##### Parameters

| Key | Type | Description | Required |
| --- | ---- | ----------- | -------- |
| version | T_INT | Protocol version ("0x3" for v3) | Required |
| from | T_ADDR_EOA | EOA address that created transaction | Required |
| to | T_ADDR_EOA or T_ADDR_SCORE | EOA address to receive coins, or SCORE address to execute the transaction. | Required |
| value | T_INT | Amount of ICX coins in to transfer. When omitted, assumes 0. (1 icx = 1 * 10^18 loop) | Required |
| stepLimit | T_INT | Maximum Step allowance that can be used by the transaction. | Required |
| timestamp | T_INT | Transaction creation time. timestamp is in microsecond. | Required |
| nid | T_INT | Network ID ("0x1" for Mainnet, "0x2" for Testnet, etc) | Required |
| nonce | T_INT | An arbitrary number used to prevent transaction hash collision. | Required |
| dataType | T_DATA_TYPE | Type of data (call, deploy or message) | Optional |
| data | T_DICT or String | The content of data varies depending on the dataType. | Optional |

##### Returns
Signature of transaction hash. (T_SIG)

#### sendICX
* Send ICX via ICONex and returns transaction hash.

##### Parameters

| Key | Type | Description | Required |
| --- | ---- | ----------- | -------- |
| from | T_ADDR_EOA | EOA address that will send ICX coins. | Required |
| to | T_ADDR_EOA | EOA address to receive coins. | Required |
| value | T_INT | Amount of ICX coins in to transfer | Required |
| dataType | T_DATA_TYPE | Type of data (call, deploy or message) | Optional |
| data | T_DICT or String | the content of data varies depending on the dataType. | Optional |

##### Returns
Transaction hash (T_HASH)

#### sendToken
* Send IRC token via ICONex and returns transaction hash.

##### Parameters

| Key | Type | Description | Required |
| --- | ---- | ----------- | -------- |
| from | T_ADDR_EOA | EOA address that will send IRC token. | Required |
| to | T_ADDR_EOA | EOA address to receive tokens. | Required |
| value | T_INT | Amount of IRC token | Required |
| contractAddress | T_ADDR_SCORE | SCORE contract address | Required |

##### Returns
Transaction hash (T_HASH)

#### Error Code

| Code | Message | Description |
| ---- | ------- | ----------- |
| -1000 | Operation canceld by user. | User cancel |
| -1001 | Parse error (Invalid JSON type) | Data is not a JSON type |
| -1002 | Invalid request. | JSON must contains required parameters. |
| -1003 | Invalid method | Invalid method |
| -1004 | Not found caller. | Could not found caller in request. |
| -2001 | ICONex has no ICX wallet. | ICONex has no ICX wallet for support. |
| -2002 | Not found parameter. ('$key') | '$key' is not found in params |
| -3001 | Could not find matched wallet. ('$address') | There is no wallet matched with given address. |
| -3002 | Sending and receiving address are same. | Sending and receiving address are same. |
| -3003 | Insufficient balance. | Requested value is bigger than balance. |
| -3004 | Insufficient balance for fee. | Insufficient balance for fee. |
| -3005 | Invalid parameter. ('$key') | Something is wrong with value. |
| -4001 | Failed to sign. | Error occurs while signing |
| -9999 | Somethings wrong with network ($message) | All errors while networking. $message may contains the reason of error. |

## Implementation
For more details, please visit ICONex for mobile.
* [ICONex Connect for Android](https://github.com/icon-project/iconex_android/tree/master/docs/Connect)
* [ICONex Connect for iOS](https://github.com/icon-project/iconex_ios/tree/master/docs/Connect)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
