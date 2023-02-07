---
iip: <to be assigned>
title: ICON Provider JS API
author: Fidel (github.com/FidelVe), Ben (github.com/bkbooth), Eric (github.com/han-so1omon)
discussions-to: https://forum.icon.community/t/creating-a-spec-in-the-iip-for-an-icon-provider-js-api-and-bip44-coin-type-and-path-derivation-discussion/2700/21
status: Draft
type: Standards Track
category: IRC
created: 2022-06-04
---
# Simple Summary

An specification to provide consistency across clients and applications in the ICON network for communication between *software wallets* and *web applications*. 

## Abstract

To allow communication between *software wallets* and *web applications* is necessary for these *software wallets* to expose an API via Javascript in the web page. For the wallets and web apps to communicate it is necessary the creation of dispatching and listening events using Javascript in the browser.



## Motivation

To ensure seamless user experience and avoid implementation of several API communication protocols in each web application to support several software wallets, it is necessary to implement a standardized protocol for all software wallets and web applications to adhere to.

## Specification

The following specification requires that a *dispatching* and a *listening* event be created using *CustomEvent* in the web applications. The type and payload of the events are assigned to the detail field in the *CustomEvent*.

*Data in detail field:*

| Field | Type | Description |
| ----- | ---- | ----------- |
| type | string | Predefined type of events |
| payload | any | Data required for the request or response. |

### Dispatch Event for Request

```javascript
const customEvent = new CustomEvent('ICONEX_RELAY_REQUEST', {
	detail: {
		type: '...',
		payload: {...}
	}
});
window.dispatchEvent(customEvent);
```

### Listening Event for Response

```javascript
const eventHandler = event => {
	const { type, payload } = event.detail;
	switch (type) { ...	}
}
window.addEventListener('ICONEX_RELAY_RESPONSE', eventHandler);
```

### Methods

**HAS_ACCOUNT**

`REQUEST_HAS_ACCOUNT` Requests for whether the software wallet has any ICON wallet.

`RESPONSE_HAS_ACCOUNT` Returns boolean-typed results.

```javascript
const customEvent = new CustomEvent('ICONEX_RELAY_REQUEST', {
	detail: {
		type: 'REQUEST_HAS_ACCOUNT'
	}
});
window.dispatchEvent(customEvent);

const eventHandler = event => {
	const { type, payload } = event.detail;
	if (type === 'RESPONSE_HAS_ACCOUNT') {
		console.log(payload.hasAccount); // true or false
	}
}
window.addEventListener('ICONEX_RELAY_RESPONSE', eventHandler);
```

**HAS_ADDRESS**

`REQUEST_HAS_ADDRESS` Requests for whether the software wallet has the specified wallet address.

`RESPONSE_HAS_ADDRESS` Returns boolean-typed results.

```javascript
const customEvent = new CustomEvent('ICONEX_RELAY_REQUEST', {
	detail: {
		type: 'REQUEST_HAS_ADDRESS',
		payload: 'hx19870922...'
	}
});
window.dispatchEvent(customEvent);

const eventHandler = event => {
	const { type, payload } = detail
	if (type === 'RESPONSE_HAS_ADDRESS') {
		console.log(payload); // true or false
	}
}
window.addEventListener('ICONEX_RELAY_RESPONSE', eventHandler);
```

**ADDRESS**

`REQUEST_ADDRESS` Requests for the address to be used for service.

`RESPONSE_ADDRESS` Returns the icx address selected by the user.

```javascript
const customEvent = new CustomEvent('ICONEX_RELAY_REQUEST', {
	detail: {
 		type: 'REQUEST_ADDRESS'
	}
});
window.dispatchEvent(customEvent);

const eventHandler = event => {
	const { type, payload } = detail;
	if (type === 'RESPONSE_ADDRESS') {
		console.log(payload); // e.g., hx19870922...
	}
}
window.addEventListener('ICONEX_RELAY_RESPONSE', eventHandler);
```

**JSON-RPC**

`REQUEST_JSON-RPC` Requests for calling standard ICON JSON-RPC API. (User confirmation is required in some cases.)

`RESPONSE_JSON-RPC` Returns the JSON-RPC response.

`CANCEL_JSON-RPC` User canceled the JSON-RPC request.

```javascript
const customEvent = new CustomEvent('ICONEX_RELAY_REQUEST', {
	detail: {
		type: 'REQUEST_JSON-RPC',
		payload: {
			jsonrpc: "2.0",
			method: "icx_method",
			id: 6339,
			params: {
				from: "hx19870922...",
				...
			}
		}
	}
});
window.dispatchEvent(customEvent);

const eventHandler = event => {
	const { type, payload } = detail;
	if (type === 'RESPONSE_JSON-RPC') {
		console.log(payload); // e.g., {"jsonrpc": "2.0", "id": 6339, "result": { ... }}
	}
	else if (type === 'CANCEL_JSON-RPC') {
		console.error('User canceled JSON-RPC request')
	}
}
window.addEventListener('ICONEX_RELAY_RESPONSE', eventHandler);
```

**SIGNING**

`REQUEST_SIGNING` Request for only signing tx hash. (User confirmation is always required.)

`RESPONSE_SIGNING` Returns signature.

`CANCEL_SIGNING` User canceled the signing request.

```javascript
const customEvent = new CustomEvent('ICONEX_RELAY_REQUEST', {
	detail: {
		type: 'REQUEST_SIGNING',
		payload: {
			from: 'hx19870922...',
			hash: '0x13979...'
		}
	}
});
window.dispatchEvent(customEvent);

const eventHandler = event => {
	const { type, payload } = detail
	if (type === 'RESPONSE_SIGNING') {
		console.log(payload) // e.g., 'q/dVc3qj4En0GN+...'
	}
	else if (type === 'CANCEL_SIGNING') {
		console.error('User canceled signing request')
	}
}
window.addEventListener('ICONEX_RELAY_RESPONSE', eventHandler);
```

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All IIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The IIP must explain how the author proposes to deal with these incompatibilities. IIP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

## Test Cases
<!--Test cases for an implementation are mandatory for IIPs that are affecting consensus changes. Other IIPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any IIP is given status "Final", but it need not be completed before the IIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
