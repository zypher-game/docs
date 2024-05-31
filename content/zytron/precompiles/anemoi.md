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

```javascript
const wasm = require('./pkg/wasm.js');

let f1 = "0x2f8dd1f1a7583c42c4e12a44e110404c73ca6c94813f85835da4fb7bb1301d4a";
let f2 = "0x1ee678a0470a75a6eaa8fe837060498ba828a3703b311d0f77f010424afeb025";
let f3 = "0x2042a587a90c187b0a087c03e29c968b950b1db26d5c82d666905a6895790c0a";

ret = wasm.anemoi_hash([f1, f2, f3]);
console.log(ret)


```

## C SDK
```c
#include "c-uzkge.h"

int main(int argc, char *argv[])
{
    char f1_array[] = {47, 141, 209, 241, 167, 88, 60, 66, 196, 225, 42, 68, 225, 16, 64, 76, 115, 202, 108, 148, 129, 63, 133, 131, 93, 164, 251, 123, 177, 48, 29, 74};
    struct Bytes f1;
    f1.len = sizeof(f1_array);
    f1.data = f1_array;
    char f2_array[] = {30, 230, 120, 160, 71, 10, 117, 166, 234, 168, 254, 131, 112, 96, 73, 139, 168, 40, 163, 112, 59, 49, 29, 15, 119, 240, 16, 66, 74, 254, 176, 37};
    struct Bytes f2;
    f2.len = sizeof(f2_array);
    f2.data = f2_array;
    char f3_array[] = {32, 66, 165, 135, 169, 12, 24, 123, 10, 8, 124, 3, 226, 156, 150, 139, 149, 11, 29, 178, 109, 92, 130, 214, 102, 144, 90, 104, 149, 121, 12, 10};
    struct Bytes f3;
    f3.len = sizeof(f3_array);
    f3.data = f3_array;
    Bytes data[3] = {f1, f2, f3};

    char ret_val[32] = {0};
    int res = __anemoi_hash(data, sizeof(data) / sizeof(data[0]), ret_val);
    printf("hash:");
    for (int i = 0; i < sizeof(ret_val); i++)
    {
        printf("%u,", ret_val[i]);
    }
    printf("\n");
    return res;
}
、、、