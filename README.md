<img alt="land battle chess" width="1412" src="resources/land_battle_chess.webp">

- [Summary](#summary)
- [Build and Run Guide](#build-and-run-guide)
- [Aleo Program Design](#aleo-program-design)

# Summary
Land Battle Chess is a two-player Chinese board game. It is an abstract strategy game of incomplete information, since each player has only limited knowledge concerning the disposition of the opposing pieces.
The rules are complicated; for more details, please refer to [Wikipedia](https://en.wikipedia.org/wiki/Luzhanqi). 

Our game fully demonstrates the advantages of aleo’s privacy protection and programmability, letting players and developers experience the innovation and value of aleo.

# Build and Run Guide

env

- snarkos 

  - https://github.com/AleoHQ/snarkOS.git [testnet3 9b8191327c0002ea7abb4252e21b4bff5b074700]

- leo

  - https://github.com/AleoHQ/leo [testnet3 a2551132f52091567a765107724746e0a6f0e39c]

- rust 

  ```bash
  rustup -V
  cargo -V
  cargo add wasm-pack
  ```

- node

  ```bash
  node -v
  v16.20.2
  ```

  

 setup

- Use  snarkOS version to build a local chain.

  ```bash
  snarok start --beacon "" --nodisplay --dev 0
  ```

- Build and Deploy the program based on the specific leo version

  ```bash
  #build land_battle_chess_aleo program
  git clone git@github.com:zksprint/land_battle_chess_aleo.git
  cd land_battle_chess_aleo && git pull 
  leo build
  
  #deploy it to testnet3
  snarkos developer deploy --query "http://127.0.0.1:3030" --private-key APrivateKey1zkp8CZNn3yeCseEtxuVPbDCwSyhGW6yZKUYKfgXmcpoGPWH --path ./build --broadcast "http://127.0.0.1:3030/testnet3/transaction/broadcast" --fee 1 --record  "{  owner: aleo1rhgdu77hgyqd3xjj8ucu3jj9r2krwz6mnzyd80gncr5fxcwlh5rsvzp9px.private,  microcredits: 93750000000000u64.private,  _nonce: 6886137302873465030715311586140142512237913705723767478581818713532726333104group.public}" land_battle_chess_v2.aleo
  
  #build land-battle-chess-rs
  git clone git@github.com:cryptodegen101/land-battle-chess-rs.git
  cd land-battle-chess-rs && git pull 
  cargo run build --release
  
  #build wasm and sdk and land-chess-ui
  git clone git@github.com:zksprint/land-chess-ui.git
  cd land-chess-ui && git pull
  #build  wasm
  cd aleo/wasm && wasm-pack build --target web
  #build sdk
  cd ../sdk && npm i
  #build  land-chess-ui
  cd ../../ && npm i 
  
  ```
  

run project

```bash
#run land-battle-chess-rs
cd lan-battle-chess-rs && cargo run --bin land-battle-chess-rs
#run land-chess-ui
cd ../land-chess-ui && npm run dev
```



# Aleo Program Design 

In the game, there are 3 parties: player1, player2 and the arbiter. When a piece lands on a space occupied by an opposing piece, the arbiter is responsible for comparing pieces. The lower-ordered piece is removed from the board; if the two are of equal order, both are to be removed from the board.

<img alt="land battle chess" width="1412" src="resources/land_battle_chess_steps.png">

1. Both Players Initialize their Board
2. Player1 Moves a piece
3. Player2 Whispers the Target Piece to the Arbiter
4. Aribter Compares pieces
5. Player2 Moves a piece
6. Player1 Whispers the Target Piece to the Arbiter 
7. Aribter Compares pieces
8. Continue until Game Ends

```
// The board consists of five lines (line0, line1, line2, line3, line4).
// flag_x, flag_y indicate the position of the flag on the board.
transition player_initialize_board(line0: u64, line1: u64, line2: u64, line3: u64, line4: u64, flag_x: u64, flag_y: u32, public game_id: u64, public player_index: u32, public arbiter: address) -> (player_state, bool);

// state is the player state.
// prev_move is the information of the movement of player’s piece in the previous round.
// x,y are the position of the piece to be moved.
// target_x,target_y are the target position.
transition move_piece(state: player_state, public prev_move: move, public x: u64, public y: u32, public target_x: u64, public target_y: u32) -> (player_state, piece_info);

// state is the player state.
// prev_move is the information of the movement of player’s piece in the previous round.
// x,y are the position of the piece to be moved.
// target_x,target_y are the target position.
transition whisper_piece(state: player_state, public prev_move: move, public target_x: u64, public target_y: u32) -> (player_state, piece_info);

// attacker is the moved piece.
// target is the piece that is attacked, If there's no piece in the target position, target piece is 0.
transition compare_pieces(attacker: piece_info, target: piece_info) -> move;

```