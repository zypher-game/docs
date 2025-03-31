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
```javascript
const wasm = require('./pkg/wasm.js');

let x1 = "0x0d52c3aa573af39845660735de0d3d9efb481a112cf00623ab22546122d4e16a";
let y1 = "0x0e7e20b3cb30785b64cd6972e2ddf919db64d03d6cf01456243c5ef2fb766a65";
let x2 = "0x242cbada3ae8d6e90056e73e4941eeccee72cb99945a194f754205b3678bd769";
let y2 = "0x2d7690deeaa77c9d89b0ceb3c25f7bb09c44f40b4b8cf5d6fcb512c7be8fcba9";

ret = wasm.point_add(x1, y1, x2, y2);
console.log(ret)

let s = "0x008d7a42a4dde1d8f8bcacddcae9bc78b1480eb547d4a490d9cfa5c268a076c7";
let x = "0x1738fd301654d891e32235d03a64b7ebe0c3f37df67db0b798f2664783b1bac9";
let y = "0x22a689a1c0aebf70ceee76fe7891729002e072ceb7ba94a32b1fce79f8c009d9";

ret = wasm.scalar_mul(s, x, y);
console.log(ret)
```
## C SDK
```c
#include "c-uzkge.h"

int point_add()
{
    char x1[] = {13, 82, 195, 170, 87, 58, 243, 152, 69, 102, 7, 53, 222, 13, 61, 158, 251, 72, 26, 17, 44, 240, 6, 35, 171, 34, 84, 97, 34, 212, 225, 106};
    char y1[] = {14, 126, 32, 179, 203, 48, 120, 91, 100, 205, 105, 114, 226, 221, 249, 25, 219, 100, 208, 61, 108, 240, 20, 86, 36, 60, 94, 242, 251, 118, 106, 101};
    char x2[] = {36, 44, 186, 218, 58, 232, 214, 233, 0, 86, 231, 62, 73, 65, 238, 204, 238, 114, 203, 153, 148, 90, 25, 79, 117, 66, 5, 179, 103, 139, 215, 105};
    char y2[] = {45, 118, 144, 222, 234, 167, 124, 157, 137, 176, 206, 179, 194, 95, 123, 176, 156, 68, 244, 11, 75, 140, 245, 214, 252, 181, 18, 199, 190, 143, 203, 169};
    char ret_val[32] = {0};
    int res = __point_add(x1, x2, y1, y2, ret_val);
    printf("hash:");
    for (int i = 0; i < sizeof(ret_val); i++)
    {
        printf("%u,", ret_val[i]);
    }
    printf("\n");
    return res;
}

int scalar_mul()
{
    char s[] = {0, 141, 122, 66, 164, 221, 225, 216, 248, 188, 172, 221, 202, 233, 188, 120, 177, 72, 14, 181, 71, 212, 164, 144, 217, 207, 165, 194, 104, 160, 118, 199};
    char x[] = {23, 56, 253, 48, 22, 84, 216, 145, 227, 34, 53, 208, 58, 100, 183, 235, 224, 195, 243, 125, 246, 125, 176, 183, 152, 242, 102, 71, 131, 177, 186, 201};
    char y[] = {34, 166, 137, 161, 192, 174, 191, 112, 206, 238, 118, 254, 120, 145, 114, 144, 2, 224, 114, 206, 183, 186, 148, 163, 43, 31, 206, 121, 248, 192, 9, 217};
    char ret_val[32] = {0};
    int res = __scalar_mul(s, x, y, ret_val);
    printf("hash:");
    for (int i = 0; i < sizeof(ret_val); i++)
    {
        printf("%u,", ret_val[i]);
    }
    printf("\n");
    return res;
}

int main(int argc, char *argv[])
{
    printf("point_add:%d\n", point_add());
    printf("scalar_mul:%d\n", scalar_mul());
}
```
