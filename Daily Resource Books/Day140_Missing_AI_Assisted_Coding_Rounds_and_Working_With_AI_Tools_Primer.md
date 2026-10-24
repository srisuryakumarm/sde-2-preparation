# Day 140 — Missing Primer: AI-Assisted Coding Rounds, and Working With AI Tools Without Losing the Fundamentals

**Do this first on Day 140 (≈ 60 min), then continue with Day 140's regular plan.**
**Needed by:** Days 145–148 (final mocks), any interview where an assistant is allowed — and your own daily practice from today (how you use AI while studying decides whether you learn).
**Why it is missing:** no book or primer discusses interviews or study workflows that involve AI tools. In 2026 there are reports that some large companies pilot rounds where an assistant is available, while others ban AI entirely, and policies change often — so this is a skill *and* a compliance topic.
**Already in place:** debugging (Day 30's file), practical coding rounds (Day 108's file), testing (Days 45–47), the Boot 3→4 alignment table (Day 34's file), your own repos.

---

## 1. First, the rules (compliance beats cleverness)
- **Ask the recruiter in writing** which rounds allow AI, which tools or platform assistant is provided, and what must be on screen. Policies differ per company and per round, and can contradict each other.
- **Where AI is banned, use none** — including a second device or a hidden tool. A rescinded offer is the cost.
- Where it is allowed, **you remain accountable** for every line you submit.

## 2. What an AI-allowed round actually measures
Not typing speed. Interviewers watch whether you can **frame the problem, decompose it, direct the tool precisely, read and verify its output, debug, test, and decide what to accept or reject** — and explain why. Weak fundamentals show up fast: you cannot spot a wrong complexity, a missed edge case, a hallucinated method, or an outdated API.

## 3. A workflow that scores well
1. **Restate** the task and constraints aloud; propose your *own* short plan first.
2. **Delegate narrowly:** scaffolding, boilerplate, test cases, unfamiliar-API lookups. Give context, constraints, examples, and ask for tests.
3. **Read every line** it produces; predict what it does before running it.
4. **Run it** — happy path, then edge cases (empty, one element, duplicates, large, concurrent).
5. **Fix and own it:** rewrite what you cannot explain; keep what you can.
6. **Narrate:** what you asked, what was wrong, why you changed it, trade-offs.
Never wait silently for the tool; never paste code you cannot defend.

## 4. Typical failure modes (and one specific to this plan)
Plausible-but-wrong logic; an off-by-one that passes the sample; missing validation and error handling; insecure defaults; invented library methods; **outdated versions** — assistants often emit Spring Boot 3 names (`spring-boot-starter-web`, `@MockBean`, `resilience4j-spring-boot3`, `Jwts.parserBuilder()`) that fail on Boot 4.1. Check any suggestion against the alignment table in Day 34's file.

## 5. Using AI to learn from the basics (your daily practice)
- **Attempt first** for 15–20 minutes. Ask for a **hint**, not the solution; then ask for the next hint.
- **Explain it back** in your own words (aloud). If you can't, you haven't learned it.
- Ask the tool to **quiz you**, generate **edge-case tests**, or **review your solution** — and compare with your own reasoning.
- Keep a **mistakes log**: bugs *you* made and bugs *the AI* made.
- Alternate: **AI-off sessions** (fundamentals; many rounds ban AI) and **AI-on sessions** (practical tasks).

## 6. Practice protocol
Weekly, one 45-minute **AI-allowed** session: take a feature task in `todo-api` (for example, a `dueSoon` filter with migration, DTO, validation and tests); use the assistant; review; **list at least three issues in its output** and write the tests that would catch them; record yourself and check how much of the time you were silent or just accepting. Then redo a smaller task with **AI off**.

## 7. Exercise (≈ 30 min)
Do the `dueSoon` task above. Deliverable: the diff, your tests, and a half-page note: *what I asked, what was wrong, how I verified, what I'd do differently*.

## Checklist
- [ ] Can state the compliance rule and what to ask a recruiter about AI use.
- [ ] Can describe what interviewers score in an AI-allowed round.
- [ ] Can list the six-step workflow and three typical AI failure modes.
- [ ] Can describe your own AI-off/AI-on study routine. Continue with Day 140's regular plan.
