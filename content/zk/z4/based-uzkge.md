+++
title = "Customized Node"
description = "A customized zk & room for game."
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 3001
sort_by = "weight"
template = "docs/page.html"

[extra]
section = "zk"
lead = 'A customized zk & room for game'
toc = true
top = false
+++

## Contents
In z4 engine, developers only need to implement an interface and configure some startup parameters, and then will get a complete z4 game node.

### Handler

`Handler` trait is the core interface. Developers can implement the game logic in it, and then they can directly use the z4 engine to run it. This interface including `accept` room events from the chain, `create` room events when the room is accepted by this node, and also supporting events when nodes go online and offline (connected and disconnected with websocket), and finally how to handle user request events in general.

```rust
pub trait Handler: Send + Sized + 'static {
    type Param: Param;

    /// accept params when submit to chain
    async fn accept(peers: &[(Address, PeerId, [u8; 32])]) -> Vec<u8>;

    /// create new room scan from chain
    async fn create(peers: &[(Address, PeerId, [u8; 32])], params: Vec<u8>, rid: RoomId) -> (Self, Tasks<Self>);

    /// when player online
    async fn online(&mut self, _player: PeerId) -> Result<HandleResult<Self::Param>> {
        Ok(HandleResult::default())
    }

    /// when player offline
    async fn offline(&mut self, _player: PeerId) -> Result<HandleResult<Self::Param>> {
        Ok(HandleResult::default())
    }

    /// handle message in a room
    async fn handle(&mut self, player: PeerId, method: &str, param: Self::Param) -> Result<HandleResult<Self::Param>>;
}
```

Let's explain the parameters and methods one by one.

#### Param
Firstly, `type Param: Param`, this method defines the type which the data in the network enters the `handle`. The specific `Param` trait is defined as follows:

```rust
/// serialize & deserialize for params
pub trait Param: Sized + Send + Default {
    fn to_value(self) -> Value;

    fn from_value(value: Value) -> Result<Self>;

    fn to_bytes(&self) -> Vec<u8>;

    fn from_bytes(bytes: &[u8]) -> Result<Self>;
}
```

It defines how to convert to and from `serde_json::Value`, and also how to convert to and from `Vec<u8>`. We provide a default params, as follows:

```rust
pub struct DefaultParams(pub Vec<Value>);
```

If developers don't want to define it specially, can directly use the `DefaultParams` structure.

#### accept
When z4 node finds that a room on the chain is in an acceptable state, it will actively call the accept method and pass in the players information as a parameter. The return value is what is carried when the accept transaction is submitted to the chain. The content of this information is completely defined by the developer based on game logic, and can also be empty.

The player's information includes: the player's eth address, the player's locally generated temporary account address and the public key of the temporary account.

#### create
When the z4 node finds that it has successfully accepted a room on the chain, it will call the create method to create the room. Parameters include: room players information, data submitted when accepting, room id, and the return value is the room structure and the scheduled task for the room. Regarding the room definition tasks, the following will explain in detail how to define scheduled tasks.

#### online and offline
When z4 node discovers that a player has connected, it will call the online method. Developers can handle the logic of the player's first online connection or reconnection, such as synchronizing the latest status of the room, notifying others that the player is online, etc.

When z4 node finds that a player has disconnected, it will call the offline method. Developers can handle the player's disconnection status.

#### handle
When the z4 node receives a specific interaction request from the player, it will call the handle method. Developers can specifically define the game processing logic corresponding to different methods.

It should be noted that online, offline, and handle all using the temporary account generated locally by the player, not the player's eth account.

The interaction between the player and z4 node adopts the jsonrpc format. Therefore, the parameters are method and param, and the return value is also two method and param. z4 will help handle others.

### Tasks
Game will have some scheduled tasks, including countdown and recurring tasks, can be defined as `Task` and added to the z4 engine. The z4 engine will automatically execute various scheduled tasks in the game.

```rust
pub trait Task: Send + Sync {
    type H: Handler;

    fn timer(&self) -> u64;

    async fn run(&mut self, state: &mut Self::H) -> Result<HandleResult<<Self::H as Handler>::Param>>;
}
```

Developers need to specify `Handler`, which is the core interface defined above. Each timer is set, which supports both fixed numbers and numbers that change according to status. The core `run` structure determines how to run the scheduled task. This task will affect the state of the core structure `Handler` and return a general `HandleResult`.

We also encapsulate a more friendly format
```rust
pub type Tasks<H> = Vec<Box<dyn Task<H = H>>>;
```
Therefore, if there are no scheduled tasks in the game, developers can directly give an empty array.

### HandleResult
This is a common return value of z4 engine. Z4 will perform different network transfers and status processing according to different fields in the return value, including sending messages to all players in the current room and sending messages to a specific player in the room. When the game ends, subsequent certification and on-chain operations are automatically performed.

This is a common return value of z4 engine. Z4 will perform different network transmission and status processing according to different fields in the return value, including sending messages to all players in the current room and sending messages to a specific player in the room. When the game over, will execute the verify and submit result to chain automatically.

```rust
pub struct HandleResult<P: Param> {
    /// need broadcast the msg in the room
    all: Vec<(String, P)>,
    /// need send to someone msg
    one: Vec<(PeerId, String, P)>,
    /// when game over, need prove the operations & states
    over: Option<(Vec<u8>, Vec<u8>)>,
}
```

Supports three methods, corresponding to different return value types. `add_all`, `add_one` and `over`.

### Run on Z4 !
Run the defined `Handler` to get a complete z4 node, it's simple!

```rust
use z4_engine::{Config, Engine};

struct MyHandler { ... }

#[async_trait::async_trait]
impl Handler for MyHandler { ... }

#[tokio::main]
async fn main() {
    dotenv::dotenv().ok();

    let network = std::env::var("NETWORK").unwrap();
    let game = std::env::var("GAME").unwrap();
    let secret_key = std::env::var("SECRET_KEY").unwrap();
    let start_block = std::env::var("START_BLOCK").ok().map(|v| v.parse().unwrap());
    let server = std::env::var("PUBLIC_SERVER").unwrap();
    let http_port = std::env::var("HTTP_PORT").unwrap_or("8080".to_owned()).parse().unwrap();
    let ws_port = std::env::var("WS_PORT").unwrap_or("8000".to_owned()).parse().unwrap();

    let mut config = Config::default();
    config.http_port = http_port;
    config.ws_port = Some(ws_port);
    config.secret_key = secret_key.to_owned();
    config.chain_network = network;
    config.chain_start_block = start_block;
    config.games = vec![game.to_owned()];
    config.auto_stake = true;
    config.http = server;

    Engine::<MyHandler>::init(config).run().await.expect("Down");
}
```

Great, this is all about using z4 engine to develop a z4 node. For more information, you can refer to the game cases and API documentation.