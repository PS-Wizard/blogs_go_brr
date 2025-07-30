+++
date = '2025-07-19T11:06:12+05:45'
draft = false
title = 'Move Generation: Magic Bitboards'
description = 'Exploring Magic Bitboards, how they function, what they solve, and generating moves for sliding pieces + packaging it all into a handy crate for everyone.'
tags = ['rust', 'chess', 'magic bitboards', 'move generation']
code_link='[https://github.com/PS-Wizard/EvalDeez/tree/main/crates/magician](https://github.com/PS-Wizard/EvalDeez/tree/main/crates/magician)'
+++

# Magic Bitboards

Magic Bitboards are a go-to technique in modern bitboard engines for generating moves for sliding pieces. They’re used in top-tier engines like [Stockfish](https://www.chessprogramming.org/Stockfish), [Crafty](https://www.chessprogramming.org/Crafty), and [Arasan](https://www.chessprogramming.org/Arasan), and are widely regarded as one of the fastest ways to generate legal moves.

This post aims to break down the concept of Magic Bitboards—what they are, how they work, and why they're worth the effort. I’m still getting my feet wet with this stuff myself, so I’ll explain it in a way that hopefully clicks for anyone just diving in.

---

## The Basic Approach

Let’s begin with how move generation typically works for sliding pieces.

For sliding pieces like rooks and bishops, the first instinct is to generate moves using a loop in the piece’s movement directions until a blocking piece is encountered. For rooks, these directions are `[8, -8, 1, -1]`. For bishops, `[9, 7, -9, -7]`. These numbers correspond to bitboard offsets.

![Chess Board](/images/chess_engine/chess_board.png)

Here's how a bishop on `e4` (index `28`) would move:

* `d5` (index 35): offset `+7`
* `f5` (index 37): offset `+9`
* `d3` (index 19): offset `-9`
* `f3` (index 21): offset `-7`

Any additional reachable squares like `c6` are just further steps in those directions. This pattern holds no matter the starting square—until a piece blocks the path.

![Bishop Offsets](/images/chess_engine/bishop_offests.png)

So, in a simple loop, we step along each of the 4 diagonal directions for a bishop (or 4 orthogonal ones for a rook) and stop when blocked. That’s your basic sliding move generation.

{{<note title="Note About Queens">}}
We don't generate queen moves separately. A queen's movement is just the union of a bishop’s and a rook’s moves.
{{</note>}}

---

## The Bottleneck

While looping works fine for a human player or a simple one-off calculation, it becomes inefficient when generating millions of moves per second during deep search trees. That’s where this approach falls short. It simply isn’t fast enough.

So how do we make it fast?

> When in doubt, use a hash map. If you're still in doubt, use two.

Let’s say we try using a hash map. We make the key the bitboard representing all occupied squares, and the value the attack bitboard. Conceptually great. But practically?

64 bits means `2^64` possible combinations.

That’s about `1.8e+19` entries. Not feasible at all.

---

## The Key Insight

We don’t need to consider the whole board. A sliding piece is only influenced by the blockers *along its movement rays*—not the entire board state.

This reduces the number of relevant bits to 13 or 14 per square:

* Bishop: 13 potential blocker squares -> `2^13 = 8192` possibilities
* Rook: 14 -> `2^14 = 16384`

{{<accordion title="Why 13 and 14 blocker squares?">}}
Each square a piece can move to lies along a ray. For central squares, bishops have 13 such squares along diagonals, rooks have 14 along ranks and files. Each square can either be empty or blocked, so we get `2^n` blocker configurations.
{{</accordion>}}

That’s way more manageable.

---

## Building the Magic

To use Magic Bitboards, we need:

* A function to enumerate all blocker configurations
* A function to generate attack maps for each configuration

### Generating Blocker Configurations

```rust
pub fn enumerate_blocker_configs(mask: u64) -> Vec<u64> {
    let mut relevant_bits = Vec::new();
    for i in 0..64 {
        if (mask >> i) & 1 == 1 {
            relevant_bits.push(i);
        }
    }

    let num_bits = relevant_bits.len();
    let num_configs = 1 << num_bits;
    let mut configs = Vec::with_capacity(num_configs);

    for i in 0..num_configs {
        let mut blocker = 0u64;
        for j in 0..num_bits {
            if (i >> j) & 1 == 1 {
                blocker |= 1u64 << relevant_bits[j];
            }
        }
        configs.push(blocker);
    }

    configs
}
```

This function takes a mask (of relevant squares) and spits out every possible blocker configuration using binary counting.

### Generating the Mask

```rust
pub fn rook_occupancy_mask(square: u8) -> u64 {
    let rank = square / 8;
    let file = square % 8;
    let mut mask = 0u64;

    for r in (rank + 1)..7 {
        mask |= 1u64 << (r * 8 + file);
    }
    for r in (1..rank).rev() {
        mask |= 1u64 << (r * 8 + file);
    }
    for f in (file + 1)..7 {
        mask |= 1u64 << (rank * 8 + f);
    }
    for f in (1..file).rev() {
        mask |= 1u64 << (rank * 8 + f);
    }

    mask
}
```

```rust
pub fn bishop_occupancy_mask(square: u8) -> u64 {
    let rank = square / 8;
    let file = square % 8;
    let mut mask = 0u64;

    for (dr, df) in [(-1, -1), (-1, 1), (1, -1), (1, 1)] {
        let mut r = rank as i8 + dr;
        let mut f = file as i8 + df;

        while (1..7).contains(&r) && (1..7).contains(&f) {
            mask |= 1u64 << (r as u8 * 8 + f as u8);
            r += dr;
            f += df;
        }
    }

    mask
}
```

---

### Generating Attack Maps

```rust
pub fn rook_attacks_from(square: u8, blockers: u64) -> u64 {
    let rank = square / 8;
    let file = square % 8;
    let mut attacks = 0u64;

    for r in (rank + 1)..8 {
        let sq = r * 8 + file;
        attacks |= 1u64 << sq;
        if (blockers >> sq) & 1 == 1 { break; }
    }
    for r in (0..rank).rev() {
        let sq = r * 8 + file;
        attacks |= 1u64 << sq;
        if (blockers >> sq) & 1 == 1 { break; }
    }
    for f in (file + 1)..8 {
        let sq = rank * 8 + f;
        attacks |= 1u64 << sq;
        if (blockers >> sq) & 1 == 1 { break; }
    }
    for f in (0..file).rev() {
        let sq = rank * 8 + f;
        attacks |= 1u64 << sq;
        if (blockers >> sq) & 1 == 1 { break; }
    }

    attacks
}
```

```rust
pub fn bishop_attacks_from(square: u8, blockers: u64) -> u64 {
    let rank = square / 8;
    let file = square % 8;
    let mut attacks = 0u64;

    for (dr, df) in [(-1, -1), (-1, 1), (1, -1), (1, 1)] {
        let mut r = rank as i8 + dr;
        let mut f = file as i8 + df;

        while (0..8).contains(&r) && (0..8).contains(&f) {
            let idx = r as u8 * 8 + f as u8;
            attacks |= 1u64 << idx;
            if blockers & (1u64 << idx) != 0 { break; }
            r += dr;
            f += df;
        }
    }

    attacks
}
```

---

## The Actual Magic

Now that we have all blocker configurations and attack maps, we want a way to map each blocker config to a unique index into an array of precomputed attack maps. That’s where the magic number comes in.

```rust
fn generate_sparse_u64(min_bits: u32, max_bits: u32) -> u64 {
    let mut rng = rand::rng();
    let num_bits = rng.random_range(min_bits..=max_bits);
    let mut candidate = 0;
    let mut set_bits = 0;
    while set_bits < num_bits {
        let bit = 1 << rng.random_range(0..64);
        if candidate & bit == 0 {
            candidate |= bit;
            set_bits += 1;
        }
    }
    candidate
}
```

```rust
pub fn find_magics_number(square: u8, mask: &u64) -> u64 {
    let relevant_bits = mask.count_ones();
    let blocker_configs = enumerate_blocker_configs(*mask);
    let shift = 64 - relevant_bits;
    let table_size = 1 << relevant_bits;
    let index_mask = table_size - 1;

    'search: for _ in 0..1_000_000 {
        let magic_candidate = generate_sparse_u64(6, 10);
        let mut used_indices = vec![false; table_size];

        for &blockers in &blocker_configs {
            let index = (blockers.wrapping_mul(magic_candidate) >> shift) as usize & index_mask;

            if used_indices[index] {
                continue 'search;
            }
            used_indices[index] = true;
        }

        return magic_candidate;
    }
    panic!("No magic number found for square {}", square);
}
```

This brute-forces random sparse `u64` values until we find one that produces a perfect hash for all blocker configs.

---

And that's it. That's the core of how Magic Bitboards work. The rest is wiring it all together—writing to disk, loading the tables, etc.—which you can find in the full implementation [here](https://github.com/PS-Wizard/EvalDeez/tree/main/crates/magician).

Hope this helped clarify what the magic is all about.
