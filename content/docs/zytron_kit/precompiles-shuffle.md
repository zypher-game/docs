+++
title = "Precompiles Shuffle"
description = "The precompiles Shuffle in zytron kit"
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 4013
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'The precompiles Shuffle in zytron kit'
toc = true
top = false
+++

## Shuffle SDK

We provide Solidity, Rust, Javascript (WebAssembly based) and C (FFI support) SDK.

## Rust Crate

| Crate Name | crates | docs.rs |
| - | - | - |
| uzkg | ![](https://img.shields.io/crates/v/uzkg) | ![](https://img.shields.io/docsrs/uzkg) |

Add dependency to Cargo.toml

```toml
zshuffle = { git = "https://github.com/zypher-game/uzkge.git" }
```

Use the following code:

```rust
    let mut rng = ChaChaRng::from_seed([0u8; 32]);

    let card_mapping = encode_cards(&mut rng);

    let alice = Keypair::generate(&mut rng);

    let keys = [alice.public].to_vec();

    // Each player should run this computation. Alternatively, it can be ran by a smart contract
    let joint_pk = aggregate_keys(&keys).unwrap();

    // Each player should run this computation and verify that all players agree on the initial deck
    let mut deck = Vec::new();
    for card in card_mapping.keys() {
        let (masked_card, masked_proof) =
            mask(&mut rng, &joint_pk, card, &ark_ed_on_bn254::Fr::one()).unwrap();
        verify_mask(&joint_pk, card, &masked_card, &masked_proof).unwrap();

        deck.push(masked_card)
    }

    let mut prover_params = gen_shuffle_prover_params(N_CARDS).unwrap();

    refresh_prover_params_public_key(&mut prover_params, &joint_pk).unwrap();

    let mut verifier_params = get_shuffle_verifier_params(N_CARDS).unwrap();
    verifier_params.verifier_params = prover_params.prover_params.verifier_params.clone();

    // Alice, start shuffling.
    let (proof, alice_shuffle_deck) =
        prove_shuffle(&mut rng, &joint_pk, &deck, &prover_params).unwrap();

    let proof = proof.to_bytes_be();

    let verifier_params = bincode::serialize(&verifier_params).unwrap();
    let deck = {
        let mut ret = Vec::new();
        for it in deck.iter() {
            let mut tmp = Vec::new();

            let (x, y) = point_to_uncompress(&it.e1);
            tmp.push(Token::Bytes(x));
            tmp.push(Token::Bytes(y));

            let (x, y) = point_to_uncompress(&it.e2);
            tmp.push(Token::Bytes(x));
            tmp.push(Token::Bytes(y));
            ret.push(Token::Array(tmp))
        }
        ret
    };

    let alice_shuffle_deck = {
        let mut ret = Vec::new();
        for it in alice_shuffle_deck.iter() {
            let mut tmp = Vec::new();

            let (x, y) = point_to_uncompress(&it.e1);
            tmp.push(Token::Bytes(x));
            tmp.push(Token::Bytes(y));

            let (x, y) = point_to_uncompress(&it.e2);
            tmp.push(Token::Bytes(x));
            tmp.push(Token::Bytes(y));
            ret.push(Token::Array(tmp))
        }
        ret
    };

    let data = ethabi::encode(&[
        Token::Bytes(verifier_params),
        Token::Array(deck),
        Token::Array(alice_shuffle_deck),
        Token::Bytes(proof),
    ]);
```

## Javascript SDK

> TODO

## C SDK

> TODO
