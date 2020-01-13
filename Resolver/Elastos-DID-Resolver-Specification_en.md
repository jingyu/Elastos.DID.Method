# Elastos DID Resolver Specification v0.2

Smart Web Powered by Blockchain

----

Elastos Foundation

30 November 2019

**Version**

0.2

**Description of Contents**

This document is the Elastos DID Resolver specification, which is published and maintained by the Elastos Foundation. It mainly explains the definition of the Elastos DID Resolver, as well as the definition of related requests and response data. In the future, we will continue to upgrade this document so that it reflects the latest state of development of the Elastos DID technology.


**Copyright Declaration**

The copyright of this document belong to the Elastos Foundation. All rights reserved.

----

[TOC]

----

## Abstract

Elastos uses the [JSON-RPC](https://www.jsonrpc.org/specification) interface to provide the DID resolve method. For more details about [JSON-RPC](https://www.jsonrpc.org/specification), please see specifications on the official website.


## Resolve Request

### method

The string value contains the name of the method to be invoked. For the DID Resolve request, the value is "resolvedid".

### params

The object value, which is the parameter to be included in the call of `resolvedid`  method. The parameters are defined as follows:

- did

  DID string. Can be a complete DID string, such as *did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4*, or just a method specific string, such as: *iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4*.


- all

  Whether or not all operation history of this DID are obtained. Boolean, true or false.


### id

The request identifier set by the client must include a string, numerical or null value (if contained). The value usually should not be null, and the numerals should not include decimals.


## Resolve Response

### jsonrpc

The JSON-RPC default response property, a string value, indicates the JSON-RPC protocol version. Must be "2.0."

### result

If Resolve is executed successfully, then this member is included, and the value is the result object of resolve. If an error occurs when resolve is invoked, then this member does not exist or is null. See [Resolve Result Object](#resolve-result-object) for the definition of the DID Resolver’s resolve result object.


### error

If Resolve executes normally, then this member does not exist or is null. If there is an error in the execution of Resolve, this member is included, the value is an error object. See [Error Object](#error-object) for the definition of error objects.

### id

This member is necessary. It must be identical to the value of the id member in the request object. If an error occurs at the time of ID inspection in the request object (for example resolve error/invalid request), then this value must be null.

## Resolve Result Object

When the DID Resolve executes successfully, the response must contain a result member whose value is an object containing the following members:

### did

The request's target DID.

### status

The numerical value type status code indicates the result status of resolving target DID. This value is an integer. The value is defined as the following:

| status | meaning                                                                                                               |
| ------ | --------------------------------------------------------------------------------------------------------------------- |
| 0      | DID document is valid. Transaction member contains an array which is  the result of requesting DID transaction.       |
| 1      | DID document is expired. Transaction member contains an array which is  the result of requesting DID transaction.     |
| 2      | DID document is deactivated. Transaction member contains an array which is  the result of requesting DID transaction. |
| 3      | DID does not exist. The result object does not contain the transaction member.                                        |

> When resolving a DID, regardless of the DID status, it should return this object, and should not return the error object, as long as the resolve is executed normally. Different resolve results are expressed though different status values. For example, statuses such as DID does not exists, is expired, or is deactivated, are all normal, and should return this object. It will only return the error object when the Resolver service cannot execute resolve actions, and in general is a JSON-RPC error or a Resolver error itself.

### transaction

An array object, and the elements are the ID transaction and operation information of the target DID being requested. The element is an object containing the following members:

- **txid**

  The string member which is the current ID transaction's transaction id.

- **timestamp**

  For the transaction time, whose value must be a valid string value conforming to [RFC3339](https://tools.ietf.org/html/rfc3339) combining the date and time, and must also be normalized to UTC time, followed in "Z."

- **operation**

  The object member, which is the payload of the current ID transaction, that is, the JSON object of the DID operation.

When`all` in [the parameters of Resolve DID request](#params) is *false*, the transaction array object only contains information from the target ID's last transaction. When `all` is *true*, the transaction array object contains all transaction information of the target DID, and the array elements are stored in descending order according to ID’s transaction time, meaning that no. #0 is the newest ID transaction.

## Error object

When the Resolve encounters an error, an error member must be contained in the response, whose value is an object containing the following members:

### code

The numerical type error code indicating the error type occurred. This value is an integer.

According to the definition in [specification](https://www.jsonrpc.org/specification), error codes from -32768 to -32000 (including -32768 to -32000) are reserved for predefined errors. Any code within this scope that has not been specifically defined below is reserved for future use. 

| code             | message          | meaning                                                                                               |
| ---------------- | ---------------- | ----------------------------------------------------------------------------------------------------- |
| -32700           | Parse error      | Invalid JSON was received by the server. An error occurred on the server while parsing the JSON text. |
| -32600           | Invalid Request  | The JSON sent is not a valid Request object.                                                          |
| -32601           | Method not found | The method does not exist / is not available.                                                         |
| -32602           | Invalid params   | Invalid method parameter(s).                                                                          |
| -32603           | Internal error   | Internal JSON-RPC error.                                                                              |
| -32000 to -32099 | Server error     | Reserved for implementation-defined server-errors.                                                    |

### message

A string briefly describing the error. Generally, a concise sentence.

### data

Member optional, contains other information related to the error. The value is defined by the Resolver server (for example, detailed error information, nested errors, etc.).

## Examples

### Resolve the DID document, and the document is valid

#### Request

```json
{
    "method": "resolvedid",
    "params":{
        "did": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
        "all": false
    },
    "id": "8555cbd1afbf3b8fd8748464ee949574"
}
```

#### Response

```json
{
  "id": "8555cbd1afbf3b8fd8748464ee949574",
  "jsonrpc": "2.0",
  "result": {
    "did": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
    "status": 0,
    "transaction": [{
      "txid": "467491e6be79fbc6...ce9192d6f15ca81e",
      "timestamp": "2019-08-10T17:30:00Z",
      "operation": {
        "header": {
          "specification": "elastos/did/1.0",
          "operation": "update",
          "previousTxid": "3641de55f368583c...8917756a872093d2"
        },
        "payload": "eyJpZCI6ImRpZDplbGFzdG9zOmlWUGFk...UMDI6MDA6MDBaIn0",
        "proof": {
          "type":"ECDSAsecp256r1",
          "verificationMethod":"#primary",
          "signature":"IdcfNkS7yA_GHL6g...6lWq7eI9OJGaNPbg"
        }
      }
    }]
  }
}
```

### Resolve the DID document, the document is expired

#### Request

```json
{
    "method": "resolvedid",
    "params":{
        "did": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
        "all": false
    },
    "id": "8555cbd1afbf3b8fd8748464ee949574"
}
```

#### Response

```json
{
  "id": "8555cbd1afbf3b8fd8748464ee949574",
  "jsonrpc": "2.0",
  "result": {
    "did": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
    "status": 1,
    "transaction": [{
      "txid": "467491e6be79fbc6...ce9192d6f15ca81e",
      "timestamp": "2019-08-10T17:30:00Z",
      "operation": {
        "header": {
          "specification": "elastos/did/1.0",
          "operation": "update",
          "previousTxid": "3641de55f368583c...8917756a872093d2"
        },
        "payload": "eyJpZCI6ImRpZDplbGFzdG9zOmlWUGFk...UMDI6MDA6MDBaIn0",
        "proof": {
          "type":"ECDSAsecp256r1",
          "verificationMethod":"#primary",
          "signature":"IdcfNkS7yA_GHL6g...6lWq7eI9OJGaNPbg"
        }
      }
    }]
  }
}
```

### Resolve the DID document, the DID has been deactivated

#### Request

```json
{
    "method": "resolvedid",
    "params":{
        "did":"iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
        "all": false
    },
    "id": "8555cbd1afbf3b8fd8748464ee949574"
}
```

#### Response

```json
{
  "id": "8555cbd1afbf3b8fd8748464ee949574",
  "jsonrpc": "2.0",
  "result": {
    "did": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
    "status": 2,
    "transaction": [{
      "txid": "467491e6be79fbc6...ce9192d6f15ca81e",
      "timestamp": "2019-11-10T21:30:00Z",
      "operation": {
        "header": {
          "specification": "elastos/did/1.0",
          "operation": "deactivate"
        },
        "payload": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
        "proof": {
          "verificationMethod":"#recovery",
          "signature":"IdcfNkS7yA_GHL6g...6lWq7eI9OJGaNPbg"
        }
      }
    }]
  }
}
```

### Resolve the DID document, the DID does not exist

#### Request

```json
{
    "method": "resolvedid",
    "params":{
        "did": "iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
        "all": false
    },
    "id": "8555cbd1afbf3b8fd8748464ee949574"
}
```

#### Response

```json
{
  "id": "8555cbd1afbf3b8fd8748464ee949574",
  "jsonrpc": "2.0",
  "result": {
    "did": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
    "status": 3
  }
}
```

### Resolve all DID operation records

#### Request

```json
{
    "method": "resolvedid",
    "params":{
        "did": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
        "all": true
    },
    "id": "8555cbd1afbf3b8fd8748464ee949574"
}
```

#### Response

```json
{
  "id": "8555cbd1afbf3b8fd8748464ee949574",
  "jsonrpc": "2.0",
  "result": {
    "did": "did:elastos:iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
    "status": 0, // Maybe 0, 1, 2
    "transaction": [{
      "txid": "467491e6be79fbc6...ce9192d6f15ca81e",
      "timestamp": "2019-12-1T17:30:00Z",
      "operation": {
        "header": {
          "specification": "elastos/did/1.0",
          "operation": "update",
          "previousTxid": "3641de55f368583c...8917756a872093d2"
        },
        "payload": "eyJpZCI6ImRpZDplbGFzdG9zOmlWUGFk...UMDI6MDA6MDBaIn0",
        "proof": {
          "type":"ECDSAsecp256r1",
          "verificationMethod":"#primary",
          "signature":"IdcfNkS7yA_GHL6g...6lWq7eI9OJGaNPbg"
        }
      }
    }, {
      "txid": "3641de55f368583c...8917756a872093d2",
      "timestamp": "2019-10-18T10:30:00Z",
      "operation": {
        "header": {
          "specification": "elastos/did/1.0",
          "operation": "update",
          "previousTxid": "3a2936c4777f02a9...3f683c82a1aa378c"
        },
        "payload": "E1NndTUkR2dEQ1SEt2Q1BOcnlITszcV...ZTVTQiLCJwdWJsaW",
        "proof": {
          "type":"ECDSAsecp256r1",
          "verificationMethod":"#primary",
          "signature":"xOanVzekR2aGk5aW...l2NlJxQ0RHc1Rjc0"
        }
      }
    }, {
      "txid": "3a2936c4777f02a9...3f683c82a1aa378c",
      "timestamp": "2019-08-10T17:30:00Z",
      "operation": {
        "header": {
          "specification": "elastos/did/1.0",
          "operation": "create"
        },
        "payload": "NLZXkiOlt7ImlkIjoiI3ByaW1hcnkiLC...JwdWJsaWNLZXlCYX",
        "proof": {
          "type":"ECDSAsecp256r1",
          "verificationMethod":"#primary",
          "signature":"NlNTgiOiJwdmZBUU...68dTRnTm1hdDZHTE"
        }
      }
    }]
  }
}
```

### Resolve the DID error

#### Request

```json
{
    "method": "resolvedid",
    "params":{
        "did": "iVPadJq56wSRDvtD5HKvCPNryHMk3qVSU4",
        "all": false
    },
    "id": "8555cbd1afbf3b8fd8748464ee949574"
}
```

#### Response

```json
{
  "id": "8555cbd1afbf3b8fd8748464ee949574",
  "jsonrpc": "2.0",
  "error": {
    "code": -32001,
    "message": "Resolver internal error."
  }
}
```