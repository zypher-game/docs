+++
title = "Reward and punishment"
description = "Reward Allocation Mechanism"
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 3401
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "mining"
lead = 'Reward Allocation Mechanism'
toc = true
top = false
math = true
+++

## Reward
Reward Allocation Mechanism in the Zypher Mining Network

A fixed amount of mining rewards is released during each epoch in the Zypher Mining Network, distributed between miners and players. A key challenge is ensuring fair distribution among the diverse games (ZK provers) operating within the network, as each prover exhibits varying computational demands and execution times.

To address this challenge, we consider two primary factors:

- **Prover Workload**: This metric quantifies the computational effort required by a specific ZK prover, encompassing factors like processing time and resources consumed.
- **Staked Amount**: This represents the amount of tokens staked by participants contributing to a specific prover, reflecting their commitment and investment.

By drawing an analogy between prover workload and labor, and staked amount and capital, we employ the [Cobb-Douglas production function](https://en.wikipedia.org/wiki/Cobb%E2%80%93Douglas_production_function) to achieve a balanced and equitable distribution of rewards. The Cobb-Douglas function, widely used in economics to model production output based on labor and capital inputs, allows us to proportionally allocate rewards based on both the prover's contribution (workload) and the stake backing it (capital). This approach ensures that both computational effort and financial commitment are appropriately recognized and rewarded.

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
As the network operates in a fully decentralized environment, the potential for disputes exists. These may include instances of delayed ZK task completion, ZK service abnormalities, or other operational issues.
To maintain fairness and integrity, a robust dispute resolution system, incorporating both penalties and oversight, has been established.

This system allows both players and miners to formally submit applications outlining the nature of the dispute. These applications are then subject to review and resolution by the DAO.

## Dispute
For miners' mistakes, part of the staking amount will be slashed and rewarded to players.

For players' mistakes, miners will directly receive rewards without submitting zk task proofs.
