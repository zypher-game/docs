+++
title = "0 GAS"
description = "Zytron chain is 0 gas"
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 4003
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "zytron"
lead = 'Zytron chain is 0 gas'
toc = true
top = false
+++

## 0 Gas

If your address and the DApp you use support Zytron's 0gas service,
you only need to use Metamask to initiate a signature to authorize the transaction.

> Pic here.

## 0gas SBT

Different Zytron networks have a contract that records whether users can be
exempted from gas permissions. This permission is displayed in SBT form.
Users can check whether there is SBT in their wallet to check whether 0gas interaction is possible.

On the testnet, developers can call the `chaim()` method to claim SBT for testing. Only one SBT can be claimed per address.

If users can't use 0gas, please send transaction to zytron network.
Please refer to [Set Priority Gas in Metamask](#set-priority-gas-in-metamask) to configure low gas.

### Zytron on Linea SBT

|  Field   | Value  |
|  ----  | ----  |
| Network Name  | [Linea (Sepolia)](https://chainlist.org/chain/59141) |
| Contract Address | 0x7b00c14c7D0087C3a0d37cAbbc6924566B4481E9 |
| Claim Contract Address | 0x11026b5f5F0441d1C08e7dF49A3C58BC5f3E8d26 |
| 0gas Service Endpoint | https://rpc-zytron-testnet-linea.zypher.game/ |

### Zytron on B2 SBT

|  Field   | Value  |
|  ----  | ----  |
| Network Name  | B2 Testnet |
| Contract Address | 0x77DB62EAB363e6DEF480e4C63210f162438eeD77 |
| Claim Contract Address | 0xfd02aa4e1DB022D74c6894417f9F47C5B3DeBEd7 |
| 0gas Service Endpoint | https://rpc-zytron-testnet-b2.zypher.game/ |

### Documents of 0gas service

> You can use endpoint to access swagger document.

#### Query address balance of 0gas SBT

Use this method to check if the user has 0gas SBT. If this value is not 0, it means that the user can use 0gas services.

Request:

```
GET {endpoint}/balanceof/{address}
```

Response:

```
{
  "data": "string",
  "msg": "string",
  "code": 0,
  "uid": "string"
}
```

#### Create 0gas Contract wallet

Using this method, you can create a contract wallet that will interact with other DApps. 
The creator automatically becomes the owner of the contract wallet.
The 0gas service will also pay the transaction fee on behalf of the user through this wallet

POST /wallet

Request:

```json
{
    // The address of who can operate this wallet
    "controller": "string",
    // Signature V
    "v": 0,
    // Signature R
    "r": "string",
    // Signature S
    "s": "string",
    // Owner of this wallet
    "owner": "string"
}
```

Response:

```json
{
  "data": "string",
  "msg": "string",
  "code": 0,
  "uid": "string"
}
```

#### Make Function Call by Wallet.

POST /functioncall

Request

```json
{
    // Address of wallet
    "wallet": "string",
    // Which address you want to call
    "to": "string",
    // Calldata
    "data": "string",
    // How much token you want to send.
    "value": "string",
    // Signature V
    "v": 0,
    // Ignature R
    "r": "string",
    // Siganture S
    "s": "string",
    // The owner of wallet
    "owner": "string"
}
```

Response:

```json
{
  "data": "string",
  "msg": "string",
  "code": 0,
  "uid": "string"
}
```

#### Make Function Call in Array

Same as Function call, but use array to make multi call.

POST /functioncall_list

Request:

```json
{
  "list": [
    {
      "wallet": "string",
      "to": "string",
      "data": "string",
      "value": "string",
      "v": 0,
      "r": "string",
      "s": "string",
      "owner": "string"
    }
  ]
}
```

Response:

```json
{
  "data": "string",
  "msg": "string",
  "code": 0,
  "uid": "string"
}
```

#### Set Controller

Set controller of contract wallet.

POST /set_controller

Request:

```json
{
  "wallet": "string",
  "controller": "string",
  "owner": "string",
  "is_allow": true,
  "v": 0,
  "r": "string",
  "s": "string"
}
```

Response:

```json
{
  "data": "string",
  "msg": "string",
  "code": 0,
  "uid": "string"
}
```

## Set Priority Gas in Metamask

Since zytron allows lower transaction gas support, metamask cannot be directly adapted in many cases.
Therefore, you need to manually configure the Priority Gas Price.

To enable this function, please follow these step:

1. In the send transaction interface, click Estimated fee edit button (blue button).
2. Click Advanced.
3. You can adjust the Priority Gas Price according to your needs. The larger this value, 
    the easier it is for the transaction to be confirmed by the network.
    We recommend using 0.002 as the Priority Gas Price.
4. To avoid doing this for every future transaction, please click "Save these values as my default for the..."
5. Click "Save" and comfirm transaction.
