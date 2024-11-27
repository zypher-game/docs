+++
title = "EdOnBN254(BabyJubJub)"
description = "The precompiles EdOnBN254(BabyJubJub) in zytron kit"
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 4102
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "zytron"
lead = 'The precompiles EdOnBN254(BabyJubJub) in zytron kit'
toc = true
top = false
+++

## EdOnBN254 SDK

We provide Solidity, Rust, Javascript (WebAssembly based) and C (FFI support) SDK.

## Rust Crate

| Crate Name | crates | docs.rs |
| - | - | - |
| uzkge | ![](https://img.shields.io/crates/v/uzkge) | ![](https://img.shields.io/docsrs/uzkge) |

Add dependency to Cargo.toml

```toml
uzkge = "0.1"
```

Use the following code for `ADD`:

```rust
let mut prng = ChaChaRng::from_seed([0u8; 32]);
let p1 = EdwardsAffine::rand(&mut prng);
let p2 = EdwardsAffine::rand(&mut prng);
let (p1_0, p1_1) = p1.xy().unwrap();
let (p2_0, p2_1) = p2.xy().unwrap();

let p1_x = U256::from_big_endian(&p1_0.into_bigint().to_bytes_be());
let p1_y = U256::from_big_endian(&p1_1.into_bigint().to_bytes_be());
let p2_x = U256::from_big_endian(&p2_0.into_bigint().to_bytes_be());
let p2_y = U256::from_big_endian(&p2_1.into_bigint().to_bytes_be());

let data = ethabi::encode(&[
    Token::Uint(p1_x),
    Token::Uint(p1_y),
    Token::Uint(p2_x),
    Token::Uint(p2_y),
]);
```

Use the following code for `MUL`:

```rust
let mut prng = ChaChaRng::from_seed([0u8; 32]);
let s = Fr::rand(&mut prng);
let p1 = EdwardsAffine::rand(&mut prng);
let (p1_0, p1_1) = p1.xy().unwrap();

let scalar = U256::from_big_endian(&s.into_bigint().to_bytes_be());
let p1_x = U256::from_big_endian(&p1_0.into_bigint().to_bytes_be());
let p1_y = U256::from_big_endian(&p1_1.into_bigint().to_bytes_be());

let data = ethabi::encode(&[Token::Uint(scalar), Token::Uint(p1_x), Token::Uint(p1_y)]);
```

## Javascript SDK

> TODO

## C SDK

> TODO
