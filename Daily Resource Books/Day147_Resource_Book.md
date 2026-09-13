# Day 147 (Weekend) — Final Portfolio Polish and Job Search Strategy

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 146 Resource Book](Day146_Resource_Book.md)
**Next ▶:** [Day 148 Resource Book](Day148_Resource_Book.md)
**Companion to:** Day 147 of `Week_21_Revised.md`

---

## Recap

Day 146 closed out the technical review cycle and finalized the resume, specifically resolving the DSA-count question in favor of the verifiable 249 (or the 197-assigned-plus-extra breakdown) rather than the plan's own unreconciled "213." Everything today builds on that decision directly — your GitHub profile's public-facing numbers need to match whatever the resume now says, not introduce a third figure.

---

## Learning Objectives

By the end of today, without notes:

1. Explain what a recruiter or interviewer is actually doing in the first minute on a GitHub profile, and design your pinned-repo layout around that behavior specifically, not around what feels personally most complete.
2. Write a portfolio-facing README that leads with impact and technical depth in the first few lines, not setup instructions.
3. Explain concretely why a referral changes an application's odds, and ask for one without it reading as presumptuous.
4. Build and use a simple status-tracking structure across all active applications, distinguishing "needs a follow-up" from "still genuinely in progress" from "needs a referral ask that hasn't happened yet."

---

## Concept Dependency Map

```
Day 146: resume numbers finalized (249 DSA, 10 LLD, 16 HLD)
        │
        └──▶ Part 1: GitHub profile — same numbers, same story, different surface

Day 144: todo-api's honest positioning (early practice, not over-claimed)
        │
        └──▶ Part 1: applied again across all five repos' README framing

        NEW: Job Search Strategy — referrals, follow-up cadence, status tracking
```

---

## Part 1 — GitHub Profile / Portfolio Presentation

### What Actually Happens in the First Sixty Seconds

A recruiter or hiring-manager skim of a GitHub profile is fast and shallow by default — the realistic behavior is: glance at the pinned repos (six slots, always visible without scrolling), read the first two or three lines of whichever README looks most relevant, and check whether the profile as a whole reads as "currently active and serious" or "abandoned side-project graveyard." Nothing below the pinned row reliably gets seen at all in that first pass. This is why the pinning decision matters more than almost any individual README's internal quality.

### Pinning Strategy and Order

Five repositories this series produced: `java-fundamentals`, `dsa-java`, `todo-api`, `lld-java`, `scalable-ecommerce-platform`. Per the plan's own instruction, the capstone leads. A reasoned order for the remaining four, built around telling a coherent story to someone skimming top to bottom rather than just listing chronologically:

1. **`scalable-ecommerce-platform`** — the flagship. Real Kubernetes/Helm deployment, chaos-tested, formally analyzed at scale. This is the repo that should make someone stop scrolling.
2. **`lld-java`** — system design range, ten systems, a genuine difficulty ramp. Immediately reinforces that the capstone's architecture wasn't a one-off.
3. **`dsa-java`** — algorithmic depth. For an SDE-2 audience specifically, showing this third (after the systems work, not first) tells a "I can design *and* I can grind the fundamentals" story rather than leading with "I did a lot of LeetCode," which undersells everything else.
4. **`java-fundamentals`** — foundational, but genuinely useful as evidence of how deep the "from zero" rigor went (JVM internals, memory model, concurrency primitives built by hand).
5. **`todo-api`** — the earliest practice project, exactly as Day 144 framed it: an honest, deliberately-simple snapshot of where this all started, which is what makes the first four repos' progression legible as growth rather than a flat pile of five similar projects.

**⚠️ Common Mistake:** pinning in strict chronological order (oldest first). That buries the strongest work at the bottom of a row someone's already skimming fast, and it's an easy, low-cost fix — pin order has nothing to do with when a repo was built.

### README Anatomy for a Portfolio-Facing Repo

**Lead with impact, not setup.** The first two or three lines should answer "what is this and why should I care," not "here's how to install dependencies." A skimming reader who has to scroll past a setup guide to find out what a project actually does has usually already moved on.

**A workable shape:**
1. One-sentence description of what it does.
2. Two or three bullets on the most technically interesting decisions (not a full feature list) — the kind of thing that would make a specific interview question.
3. Tech stack, accurately stated.
4. Setup instructions — useful, but positioned after the parts that actually sell the project.

**For `todo-api` specifically**, continuing Day 144's framing directly: this README should say plainly that it's early practice, Docker/Compose only — not because it needs to apologize for being simple, but because an honest "here's where I started" framing, sitting fourth in a five-repo progression that ends with a chaos-tested Kubernetes deployment, tells a stronger growth story than pretending every repo is equally polished.

**💡 Interview Insight:** if an interviewer opens your GitHub before a call (increasingly common), a portfolio that's honest about which projects are early practice and which are the real showcase reads as more credible, not less — an interviewer who clicks into `todo-api` expecting the same sophistication as the capstone and finds a mismatch with an unearned README is a worse outcome than one who finds exactly what an accurate README told them to expect.

### Profile README

The top-level profile README (the one that shows above your pinned repos) is worth a short, current pass too: update anything that still says "currently building" to "built," and make sure it points at the capstone and at your current job-search status if you're comfortable stating it publicly (many candidates find "open to SDE-2 opportunities" phrasing useful here specifically because recruiters do search profile READMEs for exactly that kind of signal).

---

## Part 2 — Job Search Strategy: Referrals vs. Cold Applications

### Why a Referral Actually Changes the Odds

Mechanically, not just as received wisdom: most companies at this scale route applications through an ATS (applicant tracking system) that filters on keyword matching before a human ever sees a resume, and a referral very often either bypasses that initial filter entirely or routes the application directly to a recruiter's or hiring manager's queue instead of the general pool. Beyond the routing mechanism, a referral is also a small, real signal of vouching — someone already inside the company is willing to put a small amount of their own credibility behind the introduction, which a cold application has no equivalent of.

### Asking for a Referral Without It Being Awkward

The version that works: specific, low-pressure, and easy to say no to. *"I'm applying for [specific role] at [company] — I saw you're on the [team], would you be open to referring me if it seems like a fit after you take a look at my resume?"* is concrete and gives the other person an easy out (they can look and decide it's not a fit without an awkward conversation). *"Can you refer me?"* with no specifics attached puts the other person in the position of vouching blind, which understandably makes people hesitate.

**⚠️ Common Mistake:** asking a first-degree connection you haven't spoken to in years, cold, with no context re-established first — a short "hey, it's been a while, hope you're doing well" genuinely matters before the ask, not as a formality but because it's the difference between a warm request and one that reads as purely transactional.

### Follow-Up Cadence

A reasonable default: one follow-up after roughly one to two weeks of silence following an application or a referral request, polite and brief, then generally let it rest unless there's a genuine new reason to reach out again (a relevant update, a new posting at the same company). Repeated follow-ups with no new information tend to read as pressure rather than diligence.

### A Status-Tracking Structure

Since applications have been live since Day 118 across all seven target companies (Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech), today's review needs a structure to actually be useful rather than just a mental scan. A simple table, one row per company, is enough — the columns matter more than the tool:

| Company | Applied | Referral asked? | Last contact | Next action |
|---|---|---|---|---|
| *(fill in per company)* | *(date)* | *(yes/no/pending)* | *(date + what happened)* | *(specific next step, with a date)* |

Work through your actual seven companies against this structure using whatever you've already been tracking (the plan references a `Target_Company_Research_and_Interview_Guide.md` for exactly this purpose) — the useful output of today's session isn't the table itself, it's identifying, per company, which of three buckets it's actually in: **genuinely still in progress** (no action needed, just wait), **needs a follow-up** (silence past the cadence above), or **needs a referral ask that hasn't happened yet** (the highest-leverage gap to close, per the mechanism above).

**🔑 Key Takeaway:** the single highest-value output of this session is usually the shortest row in the table — whichever company has *no* referral in motion at all. That's the one action today most likely to change an outcome, more than another resume tweak or another cold application elsewhere.

---

## Project Block Guide (2 hrs)

**Task:** archive and pin all five repositories on your GitHub profile in the order reasoned through above, and update the profile README.

**Definition of done:** all five repos pinned in order, each README passes the "first three lines sell it" test, profile README current and accurate, nothing still says "currently building" that's actually finished.

## Career Block Guide (2 hrs)

**Job search strategy session.** Work through the status-tracking structure above against your real seven companies, using `Target_Company_Research_and_Interview_Guide.md` and whatever records you've kept since Day 118. Identify, concretely: where a referral hasn't been asked for yet, and where a follow-up is overdue per the cadence above. Then act on at least the referral gaps today rather than just noting them — a status review that doesn't produce an actual message sent is a review, not a strategy session.

**Thank your accountability partner.** Specific, not generic — name an actual thing they did this week (covered a mock interview, gave real feedback on a story) rather than a general thanks.

---

## Day 147 — Interview Questions

**Q1. Why does pin order matter more than individual README quality for a first impression?** A skimming recruiter typically only sees the six pinned repos without scrolling and reads a few lines of whichever looks most relevant — pin order controls what's even seen, before README quality has a chance to matter.

**Q2. What should the first few lines of a portfolio README lead with, and why?** Impact and the most technically interesting decisions, not setup instructions — a skimming reader who has to scroll past installation steps to find out what the project does has usually already moved on.

**Q3. Mechanically, why does a referral change an application's odds?** It often bypasses the ATS's initial keyword-filtering step or routes directly to a recruiter/hiring-manager queue instead of the general pool, and it carries a small, real signal of a current employee vouching for the candidate.

**Q4. What's a reasonable follow-up cadence after an application or referral request goes unanswered?** One polite follow-up after roughly one to two weeks of silence, then generally letting it rest unless there's a genuinely new reason to reach out again.

**Q5. Of the three status buckets (in progress, needs follow-up, needs a referral ask), which one is usually the highest-leverage to act on, and why?** The referral-ask gap — it's the one lever most directly shown to change an application's actual odds, versus a follow-up (which mostly just re-surfaces an already-submitted application) or waiting on something already in progress.

---

## Daily Deliverable Check

- [ ] All five repositories archived/pinned in the reasoned order above; profile README updated and accurate.
- [ ] Every pinned repo's README passes the "first three lines sell it" test — impact first, setup instructions after.
- [ ] Application/referral status reviewed across all seven target companies using the tracking structure; every identified referral gap acted on today, not just noted.
- [ ] Accountability partner thanked specifically.

---

## What Tomorrow Assumes You Already Know Cold

Day 148 assumes your portfolio and your resume now tell the same, consistent story — the same DSA count, the same system counts, the same framing of `todo-api` as early practice — since tomorrow's closing self-check and scorecard treat all of this as settled, finished work to summarize, not something still being reconciled.
