+++
title = "What's Secret Engine"
description = "A zero Knowledge Secret Engine."
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 1101
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "zk"
lead = 'A zero Knowledge Secret Engine.'
toc = true
top = false
+++

## What's is UZKGE ?
It is recognized that every zero-knowledge proof system consists of two components:
the first is arithmetization, and the second is a polynomial commitment scheme.

The initial stage in transforming a program into a Zero-Knowledge Proof (ZKP) involves arithmetization.
This process elucidates the relationship by establishing specific conditions among these polynomials.
Our zero-knowledge proof system is implemented based on the standard PlonK recipe with highly customized gates, which mainly include:

- Gadgets Supports varied gadgets utilized in game circuit development, including basic arithmetization circuits, hash, ecc, zkshuffle, zkmatchmaking and others.
We recommend visiting our GitHub repo for more information on these gadgets.

- On-chain Verifier Optimized PLONK/ZKVM for both prover and verifier, along with support for common solidity verifiers for all EVM chains.

- App-specific Plonk Using app-specific plonk as the basic scheme for ZK proof, with various gadgets provided by our SDK to write specific game circuits. 
At the same time, we also provide verification contracts on different virtual machines (EVM/WASM/. . . ),
which can run in different blockchain systems and realize off-chain proof and on-chain verification.

UZKGE open souce [here](https://github.com/zypher-game/uzkge)

## What's is Secret Engine ?
Use zk to keep information in the game private on the chain to ensure the fairness of the game and hide player information.

We developed a zk scheme and implemented a series of secret SDKs commonly used in games based on it. We call all of this toolkit secret engine.
