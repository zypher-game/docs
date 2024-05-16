+++
title = "SDK - Match 3"
description = "Match 3 circuit SDK."
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 53
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'Match 3 circuit SDK.'
toc = true
top = false
+++

## Environment

- Language: javascript/typescript

## Setup

- Use node package manager of choice
- Install the `@zypher-game/match3` package

## How it works

- Essentially what we are trying to accomplish is verifying the game state after a player has made some moves. Once the verifying input is set, call the `verify.verifyProof` to have the ZK circuits verify the validity of the game.

## Example

```typescript
import { verify, utils } from "match3";

const conf = {
  networkId: 5611 as verify.VerifierNetworkId,
};

export async function generateProofInput() {
  const { play, copyBoard } = utils;
  const fromBoard = utils.generateBoard();
  const seed = utils.generateSeed();
  const step = 10n;
  const stepAfter = 11n;

  let seedOut = seed;
  let boardOut = copyBoard(fromBoard);
  let scores = new Array(5).fill(0n);
  let moves = new Array(30).fill(0n).map(() => [0n, 0n, 0n]);
  moves[0] = [1n, 0n, 2n];

  ({
    seedOut,
    boardOut,
    scoresOut: scores,
  } = play(boardOut, seedOut, moves, scores));

  return {
    fromSeed: seed,
    toSeed: seedOut,
    fromBoard,
    toBoard: boardOut,
    step,
    stepAfter,
    scores,
    moves,
  };
}

async function main() {
  const input = await generateProofInput();
  await verify.verifyProof(conf.networkId, input);
}

main()
  .then(() => {
    console.log("verified successfully");
    process.exit(0);
  })
  .catch(console.error);
```

## Game Variables

### Constraints

- These are the current constraints to the game that will be removed in the future:
  - Board size: 6 \* 6 grid
  - Matching items: 5
  - Moves: has to be 30

Below are the input parameters needed in order to verify the game state. See [example](#example) for how they can be derived.

- fromSeed
  - An arbitrary bigint representing the inital state.
- toSeed
  - A bigint representing the game state after moves have been made.
- fromBoard[M][M]
  - A two dimensional array representing the state of the board. Note that the board is tilted 90 degrees clockwise so adding new items can easily be done by appending to the end of the array `fromBoard[column][row]`. Currently set to a 6 by 6 grid, with values ranging from 1 - 5 (representing each separate item to match).
- toBoard[M][M]
  - A two dimensional array (bigint) representing the state of the board after moves have been made.
- step
  - The count of the moves that a player has made up to this point of the game.
- stepAfter
  - The count of the moves that a player has made plus the amount of moves intended to make.
- scores
  - This is a bigint array (of fixed-size 5) representing the scores that a player has earned by making matches. Everytime a player has pattern matched an item, the array should add 1 to the counter at the corresponding index.
- moves[N][3]
  - A two dimensional array representing all the moves played. A move array looks like [horizontal-position, vertical-position, action]. Action value: 0 (no swap), 1 (horizontal swap), 2 (vertical swap). Currently the moves uploaded to the verifier has to be 30 in length. If less than 30 moves are played, fill the rest with `[0,0,0]`.

## Helper functions

- To simulate the the game state after moves are played, call `play`. See [example](#example).
