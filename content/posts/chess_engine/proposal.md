+++
date = '2025-09-02T19:11:04+05:45'
draft = false
title = 'Proposal'
description = 'Proposal For Chess Engine'
tags = ['school', 'proposal']
repo_link=''
code_link=''
image= "/images/chess_engine/chess_cover.png"
+++

![Logo](/images/chess_engine/logo.png)

# Introduction
Chess has long served as a testing ground for computer science, specially in the study of search algorithms, data structures and decision making under constraints. 

**OopsMate** aims to be a chess engine written from scratch in Rust. The engine aims to have a baseline [Elo Rating](https://en.wikipedia.org/wiki/Elo_rating_system ) of at-least 1800. The engine itself will follow a more "classical" approach to chess engines, and the main goal of this project is to explore and benchmark different techniques used in chess programming to enhance engine performance.

# Problem Statement
This project aims to fill the gap in couple a couple of things. Firstly, although chess engines like [Stockfish](https://stockfishchess.org/), [Alpha Zero](https://www.chess.com/terms/alphazero-chess-engine) and many others do exist, they are all finished products, often open sourced and backed by thousands of open source contributors or proprietary engines in private development. As such, it is often hard for new comers to answer "*how did they make this engine*", or how different components in chess engines such as [Ordering](https://www.chessprogramming.org/Move_Ordering), [Evaluation](https://www.chessprogramming.org/Evaluation), [Search](https://www.chessprogramming.org/Search) etc tie together. Furthermore, there isn't a resource directly benchmarking these different techniques for the most significant increase in each component. Finally, despite the global progress in chess engine programming, as of (September 03, 2025) there is yet to be a chess engine developed in the country Nepal, which opens up room for contribution. 

This project, therefore, aims to answer the following question:

> How different techniques involved in chess programming affect a chess engine's playing strength and efficiency

The entire development will be documented [here](https://blogs.pswoyam.com.np/), providing newcomers with a comprehensive resource to understand and implement a fully functional chess engine. 

---

# Aims and Objectives
**The Aim**
To build a chess engine in Rust from scratch and benchmark incremental performance gains by implying different techniques used in chess programming.

**The Objectives**
- Implement baseline engine
    - [BitBoards](https://www.chessprogramming.org/Bitboards)
    - [NNUE](https://www.chessprogramming.org/NNUE)
    - [Magics](https://www.chessprogramming.org/Magic_Bitboards)
    - [Negamax](https://www.chessprogramming.org/Negamax)
    - [FEN](https://www.chessprogramming.org/Forsyth-Edwards_Notation)
    - [UCI](https://www.chessprogramming.org/UCI)
    
- Incrementally integrate different techniques:
    - [Evaluation](https://www.chessprogramming.org/Evaluation)
        - [NNUE](https://www.chessprogramming.org/NNUE)
        - HCE (Hand Crafted Evaluation)
    - [Pruning](https://www.chessprogramming.org/Pruning):
        - [Alpha Beta Pruning](https://www.chessprogramming.org/Alpha-Beta)
        - [Null Move Pruning](https://www.chessprogramming.org/Null_Move_Pruning)
        - Depending on time constraints I'd like to play around with [Futility Pruning](https://www.chessprogramming.org/Futility_Pruning) too
    - [Move Ordering](https://www.chessprogramming.org/MVV-LVA)
        - Captures First (eg. [MVV-LVA](https://www.chessprogramming.org/MVV-LVA)),
        - Promotions First 
    - Other Extensions:
        - [Quiescence Search](https://www.chessprogramming.org/Quiescence_Search)
        - [Iterative Deepening](https://www.chessprogramming.org/Iterative_Deepening)
    - Benchmark each engine generation against it's own predecessor
    - Document progress and learning in a public blog.

    ---

# Research
This project aims to be a more "classical" chess engine, focusing on building a solid baseline and then incrementally integrating and benchmarking established techniques.  Each chosen technique is selected for it's proven effectiveness in chess programming. The project also emphasizes benchmarking each iteration against its predecessor to objectively measure improvements, and documenting progress to contribute to both personal learning and the broader chess programming community. 

### Board Representation
Representing the board is the foundational step in setting up the path for the rest of the chess engine. There are two different main types of board representation, **Piece Centric** and **Square Centric**. 

#### Square Centric
This approach focuses on the board itself, typically using a 2D (eg. [8x8](https://www.chessprogramming.org/8x8_Board)) or a 1D (eg. [64](https://lichess.org/@/likeawizard/blog/review-of-different-board-representations-in-computer-chess/S9eQCAWa#one-dimensional-64array)) array where each square stores what piece occupies it, if any. It is particularly useful because it is the most intuitive, easy to visualize and simple to implement as it's basically just `board [8][8]`. But, it struggles large scale computations like generating all moves for multiple pieces. 

#### Piece Centric
In this approach, each piece type is tracked separately, often using techniques like [BitBoards](https://www.chessprogramming.org/Bitboards), and they shine making certain operations like generating moves or checking attacks on a square faster. 

This project aims to use the [BitBoards](https://www.chessprogramming.org/Bitboards) representation. This is because BitBoards are performant because they allow **parallel bitwise operations** such as `AND`, `OR`, `NOT` to quickly set or query the game states [-> 1](https://en.wikipedia.org/wiki/Bitboard). This, paired with the fact that most architectures these days are 64 bits, and conveniently enough there are exactly 64 squares in a chess board. This alignment makes it so that each bitboard fits in a single machine [word](https://en.wikipedia.org/wiki/Word_(computer_architecture). Furthermore, the use of bitboards enables the use of a technique called [Magics](https://www.chessprogramming.org/Magic_Bitboards), which allows move generation for sliding pieces like queens, bishops and rooks to be constant time `O(1)` 


### Evaluation
For a engine to come up with good moves, it needs to first know what makes a position good or bad. Each engine defines a score function: `eval(position) -> number`, but there are different ways to do so. Initially, the evaluation function was handcrafted (HCE) which considered factors such as: `material`, `piece-square tables`, `king safety`, `mobility` etc. These metrics were often weighted so that it would look like this at the end:
```python
    eval = 9*queens + 5*rooks + 3*knights + ... + mobility_bonus + king_safety_penalty
```
However, a newer approach to this evaluation functions are efficiency updatable neural networks; [NNUEs](https://www.chessprogramming.org/NNUE) or sometimes styled as ƎUИИ. Originally from [Shogi](https://en.wikipedia.org/wiki/Shogi), this technique has made it's way to the world of chess programming, with one of the first introductions of it being in [Stockfish](https://www.chessprogramming.org/Stockfish_NNUE). NNUEs learn to evaluate positions from games instead of handcrafted weights, where the inputs are usually - [BitBoards](https://www.chessprogramming.org/Bitboards) or piece-square occupancy maps and the output is a score for the position. They are fast enough to update incrementally, during move search, hence the name "*Efficiently Updatabale* neural networks". 

Finally, engines like [Alpha Zero](https://www.chess.com/terms/alphazero-chess-engine) use pure reinforcement learning, and their evaluation of position comes from self play. 
This project aims to explore both classical HCE and modern NNUE approaches for the evaluation function, while deliberately not pursuing the self-play alternative due to its substantial computational requirements and the extensive data and infrastructure necessary for effective implementation.

### Search
To search for the best move, chess engines explore potential sequences of moves in the background and call the evaluation function, to get a metric of the position. The core challenge is that the number of possible move sequences grows exponentially with depth. For a zero sum game like chess, MinMax is the natural approach, it recursively evaluates the game tree by alternating between maximizing the score from the evaluation function for itself, and minimizing the score for the opponent.  [Russell, S., & Norvig, P. (2021)](http://lib.ysu.am/disciplines_bk/efdd4d1d4c2087fe1cbe03d9ced67f34.pdf). A commonly used variation is [Negamax](https://www.chessprogramming.org/Negamax), which simplifies the implementation of Minimax by taking advantage of the zero-sum property of chess. 

However, while naive Minimax or Negamax evaluates all nodes in the game tree, engines improve upon this using pruning techniques such as:
    - Alpha beta pruning, which eliminates branches that cannot affect the final decision. [-> 2](https://www.netlib.org/utk/lsi/pcwLSI/text/node351.html)
    - Null Move Pruning, which assumes doing nothing ( passing a move )  is worse than playing the *best possible legal move*, allowing engines to prune branches that are unlikely to improve the position [-> 3](https://www.chessprogramming.org/Null_Move_Pruning#core-idea)

This project aims to explore both of these techniques to improve search efficiency while maintaining optimal decision making. 

### Other Extensions
To further enhance search quality and efficiency, modern chess engines implement additional techniques beyond standard pruning and Negamax/Minimax. 
- [ Quiescence Search ](www.chessprogramming.org/Quiescence_Search): This technique extends the search at nodes for "noisy" positions, such as captures or checks to avoid the [horizon effect](https://en.wikipedia.org/wiki/Horizon_effect). 

- [ Iterative Deepening ](https://en.wikipedia.org/wiki/Iterative_deepening_depth-first_search): This technique repeatedly  searches the game tree to increasing depths, using results from shallower searches to guide move ordering in deeper searches. This improves pruning efficiency and allowing "anytime behavior", which basically means that the engine can return the last analyzed best move even if interrupted, say in a time based setting. 

This project aims to integrate both of these techniques to improve move selection under practical constraints. 
### Communication & Notations
To standardize this engine, the project aims to integrate the standard protocol [UCI](https://www.chessprogramming.org/UCI) and as such, it's dependency [FEN](https://www.chessprogramming.org/Forsyth-Edwards_Notation). 

---

# Artifacts

**Rust Crates For**:
- Bitboards 
- Move Generation Using Magic Bitboards + Look Up Tables
- Evaluation 
    - NNUE
    - HCE
- Search 
    - Alpha-Beta Pruning
    - Null Move Pruning
- Ordering
    - MVV-LVA,
    - Promotions First 
- UCI protocol & FEN

Along with blog post documenting the each stage.

---
# Testing
Testing will proceed generation by generation. While alternatives such as using a bot account on the [Lichess](https://lichess.org/) platform were considered, this approach was ultimately set aside due to the logistical and financial challenge of maintaining a personal VPS for extended testing sessions, each potentially lasting several hours.

For the initial baseline engine, a rough estimate of its Elo will be obtained by having it play against human opponents of varying skill levels. Additionally, techniques like [Perft](https://www.chessprogramming.org/Perft) will be used to verify the correctness of move generation. 

---
# Project Management Plan
The project will use Kanban as the framework for task tracking, but following a Waterfall approach for overall planning. This approach is aimed because the requirements for the chess engine are well-defined and unlikely to change, allowing for a structured, phase-based development process while still visually managing tasks in Trello.

The timeline is for ~ 7 months, 10 days.
![Estimated Timeline](/images/chess_engine/Timeline.png)
( Subject to change, as I am increasingly realising how i might have to push UCI to one of the earlier phases)
    
---
# Diagram Showing The Overall Program Flow

![Design Diagram](/images/chess_engine/design_diagram.png)
