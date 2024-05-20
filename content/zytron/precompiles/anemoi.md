+++
title = "Anemoi Hash"
description = "The precompiles Anemoi hash function in zytron kit"
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 4102
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "zytron"
lead = 'The precompiles Anemoi hash function in zytron kit'
toc = true
top = false
+++

## Anemoi Hash Function SDK

Anemoi is a zk-friendly hash function.

We provide Solidity, Rust, Javascript (WebAssembly based) and C (FFI support) SDK.

## Rust Crate

| Crate Name | crates | docs.rs |
| - | - | - |
| uzkg | ![](https://img.shields.io/crates/v/uzkg) | ![](https://img.shields.io/docsrs/uzkg) |

Add dependency to Cargo.toml

```toml
uzkge = { git = "https://github.com/zypher-game/uzkge.git" }
```

Use the following code:

```rust
let mut prng = ChaChaRng::from_seed([0u8; 32]);
let f1 = Fr::rand(&mut prng);
let f2 = Fr::rand(&mut prng);
let f3 = Fr::rand(&mut prng);

// Build inputs

let h1 = f1.into_bigint().to_bytes_be();
let h2 = f2.into_bigint().to_bytes_be();
let h3 = f3.into_bigint().to_bytes_be();

let data = ethabi::encode(&[
    Token::Array(vec![
        Token::FixedBytes(h1),
        Token::FixedBytes(h2),
        Token::FixedBytes(h3)
    ]),
]);
```

## Javascript SDK

> TODO

## C SDK

> TODO
