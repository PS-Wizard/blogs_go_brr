+++
date = '2025-08-05T18:07:46+05:45'
draft = false
title = 'NNUE Theory'
description = 'Figuring Out How NNUEs work, trying to understand it in theory before actually trying to make a library to parse one.'
tags = ['chess', 'rust', 'ai' ,'NNUE']
repo_link=''
code_link=''
+++

# Introduction
Okay, so NNUEs, I know they are from shogi, I know it's used in stockfish, and I know that it's just one single file; that's about all that I know. NNUEs or "Efficiently Updatable Neural Network" seem nice, because my naive approach using the NNUE probe library from [dshawul](https://github.com/dshawul/nnue-probe), is getting me the same evaluations as stockfish, so I'm thrilled, but I don't quite know how it's working and I'd like to roll out my own NNUE probe native to rust. So, bear with me while I try and figure this out.

---

## Where Do I Even Begin?
Okay, so I know it's a neural network, so it's just a bunch of weights right, but okay let's figure out the gimmic, whats so *efficient*  about these *Efficiently Updatable Neural Networks*. Here is what I don't get:

- How does a file even be "Efficiently Updatable", cause its just a simple `*.nnue` file, and I doubt you update those? because well ... I don't think we tinker around with the weights for a neural network? so like ??? 
- How do I go about parsing such a file, like I couldn't find an official
    > *"ayo, this NNUE thing, is in this format, and we do this to read it"* type documentation.

---

## Answers? Hopefully?
Okay, after a not so thorough GPT (*with the study mode btw lol*), 

The *Efficiently Updatable* thing, comes from unsurprisingly not changing the actual `.nnue` , but changing the accumulator it seems. So, NNUEs are just the weights, we load em up, and once we do that then now it makes sense for it to be *Efficiently Updatable*, because after we load em up, it's the long lived program that's responsible for doing all the fancy efficient shit that I don't quite get at the moment.  Okay, it's starting to make a bit more sense now.


Okay, a bit more searching later, here's whats up. NNUEs are simply weights, they are just **raw facts**. You need to make your own stuff on how you deal with the raw facts. So, to deal with the raw facts, there is:

{{< accordion title="The Inputs " >}} 
- So, basically features is the fancy name for providing an input right, basically here is how I'm thinking of it right now, NNUE is simply a `f(x) -> evaluation` , and in this case our input is the `x`. 

But ofcourse like always, they have a catch. In this case, the catch is that we don't feed it the full raw board. I've personally always thought it was just the `FEN` notation that gets converted into like a bitboard state or something, but turns out nah. 

Turns out
> We extract **semantic features** based on **piece-square combinations** anchored to the king, also called the **HalfKP format**.

### Bro tf does that even meaaaan
![Confused Pepe The Frog](/images/memes/confused_pepe.png)


... Okay nvm, it's not that deep, so okay it's just for each side, we build inputs using:
- The king square for that side
- The square and type of every piece of that side

So, basically for our `f(x) -> eval` , the `x = (king_square, piece_type, piece_sqare)` but instead of the tuple like that, we use the following to index the vector

```rust
index = king_sq * (num_piece_types * 64) + piece_type * 64 + piece_sq
```

- `king_sq`: makes sense right, just the square that the king is on
- `num_piece_types`: basically the number of piece types, so `6` in a game of chess (pawns, rooks, knights, bishops, king, queens). **And NO, turns out it's not dynamic**, in the sense that say if white looses both of its bishop, the `num_piece_types` for white doesn't drop down to `5`, it is a constant. Which, I mean, bro why even label it if its a constant lol but sure, makes sense.

- `piece_type`: So, they have an index for each type, Pawns are `0`, Knights are `1`, Bishops are `2`, Rooks are `3`, Queens are `4` and finally the King is `5`. I'm skeptical about this tho, cause no hate ChatGPT, you're good and all, but sometimes you hallucinate, that's all I'm Saying
- `piece_sq`: That's basically the piece's square that we dealing with. Like the `x=(king_square,piece_type,*piece_square**)`
{{</accordion>}}

{{< accordion title="Hidden Layer & The Weights" >}} 

#### What is a weight matrix in a neural network?
Okay, so if you are anything like me, you have no idea what on god's green earth a neural network actually even is. But, for this context, here is what GPT is telling me:
- In a neural network, each layer transforms it's input vector `x` by multiplying it with a matrix of weights. This I get.
- This matrix, is just a big ass 2D table of numbers learnt during training. This I get as well.
- For example, if your input vector `x` has size `N` and your hidden layer has size `M`, then the weight matrix `W` has shape `N x M`. ... Okay, kidna makes sense but it's loosing me a little.
- Multiplying `x` by `W` gives you a vector of size `M`

and okay, so I got a little confused about this whole length, length of the hidden layer thing so okay, 

---

#### What is input size?
- Your input vector is a big array where each position corresponds to one possible feature.
- For NNUE, each feature is identified by `(king_sq, piece_type, piece_sq)`.
- Because the number of squares is 64 and piece types are 6, you calculate the total number of features as:
    - `input_size = 64 (king squares) × 6 (piece types) × 64 (piece squares) = 24,576`
    - So the input vector x is length 24,576, but most entries are zero because only a few pieces exist.
    - You don't actually build a full array of 24,576 values — you only track which features are “active” (non-zero) using that index formula you have.

#### What is hidden layer size, and why does it matter?
- The hidden layer is a layer in the neural network between input and output.
- It has some number of neurons (or units) — this number is called the hidden layer size.
- For example, if hidden layer size = 256, that means the hidden layer output is a vector of length 256.
- The hidden layer size controls the capacity of the network — more neurons = can learn more complex patterns (but slower and heavier).
- The weight matrix from input to hidden is therefore size `input_size × hidden_size` (e.g., 24,576 × 256).
- Each row in that matrix corresponds to one feature’s weights to all 256 hidden neurons.

#### In the context of NNUE what even is the weight_matrix?
Okay, so one thing, before we actually get into this is that our input vector `x` is mostly `0s` because only a few pieces exist on the board. So, if we were to compute `x.W` do we really need to multiply the full vector `x` with the entire weight matrix `W`?. Nah right, cause it just simply makes more sense to prune out the empty inputs, so:
- For each **active input**
- Take the corresponding row in the weight matrix -- which is a vector length `M` ( the hidden size ... bro who comes up with these names tho *tHe HiDdEn sIzE?*)
- And, **sum these rows together**  to get the hidden layer output.

#### Aite, cool jimmy, but where does this `weight_matrix` come from? and how do you even parse it from a `.nnue` file?

Okay, so I think it's pretty clear that the `.nnue` file is pretty much just, **the weights**. So, that's where the `weight_matrix` comes from. But actually parsing it?, in theory it is:
- You read it by reading the weights in the order the NNUE format expects (usually fixed offsets and sizes)
- Each weight vector (for a given input index) is stored as a fixed-size array (often i16 or f32 values).

So, again in theory, we:
- Open the `.nnue` file as a binary file.
- Read the header metadata ( which hopefully contains stuff like the input size, hidden size etc)
- Then read the weight vectors sequentially into our Rust data structure, whichever we decide to go with; which I mean is probably just gonna be a array wrapped in a `Box<>`

# Learning Happening Pls Wait. I'm getting real confused
![Crying pepe](/images/memes/pepe-cry.gif)
Here is what I need to figure out:
- Like i get a `.nnue` file is just the weights right, how do i parse it tho?, im assuming the metadata will contain offsets or atleast size of each "weight"? is that correct?
- What's this hidden layer size, the input layer size thingy. Like i get it's just matrix sizes but for some reason it ... **dont feel right**
- How does stuff correspond like:
    - > Your Input vector is a big array where each position corresponds to one possible features -- tf does this even meann
    - > For example, if hidden layer size = 256, that means the hidden layer output is a vector of length 256. -- tf does this even meaan
    - > Each row in that matrix corresponds to one feature’s weights to all 256 hidden neurons. -- Say what now
    - And like how does this all tie together, because inputs are just well inputs makes sense, hidden layer i think is the weights you parse, and so before you get the eval, turns out there is this another thing called an **activation function** and this **ReLU** which although is super simple, just simply make negatives to 0. 
    - And it also turns out that there is this second weight matrix that you need to multiply with so I gotta figure that out too

{{</accordion>}}
- Output ( The Actual Evaluation)

