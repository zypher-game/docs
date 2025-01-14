+++
title = "Become a miner"
description = "How to become a miner and earn rewards."
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 3201
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "mining"
lead = 'How to become a miner and earn rewards.'
toc = true
top = false
+++

## Introduction
Within the network, miners are tasked with generating all proofs, necessitating both significant computational resources and a stake commitment as collateral. This crucial role is reflected in the reward structure, with miners receiving the largest share of network rewards.

The network's architecture accommodates a wide range of proving programs, each with its own hardware specifications. This design allows for broader participation, enabling even devices such as laptops and mobile phones to contribute to mining and receive corresponding rewards. However, devices with higher processing power will be capable of completing a greater number of proving tasks, resulting in increased reward opportunities.

Begin running a miner program on your device to participate.

## Setup
1. Install latest [docker](https://www.docker.com)
2. Download or copy [miner docker-compose template](https://github.com/zypher-game)
```
networks:
  default:
    name: pozk

services:
  pozk-miner:
    image: zyphernetwork/pozk-miner:v0.1.11
    container_name: pozk-miner
    ports:
      - 9098:9098 # HTTP
      - 7364:7364 # P2P
      - 7364:7364/udp

    volumes:
      - ./:/usr/pozk
      - /var/run/docker.sock:/var/run/docker.sock
    command:
      - --network=testnet
      - --miner=0x0000000000000000000000000000000000000000 # set your miner address here
      # - --url=https://example.com # set the public domain for this miner, you will receive more tasks
```

> **IMPORTANT** change the miner account, and if you set the domain for this miner, you will receive tasks from proxy-service,
> and support multiple zk tasks (e.g. Z4).

Others `command` parameters list here, maybe you need:
```
# if you want to use a proxy to speed up docker download
- --docker-proxy=docker.unsee.tech

# if you want to change the data storage directory
- --base-path=/usr/pozk

# if you want to change the RPC endpoints
- --endpoints=https://rpc.zypher.network

# if you want to use another 0 gas server to pay gas
- --zero-gas=https://gas.zypher.network

# if you want to use more advanced configuration
- --config=/usr/pozk/config.toml
```

3. Run `docker compose up -d`

TIPS: if you use nginx, you can use the config to proxy:
```
server {
    listen 80;
    server_name xxx.com;

    location / {
        proxy_pass http://127.0.0.1:9098;
    }

    location /inner/ {
        return 400;
    }
}
```

## Stake a prover (game)
Open browser, and visit: `http://localhost:9098`

1. Connect wallet use your miner account
<img src="../miner-0.png" alt="Miner 0" width="100%"/>

2. You need to setup a local controller for mining
<img src="../miner-1.png" alt="Miner 1" width="100%"/>

3. You can choose generate new or import one
<img src="../miner-2.png" alt="Miner 2" width="100%"/>

4. You need enable the controller and submit it to the chain, make sure there is enough balance in this account if this account does not has [0 gas SBT](https://zytron.zypher.network/layer3).
<img src="../miner-3.png" alt="Miner 3" width="100%"/>

5. Now, you can view the dashboard
<img src="../miner-4.png" alt="Miner 4" width="100%"/>

## Install the prover
1. You will see the provers list in the dashboard, and your staking status.
<img src="../miner-5.png" alt="Miner 5" width="100%"/>

2. You must download the prover in local device, AND THEN you can stake the prover.

3. It will automatically start accepting orders for mining after you download the prover.

## Collect rewards

1. You can see all rewards which can collect by epoch.
<img src="../miner-6.png" alt="Miner 6" width="100%"/>

2. After you collected the rewards, it will in `collected rewards`, and after an epoch, you can make a claim.
<img src="../miner-7.png" alt="Miner 7" width="100%"/>

3. After claim, it will send to you wallet directly.
