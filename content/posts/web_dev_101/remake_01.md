+++
date = '2025-09-01T07:53:18+05:45'
draft = false
title = 'Remake_01'
description = "Remaking The Page Transition From 'The Art Of Documentary'"
tags = ['gsap', 'page transition', 'sveltekit']
repo_link=''
code_link=''
+++

# Introduction

Okay, so this website [The Art Of Documentary](https://theartofdocumentary.com/) has this really cool animation where one page slides up, and another one takes it's place but, the catch is that both pages are visible, during the transition, and it gives a really unique effect. You have to check it out yourself to get what I mean since I can't actually show a video over here ( im on vercel free plan :< ), but I will be showing how to acheive a similar effect with `in:` and `out:` directives in svelte.

## Step 1: Making Routes Accessible
So, thinking this through, here's what we need, on **route change** we need to do a custom `out:` transition for the container holding the current page, and a custom `in:` transition for the page coming in. Simple enough, so first thing is we need to make use of the `#key` block. The `#key` block, destroys the content inside of it, and recreates it when the value of the expression changes, in our case, when the route of the page changes, this way, we trigger the `in:` and `out:` transitions. So, first things first, in our root `+layout.ts` we throw in the following
```svelte
export const load = ({ url }) => {
    return { pathname: url.pathname };
};
```
so, each other page has easy access to the current url, granted that we could probably have just used this in our `+layout.svelte` too since no other pages actually use this but still.

## Step 2: The `#key` block

So, now that we have our route accessible, we plop this in our `+layout.svelte` file:

```svelte
<div class="container">
    {#key data.pathname}
        <div
            class="absolute flex justify-center items-center"
            in:animateIn
            out:animateOut
        >
            {@render children()}
        </div>
    {/key}
</div>
```
where, the `data.pathname` comes from:
```svelte
<script>
    let { data, children } = $props();
</script>
```

## Step 3: The `AnimateOut` function
For me it helped thinking the outro animation first, because the effect isnt visible on first page load, although its super easy to do as well, it's basically just `onMount(animateIn)`. But, actually breaking down the animation, it has 3 steps:
- Zoom out / Scale down
- Slide Out
- While Another Page Slides In.
With that, we have

```typescript
function animateOut(node: HTMLElement): TransitionConfig {
    let tl = gsap.timeline();

    // step 1: scale down
    tl.to(node, {
        scale: 0.9,
        duration: 0.3,
        ease: "sine.out",
    });

    // step 2: move up after scale
    tl.to(
        node,
        {
            y: "-100vh",
            duration: 0.75,
            ease: "sine.in",
        },
        "-=0.2",
    ); // negative offset = overlap

    const total = tl.duration(); // total duration for the animation, in seconds.
    return {
        duration: total*1000, // svelte expects ms , 
        tick: (t: number, u: number) => {
            tl.progress(u);
        },
    };
}
```
and that's basically it for our outro animation

## Step 4: Animating In
Almost identical to the `animateOut` function, except we just go the other way right, 

```svelte
function animateIn(node: HTMLElement): TransitionConfig {
    let tl = gsap.timeline();
    gsap.set(node, { y: "100vh", scale: 0.9 });

    // 1. slide in (with your delay)
    tl.to(node, {
        y: "0vh",
        duration: 0.65,
        delay: 0.4,
        ease: "sine.out",
    });

    // 2. zoom in after slide (slightly overlapping)
    tl.to(
        node,
        {
            scale: 1,
            duration: 0.3,
            ease: "sine.in",
        },
        "-=0.25",
    );

    const total = tl.duration();
    return {
        duration: total * 1000, // keep your total duration
        tick: (t: number) => tl.progress(t),
    };
}
```
Everything here is pretty identical, the only thing is the `delay` when sliding in, this is so that the previous animation that's running; our `animateOut` has enough time to slide out enough for this sliding animation to not fully overlap.

---
## Step 5: Use it

That's basically it for our animations, now in every child component, have something like:


```svelte
<div class="relative h-screen w-screen bg-neutral-900 text-white flex flex-col justify-between overflow-hidden z-8" >
    <img src="https://images.unsplash.com/photo-1740686004244-e9bc7c75d8e5?fm=jpg&q=60&w=3000&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLWxpa2VkfDExfHx8ZW58MHx8fHx8"
        alt="background"
        class="absolute inset-0 w-full h-full object-cover brightness-69 z-4"
    />

    <!-- Overlay content -->
    <div class="relative z-10 flex flex-col justify-between h-full">
        <Navbar text="AWWWARDS Remake" number={1} />

        <div class="h-[70vh] flex flex-col justify-center items-center gap-8">
            <h1 class="uppercase text-7xl">Another Page</h1>
            <p class="max-w-3xl text-center">
                Lorem ipsum dolor sit amet consectetur adipisicing elit.
                Assumenda reprehenderit vel eligendi aut repudiandae laudantium
                nesciunt. Nesciunt eligendi facere itaque reiciendis
            </p>
            <div class="flex flex-col">
                <a href="/" class="px-4 py-2 bg-white rounded-lg text-black">
                    Click Me
                </a>
            </div>
        </div>

        <Footer link="https://theartofdocumentary.com/">
            The Art Of Documentary.
        </Footer>
    </div>
</div>
```
and youre good to go, the child pages can have basically anything so this part isnt all that important.
