+++
date = '2025-09-03T07:20:45+05:45'
draft = false
title = 'Remake_02: ft. Inkfish'
description = 'Remaking The Page Transition In Inkfish'
tags = ['gsap', 'sveltekit', 'inkfish', 'staggered']
repo_link=''
code_link=''
+++

# Introduction
Allrighty, let's get right to it, [InkFishNYC](https://inkfishnyc.com/) got a great looking page transition ... **I WANT IT**.

{{< accordion title="Exited Pepe" open="y">}} 
![Exited pepe the frog](/images/memes/hype-pepe.gif)
{{< /accordion >}}
## Understanding The Idea
So, it's really not that complicated as it looks at first glance. The thing you need to do is have a container that holds a bunch of divs based on screen size. In my case I opted to have a 5x5 grid of divs that fit the screen perfectly. Then before navigating, make the blocks visible, by animating their height; stagger them randomly, and after navigate do the opposite, and voila.


### Step 1: Generating Divs

```svelte
let blocks: { x: number; y: number; w: number; h: number }[] = $state([]);
let divBlocks: HTMLElement[] = $state([]);
onMount(() => {
    const cols = 5;
    const rows = 5;
    const blockW = window.innerWidth / cols;
    const blockH = window.innerHeight / rows;

    blocks = [];
    for (let y = 0; y < rows; y++) {
        for (let x = 0; x < cols; x++) {
            blocks.push({
                x: x * blockW,
                y: y * blockH,
                w: blockW,
                h: blockH,
            });
        }
    }
});
```
and to use it, 


```svelte

<div>
    {#each blocks as block, i}
        <div
            class="absolute bg-black z-10"
            style="
                left: {block.x}px;
                top: {block.y}px;
                width: {block.w}px;
                height: 0px; 
                overflow: hidden;
                pointer-events: none; 
            "
            bind:this={divBlocks[i]}
        ></div>
    {/each}
</div>
```

Likely not the best way to do this, but oh well ( skill issues I know :< , I just didn't wana bother with this so I just gave it to GPT). Either way, once you’ve got your 5x5 grid set up, each div basically acts like a little “tile” that you can animate in and out whenever you navigate between pages. 

### Final Step: Animate The Divs Before And After Navigation

```svelte
onNavigate(async () => {
    return new Promise((res) => {
        gsap.to(divBlocks, {
            height: (i) => blocks[i].h,
            duration: 0.69,
            stagger: { each: 0.03, from: "random" },
            pointerEvents: "auto", 
            onComplete: () => {
                res(() => {
                    gsap.to(divBlocks, {
                        height: 0,
                        stagger: { each: 0.03, from: "random" },
                        duration: 0.69,
                        pointerEvents: "none", 
                    });
                });
            },
        });
    });
});
```

---

Annd that's pretty much it, fairly simple but it looks cool. If you wanna know more about these functions, always refer to the [GSAP Docs](https://gsap.com/docs/v3/) & [SvelteKit Docs](https://svelte.dev/docs/kit/introduction), specifically the docs for [navigation hooks](https://svelte.dev/docs/kit/$app-navigation)


---

# Making The Page Transition Like A Preloader
So, right now we are using the `onNavigate` funtion, and what it does is that it waits for the other page to fully load before the transition, which I mean doesn't make sense right, because the page transition should also behave like a preloader, so to acheive this preloading effect, where the page transitions out -> waits for however long it takes for the other page to load -> transitions in, we need to use the `beforeNavigate` and `onNavigate` funtions:

```svelte
<script lang="ts">
    import { onMount } from "svelte";
    import "../app.css";
    import {
        afterNavigate,
        beforeNavigate,
        goto,
        onNavigate,
    } from "$app/navigation";
    import { gsap } from "gsap";
    let { children } = $props();
    let blocks: { x: number; y: number; w: number; h: number }[] = $state([]);
    let divBlocks: HTMLElement[] = $state([]);
    onMount(() => {
        const cols = 5;
        const rows = 5;
        const blockW = window.innerWidth / cols;
        const blockH = window.innerHeight / rows;

        blocks = [];
        for (let y = 0; y < rows; y++) {
            for (let x = 0; x < cols; x++) {
                blocks.push({
                    x: x * blockW,
                    y: y * blockH,
                    w: blockW,
                    h: blockH,
                });
            }
        }
    });

    onNavigate((navigation) => {
        if (navigation.type == "link") {
        }
    });

    beforeNavigate(async (nav) => {
        if (nav.type === "link") {
            nav.cancel(); // cancel the nav From my testing, without this, the page attempts to first fully load before the animation, thus we stop that default behaviour with this
                         // and proceed with the outro animation
            await gsap.to(divBlocks, { // await transition so that it doesn't overlap with the intro animation if the other page happens to load instantly
                height: (i) => blocks[i].h,
                duration: 0.69,
                stagger: { each: 0.03, from: "random" },
                pointerEvents: "auto",
                onComplete: () => {
                    if (nav.to) {
                        goto(nav.to.url); // and ofcourse, if we cancel the navigation, it needs to resume so doing this will navigate it to where the user intended
                    }
                },
            });
        }
    });

    afterNavigate(async () => {
        gsap.to(divBlocks, { // no need to await here
            height: 0,
            stagger: { each: 0.03, from: "random" },
            duration: 0.69,
            pointerEvents: "none",
        });
    });
</script>

<div>
    {#each blocks as block, i}
        <div
            class="absolute bg-black z-10"
            style="
                left: {block.x}px;
                top: {block.y}px;
                width: {block.w}px;
                height: 0px; 
                overflow: hidden;
                pointer-events: none; 
            "
            bind:this={divBlocks[i]}
        ></div>
    {/each}
</div>
<div class="container">
    <div class="absolute flex justify-center items-center">
        {@render children()}
    </div>
</div>

```

In hindsight, it looks like we could just use `onNavigate` here as well, but the thing is that `onNavigate` doesnt allow us to use `cancel()`. 
