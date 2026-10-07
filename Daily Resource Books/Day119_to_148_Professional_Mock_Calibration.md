# Gap Fill: Day 119 to 148 — Professional Mock Interview Calibration

**Gap addressed (#4):** ten mocks with the same accountability partner build real reps, but a peer who also hasn't interviewed at these specific companies can't fully calibrate what "Strong Hire" actually looks like at Google or Databricks. This file is a reference guide, not an hour-by-hour schedule — the sessions it describes are self-booked around interviewer availability, not fully controllable the way the rest of the plan is.

**How this slots in:** three checkpoints, mapped to natural phase boundaries already in the plan. Nothing here replaces existing content; it's a parallel track the person books and runs independently, informed by what's below.

---

## Why peer feedback has a ceiling

The accountability partner is real, valuable, and should stay the primary volume driver — ten LLD/HLD mocks plus the four new DSA mocks in `Day_105_to_132_DSA_Coding_Mocks.md` don't happen without someone showing up consistently, and that reps-per-dollar ratio (free) is unbeatable. What a peer genuinely can't provide is calibration: has this specific answer actually cleared a real bar at a real company, or does it just sound reasonable to another candidate who's also preparing? That gap only closes with input from someone who's sat on the interviewer's side of the table.

One honest caution worth building into the plan itself: paid mock interviews are a *diagnostic* tool, not a *skill-building* one. Booking one before core fundamentals are solid mostly tests things that free practice would have surfaced anyway, for a much higher price per data point. That's the reasoning behind anchoring all three checkpoints to *after* a major phase has already closed, not scattered earlier.

---

## The three checkpoints

### Checkpoint 1 — Post-LLD (~Day 119–120)
LLD phase has just closed: 10 systems built, 5 mocks done with the peer partner, including one Machine Coding-format session. A professional session here should specifically target:
- One LLD system with a concurrency question attached (Parking Lot or BookMyShow are the strongest candidates, since both were built with a dedicated thread-safety phase).
- Explicit ask to the interviewer/coach: "push me on trade-offs the way a real interviewer would, don't just check if the design is correct" — since correctness is already well-covered by the peer mocks; what's missing is pressure-testing the *defense* of a design choice.

### Checkpoint 2 — Post-HLD (~Day 133–134)
HLD phase has just closed: 16 systems, 5 mocks with the peer partner. A professional session here should specifically target:
- A system-design round run in the "why not simpler" interrogation style — Uber's real loop is reported to reward exactly this kind of ruthless-clarity, anti-over-engineering pressure, and a professional interviewer is far better positioned to actually apply it convincingly than a peer who's learning the same material at the same time.
- If budget allows a second session at this checkpoint rather than one, split it: one HLD-style session, one on the capstone platform itself — walking a stranger through `scalable-ecommerce-platform`'s architecture cold is a different, harder test than walking the accountability partner through it, since the partner has already seen it evolve over months and can't react with genuinely fresh eyes.

### Checkpoint 3 — Pre-interview-execution (~Day 140–148, before or during Week 21)
Right before real interviews are likely to intensify. A professional session here should specifically target whichever of the target companies has the most distinctively hard, well-documented round:
- **Databricks-flavored:** a genuine from-scratch concurrency implementation (a thread-safe bounded queue, a rate limiter with real locking) — real candidates consistently describe this round as the hardest part of Databricks' loop, and it's real implementation, not a LeetCode pattern.
- **Uber-flavored:** think-aloud system design, justifying every choice as it's made rather than presenting a finished answer.
- **Atlassian-flavored:** a values-round rehearsal specifically — real candidates report technically strong people getting downleveled or rejected here for sounding arrogant or unprepared, and this round rewards the same rehearsal discipline as a technical round.

Pick whichever of these three feels least rehearsed after everything else in the plan, rather than defaulting to the one that would go smoothest.

---

## Platforms (researched current state, worth re-checking pricing directly before booking since it shifts)

**interviewing.io** — anonymous, live mock interviews matched with vetted senior/staff engineers who've conducted real interviews at companies like Google, Meta, and Amazon (the platform requires interviewers to have run at least 20 real interviews at a target-tier company before they're allowed to coach). Sessions are recorded with written feedback afterward. As of 2026, individual sessions run roughly $179–$339 depending on interviewer seniority and whether a specific target company is requested; a 3-session package runs close to $2,000. One free peer mock is included on signup. There's also a "Pay Later" option that defers payment until after landing a role, worth looking into if cash flow is a real constraint right now. Best fit: Checkpoints 2 and 3, where genuine FAANG-caliber calibration is the actual point.

**Exponent** (formerly Pramp) — runs **free**, scheduled peer-to-peer mock interviews multiple times daily (DSA, System Design, Behavioral, SQL slots each have dedicated daily times), reciprocal format where each person takes a turn as interviewer and interviewee. Also offers paid AI-graded mock sessions and paid 1:1 expert coaching, plus — notably — a dedicated **negotiation coaching package** (unlimited sessions against their own salary-data set), which is worth cross-referencing with `Day_146A_Offer_Negotiation_Practice.md` once a real offer exists. Best fit: as a genuinely free supplement to the peer-partner reps at any point, and specifically as a lower-cost alternative to interviewing.io if budget is the binding constraint at any of the three checkpoints — three free Exponent sessions plus one paid interviewing.io session is a reasonable way to stretch a limited budget across all three checkpoints rather than skipping the paid layer entirely.

---

## Getting real value out of each session

**Before:** state the specific thing being tested (see each checkpoint above) explicitly to the interviewer/coach at the start, rather than letting them default to a generic round — most platforms allow this kind of framing request.

**After — the 3-line delta log:** immediately after each session, before the specifics fade, write exactly three lines in a running note:
1. What the professional feedback actually said (verbatim where possible, not a paraphrase that's already softened it).
2. What was genuinely surprising — not what confirmed an existing suspicion, but what wouldn't have been guessed.
3. One specific, concrete action item — not "communicate better," but something checkable, like "state time/space complexity before being asked, every time" or "stop hedging the opening sentence of a system design answer with 'I think maybe.'"

Bring the accumulated delta log into the next peer mock with the accountability partner, and ask them to specifically watch for whether the flagged behaviors are still showing up. This is what turns three expensive, isolated data points into something that actually compounds.

## Budget note
Given the real cost, prioritize 2–3 genuinely well-targeted sessions over spreading a similar budget thin across many. The free peer and Exponent-peer volume should keep doing the heavy lifting; the paid layer's whole value is calibration against a real bar at the specific moments listed above, not additional reps.
