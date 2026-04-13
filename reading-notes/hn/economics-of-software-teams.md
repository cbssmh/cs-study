# The Economics of Software Teams: Why Most Organizations Are Flying Blind

## Source

- Author: Viktor Cessan
- Type: Blog post
- Topic: Software team economics, ROI, engineering management, LLM impact

---

## Why I Saved This

This article explains software teams not as pure builders, but as investment units that consume capital and therefore should be evaluated by financial return.

It is especially useful for backend, platform, and product-oriented engineers because it connects technical work to business value instead of stopping at delivery metrics.

---

## Core Claim

Most software organizations do not know two basic things clearly:

1. what a team actually costs
2. what economic value that team actually generates

Because of that, many teams operate with activity metrics rather than investment logic. :contentReference[oaicite:1]{index=1}

---

## What a Team Actually Costs

The article gives a simple mental model:

- One software engineer in Western Europe ≈ **€130,000/year**
- Team of 8 engineers ≈ **€1,040,000/year**
- Monthly team cost ≈ **€86,667**
- Daily team cost ≈ **€4,000**

This framing is powerful because it turns vague engineering work into explicit economic decisions.

For example:

- 3 weeks spent on a low-impact feature is not just "3 weeks of work"
- it is a capital allocation decision worth tens of thousands of euros :contentReference[oaicite:2]{index=2}

---

## Internal Platform Team Example

The article uses an internal platform team serving 100 engineers.

If the team costs about **€87,000 per month**, then to merely break even, it must create at least that much value for the rest of engineering.

The simplest way to estimate that is time saved:

- engineer monthly cost ≈ **€10,800**
- engineer hourly cost ≈ **€65**
- break-even requirement ≈ **1,340 hours saved per month**
- across 100 engineers, that is about **13.4 hours per month per engineer**
- roughly **3 hours per week per person** :contentReference[oaicite:3]{index=3}

This is a very useful lens for platform or tooling work:
the question is not whether a platform is elegant, but whether it saves enough time or prevents enough operational damage to justify its cost.

---

## Break-Even Is Not Enough

One of the strongest ideas in the article is that **break-even is too low a bar**.

The argument is:

- many initiatives fail
- successful initiatives must cover unsuccessful ones
- systems create long-tail maintenance costs
- operational risk and ownership cost continue after initial delivery

So the realistic threshold is closer to **3x to 5x annual cost**, not 1x. :contentReference[oaicite:4]{index=4}

That means a team costing about **€87,000/month** should ideally generate something like:

- **€260,000/month** at the low end of viability
- **€433,000/month** to look financially healthy :contentReference[oaicite:5]{index=5}

This changes how we think about engineering priorities:
a team should not just "do useful things" but should work on problems large enough to clear a real financial hurdle.

---

## Customer-Facing Team Economics

For product teams, the value levers are different, but the math is the same.

The article shows examples using:

- churn reduction
- activation improvement
- sales conversion improvement

Example logic:

- if a product earns **€50/month per user**
- then a team costing **€87,000/month** needs the equivalent of **1,740 users worth of monthly value** just to break even

Improvements in churn, activation, or conversion can therefore be translated directly into economic value. :contentReference[oaicite:6]{index=6}

This is important because it shows that engineering value should be tied to business levers, not just delivery volume.

---

## The Wrong Metrics Problem

A major criticism in the article is that most teams track metrics that are easy to instrument but weakly connected to economic return.

Examples:

- velocity
- tickets closed
- features shipped
- NPS
- CSAT
- engagement

These may be useful operationally, but they do not automatically prove that the team is creating enough value to justify its cost. :contentReference[oaicite:7]{index=7}

The article’s point is sharp:

- a team can ship more and still build the wrong thing
- engagement can rise while revenue quality worsens
- velocity can increase while financial performance deteriorates

So activity metrics are not substitutes for investment logic.

---

## Why Organizations Became Like This

The article explains the historical reason.

From roughly **2011 to 2022**, software companies operated in an environment of:

- cheap capital
- strong risk appetite
- a dominant SaaS growth mindset

In that environment, companies could expand headcount aggressively, miss on many roadmap bets, and still appear healthy because growth masked inefficiency. :contentReference[oaicite:8]{index=8}

As a result, an entire generation of leaders learned to manage software teams without requiring explicit financial return.

When conditions changed after **2022**, those habits did not automatically adjust. :contentReference[oaicite:9]{index=9}

---

## The Big Reframe: Asset vs Liability

Traditionally, organizations treat these as assets:

- large codebases
- large engineering teams
- complexity built over time

The article argues that these also contain a hidden liability side:

- maintenance burden
- coordination overhead
- organizational inertia
- slower decision-making
- rising cost without rising return :contentReference[oaicite:10]{index=10}

This is a useful systems-thinking point:
scale is not automatically a moat if it also increases drag.

---

## Why LLMs Make This More Urgent

The article then connects this to LLMs.

Its core claim is not simply that AI makes coding faster, but that AI changes the economics of software production enough to challenge old assumptions about large teams and large codebases.

The example given is that one developer, using LLM agents, built a functional replica of most of Slack’s core product in a short period. The article uses that as evidence that functional approximation cost is collapsing. :contentReference[oaicite:11]{index=11}

The implication is:

- scale alone is no longer a durable defense
- accumulated complexity may no longer justify itself
- companies must become clearer about what exactly their teams are economically defending or creating

This is one of the strongest strategic insights in the piece.

---

## Best Insight

The best sentence-level takeaway from this article is:

> Engineering teams are not just delivery units. They are capital allocation vehicles.

That means every roadmap choice is also a financial decision.

This is the mindset shift the article keeps pushing:
from "what are we building?" to "what return does this work generate relative to its cost?"

---

## My Interpretation

This article is valuable because it bridges three levels at once:

### 1. Team level
It gives a practical method to think about whether a team is economically justified.

### 2. Management level
It criticizes the habit of using delivery and sentiment metrics as if they were business proof.

### 3. Industry level
It argues that the cheap-capital era trained organizations to ignore return discipline, and that AI now makes that avoidance harder to sustain.

In other words, this is not just a management article.
It is an argument about how software organizations should think under new economic and technical constraints.

---

## What I Learned

A few lessons stand out:

### Engineering work should be translated into value language
If work cannot be linked to time saved, revenue gained, churn reduced, risk avoided, or reliability improved, it becomes hard to justify economically.

### Platform and backend work need clearer value narratives
These teams often create real value, but that value is hidden unless explicitly measured.

### Shipping is not enough
A fast team can still be a bad investment if the work does not connect to meaningful outcomes.

### LLMs increase pressure on justification
As implementation cost falls, organizations will increasingly ask why a specific team, system, or codebase must exist in its current form.

---

## How I Can Apply This

This framework is especially useful for my own projects.

### Example: Job crawler / AI analysis service
Instead of describing it only as a crawler + LLM pipeline, I can frame value as:

- reduced manual search time
- better relevance ranking
- faster discovery of high-signal job postings
- structured extraction for decision support

### Example: notification automation
The value is not just automation itself, but:

- elimination of repeated manual checking
- lower probability of missing time-sensitive notices
- faster response to opportunities

### Example: internal tooling or platform work
The correct question becomes:
how many hours, incidents, or repeated setup steps does this remove?

This makes project descriptions much stronger in both portfolio and interview settings.

---

## Final Takeaway

The main takeaway is simple:

**Software teams should be evaluated less like busy builders and more like investments.**

A team that costs real money must be able to explain, in concrete terms, what value it creates and why that value is large enough to justify continued investment.

That is the core message of this article, and it becomes even more important in a world where LLMs reduce the cost of producing software. :contentReference[oaicite:12]{index=12}
