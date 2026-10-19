# Day 141 — Missing Primer: The Engineering-Process Vocabulary an SDE-2 Is Expected to Show

**Do this first on Day 141 (≈ 60 min, before the STAR/competency work), then continue with Day 141's regular plan.**
**Needed by:** Days 141–145 (behavioral stories), Day 147–148 (final mocks), and SDE-2 behavioral and "how does your team work?" rounds.
**Why it is missing:** design docs/RFCs/ADRs, estimation and planning, code review, on-call and blameless postmortems, and Agile ceremonies are mentioned in passing (Days 129, 143) but never taught, even though SDE-2 stories draw on them.
**Already in place:** STAR and behavioral prep (Days 140–148), architecture decisions in practice (ADR-001 from Day 34's file), testing, CI/CD, observability (Days 45–47, 76, 139).

---

## 1. Design docs (RFCs)
A short document written **before** building anything big, so the team can critique the idea cheaply. Typical sections: **context & problem · goals and non-goals · proposed design (diagram, APIs, data model) · alternatives considered and why rejected · trade-offs · rollout and rollback plan · risks & open questions · estimates**. The *alternatives* and *non-goals* sections show seniority. An **ADR** (architecture decision record) is the one-page version of a single decision: **Title · Status · Context · Decision · Consequences** (you wrote one on Day 34).

## 2. Estimation and planning
Break work into pieces of **≤ 2 days**; estimate each as a **range** with the main risk named; add buffer for review, testing, rollout, and unknowns; re-estimate when you learn something. **Story points** measure relative effort, not time; **velocity** is the average points per sprint. Say "I'm 70 % confident in 3 weeks; the risk is the payments API" — not a single number. Slipping early and loudly beats slipping late and quietly.

## 3. Code review
**Authors:** small PRs, a description of *why*, tests, self-review first. **Reviewers:** check correctness, edge cases, tests, readability, security, and blast radius; ask questions instead of issuing orders; separate *must-fix* from *nit*; respond within a day. SDE-2s are expected to review carefully and to receive feedback without defensiveness.

## 4. On-call and incident response
**Severity levels** (SEV1 = customer-impacting outage). During an incident: **mitigate first** (rollback, feature-flag off, scale), diagnose second; one incident commander; communicate status on a schedule; keep a timeline. Afterwards a **blameless postmortem**: **summary · impact · timeline · root cause (the "5 whys") and contributing factors · what went well / badly · action items with owners and dates**. Blameless means systems and processes failed, not a person. Runbooks and alerts tied to SLOs (Day 76's file) make on-call sustainable.

## 5. Agile in one page
**Scrum:** fixed-length **sprints**; ceremonies — **planning** (commit to scope), **daily stand-up** (sync, unblock), **review/demo**, **retrospective** (improve the process), plus backlog **refinement**. **Kanban:** a continuous flow with work-in-progress limits. Neither is a religion — know why each ceremony exists and what you would change.

## 6. What "SDE-2" signals
Owns features end-to-end (design → ship → operate); handles ambiguity by clarifying, proposing and getting alignment; reviews others' designs; mentors juniors; writes things down; measures impact; escalates risks early.

## 7. Exercise (≈ 30 min)
1. Write a one-page **design doc** outline for "add product search" (Day 131's file) with goals, non-goals, two alternatives, and a rollout plan.
2. Write a **postmortem** (any format above) for a fictional "Order service returned 500s for 20 minutes after a deploy" — include a 5-whys chain and three action items.
3. Map five bullets from your own experience to **STAR** stories that use these ideas: a design you proposed, an estimate that slipped, an incident you handled, a review disagreement, a process you improved.

## Checklist
- [ ] Can outline a design doc and an ADR and say why "alternatives" matters.
- [ ] Can explain how you estimate and communicate risk.
- [ ] Can describe incident response and a blameless postmortem.
- [ ] Can explain Scrum ceremonies and Kanban in one sentence each.
- [ ] Continue with Day 141's regular plan.
