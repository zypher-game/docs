+++
title = "Shuffle"
description = "A zk-powered shuffle for cards"
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 1102
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "zk"
lead = 'A zk-powered shuffle for cards'
toc = true
top = false
+++

## Contents
Players take turns to encrypt and shuffle cards, resulting in a deck which is “sealed” & randomly ordered; it offers a superior solution that traditional RNGs, i.e.VRF cannot.

### mental poker
The mental poker protocol is not a single protocol, but rather a collection of sub-protocols, each responsible for a specific action.

- Key Generation:
  – Generate a public-private key pair for each player.
  – Aggregate all players’ public keys to generate a single public key.

- Masking:
  - A masking function is an ElGamal encryption function under an ag-
gregated public key.

- Shuffle and re-masking:
  – Perform shuffling operations to randomize the cards.
  – Re-mask the shuffled cards.

- Unmask:
  – Decode the cards to obtain the real card information.

## zk-shuffle as a service
