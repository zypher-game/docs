+++
title = "SDK - Tile Match 2048"
description = "2048 circuit SDK."
date = 2024-05-01T08:00:00+00:00
updated = 2024-05-01T08:00:00+00:00
draft = false
weight = 52
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = '2048 circuit SDK.'
toc = true
top = false
+++

## Environment

- Language: javascript/typescript

## How it works
- Essentially what we are trying to accomplish is verifying the game state after a player has made some moves. Once the verifying input is set, call the verify.verifyProof to have the ZK circuits verify the validity of the game.


## Game2048Step60CircomVerifier.sol
```
pragma solidity >=0.7.0 <0.9.0;

contract Game2048Step60CircomVerifier {
    // Scalar field size
    uint256 constant r    = 21888242871839275222246405745257275088548364400416034343698204186575808495617;
    // Base field size
    uint256 constant q   = 21888242871839275222246405745257275088696311157297823662689037894645226208583;

    // Verification Key data
    uint256 constant alphax  = 20491192805390485299153009773594534940189261866228447918068658471970481763042;
    uint256 constant alphay  = 9383485363053290200918347156157836566562967994039712273449902621266178545958;
    uint256 constant betax1  = 4252822878758300859123897981450591353533073413197771768651442665752259397132;
    uint256 constant betax2  = 6375614351688725206403948262868962793625744043794305715222011528459656738731;
    uint256 constant betay1  = 21847035105528745403288232691147584728191162732299865338377159692350059136679;
    uint256 constant betay2  = 10505242626370262277552901082094356697409835680220590971873171140371331206856;
    uint256 constant gammax1 = 11559732032986387107991004021392285783925812861821192530917403151452391805634;
    uint256 constant gammax2 = 10857046999023057135944570762232829481370756359578518086990519993285655852781;
    uint256 constant gammay1 = 4082367875863433681332203403145435568316851327593401208105741076214120093531;
    uint256 constant gammay2 = 8495653923123431417604973247489272438418190587263600148770280649306958101930;
    uint256 constant deltax1 = 18909290623589372212608388513684575758995519969001892131945528937291220952182;
    uint256 constant deltax2 = 2875203449378516743109609553050026568133321502890853732994375713553320658221;
    uint256 constant deltay1 = 2489150813694632276856612536291067589951037720943945513539803655879400209720;
    uint256 constant deltay2 = 19276775233044448710962620944521828097017666380262997869520596730242392735386;

    uint256 constant IC0x = 11128506941297822501254433205268267539034243891395423859217363107759883248654;
    uint256 constant IC0y = 3872357426528296580277109136192992878606687753367907579323499892857106028023;
    uint256 constant IC1x = 1325018593988210577661405641566795036098275828122695304685926714336229517577;
    uint256 constant IC1y = 18686319433822603781462504439484601927718937069209765373470951610790846398728;
    uint256 constant IC2x = 17589504352492591802413258511716791489617460014098384868268995008340248338890;
    uint256 constant IC2y = 7027550309711617589112326546570913722644950087163227925974727780133140226894;
    uint256 constant IC3x = 4588928733530151135947791164368553161142983039351091119141366306492203211553;
    uint256 constant IC3y = 18420049025099470438629579198889786216543247788312803323852980551532491788310;
    uint256 constant IC4x = 11633399819570956868006897176291153264879053258454880109266347340084667659269;
    uint256 constant IC4y = 19112055496166407834117839841578227460244067316266275527869120299506331868886;
    uint256 constant IC5x = 2615038044687269628522971510801981316302313068348848272418961384096001575368;
    uint256 constant IC5y = 5218493524843539969617657172992904866318833156180894217901135613096728282798;
    uint256 constant IC6x = 9270007476823418702361089176679204312677819763479617887398545340750394759725;
    uint256 constant IC6y = 14692585592693406186322812589264956004146883092056905736778714505217877929575;
    uint256 constant IC7x = 7041501423034499493782096928158436615884067989639971865426967705870706974226;
    uint256 constant IC7y = 2171352084991153726633202556319966754825846942596088172174758070182697960381;

    // Memory data
    uint16 constant pVk = 0;
    uint16 constant pPairing = 128;

    uint16 constant pLastMem = 896;

    function verifyProof(uint[2] calldata _pA, uint[2][2] calldata _pB, uint[2] calldata _pC, uint[7] calldata _pubSignals) public view returns (bool) {
        assembly {
            function checkField(v) {
                if iszero(lt(v, q)) {
                    mstore(0, 0)
                    return(0, 0x20)
                }
            }

            // G1 function to multiply a G1 value(x,y) to value in an address
            function g1_mulAccC(pR, x, y, s) {
                let success
                let mIn := mload(0x40)
                mstore(mIn, x)
                mstore(add(mIn, 32), y)
                mstore(add(mIn, 64), s)

                success := staticcall(sub(gas(), 2000), 7, mIn, 96, mIn, 64)

                if iszero(success) {
                    mstore(0, 0)
                    return(0, 0x20)
                }

                mstore(add(mIn, 64), mload(pR))
                mstore(add(mIn, 96), mload(add(pR, 32)))

                success := staticcall(sub(gas(), 2000), 6, mIn, 128, pR, 64)

                if iszero(success) {
                    mstore(0, 0)
                    return(0, 0x20)
                }
            }

            function checkPairing(pA, pB, pC, pubSignals, pMem) -> isOk {
                let _pPairing := add(pMem, pPairing)
                let _pVk := add(pMem, pVk)

                mstore(_pVk, IC0x)
                mstore(add(_pVk, 32), IC0y)

                // Compute the linear combination vk_x
                g1_mulAccC(_pVk, IC1x, IC1y, calldataload(add(pubSignals, 0)))
                g1_mulAccC(_pVk, IC2x, IC2y, calldataload(add(pubSignals, 32)))
                g1_mulAccC(_pVk, IC3x, IC3y, calldataload(add(pubSignals, 64)))
                g1_mulAccC(_pVk, IC4x, IC4y, calldataload(add(pubSignals, 96)))
                g1_mulAccC(_pVk, IC5x, IC5y, calldataload(add(pubSignals, 128)))
                g1_mulAccC(_pVk, IC6x, IC6y, calldataload(add(pubSignals, 160)))
                g1_mulAccC(_pVk, IC7x, IC7y, calldataload(add(pubSignals, 192)))

                // -A
                mstore(_pPairing, calldataload(pA))
                mstore(add(_pPairing, 32), mod(sub(q, calldataload(add(pA, 32))), q))

                // B
                mstore(add(_pPairing, 64), calldataload(pB))
                mstore(add(_pPairing, 96), calldataload(add(pB, 32)))
                mstore(add(_pPairing, 128), calldataload(add(pB, 64)))
                mstore(add(_pPairing, 160), calldataload(add(pB, 96)))

                // alpha1
                mstore(add(_pPairing, 192), alphax)
                mstore(add(_pPairing, 224), alphay)

                // beta2
                mstore(add(_pPairing, 256), betax1)
                mstore(add(_pPairing, 288), betax2)
                mstore(add(_pPairing, 320), betay1)
                mstore(add(_pPairing, 352), betay2)

                // vk_x
                mstore(add(_pPairing, 384), mload(add(pMem, pVk)))
                mstore(add(_pPairing, 416), mload(add(pMem, add(pVk, 32))))


                // gamma2
                mstore(add(_pPairing, 448), gammax1)
                mstore(add(_pPairing, 480), gammax2)
                mstore(add(_pPairing, 512), gammay1)
                mstore(add(_pPairing, 544), gammay2)

                // C
                mstore(add(_pPairing, 576), calldataload(pC))
                mstore(add(_pPairing, 608), calldataload(add(pC, 32)))

                // delta2
                mstore(add(_pPairing, 640), deltax1)
                mstore(add(_pPairing, 672), deltax2)
                mstore(add(_pPairing, 704), deltay1)
                mstore(add(_pPairing, 736), deltay2)


                let success := staticcall(sub(gas(), 2000), 8, _pPairing, 768, _pPairing, 0x20)

                isOk := and(success, mload(_pPairing))
            }

            let pMem := mload(0x40)
            mstore(0x40, add(pMem, pLastMem))

            // Validate that all evaluations ∈ F
            checkField(calldataload(add(_pubSignals, 0)))
            checkField(calldataload(add(_pubSignals, 32)))
            checkField(calldataload(add(_pubSignals, 64)))
            checkField(calldataload(add(_pubSignals, 96)))
            checkField(calldataload(add(_pubSignals, 128)))
            checkField(calldataload(add(_pubSignals, 160)))
            checkField(calldataload(add(_pubSignals, 192)))
            checkField(calldataload(add(_pubSignals, 224)))

            // Validate all evaluations
            let isValid := checkPairing(_pA, _pB, _pC, _pubSignals, pMem)

            mstore(0, isValid)
             return(0, 0x20)
         }
     }
 }
```

## Parameter interpretation
- board<br/>
    - Example of a normal board:<br/>
    [4,2,16,0,0,0,2,4,0,32,1024,0,0,0,0,0]
    - We use the nth power of 2 to represent it:<br/>       [2,1,4,0,0,,0,1,2,0,5,10,0,0,0,0,0]
- packedBoard
```
export function packBoard(board: bigint[]) {
    let packed = 0n;
    for (let i = 0; i < board.length; i++) {
        packed = packed * 32n + board[i];
    }
    return packed;
}
```
- packedDir
```
export function packDirection(direction: bigint[]) {
    let packed = 0n;
    for (let i = 0; i < direction.length; i++) {
        packed = packed * 4n + direction[i];
    }
    return packed;
}
```
- direction
    - 0: up
    - 1:left
    - 2：down
    - 3：right
- address 
    - msg.sender
- step
    - current Steps
- stepAfter
    - next Steps
- nonce
    - last execution of interactive contract returned
    ```
    uint256 constant NONCE_MOD = 21888242871839275222246405745257275088548364400416034343698204186575808495617;

    struct VerifyData {
        uint[2] _pA;
        uint[2][2] _pB;
        uint[2] _pC;
        uint[7] _pubSignals;
    }

    function generateNonce(
        VerifyData calldata proof
    ) internal view returns (uint256) {
        uint256 nonce = uint256(
            keccak256(
                abi.encodePacked(
                    msg.sender,
                    blockhash(block.number - 1),
                    proof._pubSignals[5]
                )
            )
        ) % NONCE_MOD;
        return nonce;
    }
    ```


## Assembly parameters
```
input = {
board,
[packBoard(board[0]), packBoard(board[1])], // is packedBoard
packDirection(direction), // is packedDir
direction,
address,
step,
stepAfter,
nonce
}

```

## Generate proof and publicSignals
```
const { proof, publicSignals } = await snarkjs.groth16.fullProve(
      input,
      "game2048_60.wasm",
      "game2048_60.final.zkey"
    );

```

## How to call verify smart contract
```
await verifier.verifyProof(
        proof.pi_a.slice(0, 2),
        [proof.pi_b[0].slice(0).reverse(), proof.pi_b[1].slice(0).reverse()],
        proof.pi_c.slice(0, 2),
        publicSignals
      )

```
