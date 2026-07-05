---
title: "How a $5M Bet Saved Nvidia Before It Ever Built an AI Chip"
description: "The 1997 bet that let Nvidia survive long enough to build the chips training AI today. What it means for shipping AI infrastructure under real pressure."
date: "05/07/2026"
author: "Netanel Eliav"
series: "AI Strategy"
tags: ["Nvidia", "AI chips", "AI infrastructure", "hardware history", "decision-making", "production AI", "London"]
slug: "riva-128-chip-that-saved-nvidia"
featured_image: "https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/riva-128-chip-that-saved-nvidia.jpeg"
featured: true

faqs:
  - q: "Did Nvidia really almost go bankrupt before it built anything AI-related?"
    a: "Yes, and the numbers are specific, not folklore. Nvidia's first chip, the NV1, shipped 250,000 units to Diamond Multimedia in 1995. Roughly 249,000 came back. The company had bet its architecture on quadratic surfaces instead of the triangle-based rendering that Microsoft's DirectX standardized on, and the market rejected it. Nvidia then spent over a year on the NV2, a chip Sega had contracted for the Dreamcast, before Jensen Huang concluded it wouldn't meet Sega's requirements. He flew to Japan, told Sega's leadership the chip wouldn't work, asked to be released from the contract, and then asked Sega to pay the final milestone anyway. Sega America's CEO, Shoichiro Irimajiri, agreed. That payment, reported at around $5 million, arrived when Nvidia had roughly one month of payroll left. This is 29 years before Nvidia's data center chips would make up the majority of its revenue. It didn't make Nvidia profitable. It bought the runway to build one more chip. That chip had to work."

  - q: "What was Nvidia's NV1, and why did it fail?"
    a: "The NV1 was Nvidia's first graphics chip, released in 1995, combining 2D, 3D, and audio on one card. Its core technical bet was quadratic texture mapping, rendering curved surfaces directly, instead of the triangle-based polygon rendering the rest of the industry was converging on. When Microsoft standardized DirectX around triangles, the NV1 was architecturally incompatible with where every game studio was heading. Diamond Multimedia had ordered 250,000 units. About 249,000 were returned. Nvidia laid off most of its staff and shrank to roughly 40 people. The failure wasn't a marketing problem or a manufacturing defect, it was a bet on the wrong compute primitive at the exact moment the industry was converging on the opposite one. That's the part of the story that gets skipped when people jump straight to Nvidia's current position training the industry's AI models: the company came within one bad quarter of not existing, over an architecture decision that looked reasonable at the time it was made."

  - q: "What technical risk did Nvidia take by skipping hardware validation on the chip that saved it?"
    a: "A real one. Standard chip development at the time ran one to two years and included physical prototype runs before committing to full production, catching design flaws before they're baked into silicon. Nvidia didn't have that runway. It compressed the cycle to roughly seven months by validating the design entirely in simulation and committing to the production run without ever holding a physical prototype, called the RIVA 128. That is a bet with no safety net, if the simulation had missed something, the fab run would have failed and the company would have been out of money and out of time. It wasn't a clean win either: of the 32 blend modes in DirectX at the time, the RIVA 128 only supported 8, and Nvidia had to convince developers to build within that constraint rather than wait for full compliance. The bet paid off because the core architecture was right, not because the execution was flawless. That distinction, right bet versus perfect execution, is the one worth sitting with."

  - q: "What's the actual lesson here for people running AI infrastructure today, not just a Nvidia trivia point?"
    a: "The lesson isn't 'take big risks.' Most companies that skip validation under financial pressure don't get a comeback chip, they get a second failure. The lesson is narrower: when the safe option (another year of runway you don't have, another QA cycle you can't afford) doesn't exist, the quality of the decision comes down to how well you understood the one variable that actually mattered, in Nvidia's case, that triangle-based rendering had already won. I think about this constantly in my own work shipping production AI systems. Neither carries Nvidia's 1996 stakes, but the shape of the decision repeats constantly: ship the best version of a system you can build with the information you have, under a deadline that won't move, and find out in production whether you read the constraint correctly. The rule that generalizes: bet big only on the thing you're most certain about, and make sure you can tell the difference between certainty and hope before you commit the budget."

  - q: "Where does this same pattern, the unglamorous bet that buys survival, show up in AI infrastructure decisions today?"
    a: "Constantly, and it's usually invisible until years later. Teams under budget or timeline pressure right now are making the same kind of call: ship a narrower model integration that handles the core workflow reliably, instead of waiting for the broader, more impressive platform play. The unglamorous choice, tighter scope, faster validation cycle, honest about what it doesn't cover, is frequently the one that keeps a team's AI initiative funded past the next budget review. The ambitious version usually only becomes viable once the constrained version has already proven the architecture works and the company is still solvent enough to build it. If you're deciding between the ambitious AI bet and the constrained one under a real deadline, the question to ask is which one is actually right about the one thing that matters, not which one is more impressive in the board deck."

  - q: "How much more powerful is a modern AI-training GPU than the chip that saved Nvidia in 1997?"
    a: "By transistor count, the cleanest apples-to-apples spec across both eras, the 1997 chip carried 3.5 million transistors on a 350nm process. The RTX PRO 6000 Blackwell, a current workstation card built to train and serve AI models, carries 92.2 billion, on the GB202 die, roughly 26,000 times more. Memory went from 4MB of SGRAM to 96GB of GDDR7, about 24,000 times more, and memory bandwidth went from 1.6GB/s to 1,792GB/s, roughly 1,120 times more. Even price, adjusted for inflation, moved from around $415 in today's money to about $8,500, roughly 20x, though it's not really the same product category, one rendered game triangles, the other trains models. The growth wasn't one leap. It compounded across nine flagship generations over 29 years, each funded by the commercial success of the one before it. None of that compounding happens if the 1997 chip doesn't sell a million units in its first four months."

  - q: "How much of Nvidia's business today is actually AI, not gaming?"
    a: "Almost all of it, and the shift happened faster than most outside the industry noticed. Data center chips, the ones training and serving AI models, made up roughly 88% of Nvidia's revenue in the first half of its 2026 fiscal year, versus a small single-digit share from gaming GPUs. Nvidia holds over 90% share of the AI training chip market specifically, and 60–75% of AI inference, where custom silicon and CPUs compete harder. Nvidia is, functionally, an AI infrastructure company that still happens to sell graphics cards, not a graphics card company that stumbled into AI. That reversal only exists because the company survived long enough, chip generation by chip generation, to still be standing when the AI training workload arrived. The near-bankruptcy in 1996 isn't a footnote to that story. It's the reason there was a company left to have the story happen to."
---

# How a $5M Bet Saved Nvidia Before It Ever Built an AI Chip

Nvidia's data center chips, the ones training and serving today's AI models, now make up roughly 88% of its revenue. It holds over 90% share of the AI training chip market. None of that exists if one chip most people have never heard of hadn't worked, on the first try, in 1997, when the company had about a month of payroll left.

I still have that chip. An ASUS AGP-V3000, built on the Nvidia RIVA 128. Four EtronTech memory chips, 4MB of SGRAM, a passive heatsink, a single VGA port. It never trained anything. It never ran a model. It rendered triangles for late-90s PC games, and it's the reason there's still a company called Nvidia to build the chips that do the training now.

---

## The Bet Nobody Remembers

Everyone knows Nvidia as the company behind the AI boom. Fewer people know it almost didn't survive long enough to get there.

In the mid-90s, Nvidia bet its future on the NV1, a chip built around quadratic surfaces instead of the triangle-based rendering Microsoft's DirectX was standardizing on. The bet was wrong. Diamond Multimedia had ordered 250,000 units. Roughly 249,000 came back. Nvidia shrank to about 40 people.

Sega had contracted Nvidia to build the graphics chip for the Dreamcast, the NV2. After more than a year of work, Jensen Huang told Sega it wouldn't meet their requirements, and asked to be released from the contract, then asked Sega to pay the final milestone anyway. Sega America's CEO, Shoichiro Irimajiri, agreed to a payment reported at around $5 million. Nvidia had roughly one month of payroll left when it landed, close enough to the edge that the company's unofficial motto at the time, by its own account, was "our company is thirty days from going out of business."

That payment didn't fix anything. It bought one more shot. It had to work.

## One Chip, Seven Months, No Safety Net

The chip that came out of that shot was the RIVA 128. Standard development at the time ran one to two years, with physical prototype runs to catch design flaws before committing to production. Nvidia didn't have that runway.

It compressed the cycle to roughly seven months by validating the entire design in simulation and committing to the production run without ever holding a physical chip. If the simulation had missed something, there was no money left to recover from it.

It wasn't flawless. Of the 32 blend modes DirectX supported at the time, the RIVA 128 only handled 8. Nvidia had to convince developers to build within that constraint rather than wait for full compliance. The bet worked because the core architecture, triangles, was right, not because the execution was clean.

The RIVA 128 sold over a million units in its first four months. That's not a footnote in gaming hardware history, it's the reason Nvidia was still solvent two years later to keep iterating, generation after generation, toward the architecture line that eventually became CUDA, and eventually became the hardware training GPT-scale models.

History usually credits the GeForce 256, which shipped two years later, with saving the company. It didn't need saving by then, this chip had already done that. GeForce made Nvidia dominant. This one made sure there was a company left to dominate anything with. Nobody outside a hardware forum needs to care about that distinction. What's worth caring about is what it says: the move that keeps a company alive and the move that makes it famous are rarely the same decision, and the one that keeps you alive usually gets forgotten first.

## Why I Kept It

I'm not a collector. I don't have a shelf of retro hardware. I kept this card because I bought it when 3D acceleration in a home PC was still a novelty, when nobody was mining anything with a GPU, and when "AI chip" wasn't a phrase that existed. It was just a card that made games look better.

Holding it next to what Nvidia is now, a company whose chips train the models the entire industry runs on, is a strange feeling. Not because the jump is impressive, though it is. Because the jump wasn't inevitable. It happened because a company on the edge of shutting down made one correct bet on the one variable that mattered, shipped it under pressure with no room to recover from a miss, and turned out to be right.

## What 4MB Became

The RIVA 128 ran 3.5 million transistors on a 350nm process, clocked at 100MHz, moving data through 4MB of SGRAM at 1.6GB/s. That was the entire chip.

The workstation-class card a good chunk of today's AI infrastructure work eventually depends on somewhere in the stack is the RTX PRO 6000 Blackwell: 92.2 billion transistors, 24,064 CUDA cores, 96GB of GDDR7, 1,792GB/s of memory bandwidth, 600W under load.

| | RIVA 128 (1997) | RTX PRO 6000 Blackwell (2026) |
|---|---|---|
| Transistors | 3.5 million | 92.2 billion (~26,000x) |
| Memory | 4MB SGRAM | 96GB GDDR7 (~24,000x) |
| Memory bandwidth | 1.6 GB/s | 1,792 GB/s (~1,120x) |
| Process node | 350nm | ~4nm-class |
| Power draw | Passive, no fan | 600W |
| Launch price | ~$200 (~$415 in 2026 dollars) | ~$8,500 |
| Job | Render triangles for games | Train and serve production AI models |

*Launch price adjusted for inflation using US CPI, 1997 to 2026 (~107% cumulative). Even correcting for that, the RTX PRO 6000 costs roughly 20x what the RIVA 128 did in real terms, for a card built for a completely different job.*

Transistor count is the cleanest single line to draw across all nine generations in between, it's the one spec that's been measured the same way since 1997:

![Nvidia flagship GPU transistor count, 1997 to 2026, log scale, from 3.5 million transistors to 92.2 billion](https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/riva-128-chip-that-saved-nvidia-chart.png)

*Flagship-generation transistor counts, not every calendar year, Nvidia doesn't ship a new architecture annually.*

Nvidia didn't jump from 1997 to Blackwell in one step. It compounded, one funded generation at a time. Data center chips, the direct descendants of that lineage, now generate roughly 88% of Nvidia's revenue and cover over 90% of the AI training chip market. None of that compounding, and none of that revenue, happens if the chip in my hand doesn't work on the first try.

## The Rule

Most companies that skip validation under financial pressure don't get a comeback story. They get a second failure. The lesson isn't "take the big risk." It's narrower: when the safe option doesn't exist, the quality of the call comes down to how well you understood the one thing that actually mattered, and whether you can tell the difference between being certain and just hoping.

I think about that constantly in my own work shipping production AI systems. Neither carries Nvidia's 1996 stakes. But the shape repeats: ship the best version of a system you can build, with the information you have, under a deadline that won't move, and find out in production whether you read the constraint correctly.

Nvidia was right. Most companies in that position aren't. Worth remembering next time someone describes their AI infrastructure bet as obvious in hindsight. It rarely was at the time.

The card still works, as far as I know. I haven't plugged it in. Some things are better left as proof than as demo.

---

**About the Author**

Netanel Eliav builds production AI systems — agentic workflows, RAG pipelines, LLM infrastructure, and evaluation frameworks. London-based, global AI leader.

[LinkedIn](https://linkedin.com/in/inetanel) · [GitHub](https://github.com/inetanel) · [inetanel.com](https://inetanel.com)

---

**Related Resources**
- [NVIDIA Corporate Timeline](https://www.nvidia.com/en-us/about-nvidia/corporate-timeline/) — Nvidia's own record of the RIVA 128 launch and the "thirty days from going out of business" period
