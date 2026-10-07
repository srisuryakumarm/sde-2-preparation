# Gap Fill: Day 105 to 132 — DSA Coding Mocks (Live, Narrated, Timed)

**Gap addressed (#1 — highest priority):** the 148-day plan runs 5 LLD mocks and 5 HLD mocks, but zero mocks where the format is "here are two LeetCode-style problems, solve them out loud, live, in 45 minutes." Solving a problem alone and solving it while narrating to another person under time pressure are different skills, and right now only LLD/HLD get the second one. This file adds four sessions.

**How this slots in:** four standalone bonus sessions, inserted between existing days, not replacing any existing content. Each is labeled with a letter suffix (105A, 113A, 123A, 132A) to mark it as an insertion — Week_XX_Revised.md files are untouched. Each session runs ~60–75 minutes total (45 min mock + ~15–20 min debrief) and can be scheduled on the evening of, or the day after, the anchor day it's attached to, whichever fits the week better.

---

## The Format (same shell for all four sessions)

This isn't a new pattern to learn — every problem in these four sessions is already-covered material. What's new is the conditions: live, narrated, timed, and judged by someone else.

**Structure (45 min):**
1. **0:00–0:05 — Cold open.** Partner reads the prompt exactly as written below, cold, with no framing. No "this is a sliding window problem." A real phone screen doesn't tell you the pattern.
2. **0:05–0:10 — Clarify.** Ask clarifying questions out loud before writing anything. This is where the Problem_Comprehension_Drill's restate-and-formalize habit gets tested under pressure — not in a vacuum with unlimited time, but with a partner watching the clock.
3. **0:10–0:35 — Solve, narrating continuously.** State the approach before coding it. If a brute force is the honest starting point, say so and say why you're improving it, rather than jumping straight to the optimal solution as if it arrived by magic.
4. **0:35–0:40 — Test it.** Walk through a self-generated example by hand. State time and space complexity unprompted — don't wait to be asked.
5. **0:40–0:45 — Partner questions.** Partner asks one deliberate follow-up: a variant of the problem, an edge case, or "how would this change if the input were a stream instead of an array?"

**Format variants to rotate through** (real loops don't all look the same, and practicing only one format is itself a gap):
- **Execution-off, shared doc (Google-style):** partner opens a blank Google Doc, no syntax highlighting, no autocomplete, code cannot be run. This is still Google's real phone-screen format as of 2026 — one or two problems, 45–60 minutes, calibrated around LeetCode Medium. A newer wrinkle worth knowing about: Google has been piloting a "code comprehension" round for some junior/mid-level US roles in the second half of 2026, which replaces one coding round with reading and explaining an unfamiliar code excerpt rather than writing new code — not yet confirmed for all locations or levels, but worth a mention to the accountability partner so it's not a total surprise if it shows up.
- **Execution-on, real IDE/CoderPad (Microsoft-style):** partner shares a CoderPad-style link or just watches a shared IDE window; code can be run and debugged live. Microsoft's loop for SDE2 typically opens with a timed online assessment (historically Codility, ~70 minutes, 2 problems with hidden test cases), then 3–4 separate ~1-hour live coding rounds where execution is allowed — meaning bugs get caught by running the code, not just by careful narration, which is a genuinely different skill to rehearse.
- **Whiteboard/notepad, no editor at all:** partner has you type directly into a plain text editor or literally describes the problem verbally with you writing on paper held up to camera. Several Walmart Global Tech candidate reports describe coding directly into a plain notepad on screen share, with no IDE support at all — closer to Google's format than Microsoft's, but worth its own rep since notepad-with-zero-tooling is its own adjustment.

Assign one variant per session below; don't let all four default to the easiest one (execution-on).

---

## Session 1 (Day 105A) — Baseline

**Placement:** right after the DSA curriculum closes (Day 105) and before LLD begins (Day 106). The point of running this immediately, before any LLD content has a chance to dilute DSA sharpness, is to get an honest baseline reading of the *narration* skill specifically — not the DSA skill, which is already solid at 213 problems deep.

**Format variant:** execution-off, shared doc.

**Problems** (foundational patterns, Week 1–2 material — deliberately easy on the DSA itself, so any struggle is diagnosably about the *format*, not the algorithm):
- Group Anagrams (LeetCode #49, Medium) — HashMap-keyed pattern.
- Container With Most Water (LeetCode #11, Medium) — Two Pointers.

**What "good" looks like at this checkpoint:** clarifies before coding, narrates the brute force before jumping to the optimization, doesn't go silent for more than ~20–30 seconds at a stretch while thinking. It's fine — expected, even — if the code has a small syntax slip in a no-execution doc; what matters is catching it while reading the code back aloud, the way the format actually tests.

**Debrief questions for the partner to ask:**
- Did clarifying questions happen before or after code started?
- Was there a point where narration stopped and it became silent heads-down coding?
- Could you (the partner) have followed the logic without seeing the screen, from the narration alone?

---

## Session 2 (Day 113A) — Mid-LLD Checkpoint

**Placement:** after Day 113 (ATM Machine LLD) and before Day 114 (Elevator + LLD Mock #3). Roughly a week after Session 1, deliberately positioned so DSA muscle doesn't go quiet for the first full week of LLD-only focus.

**Format variant:** execution-on, real IDE/CoderPad.

**Problems** (mid-curriculum patterns — Trees, Graphs):
- Binary Tree Right Side View (LeetCode #199, Medium)
- Course Schedule (LeetCode #207, Medium)

**Why these two together:** neither is hard in isolation, but Course Schedule specifically tests whether "graph problem" correctly resolves to "topological sort / cycle detection" rather than a more generic DFS-and-hope — exactly the pattern-recognition-under-ambiguity muscle worth checking on a live clock, not just in solo practice.

**Debrief questions:**
- On Course Schedule, was the first instinct correct, or was there a false start down a different approach before landing on the right one? A false start under time pressure is normal — what matters is how fast it got corrected.
- Rate narration quality 1–5 versus Session 1. If it's not visibly better, that's worth a second look before Session 3, not a reason to assume it's fine on its own.

---

## Session 3 (Day 123A) — Mid-HLD Checkpoint

**Placement:** after Day 123 (Distributed Cache) and before Day 124 (Distributed ID Generation). This sits roughly where real recruiter responses to the Day 118 applications are likely starting to land, per the plan's own Day 122 note — so this session doubles as a readiness check right as phone screens could plausibly start getting scheduled for real.

**Format variant:** whiteboard/notepad, zero tooling.

**Problems** (DP + Heaps — harder tier, deliberately):
- Coin Change (LeetCode #322, Medium)
- Kth Largest Element in an Array (LeetCode #215, Medium)

**What to push on:** with zero editor support, small mistakes (an off-by-one, a missing base case) are much easier to make and much harder to spot without reading the code back line by line. This session is as much about the discipline of a deliberate "read it back before declaring done" pass as it is about the algorithms themselves.

**Debrief questions:**
- Count actual syntax/logic slips. Were they caught by the candidate, or pointed out by the partner?
- Did the DP recurrence get stated in one precise sentence before code started (`dp[i] = ...`), matching the habit built during the DP block itself — or did coding start before the recurrence was actually pinned down?

---

## Session 4 (Day 132A) — Final Pre-Behavioral Checkpoint

**Placement:** after Day 132 (Leaderboard + Search Autocomplete) and before Day 133 (Google Drive HLD + HLD Mock #5 + HLD phase close). This is deliberately the last DSA-specific check-in before Week 20 (zero DSA content) and Week 21 (behavioral-focused, aside from Day 146's brief cold spot-check) — the last chance to catch a slipping skill before a long quiet stretch.

**Format variant:** candidate's choice — pick whichever of the three formats felt weakest across Sessions 1–3, and drill that one again rather than defaulting to the most comfortable one.

**Problems** (broadest pattern mix, hardest tier of the four sessions):
- Course Schedule II (LeetCode #210, Medium) — Topological Sort, returning actual order, not just feasibility.
- Longest Palindromic Substring (LeetCode #5, Medium) — could plausibly be mistaken for a pure two-pointer problem at first glance; recognizing the Expand-Around-Center shape correctly, fast, is the actual test.

**Debrief questions:**
- Across all four sessions, has the time-to-first-correct-approach gotten shorter?
- Is there a specific pattern family that's underperformed across more than one session? If so, that's worth a solo cold-review pass before Week 20 swallows the calendar with zero DSA content.

---

## A note on rubric consistency

Across all four sessions, have the partner score the same four dimensions every time, on a simple 1–5 scale, so the four scores are actually comparable to each other:

1. **Comprehension** — restated the problem correctly before coding, asked the clarifying question(s) that mattered.
2. **Pattern recognition** — landed on a reasonable approach without needing more than one false start.
3. **Communication** — narrated continuously; a stranger listening without the screen could follow the logic.
4. **Correctness under the format's constraints** — final code is right, or gets there via a self-caught correction, appropriate to whichever format variant was used that session.

Four data points isn't a lot, but a visible trend (or a stalled one) across Sessions 1 → 4 is worth more than any single session's score in isolation.
