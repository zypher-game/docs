+++
title = "What's AW Engine"
description = "ZK aggregate world engine."
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 1201
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "zk"
lead = 'ZK aggregate world engine.'
toc = true
top = false
+++

## Contents
The Aggregate World Engine is a blockchain scaling solution designed for the real-time and cohesive demands of gaming.

we have developed an AW engine that not only addresses scalability issues but also supports real-time multiplayer online gaming.
Firstly, by utilizing zero-knowledge proofs to bundle game actions, they are uploaded to the chain in one go and verified.
This allows players to take more actions and play for longer periods before interacting with the blockchain,
thereby reducing transaction costs and congestion on the main blockchain.

For example, in turn-based games, players can make multiple moves, allowing multiple moves to generate a proof. Formally, we have:

```bash
Xn = F(F(F(F(F(...F(x0))))))
```

Alternatively, denoted as `X_n = F(n)(X_0)`, where `X_0` is the input, `X_n` is the output,

`F` is the turn function, and `n` is the number of moves. Or more precisely

```bash
X_i+1 = F(X_i)

where i = {0, 1, ..., n − 1}
```

The most straightforward way to verify such iterative computation is to throw n moves into a separate circuit and then use SNARK to complete the proof,
significantly increasing the number of moves that can be placed within a single transaction.
This opens up the possibility for more types of games to be developed into fully on-chain games.

Here, we take single-player (SDKs) and multiplayer games(Z4) as examples.
