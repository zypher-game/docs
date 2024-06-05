+++
title = "Workshop - Z4 & ZK games"
description = "Z4 is a multiplayer online real-time game development framework. This article will introduce how to use the z4 framework through a specific small game"
date = 2024-05-21T09:19:42+00:00
updated = 2024-05-21T09:19:42+00:00
draft = false
template = "blog/page.html"

[taxonomies]
authors = ["Zypher Dev"]

[extra]
toc = true
lead = "Z4 is a multiplayer online real-time game development framework. This article will introduce how to use the z4 framework through a specific small game"
+++

[Here](../../zk/z4/overview) is more Z4 introduction.

Z4 is suitable for low-latency, high-performance games, such as FPS. The core idea of ​​Z4 is off-chain games, zk proofs, and on-chain verification.

Using the Z4 development framework, developers do not need smart contracts, no on-chain and off-chain interactions, and no network communication between the server and the client.
They only need to design and implement the core logic of the game and the game client to get a high-performance, low-latency game on web3.
Players do not need to pay gas, there is no wallet interaction, no block time, and a truly smooth gaming experience.

Next, take a Z4 tour, we will build a shoot game with Z4!

## Step 0: Setup coding environment
Z4 is develped in Rust, so we need install rust and cargo firstly. It is recommended to use [rustup](https://rustup.rs).

```bash
# install rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# use stable rustc
rustup default stable

# install z4 command
cargo install z4

# generate new game with z4 command
z4 new --name my-game

cd my-game
```

In the demo project, we will see z4 in dependency.
`z4-engine = "0.1"`

## Step 1: Deploy standard Z4 game contract
We have two options:
- we can use the standard game contract that comes with Z4 and deploy it directly through scripts.
```bash
# download the z4 code
git clone git@github.com:zypher-game/z4.git

cd z4/contracts/solidity

npm install

# deploy stardard game contract
npm run deploy
```
Then, we will get the game contract address in terminal output.

- we can also do some custom development based on the Z4 standard game contract.
Contract is [here](https://github.com/zypher-game/z4/blob/main/contracts/solidity/contracts/SimpleGame.sol).
And now the default contracts will at `contracts/` and only the solidity files, because solidity is default, you can choose other smart contract template when new a game project.

```solidity
contract SimpleGame is RoomMarket {
    ...
}
```

## Step 2: Game logic with Z4 trait
Firstly, we defined the Shoot game basic structs. The player has hp and bullet. and everyone has 5 hp and 5 bullet when starting.

```rust
#[derive(Default)]
pub struct ShootPlayer {
    // Hit Points or Health Points
    hp: u32,
    bullet: u32,
}

#[derive(Default)]
pub struct ShootHandler {
    accounts: HashMap<PeerId, (Address, ShootPlayer)>,
    operations: Vec<ShootOperation>,
}
```

As you see, we also collected all operations when happened in the game for do zkp.

Next, we can implement the main trait provided by Z4 engine.
```rust
#[async_trait::async_trait]
impl Handler for ShootHandler {
    type Param = DefaultParams;

    // when z4 accept the room task, no need other information when submit transaction to chain.
    async fn accept(_peers: &[(Address, PeerId, [u8; 32])]) -> Vec<u8> {
        vec![]
    }

    // create new game room
    async fn create(
        peers: &[(Address, PeerId, [u8; 32])],
        _params: Vec<u8>,
        _rid: RoomId,
        _seed: [u8; 32],
    ) -> (Self, Tasks<Self>) {
        // here we create ShootHandler
    }

    // we donnot override the online and offline function

    // main handle player's message
    async fn handle(
        &mut self,
        player: PeerId,
        method: &str,
        params: DefaultParams,
    ) -> Result<HandleResult<Self::Param>> {
        // here we handle every message from player, now, only handle `shoot` operation.
        if method == "shoot" {
            // reduce players bullet
            // reduce shooted players hp
            // check if game is over: only one player alive or no bullet
            // if game over, generate zkp
            // broadcast the events

            self.operations.push(ShootOperation { from, to });
        }
    }
}
```

## Step 3: ZK game logic
Here we use the standard plonk scheme and cs provided by zypher to write zk circuits, because it has complete on-chain support. Of course, it also supports any other zk schemes to write logic circuits. The detailed circuit code can be found in `z4/engine/examples`.

```rust
if game_over {
    // zkp
    let players: Vec<Address> = self
        .accounts
        .iter()
        .map(|(_, (account, _))| *account)
        .collect();

    let mut prng = ChaChaRng::from_seed([0u8; 32]); // this rng will fetch from chain when start.
    let prover_params = gen_prover_params(&players, &self.operations).unwrap();
    println!("SERVER: zk key ok, op: {}", self.operations.len());

    let (proof, results) = prove_shoot(&mut prng, &prover_params, &players, &self.operations).unwrap();
    println!("SERVER: zk prove ok, op: {}", self.operations.len());
    let verifier_params = get_verifier_params(prover_params);
    verify_shoot(&verifier_params, &results, &proof).unwrap();
    println!("SERVER: zk verify ok, op: {}", self.operations.len());

    // results serialize to bytes
    let proof_bytes = bincode::serialize(&proof).unwrap();
    let rank = simple_game_result(&players);
    result.over(rank, proof_bytes);
}

//////// ------- ZK -------- ////

#[derive(Clone)]
pub struct ShootOperation {
    pub from: Address,
    pub to: Address,
}

pub struct ShootResult {
    pub player: Fr,
    pub hp: u32,
    pub bullet: u32,
}

pub struct ShootResultVar {
    pub p: VarIndex,
    pub hp: VarIndex,
    pub bullet: VarIndex,
    pub r_hp: u32,
    pub r_bullet: u32,
}

pub const TOTAL: u32 = 5;
const PLONK_PROOF_TRANSCRIPT: &[u8] = b"Plonk shoot Proof";
const N_TRANSCRIPT: &[u8] = b"Number of players";

pub(crate) fn build_cs(
    players: &[Address],
    inputs: &[ShootOperation],
) -> (TurboCS<Fr>, Vec<ShootResult>) {
    let mut cs = TurboCS::new();
    // build shoot cs with opeartions and output result
}

```

After we finished writing the circuit, we found that it is actually very difficult to write zk circuits manually, which is not conducive to game developers to develop complex game logic. So, we provide a set of support for zkvm, including usage and integration, and also provide on-chain verification contracts for common zkvm, such as risc-v zkvm of risc0, so that developers do not need to worry about and deal with any zk knowledge. We will use games to demonstrate it later.

## Step 4: Running Z4 node

After we have completed the logic of the game, we need to run it. Configure some parameters in `.env` file, if not exists, copy `.env-template`.
in .env:
```
NETWORK=localhost
GAMES=0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512
SECRET_KEY=0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 # Hardhat default sk!
# START_BLOCK=1 # if not set, will sync from latest block
# RPC_ENDPOINTS=https://xxx.com,https://xxxx.com
# URL_HTTP=http://127.0.0.1:8080
# URL_WEBSOCKET=ws://127.0.0.1:8000
# HTTP_PORT=8080
# WS_PORT=8000
# P2P_PORT=7364
# AUTO_STAKE=true # if true, will stake when starting with URL_HTTP & URL_WEBSOCKET
# ROOM_MARKET=0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512 # if game and room market not the same contract
# RUST_LOG=info
```
we need change games contract address, z4 node account secret key, and other configures. Next we run z4!

```rust
let config = Config::from_env().unwrap();
Engine::<ShootHandler>::init(config).run().await.expect("Down");
```

In this way, a completed z4 node will be running. When a new `room` created on the chain and is ready, the node will automatically accept the order and inform the players. The player program will automatically connect and play the game. When the game is over, the proof and result will be automatically submitted to the chain by the z4 node, and the whole process will be guaranteed by zk for security.

## Step 5: Game front-end with Z4 node

The game front end will do two functions.

One is when the player clicks `create room` and `join room`, tx is sent to the chain and listening the status of the room. When the room is accepted, a websocket connection is made according to the z4 node information.

The other is to interact with the z4 node for a websocket connection. The interaction process uses the jsonrpc format. After the connection is completed, the `connect` method need be called. The parameters of this function are formulated according to the game `online` logic. If the online method of the `Handler` is not implemented, it is empty, and then the game interaction is carried out.

> Tip: If you want to get a list of all currently created and ongoing rooms, you can get it from any z4 node through the room_market method.

## Step 6: Play and more ecological libraries
Now we know how to develop a blockchain-based decentralized game using the z4 framework. We will continue to add more component integrations and development kits.

We have currently implemented the front-end, back-end and on-chain game development in the rust environment. For the front-end, we provide the [bevy-web3](https://github.com/zypher-game/bevy-web3) and [z4-bevy](https://github.com/zypher-game/z4-bevy) libraries, which encapsulate the [BevyEngine](https://bevyengine.org).

For more game engine and front-end toolkits, please check z4 documentation.

