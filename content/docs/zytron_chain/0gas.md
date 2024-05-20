+++
title = "Zytron Chain - 0 GAS"
description = "Zytron chain is 0 gas"
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 4501
sort_by = "weight"
template = "docs/page.html"

[extra]
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

### Zytron on Linea SBT

|  Field   | Value  |
|  ----  | ----  |
| Network Name  | [Linea(Sepolia)](https://chainlist.org/chain/59141) |
| Contract Address | 0x |

### Zytron on B2 SBT

|  Field   | Value  |
|  ----  | ----  |
| Network Name  | B2 |
| Contract Address | 0x |

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
