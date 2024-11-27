+++
title = "Precompiles"
description = "The precompiles in zytron kit"
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 4101
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "zytron"
lead = 'The precompiles in zytron kit'
toc = true
top = false
+++

## Precompiles

Currently zytron provides the following precompilers, which are placed at different addresses:

- anemoi
- ed_on_bn254
- shuffle
- matchmaking
- shuffle

## anemoi

| Contract Name | Address |
| - | - |
| AnemoiJive254 | 0x00000000000000000000000000000000000000014 |

Input parameters are encoded in the following form:

```solidity
bytes32[] memory data = /* .. */
bytes memory input = abi.encode(data)

address precompile = 0x00000000000000000000000000000000000000014;
precompile.staticcall(input);
```

## ed_on_bn254

| Contract Name | Address |
| - | - |
| EdOnBN254PointAdd | 0x00000000000000000000000000000000000000015 |
| EdOnBN254ScalarMul | 0x00000000000000000000000000000000000000016 |


EdOnBN254PointAdd:

```solidity
uint256 x1 = /* */;
uint256 y1 = /* */;
uint256 x2 = /* */;
uint256 y2 = /* */;

address precompile = 0x00000000000000000000000000000000000000015;
bytes memory output = precompile.staticcall(abi.encode(x1, y1, x2, y2));
(uint256 x, uint256 y) = abi.decode(output, (uint256 ,uint256));
```

EdOnBN254PointMul:

```solidity
uint256 x = /* */;
uint256 y = /* */;
uint256 s = /* */;

address precompile = 0x00000000000000000000000000000000000000016;
bytes memory output = precompile.staticcall(abi.encode(x, y, s));
(uint256 x, uint256 y) = abi.decode(output, (uint256 ,uint256));
```

## matchmaking

| Contract Name | Address |
| - | - |
| VerifyMatchmaking | 0x00000000000000000000000000000000000000017 |

Input parameters are encoded in the following form:

```solidity
bytes memory verifier_params = /*  */;
bytes[] memory inputs = /*  */;
bytes[] memory outputs = /*  */;
bytes memory commitment = /*  */;
bytes memory random_number = /*  */;
bytes memory proof = /*  */;

address precompile = 0x00000000000000000000000000000000000000017;

require(precompile.staticcall(abi.encode(verifier_params, inputs, outputs, commitment, random_number, proof)));
```

## shuffle

| Contract Name | Address |
| - | - |
| Shuffle | 0x00000000000000000000000000000000000000018 |

Input parameters are encoded in the following form:

```solidity
bytes memory verifier_params = /*  */;
bytes[] memory input_cards = /*  */;
bytes[] memory output_cards = /*  */;
bytes memory proof = /*  */;

address precompile = 0x00000000000000000000000000000000000000018;

require(precompile.staticcall(abi.encode(verifier_params, input_cards, output_cards, proof)));
```
