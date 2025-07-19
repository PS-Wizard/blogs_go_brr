+++
date = '2025-07-18T21:17:57+05:45'
draft = false
title = 'Initial setup & Board Representation'
description = 'Figuring Out Board + Piece Representation For A Bitboard Implementation'
tags = ["rust", "chess", "arena"]
code_link = "https://github.com/PS-Wizard/EvalDeez/tree/main/crates/arena"
+++

# Initial Setup
Allrighty, so each of the different sections of this engine is going to be in their own seperate crate. 
{{<danger title="A Quick Disclaimer">}}
**I don't know rust.** So, some of the things I be doing might not be the best practice, or even remotely a good practice
- Have I went through the [rust-lang's book](https://doc.rust-lang.org/book/)? **Yes.**
- Have I made other projects in rust? [Git Sketch](https://github.com/PS-Wizard/GitSketch) [Rust Down](https://github.com/PS-Wizard/rust_down) **Yes.**
- **But I don't fully know rust.**
{{</danger>}}

With that outa the way, the way I've structured this project is that, I have a workspace setup.
```rust
~ Cargo.toml 
[workspace]
resolver = "3"
members = [ "crates/arena", "crates/magician", "crates/prophet","crates/tactition", "crates/translator", "crates/warden"]
[wizard@archlinux evaldeez]$
```
```
.
├── assets
│   └── nnue-probe                                          // Pulled straight from: https://github.com/dshawul/nnue-probe. 
|                                                           // For now compiling it and using it as a binary, later will probably 
|                                                           // create own port in rust or FFI bindings
├── Cargo.lock
├── Cargo.toml
├── crates
│   ├── arena                                               // Will Handle Board Representation
│   ├── magician                                            // Will Handles Magic Bitboards
│   ├── prophet                                             // Will Handle evaluations
│   ├── tactition                                           // Will Handle Move Gen (depends on `magician` for sliding pieces)
│   ├── translator                                          // Will Handle Grammars ( eg. FEN, PGN, UCI ...)
│   └── warden                                              // Will Handle Legalities ( stalemates, checkmates ... )
├── README.md
└── target
    ├── CACHEDIR.TAG
    └── debug
```

---

# Board Representation:

### Bitboards / Bitsets / Bitmaps :
So, our first intuition when we are tasked to represent an `8x8` board is probably an array. That's Valid But turns out there is a better way to represent the boards, **Bitboards**. 
Bitboards are basically a `u64`, an unsigned 64 bit long integer that we use to represent board state. Makes sense ... or does it?

####  **How does a integer even represent a board?**
A chess board is `8x8`, 8 rows & 8 columns, that is a total of 64 squares. A `u64` has 64 bits, and thus it is possible to represent the board with a `u64`. Okay, but wait,

#### **How do u place pieces in an integer?**: 
A `u64` is basically `0000000000000000...repeated 64 times` right? So if we were to draw it out as a board, we would get the following:
```
8  0 0 0 0 0 0 0 0
7  0 0 0 0 0 0 0 0
6  0 0 0 0 0 0 0 0
5  0 0 0 0 0 0 0 0
4  0 0 0 0 0 0 0 0
3  0 0 0 0 0 0 0 0
2  0 0 0 0 0 0 0 0
1  0 0 0 0 0 0 0 0
   a b c d e f g h
```
That's basically a `0u64`, where the MSB is is `a8` and the LSB is `a1`. 

Say you were to represent all the white pawns, that would be `0000000000000000000000000000000000000000000000001111111100000000` in binary, or `0x0000_0000_0000_FF00` in hexadecimal. That might start to make sense, but for better visualization, that corresponds to the following: 

```
8  0 0 0 0 0 0 0 0
7  0 0 0 0 0 0 0 0
6  0 0 0 0 0 0 0 0
5  0 0 0 0 0 0 0 0
4  0 0 0 0 0 0 0 0
3  0 0 0 0 0 0 0 0
2  1 1 1 1 1 1 1 1
1  0 0 0 0 0 0 0 0
   a b c d e f g h
```
So, that's how Bitboards represent pieces, simply a `1` set in all the corresponding squares. Thus a starting position would look something like:
```
8  1 1 1 1 1 1 1 1
7  1 1 1 1 1 1 1 1
6  0 0 0 0 0 0 0 0
5  0 0 0 0 0 0 0 0
4  0 0 0 0 0 0 0 0
3  0 0 0 0 0 0 0 0
2  1 1 1 1 1 1 1 1
1  1 1 1 1 1 1 1 1
   a b c d e f g h
```
That makes sense!, but it's not quite how we do things. We typically don't set all the pieces in one `u64`. Could we do it? **technically yes** But is it a hassle? **also yes.**.
Instead of having a single `u64` represent everything, we split it into a bunch of different boards based on piece type:
- `white_pawns`: `u64(0x0000_0000_0000_FF00)`,
- `white_knights`: `u64(0x0000_0000_0000_0042)`,
- `white_bishops`: `u64(0x0000_0000_0000_0024)`,
- `white_rooks`: `u64(0x0000_0000_0000_0081)`,
- `white_queens`: `u64(0x0000_0000_0000_0008)`,
- `white_king`: `u64(0x0000_0000_0000_0010)`,
- `black_pawns`: `u64(0x00FF_0000_0000_0000)`,
- `black_knights`: `u64(0x4200_0000_0000_0000)`,
- `black_bishops`: `u64(0x2400_0000_0000_0000)`,
- `black_rooks`: `u64(0x8100_0000_0000_0000)`,
- `black_queens`: `u64(0x0800_0000_0000_0000)`,
- `black_king`: `u64(0x1000_0000_0000_0000)`,

#### So ... Why not one single board?
Because splitting just makes sense:
{{< accordion title="Clean Logic" open="y">}} A bitboard is a 64-bit integer where each bit represents a square on a chess board. {{< /accordion >}}
{{< accordion title="Piece Detection" >}}
Wanna know what piece is sitting on a square? Easy. You just do:
```rust
if (white_pawns | white_knights | white_rooks | ...) & (1 << e4_index) != 0
```
##### **Bro what even is this?**
Basically, since each of our bitboards is a `u64`, meaning every square on the board maps to a bit — `a1` is bit 0, `b1` is bit 1, ..., `h8` is bit 63.

If you wanna check if a piece exists on, say, `e4`, you create a mask like `1 << e4_index`, and then `AND` it with a bitboard. If the result isn’t zero, that piece exists on that square. 

Now, because we split bitboards by piece type (and color), if `white_pawns & mask != 0`, then boom — it’s a white pawn. And since no two pieces ever share a square, this check is totally safe.
{{< /accordion >}}
{{<accordion title="Cleaner Occupancy Masks">}}
**Want a map of all white pieces?**
```rust
let white_occupancy = white_pawns | white_knights | ... | white_king;
```
**Want All Pieces?**
```rust
let all_pieces = white_occupancy | black_occupancy;
```
{{< /accordion>}}
{{<accordion title="Move Generation Becomes Modular">}}
Since each piece type has its own set of rules, If everything was crammed into one board, you'd be constantly untangling who's who. 

Split boards = plug-and-play logic per piece, and it comes especially in handy later when we are doing move gen with magic bitboards. 
{{< /accordion>}}

### The Code:
Okay so now that the basics are done, let's move on with the code:
```rust {title = "board.rs"}
pub struct Board(pub(crate) u64);

impl Board {
    pub fn has_bit(&self, idx: u8) -> bool {
        if idx >= 64 {
            return false;
        }
        (self.0 & (1 << idx)) != 0
    }

    pub fn set_bit(&mut self, idx: usize) {
        self.0 |= 1 << idx;
    }
}
```
In my case, I made the `Board` a tuple struct. This way, it is the same memory as a `u64`, but just wrapped so that I can add helper functions such as `has_bit`, `set_bit` and anything else I choose down the line.

The `has_bit` and `set_bit` functions are pretty self explanatory, they both make use of logic gates, to either set a bit using `OR` or to check if a bit is set at a certain index using a combination of left shifts `<<` and `AND` gate

#### Is it enough?
You might have noticed, that although bitboards help represent the piece's occupancies, there are other aspects of a board that don't concern the pieces exactly, thus, we need another wrapper around these boards:

### The Game State:
```rust
pub struct Game {
    pub white_pawns: Board,
        pub white_rooks: Board,
        pub white_knights: Board,
        pub white_bishops: Board,
        pub white_queens: Board,
        pub white_king: Board,

        pub black_pawns: Board,
        pub black_rooks: Board,
        pub black_knights: Board,
        pub black_bishops: Board,
        pub black_queens: Board,
        pub black_king: Board,

        pub castling_rights: u8, // 4 bits: WK, WQ, BK, BQ
        pub side_to_move: bool,  // false = white, true = black

        //TODO:
        // - Enpassant flag
        // - Move Counters
}

```
The game state is simply the pieces + other stuff such as castling rights, En passant , side to move, etc. I am still yet to figure out how best to do En passant & the Move Counters. So, I will set those off to the future for now. Apart from that, I've also thrown in a few methods:

{{<accordion type="func" title="Constructor">}}
Creates a new game, sets all the pieces in their default position
```rust
pub fn new() -> Self {
    Game {
        // White pieces on ranks 1 and 2
        white_pawns: Board(0x0000_0000_0000_FF00),
        white_knights: Board(0x0000_0000_0000_0042),
        white_bishops: Board(0x0000_0000_0000_0024),
        white_rooks: Board(0x0000_0000_0000_0081),
        white_queens: Board(0x0000_0000_0000_0008),
        white_king: Board(0x0000_0000_0000_0010),

        // Black pieces on ranks 7 and 8
        black_pawns: Board(0x00FF_0000_0000_0000),
        black_knights: Board(0x4200_0000_0000_0000),
        black_bishops: Board(0x2400_0000_0000_0000),
        black_rooks: Board(0x8100_0000_0000_0000),
        black_queens: Board(0x0800_0000_0000_0000),
        black_king: Board(0x1000_0000_0000_0000),

        castling_rights: 0,
        side_to_move: true,
    }
}
{{</accordion>}}
{{<accordion type="func" title="Get All Pieces">}}
Simply uses an `OR` gate to get all pieces, particularly useful when printing the board
```rust
    pub fn all_pieces(&self) -> Board {
        Board(
            self.white_pawns.0
                | self.white_knights.0
                | self.white_bishops.0
                | self.white_rooks.0
                | self.white_queens.0
                | self.white_king.0
                | self.black_pawns.0
                | self.black_knights.0
                | self.black_bishops.0
                | self.black_rooks.0
                | self.black_queens.0
                | self.black_king.0,
        )
    }
```
{{</accordion>}}
{{<accordion type="func" title="Return A Piece At Certain Square">}}
Takes in a `idx:u8` and returns and Optional `Piece`, The `Piece` is defined below.
```rust
    pub fn get_piece_at(&self, idx: u8) -> Option<Piece> {
        if idx >= 64 || (self.all_pieces().0 & 1 << idx == 0) {
            return None;
        }

        // Helper closure to check if bit is set and create Piece
        let check = |board: &Board, piece_type: PieceType, color: Color| {
            if board.has_bit(idx) {
                Some(Piece::new(piece_type, color))
            } else {
                None
            }
        };

        // Check white pieces
        if let Some(p) = check(&self.white_pawns, PieceType::Pawn, Color::White) {
            return Some(p);
        }
        if let Some(p) = check(&self.white_knights, PieceType::Knight, Color::White) {
            return Some(p);
        }
        if let Some(p) = check(&self.white_bishops, PieceType::Bishop, Color::White) {
            return Some(p);
        }
        if let Some(p) = check(&self.white_rooks, PieceType::Rook, Color::White) {
            return Some(p);
        }
        if let Some(p) = check(&self.white_queens, PieceType::Queen, Color::White) {
            return Some(p);
        }
        if let Some(p) = check(&self.white_king, PieceType::King, Color::White) {
            return Some(p);
        }

        // Check black pieces
        if let Some(p) = check(&self.black_pawns, PieceType::Pawn, Color::Black) {
            return Some(p);
        }
        if let Some(p) = check(&self.black_knights, PieceType::Knight, Color::Black) {
            return Some(p);
        }
        if let Some(p) = check(&self.black_bishops, PieceType::Bishop, Color::Black) {
            return Some(p);
        }
        if let Some(p) = check(&self.black_rooks, PieceType::Rook, Color::Black) {
            return Some(p);
        }
        if let Some(p) = check(&self.black_queens, PieceType::Queen, Color::Black) {
            return Some(p);
        }
        if let Some(p) = check(&self.black_king, PieceType::King, Color::Black) {
            return Some(p);
        }

        None
    }
```
{{</accordion>}}

---

# Piece Representation:

```rust
#[repr(u8)]
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum Color {
    White = 0,
    Black = 1,
}
```
The Only Confusing Part Might Be The `repr(u8)`. Basically turns out, this just tells rust, "Yo store this enum as a u8 (1 byte) in memory". By default, rust chooses how to store enums in memory based on safety/debug info. This forces it to just 1 byte -- clean & compact.

Furthermore, this apparantly helps if later I am generating bindings for FFI (Foreign Function Interface), which I'm not too sure if I'm gonna do, I kinda want to just port the [disawul's nnue proble](https://github.com/dshawul/nnue-probe) into rust, but if that fails then I have to fall back to FFI.

```rust
#[repr(u8)]
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum PieceType {
    Pawn = 0,
         Knight = 1,
         Bishop = 2,
         Rook = 3,
         Queen = 4,
         King = 5,
}

#[derive(Clone, Copy, PartialEq, Eq)]
pub struct Piece(u8);
```
This is pretty self explanatory too.

```rust
impl Piece {
    pub fn new(piece_type: PieceType, color: Color) -> Self {
        Self((piece_type as u8) | ((color as u8) << 3))
    }

    pub fn piece_type(self) -> PieceType {
        match self.0 & 0b111 {
            0 => PieceType::Pawn,
              1 => PieceType::Knight,
              2 => PieceType::Bishop,
              3 => PieceType::Rook,
              4 => PieceType::Queen,
              5 => PieceType::King,
              _ => unreachable!(),
        }
    }

    pub fn color(self) -> Color {
        if (self.0 & 0b1000) != 0 {
            Color::Black
        } else {
            Color::White
        }
    }

    pub fn is_white(self) -> bool {
        self.color() == Color::White
    }

    pub fn is_black(self) -> bool {
        self.color() == Color::Black
    }
}
```

In this representation the `Piece` is a `u8`. So, something like `00000000`. The first 3 bits represent the piece type eg: `00000101 = King`. and the 4th bit represents the color. So, `00000101 = King` is a black king, and `00001101 = King` is a white king. The rest of the bits, are pretty useless.

##### Alr, so that's pretty much everything
