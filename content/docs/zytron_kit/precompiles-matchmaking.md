+++
title = "Precompiles Matchmaking"
description = "The precompiles Matchmaking in zytron kit"
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 4014
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'The precompiles Matchmaking in zytron kit'
toc = true
top = false
+++

## Matchmaking SDK

We provide Rust, Javascript (WebAssembly based) and C (FFI support) SDK.

## Rust Crate

| Crate Name | crates | docs.rs |
| - | - | - |
| zmatchmaking | --- | ![](https://img.shields.io/docsrs/zmatchmaking) |

Add dependency to Cargo.toml

```toml
zmatchmaking = { git = "https://github.com/zypher-game/uzkge.git" }
```

Use the following code:

```rust
let mut rng = ChaChaRng::from_entropy();

let inputs = (1..=N)
    .into_iter()
    .map(|i| Fr::from(i as u64))
    .collect::<Vec<_>>();

let committed_seed = Fr::rand(&mut rng);

let committment = AnemoiJive254::eval_variable_length_hash(&[committed_seed]);

let random_number = Fr::rand(&mut rng);

let (proof, outputs) = prove_matchmaking(
    &mut rng,
    &inputs,
    &committed_seed,
    &random_number,
    &gen_prover_params().unwrap(),
)
.unwrap();

let verifier_params = bincode::serialize(&get_verifier_params().unwrap()).unwrap();

let inputs = inputs
    .iter()
    .map(|v| Token::Bytes(v.into_bigint().to_bytes_be()))
    .collect();

let outputs = outputs
    .iter()
    .map(|v| Token::Bytes(v.into_bigint().to_bytes_be()))
    .collect();

let committment = committment.into_bigint().to_bytes_be();

let random_number = random_number.into_bigint().to_bytes_be();

let proof = bincode::serialize(&proof).unwrap();

let data = ethabi::encode(&[
    Token::Bytes(verifier_params),
    Token::Array(inputs),
    Token::Array(outputs),
    Token::Bytes(committment),
    Token::Bytes(random_number),
    Token::Bytes(proof),
]);
```

## Javascript SDK

> TODO

## C SDK

> TODO
