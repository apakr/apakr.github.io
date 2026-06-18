---
title: "SPV Watcher: A Perpetual Private-Equity Researcher"
date: 2026-06-17 00:00:00 -0600
tags: [AI Agents, Private Markets, SPV, Automation, Finance, LLM, Private Equity]
description: I built an AI harness that watches a dedicated inbox for SPVs, secondary shares, and pre-IPO allocations, then routes informed and time-sensitive deals to me, 24/7, human-in-the-loop optional.
---

### The Start

In investing, I spend most of my time with public equities, but there's a short list of private companies I've always wished I could own a slice of. The problem is that getting access to them is its own world. SPVs, secondary shares, pre-IPO allocations, tender offers, one-off introductions, it's a complicated semi-private marketplace that runs on broker relationships and email newsletters. If I spent enough time on it I could crack it, but it was just never worth the time for me. 

So it sat in the back of my head as a "someday" interest. Until I read a certain blog post.

### Finding Private Equity Opportunities

My buddy Eric pointed me to [_Adventures in Private Investing, Pt I_](https://churningandburning.com/2026/03/adventures-in-private-investing-pt-i-space-and-defense.html), by Peter To on his blog [_Churning and Burning_](https://churningandburning.com/). The whole piece is great, but one section - "Relationship Building" - is where my brain clicked. He spelt out his methodology pretty simply. Sign up to every secondary marketplace, every SPV manager who advertises, every alternative-investment newsletter, every private-investor network you can find. Cast the widest net possible so the good, time-sensitive deals actually reach you; engross yourself in the space.

That's a great strategy, but it also has an immense drag on your time, and your inbox becomes a firehose. The whole point is to subscribe to *everything*, which means the useful stuff is buried under a mountain of newsletters, marketing, slop, etc. Meanwhile, the best deals have deadlines.

But reading that section felt like the scene in _Ratatouille_ where Remy tastes the cheese and the strawberry separately, then together, and the flavors set off fireworks in his head. Except in my head it was Peter's deal-sourcing methodology and [the Pi Agent Harness](https://pi.dev/). The boring, repeatable, every-single-day part was perfectly automatable, while the slop would be useful in creating a de-facto data bank - and I could get the benefit of hours of reading and mental RAM usage for a fraction of the time-cost.

### What I built

I called the project directory opp-watcher. It's an AI agent that watches a dedicated inbox, reads every message that comes in, and sorts each one into one of five buckets:

- **Actionable Opportunity** - a real SPV, secondary, pre-IPO allocation, tender, or time-boxed application/waitlist.
- **Market Intel** - useful company or pricing information, but nothing to act on right now.
- **Newsletter Slop** - generic commentary, marketing, and hype.
- **Admin** - logins, confirmations, billing, account noise.
- **Unknown** - not enough information to call it.

The only category that's allowed to interrupt me is the first one. When the agent spots a genuine opportunity, it sends an instant alert with the details that actually decide whether something's worth a look: company, source, opportunity type, minimum, deadline, valuation, fees, accreditation requirements, what action is needed, and a plain-English summary. Everything else either rolls into a once-a-day digest or quietly disappears into memory.

![SPV Watcher architecture: a dedicated inbox is polled on a schedule, parsed, classified by an AI agent, and routed to instant alerts, a daily digest, or archive, with everything written to a local memory store.](/assets/img/spv-watcher/architecture.png)
_The whole pipeline: inbox in, triage in the middle, the right deals out — with a local memory that builds over time._

Under the hood it's a small, always-on agent built on the Pi SDK that polls the inbox on a schedule, classifies each new message, and writes everything to a local memory store. The shape is deliberately simple. I've built a few agents using OpenClaw and Hermes and they confuse themselves enough to be annoying for projects like this. The Pi SDK was perfect because it's so light.* To quote Todd Howard, it feels like "it just works."

\* _For reference: Pi's entire system prompt and tool definitions come in [under ~1,000 tokens](https://shivamagarwal7.medium.com/agentic-ai-pi-anatomy-of-a-minimal-coding-agent-powering-openclaw-5ecd4dd6b440), while OpenClaw's system prompt and tool schemas alone run [roughly 17–30k tokens](https://openclaw-ai.com/en/docs/concepts/context/) per call before any conversation history even starts._

### What a hit looks like

Here's a real alert it produced - a SpaceX SPV that came through a private-market source:

![A Discord alert from SPV Watcher: an actionable private-market opportunity for a SpaceX SPV via Healey Pre-IPO, with structured fields for sender, company, source, type, accreditation, and a summary, plus memory context noting the company has been seen before.](/assets/img/spv-watcher/discord-alert.png)
_A real alert. Note the "memory context" at the bottom - it already knew it had seen this company before and how that source has performed._

The memory context is where I believe the real sauce comes from. The agent isn't simply saving me time by reading stuff for me, it remembers companies, sources, prior offers, and observed prices, so every new alert arrives with context: have I seen this company before? Is this source usually noise or usually real? What price have I seen quoted elsewhere?

### The privacy angle (why this project, and not the others)

Using these AI tools is incredibly powerful, but I always stall on the same instinct: I really don't like exposing personal information to public models. There's some other projects I've done with fully pirvate LLMs, but those can't use the multiplicative benefits from cloud models.

This project sidesteps the privacy problem while also using frontier models. SPV Watcher gets its own dedicated gmail - it never touches my personal email. It reads a stream of deal flow that I deliberately routed to it, nothing private. And when a deal looks real, I take the conversation *off* the bot's inbox and handle it myself like any other human-to-human negotiation. The agent is my scout, not an active particpant. This separation aligns with the privacy impulse I have while also maximizing the use of frontier models.

### Does it actually work?

It's been running for about two weeks. In that window it has read and classified roughly 130 emails - and surfaced 4 as genuinely actionable. That ~3% is the whole value proposition: about 97% of the inbox never interrupted me while still benefitting me by building up the agent's memory. Meanwhile, the handful of real, deadline-driven opportunities got pinged to me on Discord.

![A terminal log of a watcher cycle: web research running and completing for several emails, the Discord bot coming online, and a cycle summary showing 79 searched, 31 processed, 1 actionable alert sent, 0 errors.](/assets/img/spv-watcher/watcher-cycle.png)
_A live cycle: it searches, processes, runs research on the interesting ones, and reports in. One actionable alert this round, zero errors._

There are still places where I'm tuning the filtering. I've gotten some "actionable market opportunities" that were just webinars, but that's another beauty of the system, I can react to the model's messages on Discord, or tell it directly when I want to see more or less of something. Additionally, I can directly chat with the model and spitball ideas, and it can take my implicit reasoning and update it's own memory - it sort of fuzzy in a way a hard coded system could never be. It's saved me hours of reading, and more importantly it does research-grade analysis and memory consolidation while I sleep. I wake up to a tidy digest instead of an inbox I'd start avoiding after a few weeks.

### What's next

The thing works, and doesn't need to have too many features. Moving forward making it better is relatively simple: subscribe to more of the right newsletters and platforms, keep tuning the memory system, and let the deal history compound. The longer it runs, the smarter its context gets (and I spend essentially 0 time on it a week!).

The exceptionally useful aspect of this agent is more general. This Pi SDK architecture with a dedicated inbox, an agent that triages and remembers, alerts where you actually look is extenable far further than SPVs. It's a lightweight, customizable pattern for *any* domain where useful signal arrives buried in a flood of email: research gathering, sourcing, market monitoring, whatever.

Anyway. Feel free to reach out if you have questions or suggestions.
