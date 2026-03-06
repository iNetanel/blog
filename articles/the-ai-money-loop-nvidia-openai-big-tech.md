---
title: "The AI Money Loop: How Nvidia, OpenAI, and Big Tech Are Funding Each Other"
description: "Nvidia invests in OpenAI. OpenAI buys Nvidia chips. Oracle builds the data centers. SoftBank funds the whole thing. And the loop keeps feeding itself — until it doesn't."
date: "06/03/2026"
author: "Netanel Eliav"
tags: ["AI", "Investment", "Nvidia", "OpenAI", "MLOps", "ProductionAI", "AIInfrastructure"]
slug: "the-ai-money-loop-nvidia-openai-big-tech"
featured_image: "https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/openai-big-tech.jpg"
featured: true

faqs:
  - q: "What is the AI circular investment loop?"
    a: "The AI circular investment loop describes a pattern where AI infrastructure companies (like Nvidia) invest in AI model developers (like OpenAI), who then commit to buying back chips, cloud services, and compute from those same investors. The money doesn't leave the ecosystem — it rotates. Nvidia invests $100B in OpenAI → OpenAI commits to buying Nvidia GPUs → GPU demand props up Nvidia's valuation → Nvidia has more capital to reinvest. Critics call it circular. Supporters call it a flywheel. The honest answer: it's both, depending on whether demand from real users ever catches up."

  - q: "Is the AI investment boom a bubble?"
    a: "It depends how you define bubble. Valuations are clearly elevated. OpenAI is valued at $500B while losing money at scale. Oracle borrowed heavily to fund Stargate. SoftBank sold its Nvidia shares to fund OpenAI. These are not signs of conservative capital allocation. But unlike the dot-com era, there is real infrastructure being built — physical data centers, GPUs, energy infrastructure. The question isn't whether AI is real. It's whether the capital structure holding it up is sustainable if revenue growth lags the commitments made."

  - q: "What is Stargate and why does it matter?"
    a: "Stargate is a $500 billion joint venture between OpenAI, Oracle, SoftBank, and MGX to build AI data centers across the US. Announced at the White House in January 2025, it's one of the largest infrastructure projects in tech history. It matters because it crystallizes the core dynamic of the AI investment loop: companies are locking in supply by pairing long-term purchase commitments with financing arrangements. OpenAI needs the compute. Oracle builds the centers. Nvidia supplies the chips. SoftBank finances it. Everyone is betting that AI demand — from real, paying users — will eventually justify the bill."

  - q: "How much has Nvidia invested in AI companies?"
    a: "Between 2020 and late 2025, Nvidia deployed roughly $53 billion across approximately 170 deals in the AI ecosystem, ranging from LLM developers like OpenAI and Cohere to cloud providers like CoreWeave and Lambda. In 2025 alone, it completed 67 deals worth $23.7 billion — exceeding its entire 2024 investment pace in less than a year. The strategic rationale is transparent: Nvidia is investing in its own customers to ensure they have the capital to buy more of its chips."

  - q: "What happens if the AI investment loop breaks?"
    a: "If one link in the chain weakens — say Oracle's debt load becomes unserviceable, or OpenAI's revenue growth stalls — the effects cascade. Oracle can't pay Nvidia. Nvidia's guaranteed revenue disappears. Startup valuations that were predicated on GPU access reprice. The difference from 2008 is that the assets being built are real and productive, not synthetic. But the leverage ratios are significant: Goldman Sachs estimated that over $140 billion in new corporate debt was issued by AI-related firms in 2025 alone."
---

The alert doesn't fire at 3 AM this time. The loop breaks quietly — in a spreadsheet, somewhere in Oracle's finance team, at the moment the numbers stop adding up.

That's the risk nobody in the Nvidia earnings call is talking about directly.

---

## Here's What's Actually Happening

In late 2025, Nvidia announced it would invest up to $100 billion in OpenAI. In return, OpenAI committed to deploying "at least 10 gigawatts" of Nvidia compute capacity — enough electricity to power a major city.

Read that again. Nvidia is investing in OpenAI. OpenAI is buying Nvidia chips with that money. Nvidia counts those chip sales as revenue.

The money doesn't leave the room.

This isn't a conspiracy. It's not even unusual in corporate finance. What's unusual is the *scale*, the *speed*, and the degree to which the entire AI ecosystem has organized itself around this same pattern.

Between 2020 and late 2025, Nvidia deployed roughly $53 billion across approximately 170 deals in the AI space — from LLM developers like OpenAI to cloud providers like CoreWeave. In 2025 alone, it completed 67 deals worth nearly $24 billion. That's more than its entire 2024 investment activity, compressed into under twelve months.

Nvidia CEO Jensen Huang's response when pressed on whether this artificially inflates demand: Nvidia's investments are "separate" from the revenue it could see from those same companies. Seaport analyst Jay Goldberg, who holds a rare Sell rating on Nvidia, disagrees: "It's very murky. To what degree is Nvidia investing versus buying demand or subsidizing demand for its chips?"

Both are probably right.

---

## The Loop, Mapped

Here's the structure, without the PR framing:

**Nvidia → OpenAI**
Nvidia invests up to $100 billion. OpenAI commits to buying Nvidia GPUs. Nvidia takes an equity stake. OpenAI gets the cash to expand. Nvidia is guaranteed to be the hardware supplier for that expansion.

**OpenAI → Oracle → Nvidia**
OpenAI signed a $300 billion contract with Oracle for computing power over five years. Oracle needs to build data centers to honor that contract. Those data centers run on Nvidia chips. Oracle has to borrow to build them — and already carried over $80 billion in debt before this deal.

**Nvidia → CoreWeave → Microsoft → OpenAI**
Nvidia took a 7% stake in CoreWeave, a GPU cloud provider. Then agreed to pre-purchase $6.3 billion in CoreWeave services — effectively guaranteeing CoreWeave's revenue so it could confidently buy even more Nvidia chips. CoreWeave's largest customer? Microsoft. Microsoft's biggest AI bet? OpenAI, in which it holds a roughly 49% profit-sharing stake following $13+ billion in investment since 2019.

**SoftBank → OpenAI → Everything**
SoftBank sold its entire Nvidia position to fund a $30 billion investment in OpenAI. OpenAI then signed deals with Microsoft ($250B in cloud services), AMD ($90B), and AWS ($38B). OpenAI may become one of AMD's largest shareholders if milestones are met.

One out of every two dollars invested by the venture capital industry over the past two years has gone into AI startups — led by OpenAI and Anthropic.

---

## Stargate: The Physical Manifestation

All of this capital had to go somewhere. In January 2025, it went into concrete.

The Stargate Project — a $500 billion joint venture between OpenAI, Oracle, SoftBank, and Abu Dhabi's MGX — is one of the largest infrastructure commitments in tech history. Announced at the White House alongside President Trump, it intends to deploy 10 gigawatts of AI compute across the US over four years.

The flagship site is in Abilene, Texas. Oracle began delivering the first Nvidia GB200 server racks in June 2025. Early training workloads started running. By September, five more sites were announced. By the end of 2025, the project claimed to be on track toward its full 10-gigawatt commitment.

There are genuine complications. SoftBank needed to borrow $3 billion from Mizuho to fund its share. The joint venture itself reportedly stalled mid-year due to internal disagreements about site selection and operational responsibilities. OpenAI tried to build its own data centers independently, couldn't secure financing — lenders were unwilling to back billion-dollar projects from a company with unproven unit economics — and went back to its Stargate partners.

OpenAI CFO Sarah Friar's comparison at Davos: "When the internet was getting started, people kept feeling like, 'Oh, we're over-building, there's too much.' Look where we are today."

That argument might be right. It might also be the thing people say during every infrastructure bubble — because it was true during the internet, and therefore it must be true now.

---

## What's Real and What's Circular

Asset manager Janus Henderson called this a "virtuous circle" — companies locking in supply by pairing long-term buying commitments with financing. That framing isn't wrong.

The AI infrastructure being built is real. The chips exist. The data centers are going up. The energy capacity is being contracted. These are productive assets, not synthetic derivatives.

But the revenue story is still thin. OpenAI is projected to bring in $13 billion in 2025 — impressive growth, but small against $1.15 trillion in combined infrastructure commitments from hyperscalers between 2025 and 2027. The combined free cash flow of Amazon, Google, Meta, and Microsoft is expected to shrink by 43% between Q4 2024 and Q1 2026.

The capital structure holding this up assumes that AI demand from real users — enterprises, developers, consumers paying for subscriptions — will compound fast enough to service the debt and justify the valuations.

That assumption might be correct. The question for builders and engineers isn't whether to bet on AI. It's whether the specific companies you're building on, integrating with, or competing against have sustainable capital structures underneath them — or whether they're running on the loop.

---

## Why This Matters for Production AI

I've written before about [what actually breaks at scale in production AI systems](https://inetanel.com) — and the most dangerous failures are the ones that look fine until they suddenly don't.

The circular investment loop has a production-systems analog. When every component in your pipeline depends on every other component, you don't get graceful degradation. You get cascade failure.

Oracle's debt load is not a problem for an Nvidia GPU. But it's a problem for an engineering team at a startup that built their entire inference infrastructure on Oracle Cloud Infrastructure, signed a long-term contract based on pricing that assumed Oracle's scale would hold, and now faces a counterparty that is financially stressed.

That's not hypothetical. It's the kind of second-order effect that only shows up in production, after the contracts are signed and the architecture is locked.

A few things I'd actually watch:

**Revenue vs. commitments.** OpenAI's $1.15 trillion in infrastructure commitments against $13B in projected 2025 revenue is a 88x gap. That gap narrows every year if growth holds. It becomes a solvency problem if growth stalls for even two quarters.

**Debt levels at the infrastructure layer.** Goldman Sachs estimated over $140 billion in new corporate debt issued by AI-related firms in 2025 alone. The companies holding that debt are the ones building and operating the data centers your models run on.

**Concentration risk.** The loop concentrates around a handful of nodes: Nvidia for chips, Azure and OCI for cloud, OpenAI for model capability. If any single node reprices significantly — chip shortages, regulatory intervention, a major model capability jump from a competitor — the whole system rebalances fast.

**Demand from outside the loop.** The real test isn't whether AI companies will buy compute from each other. It's whether enterprises outside the loop will pay enough to justify the infrastructure being built for them. That answer is still being written.

---

## The Ouroboros Problem

In mythology, an ouroboros is a snake devouring its own tail — often used as a symbol of self-sustaining cycles. In AI investment, it's becoming a useful diagnostic.

The question isn't whether Nvidia, OpenAI, Microsoft, and Oracle are building something real. They clearly are. The question is whether the self-referential nature of the capital flows creates incentives that distort decision-making.

When Nvidia invests in OpenAI and OpenAI commits to buying Nvidia chips, both companies can report revenue and investment that looks like independent market validation. It isn't. It's a coordinated bet that user demand will catch up before the debt comes due.

That bet might pay off. The internet infrastructure overbuild of the late 1990s did eventually get consumed by actual demand — a decade later, at dramatically lower prices, often through bankruptcy proceedings that wiped out the original investors.

Production AI builders should understand which version of that story they're inside.

---

## What I'd Do

If I were architecting production AI infrastructure today, I'd think hard about a few things:

**Multi-cloud by design, not by accident.** The concentration of AI compute in a handful of providers — all financially entangled with each other — is a production risk, not just a strategic one. OpenAI learned this the hard way when it tried to diversify away from Microsoft and discovered how hard it is.

**Contracts with financial contingency clauses.** If you're signing long-term agreements with infrastructure providers who are carrying significant debt, understand what happens to your SLAs if they need to restructure. This isn't paranoia. It's due diligence.

**Track the revenue gap.** The circular investment loop is self-reinforcing while demand grows. It becomes self-defeating when demand stalls. Watch the quarterly revenue numbers from OpenAI, Anthropic, and the hyperscalers' AI-specific segments. Those numbers tell you whether the loop is spinning faster or slowing down.

**Don't confuse infrastructure investment with product-market fit.** A $500B data center commitment is not the same as a billion users paying for AI products. The former is a bet. The latter is a business.

The AI investment loop will either generate enough real revenue to justify itself — in which case the circular nature of the early capital flows won't matter — or it won't, in which case the cascade will be significant.

That uncertainty is priced into every infrastructure decision you make today.

Build accordingly.

---

*Netanel Eliav builds production AI systems — agentic workflows, RAG, and LLM infrastructure. London-based, global work. [inetanel.com](https://inetanel.com)*