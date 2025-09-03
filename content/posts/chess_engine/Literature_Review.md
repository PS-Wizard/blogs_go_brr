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

# Research Question

While modern chess engines like [Stockfish](https://stockfishchess.org/) and [AlphaZero](https://en.wikipedia.org/wiki/AlphaZero) along with many others do exist, they are all finished products, often backed by thousands of open source contributors. This makes it difficult for learners who are trying to understand how everything ties together, such as how move generation, legality checks, searching, ordering work. Furthermore, there isn't a resource benchmarking different techniques involved in chess programming, that results to the highest performance gain. 

Additionally despite the global progress in chess engine programming, as of today, (September, 02, 2025), there has not yet been a chess engine originating from Nepal, which also leaves a opportunity to contribute to this space. Therefore, the research question is:

> How does each incremental algorithmic enhancement (with techniques such as alpha-beta pruning, iterative deepening, move ordering, endgame tables etc) affect a chess engine’s playing strength and efficiency?


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
    
