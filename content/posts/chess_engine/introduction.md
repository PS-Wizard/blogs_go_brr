+++
date = '2025-07-18T21:07:45+05:45'
draft = false
title = 'Introduction'
description = 'Rough sketch on the why and the how for building a chess engine'
tags = ["rust", "chess","introduction"]
+++

# Introduction

Welcome to a new series, of me figuring out how to build a chess engine. I'm doing this because chess is something I grew up with and the idea of me building something that could consistently beat me is pretty cool. Kinda like the student becomes the master vibe. 

Furthermore, I need to do something for my Final Year Project too, so this serves that purpose as well. For our Final Year Project, they want us to build something that has meaning, like they don't want us to reinvent the wheel, they want us to have some sort of a selling point. Originally, I planned to make a chess engine from scratch and throw in a "explainer" layer, that basically explains why a certain position is better than the other, kinda like [chess.com's](https://www.chess.com) game review, but instead of like cookie cutter templates, I wanted it to explain like a human would. 

**The only problem?** It needs a dataset — one large enough to train a decent model that can explain, *in human terms*, why one position is better than another. But that ... as far as I know, doesn’t exist. I suspect that’s because only about **10% of the world’s population** (roughly **800 million** people) even know how to play chess — and among them, only around **2,000** are Grandmasters, that's like **0.0025%** of the world's population. 

Grandmasters are the ones capable of actually explaining positions in human language. A dataset of annotated games *with explanations* from them? As far as I know, it just doesn’t exist. And honestly, even if it did — today’s top chess bots destroy any human. So if you trained a model purely on grandmaster commentary, it might only justify “human” moves, not the weird, depth-20 engine stuff that leads to a +0.1 eval shift. It wouldn’t help explain why the bot made *that* move.

So yeah, that idea doesn’t feel too feasible. I’ll probably still toss in a basic game review feature — maybe pipe the PGN into ChatGPT or some other LLM via API — but I don't think its gonna be too good, the quality’s gonna be meh. LLMs kinda suck at playing chess right now. (Try it yourself — don’t be surprised if it invents pawns or eats its own pieces lol.)
So, okay if that's the case then what's my unique thing?

---

## The Goal?
Welp, turns out no one from Nepal has ever made a chess engine. Or, I mean more correctly, no one seems to have made a chess engine from Nepal, published it so that it appears on google. So, my goal, create Nepal's first, and the strongest ( well I mean, if I am the first, then implicitly I am also the strongest, but we don't talk about that) chess engine. 

{{<danger title="So, your selling point is another chess engine?">}}
**yes. bite me lol**. No one's made a chess engine from Nepal, or atleast, one that I can find so, ima look to change that.
{{</danger >}}

But with that being said, I'm not gonna just slack off, because if we are being honest, If i just make a bot and publish it, It is going to be the first bot, and as such the defacto strongest, So even if I make a bot with an elo of 50, I technically would have made the strongest and the first bot from Nepal. Sure that works, but I don't quite like that. So let's setup some ground rules

### The Chess Engine:
It's gotta:
- Atleast 2000 elo
- Support FEN, UCI, PGN all that good stuff
- Bitboards Based & Magic Bitboard
- Made from scratch, no other libraries.
- And for eval, I know we go for NNUEs these days, and I also don't know of any rust crates to parse NNUEs, so I wana make that too. 
- Gotta be hosted on like Lichess as a bot or somethin.

And, on a side note I gotta figure out how the NNUEs work. Efficiently updatable neural network, but I don't seem to find anything relating to how they update over time. Because, like my implementation of this for now, is just a binary compiled from [disawul's nnue proble](https://github.com/dshawul/nnue-probe) and I am just gonna be calling it with `./nnue <fen>` to get an eval. I mean it works but, I don't know if the *Efficiently Updatable* Neural Network is even *Efficiently Updating*.

### This Blog:
This blog aims to be well basically me figuring stuff out. I am new to rust and I am new to chess programming, so this is simply rambling and figuring this out and hopefully of being help to someone else taking on this journey down the line. 
Basically my version of the: [Rustic Chess Engine's Blogs](https://rustic-chess.org/front_matter/title.html)

---

## The Plan?
Have I Figured Everything Out Yet? Hell naw, but the rough idea is Individual Crates:
- `arena`: Board & Piece Representations
- `tactition`: Move Gen 
    - `magician`: Move Gen Magic Bitboards
- `translator`: Notations, so stuff like FEN, UCI, PGN yada yada
- `warden`: Legalities, so stuff like checkmates, stalemates, 50 move rules etc.
- `prophet`: Evaluation, So NNUE + MiniMax / NegaMax etc + Some Ordering + Some Pruning
