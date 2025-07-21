+++
date = '2025-07-19T11:06:12+05:45'
draft = false
title = 'Move Generation: Magic Bitboards'
description = 'Exploring Magic Bitboards, how they function, what they solve, and generating moves for sliding pieces + packaging it all into a handy crate for everyone.'
tags = ['rust', 'chess', 'magic bitboards', 'move generation']
code_link='https://github.com/PS-Wizard/EvalDeez/tree/main/crates/magician'
+++

# Magic Bitboards:
Magic Bitboards, a defacto standard of modern bitboard engines, as used for instance in [Stockfish](https://www.chessprogramming.org/Stockfish) [Crafty](https://www.chessprogramming.org/Crafty) [Arasan](https://www.chessprogramming.org/Arasan) etc. It is a technique to generate moves for **sliding pieces**, and it is one of, if not the fastest way to do so. In this blog, I will attempt to explain the concept of Magic Bitboards, how they work, why even bother with one. Although, I am new to this myself, perhaps that's a good thing, that way I can explain it in a way, that will perhaps "click" to someone starting out. Truth be told, I still am not sure if the way I understand magic bitboards is correct, but it makes sense to me, and made sense enough to spin up a correct implementation, so Ima just roll with it. Allrighty, with that said, lets get started:

---

## The Normal Approach
To understand Magic Bitboards, let's first understand how we generate moves normally.

So, our first intuition when tasked with generating moves for any sliding piece, is to use a loop. Basically, from any given square, we move in predefined directions, until there is a piece blocking our path. So for rooks, that would be `[8,-8,1,-1]`, for bishops, that would be `[9, 7, -9, -7]`. Those numbers might not make sense instantly, so let's look at it in a chess board:
![Chess Board](/images/chess_engine/chess_board.png)
In this board view, white pieces are shown with uppercase letters and black with lowercase—though that detail isn't important here. What's key is that we're looking at the board from white's perspective, and each square along the a-file (leftmost column) is labeled with its corresponding bit index in a 64-bit bitboard, starting from the bottom-left (a1 = 0) to the top-left (a8 = 56). With that out of the way, let's look at the bishop's offsets: [9,7,-9,-7]. 
![Bishop Offsets](/images/chess_engine/bishop_offests.png)

So, say a bishop is at `e4` (which is the `28th` index in bitboard terms), and we're only focusing on its diagonal moves.
A bishop moves in four diagonal directions, and from e4, the possible immediate steps are:

- `d5` (index `35`) — offset `+7` (35 - 28 = 7)
- `f5` (index `37`) — offset `+9` (37 - 28 = 9)
- `d3` (index `19`) — offset `-9` (19 - 28 = -9)
- `f3` (index `21`) — offset `-7` (21 - 28 = -7)
- and all other reachable squares, like `c6`, are just further steps in those same directions — for example, `c6` is offset `+7 * 2`. In other words, every square the bishop can move to lies along a **multiple of one of those base offsets** (`+7`, `+9`, `-7`, `-9`).

These offsets are consistent no matter where the bishop is, as long as you avoid wrapping off the board. In the diagram, we’ve marked those offsets `[7, 9, -7, -9]` directly on the adjacent diagonal squares from `e4`. All other reachable squares further along the diagonals are marked with `X`, showing the bishop's full potential path from that square. 

So, basically all we do is loop in each direction with said offsets until we are blocked; and that's how move generation works for bishop, and pretty much every other sliding pieces.

{{<note title="Note About Queens">}}
This is a good time to mention that we don’t explicitly generate moves for queens. Since a queen's movement is just a combination of a bishop’s and a rook’s, we simply generate moves as if a bishop and a rook were on the queen’s square, then combine the two results.
{{</note>}}

---

## The Problem?
Okay, this makes sense right? just loop in each direction based on a predefined offset, until you meet a piece that blocks you. This works, so why even bother with anything else?.

While it's fine for a human or a one-off move, it quickly becomes a **performance bottleneck** when the computer starts evaluating positions deeper in the **search tree**. Engines need to generate **millions of moves a second**, and this step-by-step offset approach starts to slow things down fast.

So, how do we speed things up?

>  When in doubt, use a hash map. If you're still in doubt, use two. 
> – *Probably not Dustin Poirier (I’m watching his final match rn)*

Like every good dev, let’s throw a hashmap at the problem. I mean, `O(1)` lookup time, right? Sounds like a win.

So what if we made the **key** the entire board state, like `all_white_pieces() | all_black_pieces()` in bitboard terms, and the **value** the possible attacks from a piece? Easy. Cool idea ... until you realize you’d need to generate **every possible board combination**. That’s like... a big hashmap, and I mean a BIIIG hashmap. 

If you use the entire board as the key:
- 64 squares, each can either be `0` (empty) or `1` (occupied)
- that's `2^64 = 1.8446744e+19` possible keys -- aka your hashmap is now way bigger than ~~DEEEEZ NUTTS~~ any storage.

### Yikes, so that doesnt work

But wait — if we think about it, we **don’t care** about the entire board now do we?. Because all that really affects a sliding piece's moves is the blockers *along piece's possible paths*. We only care 'bout:

- What **blocks** the sliding piece in the direction it wants to move?

That's a much smaller, manageable input space. That brings us down from `2^64 = 1.8446744e+19` possible keys to a max of `2^13 = 8192` possible keys for a bishop, or `2^14` for a rook **per square** . 

{{<accordion title="If you are confused about where the `2^13` and `2^14` came from">}}

Each sliding piece moves along certain directions called **rays** — for a bishop, these are the diagonals; for a rook, the ranks and files.

- For a bishop on a central square, there are up to **13 squares** along its diagonal rays that could potentially block its movement.
- For a rook, it’s up to **14 squares** along the rank and file rays.

Since each of these squares can either be **empty (0)** or **occupied (1)**, the number of possible blocker configurations along those rays is:

* Bishop: `2^13 = 8192` possible blocker patterns
* Rook: `2^14 = 16384` possible blocker patterns

By only considering these blockers **along the relevant rays** for the sliding piece, instead of the entire board, we massively reduce the number of keys we need to handle. This is the key insight behind magic bitboards.
{{</accordion>}}

### Alright, that makes sense — let’s give it a shot:

Here’s what we need:

* A function to **generate the keys**
* A function to **generate values** for those keys

In this case, “generating values for the keys” means a function that, given a **blocker configuration** (the pieces blocking a sliding piece’s path), produces all the valid moves for that setup.


# ... Damn you're here early, pls wait while I finish this :P
