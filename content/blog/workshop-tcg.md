+++
title = "Workshop - TCG"
description = "TCG is a multiplayer, multi-turn card game. This article will introduce the process of using ZK to implement it. At the same time, it will also introduce the design and implementation of web3.0 open games"
date = 2024-05-20T09:19:42+00:00
updated = 2024-05-20T09:19:42+00:00
draft = false
template = "blog/page.html"

[taxonomies]
authors = ["Zypher Dev"]

[extra]
toc = true
lead = "TCG is a multiplayer, multi-turn card game. This article will introduce the process of using ZK to implement it. At the same time, it will also introduce the design and implementation of web3.0 open games."
+++

## Step 0: Set up coding environment

```bash
$ npm init

$ npm i --save-dev hardhat
# "hardhat": "^2.22.3"

$ npx hardhat init
# ❯ Create a TypeScript project
#
# We are using TypeScript with Ethers.js here. you can choose whatever project structure you preferred.
```

Next, we will add the Shuffle SDK provided by Zypher, and the NFT interfaces provided by OpenZepplin.

```bash
$ npm i @zypher-game/secret-engine @openzeppelin/contracts
# "@openzeppelin/contracts": "^5.0.2"
# "@zypher-game/secret-engine": "^0.2.0"
```

The environment preparation is now complete.

## Step 1: NFT & Player's Deck

Our goal is to use ERC-721 specification NFT to build our card game. First, we need to declare the NFT contracts:

```solidity
contract Game {
    IERC721 public cardSet;

    constructor(address _card) {
        cardSet = IERC721(nft);
    }
}
```

Next, we will provide a basic interface for players to query, add, and remove their own decks.
At this stage, we will not check the card holder and let the player edit it at will. We will confirm it when the game starts:

```solidity
    mapping(address => uint256[]) private _decks;

    function getDeck(address deckOwner) external view returns (uint256[] memory) {
        return _decks[deckOwner];
    }

    error DuplicateCard(uint256 cardId);
    function addCard(uint256 cardId) external {
        if (_findCardIndex(msg.sender, cardId) != CARD_NOT_FOUND) {
            revert DuplicateCard(cardId);
        }

        _decks[msg.sender].push(cardId);
    }

    error CardNotFound(uint256 cardId);
    function removeCard(uint256 cardId) internal {
        uint256[] storage deck = _decks[msg.sender];
        uint256 index = _findCardIndex(msg.sender, cardId);

        if (index >= deck.length) {
            revert CardNotFound(cardId);
        }

        deck[index] = deck[deck.length - 1];
        deck.pop();
    }

    function _findCardIndex(
        address deckOwner,
        uint256 cardId
    ) internal view returns (uint256) {
        uint256[] storage deck = _decks[deckOwner];

        for (uint256 i = 0; i < deck.length; i++) {
            if (deck[i] == cardId) {
                return i;
            }
        }

        return CARD_NOT_FOUND;
    }
```

Finally, through basic testing, we ensure that this step functions as expected:

```ts
  it('Deck management', async () => {
    const { game, p1, p2 } = await loadFixture(deployGame)

    expect(await game.getDeck(p1.address)).to.be.empty

    await game.connect(p1).addCard(123)
    await game.connect(p1).addCard(456)
    await game.connect(p1).addCard(789)

    expect(await game.getDeck(p1.address)).to.deep.eq([123, 456, 789])

    await expect(game.connect(p1).addCard(123)).to.be.revertedWithCustomError(game, 'DuplicateCard')

    await game.connect(p1).removeCard(456)

    expect(await game.getDeck(p1.address)).to.deep.eq([123, 789])

    await expect(game.connect(p1).removeCard(456)).to.be.revertedWithCustomError(game, 'CardNotFound')
  })
```

At this point, our Game contract allows players to edit the deck they use for battle.

If you want the game to have a more user-friendly editing interface, or even several backup decks, you are welcome to extend it and create more powerful functions!

## Step 2: Validate Player's Deck and Start a Duel

After players get their decks ready, a battle begins.

Before joining a battle, we want to make sure the player's deck complies with our rules.
In this example, our rules are that players must build a deck with a total of 20 cards, and each card must belong to the player:

```solidity
    uint256 private constant VALID_DECK_SIZE = 20;
    function isValidDeck(address deckOwner) public view returns (bool) {
        uint256[] storage deck = _decks[deckOwner];

        if (deck.length != VALID_DECK_SIZE) {
            return false;
        }

        for (uint256 i = 0; i < deck.length; i++) {
            if (cardSet.ownerOf(deck[i]) != deckOwner) {
                return false;
            }
        }

        return true;
    }
```

In order to verify this function, we simply write an ERC-721 contract with batch mint function to facilitate our testing.

```solidity
    function batchMint(address to, uint8 amount) external {
        for (uint8 i = 0; i < amount; i++) {
            _safeMint(to, ++_tokenIdCounter);
        }
    }
```

We first replace the blank card contract in the previous test with this NFT, and provide 50 cards each to two players at the beginning:

```ts
  const nft = await ethers.deployContract('GameCard', []) as GameCard

  await nft.batchMint(p1.address, 50)
  await nft.batchMint(p2.address, 50)
```

Then we can check whether the player's deck complies with the rules:

```ts
  it('Deck validation', async () => {
    const { game, nft, p1, p2 } = await loadFixture(deployGame)

    for (const index of new Array(19).keys()) {
      await game.connect(p1).addCard(index + 1) // Add cards 1-19
    }

    // Invalid: less than 20 cards
    expect(await game.isValidDeck(p1.address)).to.be.false

    await game.connect(p1).addCard(20)

    expect(await game.isValidDeck(p1.address)).to.be.true

    await game.connect(p1).addCard(21)

    // Invalid: more than 20 cards
    expect(await game.isValidDeck(p1.address)).to.be.false

    await game.connect(p1).removeCard(21)

    // Valid again
    expect(await game.isValidDeck(p1.address)).to.be.true

    await nft.connect(p1).transferFrom(p1.address, p2.address, 20)

    // Invalid: not owned every card in the deck
    expect(await game.isValidDeck(p1.address)).to.be.false
  })
```

After ensuring that the functionality of our deck verification meets expectations, we can allow players to enter the game.

First, we first define a basic battle data structure, as well as its basic query and refresh methods:

```solidity
    enum DuelState {
        None,
        // We'll define more states later:
        __UNUSED_1__,
        __UNUSED_2__
    }
    struct Duel {
        DuelState state;
        address player1;
        address player2;
    }

    Duel private _duel;
    function duel() external view returns (Duel memory) {
        return _duel;
    }

    event DuelReset();
    function newDuel() external {
        delete _duel;
        emit DuelReset();
    }

    function _nextDuelState() internal returns (DuelState) {
        _duel.state = DuelState(uint256(_duel.state) + 1);

        return _duel.state;
    }
```

We will first define a single group of battles here. Of course, you can rewrite it into more groups of battles later.

Next, we allow players to join the battle and make sure that their decks have been ready when they join:

```solidity
    error InvalidDeck(address player);
    error InvalidDuelState(DuelState current, DuelState expected);
    error DuelAlreadyFull();
    event DuelStarted(address player1, address player2);
    function joinDuel() external {
        address player = msg.sender;

        if (!isValidDeck(player)) {
            revert InvalidDeck(player);
        }

        if (_duel.state != DuelState.None) {
            revert InvalidDuelState(_duel.state, DuelState.None);
        }

        if (_duel.player1 == address(0)) {
            _duel.player1 = player;
        } else if (_duel.player2 == address(0)) {
            _duel.player2 = player;

            _nextDuelState();
            emit DuelStarted(_duel.player1, _duel.player2);
        } else {
            revert DuelAlreadyFull();
        }
    }
```

Here we let the battle start automatically after the players arrive, and then we can test this part of the function:

```ts
  it('Start and reset a duel', async () => {
    const { game, p1, p2 } = await loadFixture(deployGame)

    for (const index of new Array(20).keys()) {
      await game.connect(p1).addCard(index + 1)
      await game.connect(p2).addCard(index + 51)
    }

    expect((await game.duel()).player1).eq(ethers.ZeroAddress)
    expect((await game.duel()).player2).eq(ethers.ZeroAddress)

    await game.connect(p1).joinDuel()

    expect((await game.duel()).player1).eq(p1.address)
    expect((await game.duel()).player2).eq(ethers.ZeroAddress)

    await expect(game.connect(p2).joinDuel()).to.emit(game, 'DuelStarted')

    expect((await game.duel()).player1).eq(p1.address)
    expect((await game.duel()).player2).eq(p2.address)
    expect((await game.duel()).state).eq(1)

    await expect(game.newDuel()).to.emit(game, 'DuelReset')

    expect((await game.duel()).player1).eq(ethers.ZeroAddress)
    expect((await game.duel()).player2).eq(ethers.ZeroAddress)
    expect((await game.duel()).state).eq(0)
  })
```

At this point, our game can start and reset correctly, and ensure that players' hands comply with the rules when players join.

> [!TIP]
> You may have noticed that according to the current design, players can trade cards to others after joining.
> To avoid this, More verifications can be added in the game, or after the player get the deck ready,
> the cards must be deposited to the game contract.
> These options will increase gas and player operation complexity.
> You can evaluate them and add them to your work appropriately.

## Step 3: Shuffle Player's Deck

Now we're going to add Zypher Shuffle feature,

First, we first add the Verifiers corresponding interface and contract address provided by Zypher to our Game contract:

```solidity
import {
    ZgShuffleVerifier,
    ZgRevealVerifier,
    Point
} from "@zypher-game/secret-engine/Verifiers.sol";

contract Game {
    // ...

    ZgShuffleVerifier private _shuffle;
    ZgRevealVerifier private _reveal;
    function setVerifiers(
        ZgShuffleVerifier shuffle,
        ZgRevealVerifier reveal
    ) external {
        _shuffle = shuffle;
        _reveal = reveal;
    }

    // ...
}
```
Before shuffling the cards, we must first collect the public keys of the two players and generate a aggregated key dedicated to the battle.

So let's slightly tweak the `joinDuel` method so that players can provide their own public key when joining the game:

```solidity
    struct Duel {
        // ...
        // Add public keys in Duel struct:
        Point publicKey1;
        Point publicKey2;
        Point gameKey;
    }

    function joinDuel(Point memory publicKey) external {
        // ...
        // Store player's public key
        if (_duel.player1 == address(0)) {
            _duel.player1 = player;
            _duel.publicKey1 = publicKey;
        } else if (_duel.player2 == address(0)) {
            _duel.player2 = player;
            _duel.publicKey2 = publicKey;

            Point[] memory playerKeys = new Point[](2);
            _duel.gameKey = _reveal.aggregateKeys(playerKeys);

            // ...
        }
        // ....
    }
```

At this time our test will fail due to lack of verifiers. In order to allow the game to run smoothly on the local side, we create a fixed set of keys and mocked verifiers for testing:

```ts
  const shuffler = await ethers.deployContract('MockShuffleVerifier', []) as MockShuffleVerifier
  const revealer = await ethers.deployContract('MockRevealVerifier', []) as MockRevealVerifier

  await game.setVerifiers(
    await shuffler.getAddress(),
    await revealer.getAddress()
  )

  // SE.generate_key()
  const key1 = {
    sk: '0x03cef05cd7c1297e6b94dbd68a370213a2badb49a7167533d0971073c58673bd',
    pk: '0x3b93e02ba3dc6029f47a34b8cb56b7740a0908db6996b0dbc11f2a2f21122813',
    pkxy: [
      '0x108f4dfc21fec087ffbb0fd807b343399ff0d3527c59b86059dbd53df37d37b8',
      '0x132812212f2a1fc1dbb09669db08090a74b756cbb8347af42960dca32be0933b'
    ]
  }
  // SE.generate_key()
  const key2 = {
    sk: '0x035834c90c89d1f13b8c33851cfeb832b7bb98ea82968073e5c26ea0a6a055df',
    pk: '0x467a434b5b376c3a0f0e14fec10c495da5061dc3887d5fb2371370238a87be06',
    pkxy: [
      '0x0fc2388eb7fce64437b65900d130b96fdb639fdd4af7f7ca12d5b2bbbef50fe0',
      '0x06be878a23701337b25f7d88c31d06a55d490cc1fe140e0f3a6c375b4b437a46'
    ]
  }
```

Then pass in the value of `.pkxy` in the previous `joinDuel` method:

```ts
await game.connect(p1).joinDuel(key1.pkxy)

// ...

await expect(game.connect(p2).joinDuel(key2.pkxy)).to.emit(game, 'DuelStarted')
```

At this point, our test can be successfully completed again, and after the game starts, we can obtain the game key necessary for next shuffling.
Here we make a simple conversion to ensure that the game key text is converted to hex form.
(Depending on the web3 library on the client side, the processing methods here will be somewhat different,
our current environment is Ethers v6, so we use the toBeHex method to convert bigint to hex string)

```ts
// 0x964d2d19f0c34046bb6cee3652dc802fddf4c5b3c465b730c224da3946c6f92a
const gameKey = await game.duel()
                          .then(duel => duel.gameKey.map(bn => ethers.toBeHex(bn)))
                          .then(SE.public_compress)

// Initialize Secret Engine with the game key:
const deckSize = 20
SE.init_prover_key(deckSize)
const pkc = SE.refresh_joint_key(gameKey, deckSize)
```

The client is now ready to shuffle the cards, so we create some interfaces in the contract to allow players to submit the shuffled decks.
In order to simplify the process, we let the first player encrypt the public cards and shuffle them directly, so the interface for shuffling his own deck and uploading it will be different from the next shuffles:

```solidity
enum DuelState {
    None,
    SubmitSelfDeck,
    ShuffleOpponentDeck,
    // ...
}

struct Duel {
    // ...
    // Add shuffled decks & ZK variables:
    uint256[24] pkc;
    MaskedCard[] player1Deck;
    MaskedCard[] player2Deck;
}

error InvalidShuffle();
function submitDeck(
    uint256[24] calldata pkc,
    uint256[4][20] calldata maskedDeck,
    uint256[4][20] calldata shuffledDeck,
    bytes calldata proof
) external {
  // Do some simple checks...

  _duel.pkc = pkc;
  uint256[] memory input = new uint256[](20 * 4 * 2);
  for (uint256 i = 0; i < 20; i++) {
      input[i * 4 + 0] = maskedDeck[i][0];
      input[i * 4 + 1] = maskedDeck[i][1];
      input[i * 4 + 2] = maskedDeck[i][2];
      input[i * 4 + 3] = maskedDeck[i][3];

      input[i * 4 + 0 + 80] = shuffledDeck[i][0];
      input[i * 4 + 1 + 80] = shuffledDeck[i][1];
      input[i * 4 + 2 + 80] = shuffledDeck[i][2];
      input[i * 4 + 3 + 80] = shuffledDeck[i][3];
  }

  if (!_shuffle.verifyShuffle(proof, input, _duel.pkc)) {
      revert InvalidShuffle();
  }

  // Store deck...
}
```

Above, we first wrote the part about shuffling the player's own deck for the first time, and then added a test to let two players shuffle their own decks and send them out:

```ts
async function submitSelfDeck({ game, wallet, pkc, gameKey, deckSize }) {
  const maskedCards = SE.init_masked_cards(gameKey, deckSize)
                        .map(({ card }) => card)

  const {
    cards: shuffledCards,
    proof,
  } = SE.shuffle_cards(gameKey, maskedCards)

  return game.connect(wallet).submitDeck(pkc, maskedCards, shuffledCards, proof)
}

expect((await game.duel()).player1Deck).have.lengthOf(0)
expect((await game.duel()).player2Deck).have.lengthOf(0)

await Promise.all([
  { game, wallet: p1, pkc, gameKey, deckSize },
  { game, wallet: p2, pkc, gameKey, deckSize },
].map(submitSelfDeck))

expect((await game.duel()).player1Deck).have.lengthOf(20)
expect((await game.duel()).player2Deck).have.lengthOf(20)
```

Next, we add a method for shuffling the next deck into the contract to ensure no one can influence the ordering of cards:

```solidity
function shuffleDeck(
    uint256[4][20] calldata shuffledDeck,
    bytes calldata proof
) external {
  // Do some simple checks...

  uint256[] memory input = new uint256[](20 * 4 * 2);
  uint256[4][] storage deck = playerId == 1 ? _duel.player2Deck : _duel.player1Deck;

  for (uint256 i = 0; i < 20; i++) {
      input[i * 4 + 0] = deck[i][0];
      input[i * 4 + 1] = deck[i][1];
      input[i * 4 + 2] = deck[i][2];
      input[i * 4 + 3] = deck[i][3];

      input[i * 4 + 0 + 80] = shuffledDeck[i][0];
      input[i * 4 + 1 + 80] = shuffledDeck[i][1];
      input[i * 4 + 2 + 80] = shuffledDeck[i][2];
      input[i * 4 + 3 + 80] = shuffledDeck[i][3];
  }

  if (!_shuffle.verifyShuffle(proof, input, _duel.pkc)) {
      revert InvalidShuffle();
  }

  // Store deck...
}
```

It can be seen that it is basically the same as `submitDeck` in the previous step.
The main difference is that we use the deck before shuffling to bring in the verification parameters to ensure that the next player actually uses the correct deck to shuffle the cards.

Then add the process of shuffling the other's cards to the test:

```ts
async function shuffleOpponentDeck({ game, wallet, gameKey, deck }) {
  const hexifiedDeck = deck.map(cardBNs => cardBNs.map(bn => ethers.toBeHex(bn)))

  const {
    cards: shuffledCards,
    proof,
  } = SE.shuffle_cards(gameKey, hexifiedDeck)

  return game.connect(wallet).shuffleDeck(shuffledCards, proof)
}

await Promise.all([
  { game, wallet: p1, gameKey, deck: duel.player2Deck },
  { game, wallet: p2, gameKey, deck: duel.player1Deck },
].map(shuffleOpponentDeck))
```

At this point, both players in the game have mixed up each other's decks and are ready to start playing cards.

Before playing the cards, we need to let the players know what cards they have in their hand. Please see the next chapter for the detailed process.

## Step 4: Compute Reveal Tokens and Show Player's Hand Cards

Next, usually the player will hold some cards in his hand, the contents of which only the player can see.

In order to achieve this common card game function, we need to perform the following steps:

1. Other players calculate and submit the reveal tokens of the player's card.

2. After the player gets the reveal tokens of his card from the contract, he uses the secret engine to decrypt them locally. Player can see which cards in the NFT deck these cards are.

3. When a player plays a card, in addition to telling the contract which card he played, he also sends the reveal token of the card at the same time. Allow other players to see the card content.

For this, we first adjust the contract structure to let each other know which cards they currently hold, and players need to calculate reveal tokens:

```solidity
struct Duel {
  // ...

  // Reveal tokens of each cards
  uint256[2][][] player1Reveals;
  uint256[2][][] player2Reveals;

  // Index of the card in the deck
  uint8[] player1Hand;
  uint8[] player2Hand;
}
```

Then  write some internal functions to verify reveal tokens and display the actual cards:

```solidity
    error InvalidRevealToken();
    function _requireValidRevealToken(
        uint8 targetPlayerId,
        uint8 targetCardIndex,
        uint8 senderPlayerId,
        uint256[2] calldata revealToken,
        uint256[8] calldata proof
    ) internal view {
        uint256[4][] storage deck = targetPlayerId == 1 ? _duel.player1Deck : _duel.player2Deck;
        Point storage publicKey = senderPlayerId == 1 ? _duel.publicKey1 : _duel.publicKey2;

        if (!_reveal.verifyRevealWithSnark([
            deck[targetCardIndex][2],
            deck[targetCardIndex][3],
            revealToken[0],
            revealToken[1],
            publicKey.x,
            publicKey.y
        ], proof)) {
            revert InvalidRevealToken();
        }
    }

    function _realCardId(
        uint8 playerId,
        uint8 cardIndex
    ) internal view returns (uint256) {
        address player = playerId == 1 ? _duel.player1 : _duel.player2;

        uint256[4] storage maskedCard = playerId == 1 ? _duel.player1Deck[cardIndex] : _duel.player2Deck[cardIndex];
        uint256[2][] storage reveals = playerId == 1 ? _duel.player1Reveals[cardIndex] : _duel.player2Reveals[cardIndex];

        if (reveals.length < 2) {
            return 0;
        }

        Point[] memory rTokens = new Point[](reveals.length);
        for (uint256 i = 0; i < reveals.length; i++) {
            rTokens[i] = Point(reveals[i][0], reveals[i][1]);
        }

        uint8 realCardIndex = _reveal.unmaskCard(MaskedCard(
            maskedCard[0],
            maskedCard[1],
            maskedCard[2],
            maskedCard[3]
        ), rTokens);

        return _decks[player][realCardIndex];
    }
```

Next, we assume that the two players will draw three cards in their hands at the beginning, that is, 0, 1, and 2 cards in their decks.

We provide an interface in the contract for players to submit reveal tokens of each other's cards and verify them:

```solidity
    error AlreadyShownCard(uint8 cardIndex);
    function showOpponentHandCard(
        uint8 cardIndex,
        uint256[2] calldata revealToken,
        uint256[8] calldata proof
    ) external {
        uint8 playerId = requireDuelPlayer(msg.sender);
        uint8 targetPlayerId = playerId == 1 ? 2 : 1;

        uint256[2][][] storage reveals = targetPlayerId == 1
            ? _duel.player1Reveals
            : _duel.player2Reveals;

        if (reveals[cardIndex].length > 0) {
            revert AlreadyShownCard(cardIndex);
        }

        _requireValidRevealToken(
            targetPlayerId,
            cardIndex,
            playerId,
            revealToken,
            proof
        );

        reveals[cardIndex].push(revealToken);

        if (_duel.state == DuelState.RevealInitialHand) {
            bool allRevealed = true;
            for (uint8 i = 0; i < INIT_HAND_SIZE; i++) {
                if (reveals[i].length == 0) {
                    allRevealed = false;
                    break;
                }
            }

            if (allRevealed) {
                if (playerId == 1) {
                    _duel.done1 = true;
                } else {
                    _duel.done2 = true;
                }
            }
        }

        if (_duel.done1 && _duel.done2) {
            _nextDuelState();
        }
    }
```

Then back in the test, we asked players to hand over each other's hand reveal tokens:

```ts
async function showOpponentHand({ game, wallet, keys, reveals, deck }) {
  for (const index of reveals) {
    const target = deck[index].map(bn => ethers.toBeHex(bn))
    const {
      card: revealToken,
      snark_proof: proof,
    } = SE.reveal_card_with_snark(keys.sk, target)

    await game.connect(wallet).showOpponentHandCard(index, revealToken, proof)
  }
}

// Run this before reveal any cards
SE.init_reveal_key()

duel = await game.duel()
await Promise.all([
  { game, wallet: p1, keys: key1, reveals: [0, 1, 2], deck: duel.player2Deck },
  { game, wallet: p2, keys: key2, reveals: [0, 1, 2], deck: duel.player1Deck},
].map(showOpponentHand))
```

After the opponent submits the reveal tokens of the cards, we can review it.

Here we present the cards in the format of NFT ID. In your product, each NFT should have its own image and value:

```ts
async function printHandCards({ name, deck, keys, hands, rTokens, nfts }) {
  const indexes = hands.map(idx => SE.unmask_card(
    keys.sk,
    deck[idx].map(bn => ethers.toBeHex(bn)),
    rTokens[idx].map(bns => bns.map(bn => ethers.toBeHex(bn)))
  ))

  const nftIds = indexes.map(idx => `[${nfts[idx]}]`)

  console.log(`${name}: ${nftIds.join(' ')}`)
}

const nftDeck1 = await game.getDeck(p1.address)
const nftDeck2 = await game.getDeck(p2.address)

await Promise.all([
  { deck: duel.player1Deck, keys: key1, hands: duel.player1Hand, rTokens: duel.player1Reveals, nfts: nftDeck1, name: 'Player 1' },
  { deck: duel.player2Deck, keys: key2, hands: duel.player2Hand, rTokens: duel.player2Reveals, nfts: nftDeck2, name: 'Player 2' },
].map(printHandCards))
```

After running the test again, you can already see the two players showing their cards at the end.

If it is a front-end interface, it should be able to convert these NFT IDs into better-looking cards and corresponding values.

Next, let's move on to the next chapter, which shows you how to play a card so that everyone can see its contents.

## Step 5: Play a Card and Open Its Content

Now, both players can see their own cards, but the other player cannot see their contents.

Let’s adjust the contract and add a column to hold the cards played by the two players:

```solidity
struct Duel {
    // ...

    // NFT IDs
    uint256[] player1Board;
    uint256[] player2Board;
}
```

Then we add a new method for players to play cards. In addition to specifying the corresponding index of the hand,
Also bring the reveal token and reveal proof so that we can decode it and store it in the board field.

```solidity
function playCard(
    uint8 cardIndex,
    uint256[2] calldata revealToken,
    uint256[8] calldata proof
) external {
    // Some simple checks...

    reveals[cardIndex].push(revealToken);

    if (playerId == 1) {
        _duel.player1Board.push(_realCardId(playerId, cardIndex));
    } else {
        _duel.player2Board.push(_realCardId(playerId, cardIndex));
    }
}
```

Then we use the test to play the card and confirm that the results match:

```ts
async function playCard({ game, wallet, keys, hand, deck, handIndex }) {
  const cardIndex = hand[handIndex]
  const target = deck[cardIndex].map(bn => ethers.toBeHex(bn))

  const {
    card: revealToken,
    snark_proof: proof,
  } = SE.reveal_card_with_snark(keys.sk, target)

  await game.connect(wallet).playCard(cardIndex, revealToken, proof)
}

// We don't have ZK verifier to unmask card here, so we just mock the unmask result
await revealer.mockNextCardId(hands[0].indexes[0])
await playCard({
  game,
  wallet: p1,
  keys: key1,
  hand: duel.player1Hand,
  deck: duel.player1Deck,
  handIndex: 0,
})
// Ensure we had correctly tranform that unmasked card index to NFT ID:
expect(duel.player1Board[0]).to.eq(hands[0].nftIds[0], 'Same NFT card as played')
```

At this point, we can correctly play cards and display them on the public card table in the form of NFT IDs.

In the future, we can add more card game logic to complete this card game.

## Step 6: Add some gaming logic and have fun!

- Let's start by adding an attack attribute to `GameCard.sol`

```solidity
contract GameCard is ERC721 {
    // ...
    mapping (uint256 => uint8) internal _attacks; // tokenId => attack

    // atk range: 1 ~ 5
    function _createAttackValue(uint256 tokenId) internal view returns (uint8) {
        uint8 atk = uint8(uint256(keccak256(abi.encodePacked(block.timestamp, tokenId, msg.sender)))) % 5 + 1;
        return atk;
    }

    function getAttack(uint256 tokenId) external view returns (uint8) {
        return _attacks[tokenId];
    }
}
```

- Back to `Game.sol`, let's give players some health

```solidity
struct Duel {
    // ...

    uint8 player1Lives;
    uint8 player2Lives;
}

uint8 private constant PLAYER_LIVES = 5;

function _initDuel() internal {
    // ...
    _duel.player1Lives = PLAYER_LIVES;
    _duel.player2Lives = PLAYER_LIVES;
}
```

- Then add some gaming logic when players play a card

```solidity
function playCard(
        uint8 cardIndex,
        uint256[2] calldata revealToken,
        uint256[8] calldata proof
    ) external {
        // ...
        uint256 cardId = _realCardId(playerId, cardIndex);
        uint8 atk = cardSet.getAttack(cardId);
        uint8 targetLives = playerId == 1 ? _duel.player2Lives : _duel.player1Lives;

        if (atk > targetLives) {
            targetLives = 0;
        } else {
            targetLives -= atk;
        }

        if (playerId == 1) {
            _duel.player2Lives = targetLives;
        } else {
            _duel.player1Lives = targetLives;
        }
}
```

There you have it, a card game on the blockchain with zk shuffle!
