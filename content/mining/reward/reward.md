+++
title = "Reward and punishment"
description = "Reward and punishment mechanism."
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 3401
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "mining"
lead = 'Reward and punishment mechanism.'
toc = true
top = false
math = true
+++

## Reward
In the mining network, a fixed amount of mining rewards will be released in each epoch, and both miners and players will receive rewards, so how are these rewards distributed?

First, because there are different games (zk provers) in the network, each prover requires different computing power and running time, so we need to make a fair distribution of the provers.
Here, the parameters include the prover's work and the staking amount. So there are two attributes of labor and assets, we adopted [Cobb–Douglas production function](https://en.wikipedia.org/wiki/Cobb%E2%80%93Douglas_production_function) when allocating the two attributes fairly.

<p>
\(W_g\) is the prover's proof of ZK work, \(W_t\) is all provers' work, \(S_g\) is the prover's staking amount, \(S_t\) is total provers' staking amount. \(a\) and \(b\) is cobb-douglas coefficients. \(n\) is epoch reward number.
</p>

$$
 \varphi = n \times (\dfrac{W_g}{W_t})^a \times (\dfrac{S_g}{S_t})^b
$$

After we have made a fair distribution among the provers, we will then distribute the miners and players within each prover.

In order to conduct macro-control between miners and players and attract more players and miners to join in, a linear ratio is adopted.
When there are fewer players, players will get more rewards, which encourages players to play more games. As more games are played, more zk tasks are created, and then rewards of miners will also increase.

<p>
Miners reward percentage \(P_m\), Miners reward amount is \( \varphi \times \dfrac{P_m}{100} \), miner maximum percentage \(p\)，miner minimum percentage \(q\)，maximum number of computed games \(t\)，minimum number is \(1\), the number of games \(x\), players reward is \(\varphi \times \dfrac{100 - P_m}{100}\):
</p>

$$
P_m = \dfrac{x - 1}{t - 1} \times (p - q) + q
$$

We adopt same distribution methods among miners and players, because there are still two parameters, one is staking and the other is labor, so we still adopt Cobb–Douglas production function.

<p>
\(W_m\) is the miner's task number, \(W_t\) is task number of this miner, \(S_m\) is the miner's staking amount, \(S_t\) is total miners' staking amount. \(c\) and \(d\) is cobb-douglas coefficients. \(m\) is prover's miner/player reward amount.
</p>

$$
 R_m = m \times (\dfrac{W_m}{W_t})^c \times (\dfrac{S_m}{S_t})^d
$$

In this way, we distribute all rewards fairly.

## Pubishment
During the process of the network, because it is completely decentralized, some disputes may arise, such as a zk task not being completed on time, an abnormality in a zk service, etc.
In these cases, for the fairness, we have added a punishment and supervision mechanism.

Both players and miners can submit applications to explain the problems of the other party. After the intervention and review by DAO, it will be processed.

## Dispute
For miners' mistakes, part of the staking amount will be slashed and rewarded to players.

For players' mistakes, miners will directly receive rewards without submitting zk task proofs.
