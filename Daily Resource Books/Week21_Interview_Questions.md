# Week 21 — Interview Questions

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Companion to:** `Week_21_Revised.md` (Days 141–148)

---

Every interview question from every Week 21 day, consolidated into one review document — 47 questions across 8 days. Week 21 has no LeetCode-style problems (behavioral, domain-knowledge, and portfolio-finalization content throughout), so this bank reads differently from Weeks 1–20's: no complexity derivations, but the same standard of "can you defend this cold if pushed" applies to every question below, whether it's a STAR-construction question, a distributed-systems mechanism, or your own program's verified numbers.

---

## Day 141 — The STAR Framework, Competency Map, and UPI's Real 2PC

**Q1. What are the four parts of a rigorous STAR answer, and what's the one-sentence failure mode for each?** Situation (fails if it rambles past a sentence or two); Task (fails if it doesn't name why the task was specifically yours); Action (fails if it's passive-voice "we" with no isolable individual decision); Result (fails if it's a mood instead of a measured outcome).

**Q2. Name the six competencies in the company-agnostic map.** Ownership, Navigating Ambiguity, Conflict & Disagreement, Failure & Learning, Technical Leadership, Cross-Functional Pushback.

**Q3. What's the one-line test that separates Ownership from Technical Leadership?** Ownership is about driving your own deliverable to completion; Technical Leadership is about influencing other people's technical direction without formal authority over them.

**Q4. What's the one-line test that separates Conflict & Disagreement from Cross-Functional Pushback?** Conflict & Disagreement is generally within your own team or reporting line; Cross-Functional Pushback is specifically against pressure from a different function or team.

**Q5. Why is "it went well" not a valid STAR Result?** It's a mood, not a measured outcome — a rigorous Result names a number wherever a real one exists, or at minimum a concrete, verifiable change in state.

**Q6. Walk through 2PC's coordinator-crash blocking failure, applied to UPI.** If NPCI (coordinator) sends Commit to Bank A but the message to Bank B is lost after Bank B already voted yes, Bank B is stuck holding its hold — it can't unilaterally commit or abort, since it doesn't know what the other participant was told, and must wait for the coordinator to recover and clarify.

**Q7. What actually resolves that blocked/ambiguous state in practice, and why is it safe to retry?** A retry tagged with the same idempotency key used for the original instruction (Week 19, Day 130's mechanism) — the receiving bank checks whether that exact key was already processed and returns the stored result if so, which is what prevents a naive retry from double-crediting the account.

**Q8. Is UPI's real-world failure handling a new consistency-vs-availability trade-off, or a familiar one in a new place?** The same one already established for Saga vs. 2PC (Week 10/12) — synchronous 2PC inside NPCI's own commit decision buys strong consistency at the cost of blocking; the asynchronous idempotent-retry reconciliation layer buys availability at the cost of a visible pending window, which is exactly Saga's own trade-off, one layer down.

**Q9. Why is Saga, by itself, a weaker fit than 2PC for the core UPI debit/credit decision specifically?** Saga's compensating-transaction model tolerates a visible partial-completion window, which is a much higher-stakes cost when the resource in question is money actually leaving an account, however briefly, versus a reversible order-status field.

---

## Day 142 — STAR Stories 3–4, and High Availability

**Q1. What's the precise difference between Active-Active and Active-Passive?** Active-Active runs multiple instances serving live traffic simultaneously, so a failure just reduces capacity with no failover delay; Active-Passive keeps a standby idle until the active instance fails, incurring a real, measurable detection-plus-promotion gap.

**Q2. Why isn't "we run three replicas" automatically evidence of High Availability?** If all three sit in the same failure domain (same rack, zone, or region), a single domain-level failure can take out all three at once — real HA requires the replicas to be in independent failure domains, not just duplicated.

**Q3. What mechanism actually triggers a failover?** A load balancer's health check polling each backend on an interval; after a configured number of failed checks, the load balancer stops routing new traffic to that instance — the same mechanism as Kubernetes removing a Pod from a Service's Endpoints on a failed readiness probe.

**Q4. Why does an HA claim need to be chaos-tested rather than just assumed from the architecture diagram?** An untested redundancy mechanism might be misconfigured in a way that only a real, induced failure reveals — a chaos experiment is what actually confirms the failover behaves as designed rather than merely as intended.

**Q5. Distinguish High Availability from Disaster Recovery.** HA is about surviving a failure with continuous or near-continuous service; DR is about recovering after a catastrophic event using backups and a recovery plan, typically accepting some downtime and data-loss window (RTO/RPO) — a system can need both, and they solve different problems.

**Q6. What new failure mode does Active-Active introduce that Active-Passive doesn't have to worry about?** Split-brain — both active sides believing they're authoritative during a network partition — resolved the same way Week 18's Distributed Consensus content resolves leader election: a quorum-based majority mechanism.

---

## Day 143 — STAR Stories 5–6, and High-Volume, Low-Margin Transaction Systems

**Q1. Why does the same set of tools — caching, sharding, rate limiting — matter *more*, not differently, in a high-volume low-margin system?** Because the cost of an inefficiency multiplies by transaction volume against a thin per-unit margin, so the same absolute inefficiency that's a minor annoyance in a high-margin system can consume a meaningful share of total margin at scale.

**Q2. Give a concrete example of how a cache-hit-rate target becomes a cost lever, not just a latency one.** At high volume, even a small cache-miss percentage represents a large absolute number of full-cost database round-trips a day — reframing the hit-rate target in terms of avoided infrastructure cost, not just avoided latency, is the point.

**Q3. Why is a naive retry-on-failure policy riskier in a high-volume low-margin system specifically?** Retries amplify load on a downstream service exactly when it's already struggling, and at high volume that amplified cost is a direct, quantifiable margin loss rather than a purely defensive stability concern.

**Q4. Name all ten LLD systems and each one's primary pattern, cold.** Coffee Ordering (Decorator), WeatherStation/Display (Observer), Tic-Tac-Toe (none — framework's first run), Vending Machine (State), Parking Lot (none — concurrency via per-spot locking), Library Management (data-driven transition table), ATM (State + Chain of Responsibility), Elevator (State + SCAN/LOOK), Splitwise (Strategy, dual-heap settlement), BookMyShow (none at the design layer — pessimistic/optimistic concurrency control).

**Q5. Why was Story 6 deliberately framed around Navigating Ambiguity rather than left to default toward Ownership?** Because the plan left it as the one story without an explicit competency label, and a running tally showed Navigating Ambiguity was the only one of the six competencies with zero coverage otherwise — the outage scenario fits it well on its own merits (acting on incomplete information) independent of the gap, which made it the natural fix.

---

## Day 144 — STAR Stories 7–8, and Multi-Tenant SaaS at Scale

**Q1. What makes a Cross-Functional Pushback story different from a Conflict & Disagreement story with a manager?** The audience is a different function without your technical context by default, so the Action has to show translating a technical concern into terms that function can actually evaluate, not just asserting technical correctness within a shared technical vocabulary.

**Q2. What's the difference between "helping" and "mentoring" in a Technical Leadership story?** Helping solves the problem for the person; mentoring guides them to find the fix themselves — a story where you did the fixing while they watched is a help story, not a mentoring one.

**Q3. Name the three-tier multi-tenant isolation spectrum and the trade-off each tier makes.** Shared database/shared schema (lowest isolation, lowest cost, highest blast radius); shared database/separate schema (middle ground); separate database per tenant (highest isolation and cost, lowest blast radius).

**Q4. Explain the noisy-neighbor problem and name two concrete mitigations.** One tenant's heavy usage consumes shared infrastructure that every other tenant also depends on, degrading their experience through no fault of their own; mitigated by per-tenant rate limiting and connection-pool partitioning, both extensions of already-known mechanisms rather than new ones.

**Q5. Why does RBAC attach permissions to roles instead of directly to users, and what extra piece does multi-tenancy add to that model?** Direct user-to-permission assignment is an unmanageable N×M problem at scale; role-based indirection keeps it to a small, stable set of roles. At multi-tenant scale, roles themselves need to be tenant-scoped, or one tenant's custom role definitions can leak into another tenant's access model.

**Q6. Why is rate limiting alone insufficient protection against the noisy-neighbor problem?** Rate limiting caps request volume, not the cost of an individual request — a low-frequency but expensive, unbounded query can still starve shared resources while staying well under a reasonable rate limit.

---

## Day 145 — Closing the Competency Map; Googleyness, Databricks, and Atlassian

**Q1. What are Google's four hiring attributes, and where does Googleyness sit among them?** General cognitive ability, leadership, Googleyness, and role-related knowledge — Googleyness is evaluated as a distinct, separate axis from technical skill, not folded into either the cognitive-ability or role-related-knowledge assessments.

**Q2. Name Databricks' six leadership principles.** Customer obsession, raising the bar, truth-seeking, operating from first principles, a bias for action, and putting the company first.

**Q3. Why is reciting a company's named values directly in your answer ("this shows my bias for action") usually a weak move?** It can read as reverse-engineering the answer to the rubric rather than genuinely embodying the value — the stronger move is technical substance and reasoning that's naturally compatible with the value without naming it outright.

**Q4. Recite Atlassian's five values in order.** Open Company, No BS; Build With Heart and Balance; Don't [Mess With] the Customer; Play, as a Team; Be the Change You Seek.

**Q5. Why does Story 7 map specifically to "Play, as a Team" rather than just to Cross-Functional Pushback generically?** Because the value isn't about being right in the disagreement — it's about disagreeing in a way that preserves the working relationship afterward, which is exactly what Story 7's Result is built to demonstrate.

**Q6. Why is it valuable that a single story (like Story 4) can be reframed for three different frameworks rather than needing three separate stories?** Because you often won't know in advance which company-specific framework a given interview is actually using — the ability to reframe on demand is a more realistic skill to have rehearsed than three narrowly-purpose-built stories.

---

## Day 146 — Final Technical Review Pass, System Design Mock, and Resume Finalization

**Q1. What four things does the 2-minute cold-recall structure cover, in order?** What it is, when you reach for it, its trade-off against the nearest alternative, and one common mistake.

**Q2. Why are Two Pointers and Binary Search good candidates for today's DSA spot-check specifically, rather than a more recently-covered pattern?** Neither appears in the curriculum map's DSA Revision Log (Weeks 16–19) — they're the longest-untouched material in the curriculum, making them the sharpest test of genuine long-term retention rather than recent-practice memory.

**Q3. What's the verified, defensible total for DSA problems solved, and what does it include?** 249 distinct problems — 197 assigned by the study plan plus additional self-driven practice across every major pattern — directly re-summable from the curriculum map's own row-by-row table and running totals.

**Q4. Why shouldn't "213" go on the resume as currently stated?** It doesn't match any of the curriculum map's directly-verifiable figures (197 required-only, 249 distinct including extras, 259 including the SQL track) — without a traceable derivation, it isn't defensible if an interviewer asks how the number was reached.

**Q5. What distinguishes a System Design mock's evaluation criteria from an LLD mock's?** An LLD mock is scored on class-level design and pattern choice within one process; a System Design mock is scored on driving the 5-step HLD framework itself, including whether Estimation numbers are reasoned through rather than skipped.

---

## Day 147 — Final Portfolio Polish and Job Search Strategy

**Q1. Why does pin order matter more than individual README quality for a first impression?** A skimming recruiter typically only sees the six pinned repos without scrolling and reads a few lines of whichever looks most relevant — pin order controls what's even seen, before README quality has a chance to matter.

**Q2. What should the first few lines of a portfolio README lead with, and why?** Impact and the most technically interesting decisions, not setup instructions — a skimming reader who has to scroll past installation steps to find out what the project does has usually already moved on.

**Q3. Mechanically, why does a referral change an application's odds?** It often bypasses the ATS's initial keyword-filtering step or routes directly to a recruiter/hiring-manager queue instead of the general pool, and it carries a small, real signal of a current employee vouching for the candidate.

**Q4. What's a reasonable follow-up cadence after an application or referral request goes unanswered?** One polite follow-up after roughly one to two weeks of silence, then generally letting it rest unless there's a genuinely new reason to reach out again.

**Q5. Of the three status buckets (in progress, needs follow-up, needs a referral ask), which one is usually the highest-leverage to act on, and why?** The referral-ask gap — it's the one lever most directly shown to change an application's actual odds, versus a follow-up (which mostly just re-surfaces an already-submitted application) or waiting on something already in progress.

---

## Day 148 — Consolidation, and the Technical Preparation Phase Closes

**Q1. What's the final, defensible total for DSA problems solved across the series, and why is it that number and not the plan's own "213"?** 249 distinct problems — 197 required by the plan plus additional self-driven practice — directly traceable to the curriculum map's row-by-row table and running totals; "213" doesn't match any of the map's verifiable figures (197, 249, or 259 with the SQL track included), so it isn't used here.

**Q2. Name the two pattern families this series added beyond what the original plan's earliest drafts contained.** Design Patterns (opened Week 15, carried through the LLD phase) and the SQL practice track (Week 15) — both confirmed as additive against the curriculum map's own extension log, not part of the plan's original scope.

**Q3. Name the four "thin-pattern" gaps from the original plan audit that this series specifically closed.** Union-Find (the audit's thinnest gap), Graphs BFS/DFS (the second-thinnest), Dijkstra's Algorithm (its required ladder expanded from 3 to 5 problems), and 1D Dynamic Programming (expanded from 12 to 14 required problems).

**Q4. What distinguishes the "technical preparation phase," which closes today, from the "interview execution phase," which doesn't start until after it?** The preparation phase — everything this series built, from DSA fundamentals through behavioral rehearsal and portfolio polish — is fully within this series' scope and complete as of today; the execution phase (attending real interviews, debriefing them, negotiating, accepting an offer) runs on company hiring calendars rather than a fixed plan, and is explicitly outside what any Resource Book in this series prepares for directly.

**Q5. If asked, cold, to summarize this entire preparation program in under thirty seconds, what's the honest, accurate version?** Something close to: "21 weeks, built from zero — 249 DSA problems across every major interview pattern plus a dedicated SQL track, 10 LLD and 16 HLD systems each backed by six mock interviews, a capstone platform hardened to a real chaos-tested Kubernetes deployment with a formal at-scale analysis, and eight true behavioral stories rehearsed against a company-agnostic map and three real companies' own frameworks."
