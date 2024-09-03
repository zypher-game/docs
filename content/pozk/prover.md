+++
title = "Build a prover (Game)"
description = "How to build a prover for game and use PoZK network."
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 5001
sort_by = "weight"
section = "pozk"
template = "docs/page.html"

[extra]
section = "pozk"
lead = 'How to build a prover for game and use PoZK network.'
toc = true
top = false
+++

## Introduction
Each prover represents a ZK prover program. A game may have one prover or multiple provers. Our network uses prover as the basic unit. Stake and reward are divided according to prover. Each prover has a different reward scale, based on the power of the prover itself, and also based on the usage of the prover.

We know that different zk schemes have different computing power requirements, and zkVMs are also significantly different from ordinary zk schemes. Therefore, for different zk provers, we have developed a set of measurement algorithms to balance the performance of different zk provers. The difference in computing power between algorithms.

| scheme      | coefficient | Game              | Circuit size / Cycle | Work      | Times (4core) | Times (96core) |
|-------------|-------------|-------------------|----------------------|-----------|---------------|----------------|
| Plonk       | 1           | Zk-shuffle        | 14094                | 14094     | 1.93s         | 0.08s          |
| Groth16     | 0.2         | 2048              | 229952               | 45990     | 6.6s          | 0.27s          |
| Groth16     | 0.2         | Crypto Rumble     | 111691               | 22338     | 1.61s         | 0.067s         |
| Groth16     | 0.2         | three!            | 1089788              | 217958    | 18.4s         | 0.79s          |
| zkEVM-RISC0 | 4           | Poker0 [1]        | 2516153              | 10064612  | 1096s         | 84.3s          |
| Plonk       | 1           | Poker0 [2]        | 871616               | 871616    | 107s          | 4.45s          |
| zkEVM-RISC0 | 4           | Alien-Cake-Addict | 4418189              | 17672756  | 2626s         | 150s           |
| zkEVM-RISC0 | 4           | Zemeroth          | 195919638            | 783678552 | 30h           | 4448s          |

## 1. Build a prover
Now you need to build a command-line prover program.

1. read input file from outside
2. parses the witness or raw data in the file
3. executes prove, and get proof and public inputs
4. serializes the proof and public inputs and stores them in the specified file

Example 2048 prover:
```rust
use input::decode_prove_input;
use std::fs::{read_to_string, write};
use zypher_circom_compat::{init_from_bytes, prove, verify, proofs_to_abi_bytes};

const WASM_BYTES: &[u8] = include_bytes!("../materials/game2048_60.wasm");
const R1CS_BYTES: &[u8] = include_bytes!("../materials/game2048_60.r1cs");
const ZKEY_BYTES: &[u8] = include_bytes!("../materials/game2048_60.zkey");

/// INPUT=test_input OUTPUT=test_output PROOF=test_proof cargo run --release
fn main() {
    let input_path = std::env::var("INPUT").expect("env INPUT missing");
    let output_path = std::env::var("OUTPUT").expect("env OUTPUT missing");
    let proof_path = std::env::var("PROOF").expect("env PROOF missing");

    let input_hex = read_to_string(input_path).expect("Unable to read input file");
    let input_bytes = hex::decode(input_hex.trim_start_matches("0x")).expect("Unable to decode input file");
    let input = decode_prove_input(&input_bytes).expect("Unable to decode input");

    init_from_bytes(WASM_BYTES, R1CS_BYTES, ZKEY_BYTES).unwrap();
    let (pi, proof) = prove(input).unwrap();
    assert!(verify(&pi, &proof).unwrap());
    let (pi_bytes, proof_bytes) = proofs_to_abi_bytes(&pi, &proof).unwrap();

    let pi_hex = format!("0x{}", hex::encode(pi_bytes));
    write(output_path, pi_hex).expect("Unable to create output file");

    let proof_hex = format!("0x{}", hex::encode(proof_bytes));
    write(proof_path, proof_hex).expect("Unable to create proof file");
}
```

## 2. Deploy to chain
After you complete the prover program, you can also write an on-chain contract to verify the prover.
Example 2048 verifier contract:
```javascript
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/utils/introspection/ERC165.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

import "./IProver.sol";
import "./IVerifier.sol";

contract Game2048Step60CircomVerifier is Initializable, OwnableUpgradeable, ERC165, IProver, IVerifier {
    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC165) returns (bool) {
        return interfaceId == type(IProver).interfaceId
            || interfaceId == type(IVerifier).interfaceId
            || super.supportsInterface(interfaceId);
    }

    function name() external view returns (string memory) {
        return "2048";
    }

    /// show how to serialize/deseriaze the inputs params
    /// e.g. "uint256,bytes32,string,bytes32[],address[],ipfs"
    function inputs() external pure returns (string memory) {
        return "uint256[]";
    }

    /// show how to serialize/deserialize the publics params
    /// e.g. "uint256,bytes32,string,bytes32[],address[],ipfs"
    function publics() external pure returns (string memory) {
        return "uint256[7]";
    }

    function verify(bytes calldata publics, bytes calldata proof) external view returns (bool) {
        uint[7] memory _pubSignals = abi.decode(publics, (uint[7]));
        (uint[2] memory _pA, uint[2][2] memory _pB, uint[2] memory _pC) = abi.decode(proof, (uint[2], uint[2][2], uint[2]));
        return this.verifyProof(_pA, _pB, _pC, _pubSignals);
    }

    function verifyProof(uint[2] calldata _pA, uint[2][2] calldata _pB, uint[2] calldata _pC, uint[7] calldata _pubSignals) public view returns (bool) {
     ...
    }
```

You can add more game logic to the contract.

After the contract is completed, deploy the contract to the blockchain where the mining network is located, and then let's make a pull request to the official repository.

## 3. Register to prover market
First, submit a register application in the PoZK network. Use this function.

```
function register(
    address prover,
    uint256 _work,
    uint256 _version,
    uint256 _overtime,
    address _verifier
);
```

And then, open a PR to PoZK repository in GitHub (coming soon), and provide the open source address of prover and the dockerfile compiled into docker image.

Example 2048 Dockerfile:
```
# Builder
FROM rust:buster AS builder
RUN update-ca-certificates
ENV CARGO_NET_GIT_FETCH_WITH_CLI=true

RUN curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash
RUN apt-get install -y git-lfs

WORKDIR /prover

RUN git clone https://github.com/zypher-network/pozk-2048.git && cd pozk-2048 && git lfs pull && cd prover && cargo update && cargo build --release && mv /prover/pozk-2048/prover/target/release/prover /prover/

# Final image
FROM debian:buster-slim

WORKDIR /prover

# Copy our build
COPY --from=builder /prover/prover .

ENTRYPOINT ["./prover"]
```

After complete the application, the official DAO will review the prover code and decide approve or reject.

## 4. Release prover
After DAO approves the prover, it will build the prover’s docker image at https://hub.docker.com/u/zyphernetwork

At this point, your game prover can be staked and minted by miners, and can be integrated to the game’s front end for players to use.

## 5. Game Integration

### 5.1 Create task on Task Market directly

First of all, you need to understand this process: players create proof task to the `TaskMarket` contract in the PoZK network, all miners will listen to the contract, when a new task appears, miners will accept the task immediately, and execute the proof, after the proof is completed, miner will submit to TaskMarket contract.

Therefore, you only need to follow the two points in the TaskMarket contract:
1. How to create a task.

```
function create(
    address prover,     // the prover address
    address player,     // the player address
    uint256 fee,        // fee for this task, it can be 0 if not want give extra fee to miner
    bytes calldata data // the raw data of this task, it uses the same format as defined in prover code.
) external returns(uint256); // return the task id
```

2. Listen the result(proof) of this task.

```
// publics and proof use the same format as defined in prover and verifier code.
event SubmitTask(uint256 id, bytes publics, bytes proof);
```

After getting the proof, you can use the proof according to the logic of the game.

### 5.2 Create task by third contracts
You can also use third-party contracts to complete these operations.
