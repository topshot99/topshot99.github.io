---
layout: post
title: "What Anthropic's Scion Made Me Rethink About My Own Developer Productivity"
date: 2026-10-14 08:00:00 +0000
categories: [ai-agents, developer-productivity, software-engineering]
tags: [anthropic, scion, git, source-control, ai-agents, agentic-sdlc, developer-productivity]
author: Manik
excerpt: "Anthropic's Scion post isn't really about replacing Git. It's a glimpse of what development looks like at agent scale — and a prompt to rethink my own workflow."
---

# What Anthropic's Scion Made Me Rethink About My Own Developer Productivity

I recently read Anthropic's *Redesigning source control at Anthropic*, and one number stopped me mid-scroll.

They went from roughly 2,500 merged changes per day to more than 30,000 — in a few months. Not because engineers suddenly got superhuman. Because agents started writing a huge share of the code.

Their argument is simple but a little unsettling: coding itself is increasingly *not* the bottleneck anymore. Once agents start producing code at that volume, the stuff we built for humans — source control, CI, build systems — starts creaking under the load.

I read that and, honestly, didn't think much about Scion itself. I thought about a different question:

**If agents can make me 10x more productive, what actually needs to change in how I work today?**

## The Git Problem (That Isn't My Problem — Yet)

Git is genuinely great at what it was built for. A clone is a full repository. You can inspect history, diff, blame, work completely offline. It assumes the client is sometimes disconnected, and it's optimized beautifully for that.

But Anthropic's clients are now mostly agents and automated jobs — always online, by definition. Their main branch is nearly 100GB. They run around 25 million checkouts a day just for CI.

So they asked: *what would source control look like if we designed it assuming the client was always online?*

Their answer is **Scion** — push more responsibility to the server, keep clients small, fetch only what's needed, let working copies be sparse or virtual, do more through APIs instead of full checkouts.

Notably, Anthropic isn't pitching Scion as "the next Git." They're saying their tooling is deeply specialized, Scion probably isn't generally useful as-is, but maybe some of these ideas filter into Git and other platforms over time.

## Does This Affect Me? Honestly, No.

I am nowhere near producing enough code for Git to be my bottleneck. Not even close.

But that's actually the interesting part. The real question isn't "do I need Scion." It's:

**What would my workflow need to look like if I could actually produce 10x more useful work?**

Anthropic drops a hint here too — their engineers increasingly review *intent* rather than reading every line. Agents handle more of the implementation; humans hold onto architecture, constraints, and judgment.

That's something I can start experimenting with today, at my own tiny scale.

## Three Loops I'm Going to Try

Instead of just telling an agent "write this code" and hoping for the best, I want to wrap my work in three small, deliberate loops.

### 1. The Build Loop

```
Problem → Spec / Intent → Agent → Implementation → Tests → Human reviews intent
```

I own the problem definition, the constraints, the expected behavior. The agent gets real autonomy over *how* it gets built. My job shifts from writing lines to writing clear intent.

### 2. The Review Loop

An agent reviewing its own code is basically an agent congratulating itself. Not useful.

```
Implementation → Independent Agent → Try to break it → Failure scenarios → Fix
```

A second, independent agent's whole job is to try to break what the first one built — edge cases, concurrency bugs, bad assumptions. Anthropic apparently leans on adversarial review heavily, while engineers stay in the loop to stop agents from over-engineering or quietly breaking system constraints.

### 3. The Observe Loop

This is the one I find most interesting, because it's the part most of us skip entirely.

Anthropic has what they call "superfans" — agents dedicated to watching the health of individual components, digging through telemetry and logs, flagging anything that looks off, not just tripping static alert thresholds.

```
Production → Telemetry/Logs → Observer Agent → Something looks strange → Evidence + Hypothesis → Engineer
```

The idea: don't just use agents *before* merge. Use them *after* deployment too, so problems surface as evidence and hypotheses instead of 3am pages.

## The Bigger Idea

For me, this was never really a story about Git versus Scion.

It's a glimpse of what development looks like when it starts operating at agent scale — and you don't need 30,000 merges a day to take something from that.

My experiment is much smaller and much more personal:

**Can I close more of the loop — idea, implementation, review, production — using agents at every stage, not just the "write code" stage?**

Git is definitely not my bottleneck right now.

Maybe the first bottleneck I should remove is myself.

---

*Reference: Anthropic — Redesigning source control at Anthropic*
