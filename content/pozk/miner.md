+++
title = "Become a miner"
description = "How to become a miner and earn rewards."
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 5002
sort_by = "weight"
section = "pozk"
template = "docs/page.html"

[extra]
section = "pozk"
lead = 'How to become a miner and earn rewards.'
toc = true
top = false
+++

## Introduction
Miners are responsible for generating all proofs in the network, so they need not only computing power support, but also stake guarantee. At the same time, miners will receive most of the rewards in the network.

Because there are various prover programs in the network, different programs have different hardware requirements, so even laptops and mobile phones also have the opportunity to participate in mining and get rewards. Of course, high-computing power devices will complete more prover tasks and get more opportunities.

Let's running a miner program in your devices.

## Setup
1. Install latest [docker](https://www.docker.com)
2. Download or copy [miner docker-compose template](https://github.com/zypher-game) - coming soon
```
version: '3'

services:
  pozk-miner:
    image: docker.registry.cyou/zyphernetwork/pozk-miner:0.0.11
    container_name: pozk-miner
    ports:
      - "9098:9098"
    volumes:
      - /home/cloud/tmp/pozk2:/home/ubuntu/pozk
      - /var/run/docker.sock:/var/run/docker.sock
    command: /home/ubuntu/pozk/config.toml

  pozk-frontend:
    image: docker.registry.cyou/zyphernetwork/pozk-frontend:0.0.5
    container_name: pozk-frontend
    ports:
      - "4000:4000"
    environment:
      - API_BASE_URL=http://localhost:9098/api
```
3. Edit the docker-compose.yml and config.toml

```
# host path, same as base_path if not use docker
prover_host_path = "/home/cloud/tmp/pozk2"

# use docker
base_path = "/home/ubuntu/pozk"

# not docker
# base_path = "/home/cloud/tmp/pozk2"

db_remove = false
endpoint = "https://opbnb-testnet-rpc.bnbchain.org"
open_monitor = true

[monitor_config]
task_market_address = "0x27DE7777C1c643B7F3151F7e4Bd3ba5dacc62793"
prover_market_address = "0x1c23e9F06b10f491e86b506c025080C96513C9f5"
stake_address = "0x003C1F8F552EE2463e517FDD464B929F8C0bFF06"
from = 0
delay_sec = 0
step = 10
wait_time = 10
block_number_type = "latest"
miner = "0x28B9FEAE1f3d76565AAdec86E7401E815377D9Cc"
docker_proxy_prefix = "docker.registry.cyou"

[api_config]
host = "0.0.0.0"
port = 9098
login_domain = "localhost:4000"
```

4. Run `docker compose up -d`

## Stake a prover (game)
Open browser, and visit: `https://localhost:4000`

## Install the prover

## Collect rewards
