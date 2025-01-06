+++
title = "Based on Mining Network"
description = "A customized zk game running on Mining Network"
date = 2024-11-26T08:00:00+00:00
updated = 2024-11-26T08:00:00+00:00
draft = false
weight = 2102
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "z4"
lead = 'A customized zk game running on Mining Network'
toc = true
top = false
+++

## Introduction
Whether it is based on mining PoZK engine or custom engine, you can reuse the code of `Handler`. You only need to specify the engine type to be used when starting.

Once you have written a program based on mining netweork, the next step is [How to register it in the mining network](/mining/prover/z4/).

## Trait for PoZK
In the previous docs, you have been learned the `Handler` trait, here we will show you another two method for PoZK. That is `pozk_create` & `pozk_join`.

```rust
pub trait Handler: Send + Sized + 'static {
    type Param: Param;

    /// Viewable for game
    /// If true, when send message to all will also send to viewers
    /// If set false, no viwers
    fn viewable() -> bool;

    /// Create new room from PoZK
    async fn pozk_create(player: Player, params: Vec<u8>, rid: RoomId) -> Option<(Self, Tasks<Self>)>;

    /// New player join from PoZK
    async fn pozk_join(&mut self, player: Player, params: Vec<u8>) -> Result<HandleResult<Self::Param>>;

    /// New Viewer online if viewable is true
    async fn viewer_online(&mut self, _peer: PeerId) -> Result<HandleResult<Self::Param>> {
        Ok(HandleResult::default())
    }

    /// New Viewer offline if viewable is true
    async fn viewer_offline(&mut self, _peer: PeerId) -> Result<HandleResult<Self::Param>> {
        Ok(HandleResult::default())
    }

    /// when player online
    async fn online(&mut self, _player: PeerId) -> Result<HandleResult<Self::Param>> {
        Ok(HandleResult::default())
    }

    /// when player offline
    async fn offline(&mut self, _player: PeerId) -> Result<HandleResult<Self::Param>> {
        Ok(HandleResult::default())
    }

    /// handle message in a room
    async fn handle(&mut self, player: PeerId, param: Self::Param) -> Result<HandleResult<Self::Param>>;

    /// Generate proof for this game result, when find game is over
    async fn prove(&mut self) -> Result<(Vec<u8>, Vec<u8>)>;
}
```
In this way, you can find that only need to implement one Handler, which can run in different modes.

## Running with PoZK
When run PoZK node, you need `INPUT` env and it will supported by PoZK miner, and the code like it.

```rust
use z4_pozk::Engine;

struct MyHandler { ... }

#[async_trait::async_trait]
impl Handler for MyHandler { ... }

#[tokio::main]
async fn main() {
    Engine::<MyHandler>::run().await.expect("Down");
}
```

Next step is [How to register it in the mining network](/mining/prover/z4/).
