+++
date = '2025-07-18T21:17:57+05:45'
draft = false
title = 'Initial setup'
description = 'Figuring Out Board + Piece Representation For A Bitboard Implementation'
tags = ["rust", "chess"]
+++

# Board Representation:

The boards are of course Bit Boards, but this time around the pieces are a bit different. Instead of structs, this time I opted for a more compact representation.


### The Actual Board:

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

That's the board, the helper functions are there to well ... help.

### The Game:

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
        // Move Counters
}

```

The game representation is just bitboards and other game states such as castling rights and a bool representing who's turn it is to move. It has 3 functions:
1. `new (constructor)` : Creates a new game, sets all the pieces in their default position
2. `all_pieces()` : Returns all the pieces by simply: `all_white_pieces() | all_black_pieces()`. 
3. `get_piece_at(idx: u8)` : Given an index, this function returns an `Option<Piece>` depending if there is a given piece at the said `idx`


### The Pieces:

```rust
#[repr(u8)]
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum Color {
    White = 0,
          Black = 1,
}
```

>[!INFO]
> **The Only Confusing Part Might Be The `repr(u8)`**
> Basically, this just tells rust, "Yo store this enum as a u8 (1 byte) in memory" . By default, rust chooses how to store enums in memory based on safety/debug info. This forces it to just 1 byte -- clean & compact.

> [!QUESTION]
> **Why Use It?**
> 1. Tight Packing
> 2. And apparantly, this helps if I am generating bindings for FFI (Foreign Function Interface); which I plan to do later.

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

In this representation the `Piece` is a `u8`. So, something like `00000000`. The first 3 bits represent the piece type eg: `00000101 = King`. and the 4th bit represents the color. So, `00000101 = King` is a black king, and `00001101 = King` is a white king.
