# Week 21 (Revised): Behavioral Mastery, Domain Knowledge, and Final Polish

**What changed:** the original behavioral week was built entirely around Amazon's 16 Leadership Principles, back when no specific target companies existed yet. They exist now — Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech — and Amazon isn't one of them. This week teaches a company-agnostic competency map first, so every story has a clean underlying home regardless of framing, then rehearses it against the three frameworks your actual targets use: Google's Googleyness, Databricks' own principles, and Atlassian's named values. Amazon's LPs are dropped entirely rather than kept as a "just in case" — time here should go toward companies you're actually applying to.

---

## Day 141 — The STAR Framework, Rigorously, and a Company-Agnostic Competency Map

### Theory Block (2.5 hrs)
- Topic: The STAR Framework, Applied Rigorously
- Situation, Task, Action, Result — the trap most candidates fall into is a vague Situation, a passive-voice Action ("we decided..." instead of "I decided..."), and a Result with no number attached. Every story should have a quantified result wherever genuinely possible (latency reduced by X%, an incident resolved Y minutes faster, team velocity up Z%) — "it went well" is not a result.
- Topic: A Competency Map That Isn't Tied to One Company
- Most behavioral rubrics, across most companies, actually cluster around the same handful of underlying things, just with different names attached: **Ownership** (did you drive something to completion without being told every step), **Navigating Ambiguity** (how you acted when requirements or direction weren't clear), **Conflict & Disagreement** (how you handled disagreeing with a peer, manager, or the data itself), **Failure & Learning** (a genuine failure, what you took from it, what changed afterward), **Technical Leadership** (influencing a technical direction without formal authority over the people involved), **Cross-Functional Pushback** (holding a position against product, design, or another team's pressure, for a good reason). Google's "Googleyness," Microsoft's growth-mindset framing, Meta's own competencies, and Amazon's Leadership Principles are all, underneath, mostly reshuffled versions of this same set. Build your story bank against *this* map first — then any specific company's framework becomes a relabeling exercise, not new preparation.
- Domain Knowledge: How UPI Works Internally
- NPCI acts as the central switch, PSPs (Payment Service Providers) form the interface layer, and bank nodes hold the actual accounts. Two-Phase Commit shows up for real here — debiting one bank and crediting another must both succeed or both roll back, and safely handling a pending or ambiguous transaction after a network failure mid-transfer is a genuinely hard, real distributed-systems problem, not a textbook exercise.
- Coding exercise: rehearse Story 1 (a time you handled a tight deadline — maps to Ownership) and Story 2 (a time you disagreed with a manager — maps to Conflict & Disagreement) out loud, timed to 90 seconds each, and note which competency each one is really demonstrating.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: write `docs/architecture.md`'s final section — explaining the Saga pattern choice, the DB/Redis split, the move to Kubernetes/Helm, and how you'd extend the design toward genuine UPI-style payment handling if asked in an interview.
- Definition of done: pushed.

### Career Block (1 hr)
- LinkedIn: Post 28 — full platform architecture diagram, tying together everything built since Week 9.
- Networking: respond to interview requests, confirm logistics.

### Daily Deliverable
- [ ] Stories 1-2 rehearsed to 90 seconds each, with a quantified result in both, and mapped to the competency map.
- [ ] UPI/2PC-in-practice knowledge reviewed. Architecture doc finalized. LinkedIn Post 28 published.

---

## Day 142 — STAR Stories 3-4, and High Availability

### Theory Block (2 hrs)
- Topic: High Availability
- Active-Active (multiple regions or instances serving traffic simultaneously — higher cost, no failover delay) vs. Active-Passive (a standby takes over on failure — cheaper, but a failover gap exists). Load balancer health checks are what actually trigger failover in practice. Chaos Engineering, which you practiced for real against a Kubernetes deployment two weeks ago, is the concrete way HA claims get verified rather than just assumed.
- Coding exercise: rehearse Story 3 (a time you failed — maps to Failure & Learning) and Story 4 (a time you went above and beyond — maps to Ownership) out loud, timed to 90 seconds each.

### Project Block (1.5 hrs)
- Repository: none today.
- Task: draft a one-paragraph answer to "tell me about a time a system you built failed," using your actual Week 20 chaos-test findings as the true story — you have real material now, not a hypothetical.
- Definition of done: written and rehearsed once out loud.

### Career Block (1 hr)
- Mock Interview: 45-minute Behavioral & Resume Deep Dive with your accountability partner.
- Networking: review feedback from the mock.

### Daily Deliverable
- [ ] Stories 3-4 rehearsed to 90 seconds each, mapped to the competency map.
- [ ] HA concepts reviewed. Behavioral mock completed.

---

## Day 143 — STAR Stories 5-6, and Domain Knowledge: High-Volume, Low-Margin Systems

### Theory Block (2 hrs)
- Topic: Domain Knowledge — High-Volume, Low-Margin Transaction Systems
- Reseller and marketplace models (many small transactions, thin per-transaction margin) push hard on cost-efficiency at scale — this is precisely why the caching, sharding, and rate-limiting work from Weeks 18-19 matters practically, not just as interview trivia: at high volume, one inefficient query or one uncached hot path directly costs real money, not just latency.
- Coding exercise: rehearse Story 5 (a time you received critical feedback — maps to Failure & Learning) and Story 6 (a time you resolved a production outage — your Week 20 chaos test gives you real material again here) out loud, timed to 90 seconds each.

### Project Block (1.5 hrs)
- Repository: `lld-java`.
- Task: finalize the `lld-java` README — list all 10 completed LLD systems with a one-sentence summary of the core pattern each centers on.
- Definition of done: pushed, portfolio-ready.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: research interviewers for upcoming rounds if names were provided.

### Daily Deliverable
- [ ] Stories 5-6 rehearsed to 90 seconds each, mapped to the competency map.
- [ ] `lld-java` README finalized and portfolio-ready.

---

## Day 144 — STAR Stories 7-8, and Multi-Tenant SaaS Architecture

### Theory Block (2 hrs)
- Topic: Domain Knowledge — Multi-Tenant SaaS at Scale
- Multi-tenant (shared infrastructure, tenant data logically isolated — usually cheaper, harder to isolate a noisy-neighbor problem) vs. single-tenant (fully isolated, more expensive, easier to reason about security boundaries). Handling massive relational graphs and Role-Based Access Control both become genuinely harder at this scale than a typical CRUD app.
- Coding exercise: rehearse Story 7 (a time you pushed back on a product requirement — maps to Cross-Functional Pushback) and Story 8 (a time you mentored someone — maps to Technical Leadership) out loud, timed to 90 seconds each.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: a final pass ensuring the repository has a clean, professional README, since it'll be visible in your GitHub profile alongside the capstone as your early practice project.
- Definition of done: pushed and reviewed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: attend any scheduled recruiter calls or initial technical screens.

### Daily Deliverable
- [ ] Stories 7-8 rehearsed to 90 seconds each — all 8 STAR stories complete and mapped to the competency map.
- [ ] `todo-api` README finalized.

---

## Day 145 — The Three Frameworks That Actually Matter: Googleyness, Databricks' Principles, Atlassian's Values

### Theory Block (2.5 hrs)
- Topic: Closing the Competency Map
- You've mapped each story to one primary competency as you built it this week — today, go back over all 8 together and check for coverage gaps. Do you have at least one strong story for each of the six competencies (Ownership, Ambiguity, Conflict, Failure & Learning, Technical Leadership, Cross-Functional Pushback)? If any competency only has a weak or borrowed story, that's worth strengthening now, not discovering mid-interview.
- Topic: Google's "Googleyness and Emergent Leadership" — Its Own Evaluated Axis
- This is not generic behavioral — it's a distinct axis Google evaluates separately from technical skill, probing comfort with ambiguity, intellectual humility (can you say "I don't know" and mean it), collaborative instinct over lone-wolf brilliance, and genuine curiosity. Map Stories 2 (disagreement) and 8 (mentoring) against this specifically — they're your strongest fits.
- Topic: Databricks' Own Leadership Principles
- Real candidates specifically recommend relating your answers to phrases like "working from first principles" and "seeking truth" — Databricks' own named framework, evaluated the way Amazon evaluates its LPs, just under different names. Map Story 5 (critical feedback → seeking truth) and Story 3 (failure → first-principles reassessment) against this.
- Topic: Atlassian's Five Values — Treat This Round Like a Technical Round
- Open Company No BS, Build With Heart and Balance, Don't [Mess With] the Customer, Play as a Team, Be the Change You Seek. Multiple real candidates specifically warn that technically strong people have been downleveled or rejected here for answering arrogantly or unpreparedly — this round gets the same rehearsal time as a technical round from today forward, not improvisation. Map Story 7 (pushback, done respectfully — Play as a Team) and Story 4 (going above and beyond — Be the Change You Seek) against this.
- Coding exercise: the three mappings above, done in writing, plus one 90-second verbal rehearsal of your strongest story reframed for each of the three frameworks.

### Project Block (1.5 hrs)
- Repository: none today — this is pure interview prep.

### Career Block (1.5 hrs)
- Mock Interview: 60-minute LLD Mock Interview, practicing clean communication of a class design under time pressure — your sixth LLD mock across the whole plan.
- Networking: send follow-up thank-you notes to any interviewers so far.

### Daily Deliverable
- [ ] All 8 STAR stories checked against the competency map for coverage gaps, gaps addressed.
- [ ] Stories explicitly mapped and rehearsed against Googleyness, Databricks' principles, and Atlassian's values — the three frameworks your actual target companies use, not a generic one.
- [ ] LLD mock interview completed.

---

## Day 146 — System Design Mock, and Resume Finalization

### Theory Block (1 hr)
- Topic: Final Technical Review Pass
- Pick one DSA pattern, one LLD system, and one HLD system you haven't touched in a while, and explain each for 2 minutes, cold — a spot check before the heavier mock below.

### Project Block (2 hrs)
- Repository: none today.
- Task: **final resume pass** — incorporating every project and metric from the full 21 weeks: the platform's real load-test numbers, the chaos-test findings, all 10 LLD systems, all 16 HLD systems, the 213-problem DSA curriculum. This is the last of the scheduled resume checkpoints seeded throughout this plan — it should be a light polish at this point, not a rewrite, since it's been kept current since Week 6.
- Definition of done: resume reflects the complete, real body of work, ready to accompany every remaining application.

### Career Block (2 hrs)
- Mock Interview: 60-minute System Design Mock Interview — pick a system not yet mock-interviewed (Payment System or Distributed Job Scheduler are good choices, given their idempotency/consistency depth) — your sixth HLD-adjacent mock.
- Networking: rest and review the schedule for interview weeks ahead.

### Daily Deliverable
- [ ] Final resume pass complete.
- [ ] System Design mock interview completed.

---

## Day 147 (Weekend) — Final Portfolio Polish

### Theory Block (1 hr)
- Topic: Final Review Pass
- Walk your own GitHub profile as a stranger would — is the pinned-repo order right (capstone first), does every README actually explain what the project demonstrates, does the profile README's "Currently Building" section need updating to "Built" now that the full system is functionally complete?

### Project Block (2 hrs)
- Task: archive/pin all five repositories (`java-fundamentals`, `dsa-java`, `todo-api`, `lld-java`, `scalable-ecommerce-platform`) on your GitHub profile; update the profile README to reflect what's actually been built, not the Day-1 aspirational version. Note: this is five repos, not six — `order-management-api` never existed as a separate repo in this revised plan, since it was absorbed directly into the platform back in Week 9.
- Definition of done: profile is genuinely interview-ready — a recruiter or interviewer clicking through sees a coherent, complete story across five repos, not a scattered one across six.

### Career Block (2 hrs)
- Networking: a genuine job-search strategy session, not just another day of applications — review status across all 7 target companies (Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech) from `Target_Company_Research_and_Interview_Guide.md`, identify where a referral still hasn't been asked for despite a warm connection existing, and where a process has gone quiet long enough to warrant a follow-up. Given current market conditions, a real referral is worth more right now than another cold application.
- Reach out to your accountability partner — their role across this entire plan has been real, six LLD mocks, six HLD-adjacent mocks, and a behavioral mock don't happen without a second person showing up consistently. A proper thank-you is worth the time.

### Daily Deliverable
- [ ] GitHub profile fully polished and pinned, five repositories.
- [ ] Application and referral status reviewed across all 7 target companies, with next steps identified for each.

---

## Day 148 (Sunday) — Consolidation, and the Technical Prep Phase Closes

### Self-Check (20 min)
- [ ] Run through all 8 STAR stories once more, from memory, unprompted, checking they still map cleanly to the competency map.
- [ ] Pick one DSA pattern, one LLD system, and one HLD system at random and explain each for 2 minutes, cold — a full-spectrum readiness check before interview execution begins in earnest.

### Career Block (2 hrs)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard, and an honest Day 148 status:** every DSA pattern is closed at real depth (213 problems, including two pattern families and four thin-pattern fixes that didn't exist in earlier drafts of this plan at all), all 10 LLD systems are built with a genuine difficulty ramp and six mock interviews behind them — including one in the exact Machine Coding format Uber and Atlassian actually use — all 16 HLD systems are designed with six mock interviews of their own, several run in a target company's real format. The capstone is hardened with real Kubernetes/Helm deployment across three environments, genuine chaos-test data, a hand-rolled concurrency component matching Databricks' actual bar, real API design work matching Stripe's, and a formal at-scale analysis of its own architecture. All 8 STAR stories are rehearsed against a company-agnostic competency map, then specifically against Google's Googleyness, Databricks' principles, and Atlassian's values — the three real named frameworks your actual target companies use. Applications have been live since Day 118 across all 7 companies (Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech), targeted LinkedIn networking and reference-cultivation have been running in parallel, and the portfolio is polished around five coherent repositories.
- **What's not yet done, honestly: the interview execution phase itself** — attending remaining interviews, debriefing them, negotiating, accepting an offer, and transitioning out of your current role. That phase is calendar-bound by company hiring processes in a way none of the last 148 days were, and current market conditions (real, and covered honestly back at the start of this process) mean it may take real, multi-month calendar time even with strong preparation behind you. Treat today as the day you became genuinely, verifiably ready for exactly the companies you're actually targeting — not the day the goal was reached. The goal was always the offer, at 45+ LPA base, at one of these seven companies; today is the day you're prepared to go get one.

### Daily Deliverable
- [ ] Full-spectrum readiness self-check complete.
- [ ] Weekly ritual and scorecard complete — **the preparation phase, in full depth, at real pace, without a single corner cut to protect a calendar date, is done.**
