# Day 142 — STAR Stories 3–4, and High Availability

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 141 Resource Book](Day141_Resource_Book.md)
**Next ▶:** [Day 143 Resource Book](Day143_Resource_Book.md)
**Companion to:** Day 142 of `Week_21_Revised.md`

---

## Recap

Day 141 established the STAR mechanism (S/T/A/R timing split, the passive-"we" trap, quantifying a Result without an obvious number) and the six-competency map, then built Story 1 (Ownership) and Story 2 (Conflict & Disagreement). Today reuses that mechanism without re-explaining it — the worksheets below assume yesterday's shape is already familiar.

On the domain-knowledge side: Chaos Engineering was fully taught last week as formal practice (Week 20, Day 138 — three real experiments: Payment-kill via scaling to zero, latency injection against Product, connection-pool saturation). Today's High Availability theory cites that work directly rather than re-teaching what chaos engineering *is* — today is about what HA actually means and how the failover mechanism works, using Day 138's own experiments as live case studies for what HA claims look like when someone actually checks them.

---

## Learning Objectives

By the end of today, without notes:

1. Tell Story 3 and Story 4 to the same rigor bar as Days 141's stories, correctly mapped to Failure & Learning and Ownership respectively.
2. Explain Active-Active vs. Active-Passive precisely enough to state which one a given system is using from its behavior alone, and argue the cost/benefit of each without hedging.
3. Explain exactly what triggers a real failover — not "the system detects a failure" as a black box, but the actual health-check-to-Service-update mechanism, citing the concrete Kubernetes example you already built.
4. Distinguish High Availability from Disaster Recovery, and explain why "we have three replicas" is not, by itself, evidence of HA.

---

## Concept Dependency Map

```
Day 141: STAR mechanism (S/T/A/R, timing, quantification) — no re-teach
        │
        └──▶ Story 3 (Failure & Learning) + Story 4 (Ownership)


Week 20, Day 138: Chaos Engineering (formal practice, 3 real experiments)
Week 20, Day 134: Kubernetes readiness probes / Endpoints removal (the actual failover trigger)
Week 18, Day 125: Quorum / split-brain (already taught, for active-active's own failure mode)
        │
        └──▶ NEW: High Availability
                   ├─ Active-Active vs. Active-Passive
                   ├─ Load-balancer health checks as the failover trigger
                   └─ Chaos Engineering as verification, not assumption
```

---

## Part 1 — Story Workshop: Story 3 (Failure & Learning) and Story 4 (Ownership)

### Story 3 — "A time you failed" → Failure & Learning

**Construction worksheet:**

1. *Situation:* What was the system or project, and what was supposed to happen?
2. *Task:* What was your specific responsibility in the part that failed?
3. *Action:* What actually went wrong, stated plainly — and then, separately, what you did once you knew?
4. *Result:* What changed afterward, concretely — a process, a safeguard, a piece of testing that didn't exist before?

**⚠️ A framing choice worth making deliberately, not by default:** the plan points you at your real Week 20 chaos-engineering findings as material for this story — and that's genuinely strong material, but it needs one careful distinction stated honestly if you use it. A chaos experiment is a **deliberately induced, controlled test**, not a spontaneous production incident. If your chaos work surfaced a real weakness (say, the connection-pool-saturation experiment revealed a cascading failure mode you hadn't anticipated), that is completely legitimate Failure & Learning material — arguably a *better* signal than a reactive incident, because it shows you went looking for weaknesses rather than waiting for one to find you. But tell it as what it actually was: *"I ran a deliberate chaos experiment specifically because I suspected our connection pool handling was fragile, and it confirmed a cascading failure mode I hadn't fully accounted for"* is honest and strong. Implying, even by omission, that this happened to live customer traffic when it didn't is a real risk — a good interviewer's natural follow-up ("was this affecting real users at the time?") will surface the truth anyway, and being caught reshaping the framing costs you far more credibility than the honest version ever would have.

**What "strong" looks like here specifically:** the Result names a concrete change, not a vague resolve. "I added a circuit breaker" is concrete. "I learned to be more careful about connection pooling" is not — it's the single most common weak-Result pattern for this competency, and it's worth actively avoiding.

**⚠️ Common Mistakes, specific to this story:**
- Choosing a "failure" that's actually someone else's fault, framed to avoid taking real ownership of your own part in it.
- A Result with no verifiable change — "I learned X" without "...and here's the specific thing I built/changed/checked afterward as a direct consequence."

**💡 Interview Insight:** expect *"if you hadn't caught this in a controlled test, what would have actually happened in production?"* — having a specific, concrete answer (not a vague "it could have been bad") is exactly the kind of severity-awareness this question is probing for.

### Story 4 — "A time you went above and beyond" → Ownership

**Construction worksheet:**

1. *Situation:* What was the actual scope of what was asked of you?
2. *Task:* What gap did you notice that fell outside that scope?
3. *Action:* What did you do about it, and — importantly — did you flag it to anyone, or just quietly do it? Either can be right, but be ready to say which and why.
4. *Result:* What was the concrete benefit, and to whom?

**⚠️ The trap unique to this story type:** "above and beyond" stories are the ones most likely to sound like bragging, or worse, like boundary-violating overreach (rewriting someone else's system without asking, for instance). The test that keeps this story credible rather than self-congratulatory: was there a **real, verifiable gap** (not just "I decided my way was better"), and did your action **actually help** rather than just represent extra unrewarded effort for its own sake? A strong version names the gap specifically enough that its reality isn't in question.

**Quantifying:** who benefited, and how — time saved for a specific team, a class of bug that stopped recurring, an onboarding process that got measurably faster. If the honest answer is "I'm not sure anyone else noticed," that's a signal this particular story may be a better fit for a different, lower-stakes moment in the conversation than a headline Ownership story.

**⚠️ Common Mistakes, specific to this story:**
- No clear "beyond what," i.e., what the actual, stated scope was — without that baseline, "above and beyond" has nothing to be measured against.
- Overreach dressed up as initiative — changing something without any communication, in a way that could have gone badly, isn't automatically Ownership; it can just as easily read as poor judgment about boundaries, depending how it's told.

---

## Part 2 — High Availability

### Active-Active vs. Active-Passive

**Active-Active:** multiple instances (or entire regions) serve live traffic *simultaneously*. If one fails, the others are already handling load — there's no failover delay in the traditional sense, because nothing needs to "take over"; capacity simply drops until the failed instance recovers or is replaced.

**Active-Passive:** one instance (or region) serves traffic; one or more standbys sit idle, ready to take over if the active one fails. There is a real, measurable gap between the failure and the standby actually taking over — detection time plus promotion time.

| | Active-Active | Active-Passive |
|---|---|---|
| Cost | Higher — every standby is fully provisioned and running, not idle | Lower — the standby can often be smaller or entirely dormant until needed |
| Failover gap | None in principle — capacity just drops | Real and measurable — detection + promotion time |
| Complexity | Higher — traffic must be correctly split/routed across all active instances, and state (if any) must be consistent across them | Lower — only one instance is ever actually serving traffic at a time |
| Best fit when... | Downtime is extremely costly and the system can tolerate the added cost and complexity of running everything twice | The failure mode is rare enough, and a short recovery window tolerable enough, that paying for full duplicate capacity isn't justified |

**⚠️ Common Mistake:** treating "we run three replicas" as automatically Active-Active HA. If all three replicas live in the same failure domain (same rack, same availability zone, same region), a single domain-level failure takes out all three simultaneously — replication without domain diversity buys you load distribution and rolling-deploy safety, not real availability against the failure modes that actually matter at scale. Real Active-Active HA specifically requires the replicas to sit in **independent failure domains**.

### What Actually Triggers a Failover — Not a Black Box

"The system detects the failure and fails over" is the sentence a weak answer stops at. The mechanism, precisely: a **load balancer's health check** polls each backend instance on a fixed interval; when an instance fails a configured number of consecutive checks, the load balancer stops routing new traffic to it. That's the entire mechanism — there's no separate, more mysterious "failure detection" system underneath it.

**You've already built exactly this, concretely, and it's worth citing by name rather than describing HA abstractly:** Kubernetes' own readiness probe (Week 20, Day 134) *is* a health check in this exact sense — when a Pod fails its readiness probe, Kubernetes removes it from the Service's Endpoints, and the Service (acting as the load balancer here) simply stops sending it traffic. This is the identical mechanism industrial HA setups use at a larger scale, not an analogy to it.

**🔑 Key Takeaway:** HA isn't a separate system bolted on top of your architecture — for anyone who's already built a real health-check-driven system, it's the same mechanism, just with the standby/redundant capacity sized and placed deliberately to survive the specific failure domains you're worried about.

### Chaos Engineering as Verification, Not Assumption

An HA claim that's never been tested against a real, induced failure is an assumption, not a verified property — this is precisely why Day 138's chaos experiments matter beyond satisfying a checklist item. Reframing that work as an HA case study directly:

**Case study — Day 138's Payment-kill experiment, reframed:** the experiment scaled the Payment module to zero replicas rather than deleting a single pod, specifically because a single pod delete would just have triggered Kubernetes' own reconcile loop (Week 15, Day 99) healing itself within seconds — not a real test of anything. Scaling to zero is the honest equivalent of "this entire failure domain is down" — closer to an Active-Passive-style total outage of the active side than a partial degradation. What this experiment verified, concretely: whatever depended on Payment correctly detected the outage (via its own health checks / circuit breaker, Week 10 Day 64's Resilience4j) and degraded or queued rather than cascading into an unrelated failure. **That's the actual content of an HA claim** — not "we have redundancy," but "we induced the failure for real and confirmed the redundancy behaved as configured."

**⚠️ Common Mistake:** conflating High Availability with Disaster Recovery. HA is about surviving a failure with continuous (or near-continuous) service; DR is about *recovering* after a catastrophic event (data center loss, a bad deployment that corrupts data) using backups and a recovery plan, typically with some acceptable downtime and data-loss window (RTO/RPO). A system can have excellent HA and still need a separate, distinct DR plan — they answer different questions about different failure classes.

### Extra Scenarios

1. **Split-brain in an Active-Active setup.** Already have the tool for this — Week 18, Day 125's Quorum/split-brain coverage applies directly: without a quorum-based mechanism to establish which side of a network partition is authoritative, two active regions can both believe they're the sole source of truth simultaneously, which is a correctness problem, not just an availability one.
2. **False-positive health checks causing a failover storm.** If a health check is miscalibrated (too aggressive a timeout, checking a dependency rather than the service's own health), a brief, unrelated blip can trigger mass failover across many instances at once — precisely the dependency-in-liveness anti-pattern Day 139 already named, now recognized as an HA risk specifically, not just a Kubernetes quirk.
3. **The failover mechanism itself becomes the single point of failure.** If there's only one load balancer and it goes down, every backend's HA setup is irrelevant — worth naming explicitly if asked "where's the weakest link in this design," since it's an easy thing to design around (a redundant load-balancer tier) but an easy thing to forget to mention.

### 💡 Interview Insight

If asked to design for "five nines" (99.999% uptime), the strong answer doesn't just say "Active-Active across multiple regions" — it names the actual annual downtime budget that implies (about five minutes a year) and reasons from there about how little room that leaves for anything except automated, health-check-driven failover; a human-in-the-loop failover process cannot realistically hit that budget, which is itself worth saying out loud.

---

## Project Block Guide (1.5 hrs)

**Repository:** none today. **Task:** draft a one-paragraph answer to "tell me about a time a system you built failed," using your real Week 20 chaos-test findings.

This is Story 3, written down formally rather than just rehearsed verbally — use the worksheet from Part 1 above. Write the paragraph, then read it back and check specifically: does it name the concrete change that resulted, and does it correctly represent this as a deliberate test rather than implying a live incident?

**Definition of done:** written, and rehearsed once out loud against the 90-second budget.

## Career Block Guide (1 hr)

**Mock Interview: 45-Minute Behavioral & Resume Deep Dive.**

What this format actually assesses, so you can prepare for the format itself rather than just the content: this round typically moves fast between multiple behavioral prompts (expect 3–4 in 45 minutes, leaving room for follow-ups) and pairs them with resume-specific questions — "walk me through this project," "what was your specific role here" — testing whether your resume bullets hold up under a direct follow-up the way today's Story 3 and Story 4 need to.

**Prep checklist:**
- Have Stories 1–4 rehearsed and ready to deploy against an unfamiliar phrasing (Day 141's "extra prompts" list is exactly for this).
- Re-read your own resume as if you were the interviewer, and for each bullet, ask yourself the follow-up an interviewer would ask — if you can't answer it cleanly, that's worth knowing *before* the mock, not during it.
- After the mock: get feedback in writing if possible, and specifically ask your accountability partner which competency, if any, felt weakest — that's today's data point feeding directly into Day 145's coverage audit.

---

## Day 142 — Interview Questions

**Q1. What's the precise difference between Active-Active and Active-Passive?** Active-Active runs multiple instances serving live traffic simultaneously, so a failure just reduces capacity with no failover delay; Active-Passive keeps a standby idle until the active instance fails, incurring a real, measurable detection-plus-promotion gap.

**Q2. Why isn't "we run three replicas" automatically evidence of High Availability?** If all three sit in the same failure domain (same rack, zone, or region), a single domain-level failure can take out all three at once — real HA requires the replicas to be in independent failure domains, not just duplicated.

**Q3. What mechanism actually triggers a failover?** A load balancer's health check polling each backend on an interval; after a configured number of failed checks, the load balancer stops routing new traffic to that instance — the same mechanism as Kubernetes removing a Pod from a Service's Endpoints on a failed readiness probe.

**Q4. Why does an HA claim need to be chaos-tested rather than just assumed from the architecture diagram?** An untested redundancy mechanism might be misconfigured in a way that only a real, induced failure reveals — a chaos experiment is what actually confirms the failover behaves as designed rather than merely as intended.

**Q5. Distinguish High Availability from Disaster Recovery.** HA is about surviving a failure with continuous or near-continuous service; DR is about recovering after a catastrophic event using backups and a recovery plan, typically accepting some downtime and data-loss window (RTO/RPO) — a system can need both, and they solve different problems.

**Q6. What new failure mode does Active-Active introduce that Active-Passive doesn't have to worry about?** Split-brain — both active sides believing they're authoritative during a network partition — resolved the same way Week 18's Distributed Consensus content resolves leader election: a quorum-based majority mechanism.

---

## Daily Deliverable Check

- [ ] Stories 3 and 4 rehearsed to 90 seconds each, mapped to Failure & Learning and Ownership, with the chaos-test framing stated honestly as a controlled experiment rather than implied to be a live incident.
- [ ] Can explain Active-Active vs. Active-Passive, the load-balancer health-check failover mechanism, and HA-vs-DR from memory, without hedging.
- [ ] The one-paragraph failure story written and pushed/saved, rehearsed once out loud.
- [ ] Behavioral mock completed, feedback captured, weakest-competency signal noted for Day 145.

---

## What Tomorrow Assumes You Already Know Cold

Day 143 assumes the STAR mechanism is now fully reflexive across four stories' worth of practice, and it assumes today's chaos-test framing distinction (controlled experiment vs. live incident) carries forward without re-explanation, since tomorrow's Story 6 draws on the same Week 20 data again from a different angle ("resolved a production outage") — where that same honesty distinction becomes even more load-bearing.
