+++
date = '2025-09-02T19:11:04+05:45'
draft = false
title = 'Proposal'
description = 'Outline For Proposal Review'
tags = ['school', 'proposal draft']
repo_link=''
code_link=''
image= "/images/chess_engine/chess_cover.png"
+++

![Logo](/images/chess_engine/logo.png)


# Introduction

Chess can be traced back to nearly 1500 years with it's earliest known predecessor called Chaturanga in India. The game of chess has long served as a testing ground for computer science, specially in the study of search algorithms, data structures and decision making under constraints. The world of chess programming has come a long way since the earliest known chess algorithm written by Alan Turing. 

The goal of this project is to build a chess engine entirely from scratch in Rust. The engine will be modular, with core components such as [ Bitboards ](https://www.chessprogramming.org/Bitboards), move generation via [Magics](https://www.google.com/search?client=firefox-b-d&channel=entpr&q=magic+bitboards), Legality checks, evaluation etc being published as a separate crates. Along side the development, the entire process will be documented in this [blog](https://blogs.pswoyam.com.np). 

---

# Problem Statement
While modern chess engines like [Stockfish](https://stockfishchess.org/) and [AlphaZero](https://en.wikipedia.org/wiki/AlphaZero) along with many others do exist, they are all finished products, often backed by thousands of open source contributors. This makes it difficult for learners who are trying to understand how everything ties together, such as how move generation, legality checks, searching, ordering work. Furthermore, there isn't a resource benchmarking different techniques involved in chess programming, that results to the highest performance gain. 

Additionally despite the global progress in chess engine programming, as of today, (September, 02, 2025), there has not yet been a chess engine originating from Nepal, which also leaves a opportunity to contribute to this space. Therefore, the research question is:

> How does each incremental algorithmic enhancement (with techniques such as alpha-beta pruning, iterative deepening, move ordering, endgame tables etc) affect a chess engine’s playing strength and efficiency?


---

# Aims and Objectives

**The Aim:**
To build a chess engine in Rust from scratch and benchmark incremental performance gains by implying different techniques used in chess programming.

**The Objectives:**
- Implement baseline engine ( Bitboards, NNUE based eval function, UCI Protocol Support, Basically an engine that can play chess)
- Incrementally integrate different techniques: ( Subject to be added upon based on the timeline)
    - Minimax / Negamax as the baseline
    - Alpha Beta Pruning
    - Move Ordering ( eg, Captures First, Checks First, Promotions First etc )
    - Quiescence Search
    - Iterative Deepening
- Benchmark each engine generation against it's own predecessor
- Publish modular crates for core components
- Document progress and learning in a public blog.

---

# Preliminary Literature Review
### This the part im confused on
![Crying pepe](/images/memes/pepe-cry.gif)

Chess engines can be categorized into traditional search-based engines like Stockfish and neural network–assisted engines like Leela, Chess Zero. Both types share several core components:

- Some Sort Of Search
- Some Sort Of Pruning
- Some Sort of Move ordering
- Evaluation functions, which are either hand-crafted (as in classic engines) or augmented with neural networks (as in Stockfish with NNUE or Leela Chess Zero with fully trained networks)

Modern chess engines are increasingly using alternative methods, such as Monte Carlo Tree Search (MCTS) paired with fully trained neural networks, as seen in engines like Leela Chess Zero. 

However, this project will focus on a classic search-based approach, implementing Minimax/Negamax as the backbone, enhanced with Alpha-Beta pruning, move ordering heuristics, and Quiescence Search. 

For the evaluation function, Neural network evaluation similar to existing NNUE architecture from Stockfish will be used, keeping the focus on understanding and benchmarking incremental algorithmic improvements. While some high-performance engines also implement parallel search techniques such as Young Brothers Wait Concept (YBWC) or Lazy SMP to leverage multicore processors, this project will focus on sequential search methods for simplicity. Neural network evaluation will use the existing NNUE architecture from Stockfish rather than training a new model from scratch, keeping the focus on understanding and benchmarking incremental algorithmic improvements.






---
# Artifacts 
**Rust Crates For**:
- Bitboards 
- Move Generation
- Evaluation 
- Search 
- UCI protocol
- Blog post documenting each stage

---
# Testing:

As before mentioned, testing will happen based on generation. While other alternatives were considered such as a bot account on the LiChess platform, it was eventaually turned down because of the inconvience of having to have a personal VPS even for testing, with each test spanning hours. 

However, for the first ancestor, to get a rough estimate on it's Elo, It will be facing random people who know how to play chess. Furthermore, techniques like [Perft](https://www.chessprogramming.org/Perft) will be applied to check the correctness of move generation.



---
# Project Management Plan
The project will use Kanban as the framework for task tracking, but following a Waterfall approach for overall planning. This approach is aimed because the requirements for the chess engine are well-defined and unlikely to change, allowing for a structured, phase-based development process while still visually managing tasks in Trello.

The timeline is for ~ 7 months, 10 days.
![Estimated Timeline](/images/chess_engine/Timeline.png)
( Subject to change, as I am increasingly realising how i might have to push UCI to one of the earlier phases)
    
