---
title: "Someone Caught the AI Money Loop - Here's What They Found"
description: "This week Jensen Huang admitted the $100B Nvidia→OpenAI deal was never signed. The official reason is IPOs. The real reason is that someone finally called it what it was: a loop feeding itself."
date: "06/03/2026"
author: "Netanel Eliav"
tags: ["AI", "Investment", "Nvidia", "OpenAI", "MLOps", "ProductionAI", "AIInfrastructure"]
slug: "the-ai-money-loop-nvidia-openai-big-tech"
featured_image: "https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/openai-big-tech.jpg"
featured: true

faqs:
  - q: "What is the AI circular investment loop?"
    a: "The AI circular investment loop describes a pattern where AI infrastructure companies (like Nvidia) invest in AI model developers (like OpenAI), who then commit to buying back chips, cloud services, and compute from those same investors. The money doesn't leave the ecosystem - it rotates. Nvidia invests in OpenAI → OpenAI commits to buying Nvidia GPUs → GPU demand props up Nvidia's valuation → Nvidia has more capital to reinvest. Critics call it circular. Supporters call it a flywheel. The honest answer: it's both, depending on whether demand from real users ever catches up."

  - q: "Why did Nvidia pull back from its $100B OpenAI deal?"
    a: "The official reason Jensen Huang gave at the Morgan Stanley TMT conference in March 2026 is that OpenAI and Anthropic are both heading toward IPOs, which closes the private investment window. The more complete picture: the circular arrangement attracted escalating scrutiny from analysts, the WSJ reported internal doubts at Nvidia about OpenAI's business discipline, Anthropic's CEO publicly compared Nvidia chip sales to 'selling nuclear weapons,' and the Trump administration then blacklisted Anthropic from Pentagon contracts. The loop became politically and relationally too complex to maintain at $100B scale."

  - q: "What is Stargate and why does it matter?"
    a: "Stargate is a $500 billion joint venture between OpenAI, Oracle, SoftBank, and MGX to build AI data centers across the US. Announced at the White House in January 2025, it's one of the largest infrastructure commitments in tech history. It crystallizes the core dynamic of the loop: companies locking in supply by pairing long-term purchase commitments with financing. OpenAI needs the compute. Oracle builds the centers. Nvidia supplies the chips. SoftBank finances it. Everyone is betting that AI demand from real, paying users will eventually justify the bill."

  - q: "Is the AI investment boom a bubble?"
    a: "It depends how you define bubble. Valuations are clearly elevated - OpenAI at $500B while losing money at scale, Oracle borrowing heavily to fund Stargate, SoftBank selling its Nvidia shares to fund OpenAI. But unlike the dot-com era, real infrastructure is being built: physical data centers, GPUs, energy capacity. The question isn't whether AI is real. It's whether the capital structure holding it up is sustainable if revenue growth lags the commitments made. Nvidia pulling back from circular deals this week is the first sign that the people inside the loop are asking the same question."

  - q: "What happens if the AI investment loop breaks?"
    a: "If one link in the chain weakens - Oracle's debt becomes unserviceable, OpenAI's revenue growth stalls, a major cloud provider reprices - the effects cascade. The difference from 2008 is that the assets being built are real and productive, not synthetic. But Goldman Sachs estimated over $140 billion in new corporate debt was issued by AI-related firms in 2025 alone. The real test now is whether OpenAI's upcoming IPO can attract enough genuine outside capital to replace the circular financing that built the ecosystem."

  - q: "What does this mean for engineers building on AI infrastructure?"
    a: "The circular investment loop is a production risk, not just a finance story. When every component in your infrastructure depends on financially entangled counterparties, you don't get graceful degradation if one of them restructures - you get cascade failure. Engineers should think about multi-cloud architectures by design, understand the financial health of their infrastructure providers, and track whether AI revenue from real users is growing fast enough to service the debt underneath the platforms they build on."
---

This week, Jensen Huang walked onto the Morgan Stanley TMT conference stage and quietly confirmed what analysts had been saying for months.

The $100 billion Nvidia→OpenAI deal - announced at a splashy press event in September 2025, celebrated as the defining moment of the AI boom - was never signed. No contract. No money changed hands. It settled at $30 billion instead. And Huang said it's probably the last time Nvidia invests in OpenAI at all.

Same story for Anthropic. The $10 billion stake announced in November? Also likely the last.

The official explanation: both companies are heading toward IPOs, which closes the private investment window.

That explanation is technically true. It's also the most convenient available exit from a situation that got very complicated, very fast.

To understand why Nvidia is pulling back - and why it matters for anyone building production AI systems - you need to understand the loop they built.

---

## The Loop Nobody Officially Named

Here's what the AI investment ecosystem looked like from mid-2024 through late 2025, stripped of the press release language:

**Nvidia → OpenAI**
Nvidia announced it would invest up to $100 billion in OpenAI. In return, OpenAI committed to deploying at least 10 gigawatts of Nvidia compute - enough electricity to power a major city. Nvidia takes an equity stake. OpenAI gets cash to expand. Nvidia is guaranteed to be the hardware supplier for that expansion.

Nvidia invests in OpenAI. OpenAI buys Nvidia chips with that money. Nvidia counts those chip sales as revenue.

The money doesn't leave the room.

**OpenAI → Oracle → Nvidia**
OpenAI signed a $300 billion contract with Oracle for computing power over five years. Oracle builds data centers to honor that contract. Those data centers run on Nvidia chips. Oracle had to borrow to build them - already carrying over $80 billion in debt before the deal.

**Nvidia → CoreWeave → Microsoft → OpenAI**
Nvidia took a 7% stake in CoreWeave, a GPU cloud provider, then pre-purchased $6.3 billion in CoreWeave services - guaranteeing its revenue so CoreWeave could confidently buy even more Nvidia chips. CoreWeave's largest customer: Microsoft. Microsoft's biggest AI bet: OpenAI, in which it holds roughly a 49% profit-sharing stake following $13+ billion invested since 2019.

**SoftBank → OpenAI → Everything**
SoftBank sold its entire Nvidia position to fund a $30 billion investment in OpenAI. OpenAI then signed deals with Microsoft ($250B in cloud services), AMD ($90B), and AWS ($38B). One out of every two venture capital dollars invested over the past two years went into AI startups.

Between 2020 and late 2025, Nvidia alone deployed roughly $53 billion across approximately 170 deals. $24 billion of that in 2025 alone - more investment activity compressed into under twelve months than in the previous five years combined.

Seaport analyst Jay Goldberg, one of the few analysts holding a Sell rating on Nvidia: "It's very murky. To what degree is Nvidia investing versus buying demand or subsidizing demand for its chips?"

Nvidia declined to comment.

---

## Stargate: $500 Billion in Concrete

All of that capital needed a physical destination. In January 2025, announced at the White House alongside President Trump, it became the Stargate Project.

A $500 billion joint venture between OpenAI, Oracle, SoftBank, and Abu Dhabi's MGX - one of the largest infrastructure commitments in tech history. Ten gigawatts of AI compute across the US over four years. The flagship site in Abilene, Texas, where Oracle began delivering the first Nvidia GB200 server racks in June 2025 and early training workloads started running.

Behind the announcements, the complications were real. SoftBank needed to borrow $3 billion from Mizuho just to fund its share. The joint venture stalled mid-year - OpenAI, Oracle, and SoftBank arguing about site selection and structure. OpenAI tried to build its own data centers independently. Couldn't get financing. Lenders weren't willing to back billion-dollar projects from a company with unproven unit economics. Went back to Oracle and SoftBank.

OpenAI CFO Sarah Friar at Davos: "When the internet was getting started, people kept feeling like, 'Oh, we're over-building, there's too much.' Look where we are today."

That argument might be right. It's also the thing people said in 1999.

---

## This Week: Someone Pulled the Thread

The loop started unraveling publicly in February, when the WSJ reported that Nvidia internally questioned OpenAI's business discipline. Huang denied it - called it "nonsense" - but the $100B deal remained unsigned.

Then this week at Morgan Stanley's conference, Huang confirmed the pullback. Thirty billion, not a hundred. Probably the last. IPOs close the private investment window.

The IPO explanation has a logic problem. Late-stage private investing - piling into a company right before it goes public - is standard practice. Firms do it routinely. Huang's real explanation, reading between the lines, is that the circular arrangement had become more liability than asset.

Here's what actually happened between September 2025 and March 2026:

Two months after Nvidia invested $10 billion in Anthropic, Anthropic's CEO Dario Amodei stood at Davos and - without naming Nvidia directly - compared US chip companies selling high-performance AI processors to approved Chinese buyers to "selling nuclear weapons to North Korea."

Then the Trump administration blacklisted Anthropic entirely, blocking federal agencies and military contractors from using its technology after the company refused to allow its models for autonomous weapons or mass domestic surveillance.

OpenAI moved within hours. It announced its own Pentagon deal.

Nvidia now held equity stakes in the two most significant AI labs on the planet. And they were at war with each other, with the US government, and with public opinion simultaneously.

The loop doesn't just break because of spreadsheets. It breaks when the relationships underneath it become too complicated to maintain.

---

## What the Loop Was Actually Hiding

Asset manager Janus Henderson called it a "virtuous circle." Supporters argued that in a market where advanced chips are hard to get, companies don't just place orders - they lock in supply by pairing purchase commitments with financing.

That framing isn't wrong. But it obscures something important.

When Nvidia invests in OpenAI and OpenAI commits to buying Nvidia chips, both companies can report revenue and investment activity that *looks* like independent market validation. It isn't. It's a coordinated bet - financially structured to resemble organic demand - that real user revenue will catch up before the debt comes due.

The numbers underneath are significant. OpenAI projected $13 billion in 2025 revenue against over $1 trillion in infrastructure commitments. An 88x gap. Goldman Sachs estimated over $140 billion in new corporate debt issued by AI-related firms in 2025 alone. The combined free cash flow of Amazon, Google, Meta, and Microsoft is expected to shrink by 43% between Q4 2024 and Q1 2026 - all of it redirected into data centers.

Nvidia, for its part, still booked $215.9 billion in FY2026 revenue, up 65% year on year. The chips are selling. The loop didn't collapse - it got called out. And being called out is enough to force a restructuring when the deals are this visible.

---

## Why This Is a Production Problem, Not Just a Finance Story

I've [written before about what breaks in production AI systems](https://inetanel.com) - the failures that look fine until they don't. The circular investment loop has an exact analog in production engineering.

When every component in your pipeline depends on every other component, you don't get graceful degradation. You get cascade failure.

Oracle's debt load is not a problem for an Nvidia GPU. But it is a problem for the engineering team at a startup that built their entire inference stack on Oracle Cloud Infrastructure, signed a long-term contract based on pricing that assumed Oracle's scale would hold, and now faces a counterparty under financial stress. That's not hypothetical. That's the second-order effect that only shows up after the contracts are signed and the architecture is locked.

A few things worth watching now:

**The IPO test.** OpenAI is reportedly targeting a $1 trillion valuation at IPO. The circular financing that got it to $500B private valuation is unwinding. Whether public market investors - who don't have the same incentive to participate in the loop - will pay a trillion for a company projecting $13B in revenue is the most important data point of 2026.

**What replaces the loop.** Nvidia is stepping back from equity investments in its customers. OpenAI's latest $110 billion funding round includes $50B from Amazon and $30B from SoftBank. Watch whether it holds together without the circular validation the Nvidia stake provided.

**Concentration in your own stack.** The loop concentrated AI infrastructure around a handful of nodes: Nvidia for chips, Azure and OCI for cloud, OpenAI for model capability. If any of those nodes reprices significantly - regulatory action, a major capability jump from a competitor, debt restructuring - the whole system rebalances fast. Multi-cloud by design isn't a best practice right now. It's a survival strategy.

**Revenue from outside the loop.** The real test was never whether AI companies would buy compute from each other. It's whether enterprises outside the loop will pay enough to justify the infrastructure built for them. That answer is being written quarter by quarter.

---

## The Ouroboros Eats Itself

In mythology, an ouroboros is a snake devouring its own tail. The symbol is usually read as continuity - a self-sustaining cycle. The part people skip is that the snake is consuming itself to survive. That only works until it runs out of tail.

The AI investment loop is not collapsing. The infrastructure is real. The chips are real. The revenue is growing. But the self-referential financing structure that made early valuations look like market consensus is being dismantled - by the same people who built it, before someone else dismantles it for them.

Huang's move this week is rational. Walk away from the circular deals before they become a regulatory or reputational problem. Let OpenAI's IPO be the validation mechanism instead. If public markets price it at a trillion dollars, that's genuine outside capital saying the bet paid off. If they don't, better to have already reduced exposure.

For builders: the question was never whether AI is real. It's whether the specific financial structures underneath your infrastructure are as stable as the infrastructure itself.

The loop was always a bet on demand catching up. This week, the people who made that bet started quietly hedging it.

Build on that accordingly.

---

*Netanel Eliav builds production AI systems - agentic workflows, RAG, and LLM infrastructure. London-based, global work. [inetanel.com](https://inetanel.com)*