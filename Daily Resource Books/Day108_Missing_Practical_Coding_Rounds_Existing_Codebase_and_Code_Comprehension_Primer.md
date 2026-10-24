# Day 108 — Missing Primer: Practical Coding Rounds — Existing Codebases, Code Comprehension, Pairing, and Online Assessments

**Do this first on Day 108 (≈ 90 min, before LLD problems begin), then continue with Day 108's regular plan.**
**Needed by:** Days 113–114 and 119 (machine-coding practice and mock), Days 145–148 (final mocks), and any loop that replaces a classic algorithm round with a practical one.
**Why it is missing:** the plan trains DS&A (Days 1–105), LLD and machine coding (Days 106–119) and HLD (120–133). Nothing teaches the *other* round shapes: implementing a feature or fixing a bug **in code you did not write**, explaining unfamiliar code, pairing, or the mechanics of online assessments. The words "existing codebase" and "pair programming" appear nowhere in the books.
**Already in place:** debugging and stack traces (Day 30's file), `todo-api` with tests (Days 34–47), TDD (Day 107), machine-coding format (Days 113–114), SOLID and patterns (Days 100–107).
**Reality check:** formats vary by company and keep changing — in 2026 there are reports of large companies piloting code-comprehension and AI-assisted rounds *alongside* the classic algorithm rounds, not instead of them. Ask each recruiter exactly which rounds you will face.

---

## 1. The round shapes you may meet
| Shape | What you do | The plan covers it with |
|---|---|---|
| Algorithm round (LeetCode-style) | solve 1–2 problems, explain complexity | Days 1–105, mocks 105A–132A |
| Machine coding / LLD build | build a small working app in 60–120 min with clean OOP | Days 106–119 |
| **Feature in an existing codebase** | read a repo, add an endpoint/field/rule, keep tests green | **this file** |
| **Bug fix / code comprehension** | read unfamiliar code, explain it, find and fix a bug, improve it | **this file** |
| **Pair programming** | solve a task *with* the interviewer, think aloud, take hints | **this file** |
| **Online assessment / take-home** | timed platform problems or a 2–4 h project | **this file** |
| AI-assisted coding | solve with an assistant available | Day 140's file |

## 2. The loop for "implement this in an existing codebase" (≈ 60 min round)
1. **Clarify (3–5 min):** restate the task, acceptance criteria, what *not* to change. Ask how to build, run and test.
2. **Orient (8–10 min):** README → build file (`pom.xml`) → package layout → the **tests**. Run the test suite first to learn the baseline (what already fails).
3. **Trace the flow (10 min):** find the entry point (controller/handler), then follow request → service → repository. Use *Find Usages*, *Go to Declaration*, *Call Hierarchy*, and a breakpoint if needed (Day 30's file). Read just enough — do not read everything.
4. **Plan aloud (3 min):** "I'll add a field to the entity, a migration, the DTO and validation, the service rule, and tests." The smallest change that satisfies the criteria.
5. **Test first:** write or adjust a failing test (or reproduce the bug), then implement in small steps, running tests after each.
6. **Review your diff:** naming, edge cases, null/empty inputs, error handling, no unrelated refactors.
7. **Explain:** trade-offs, what you would do with more time (docs, more tests, performance).
**Style rules:** match the codebase's conventions; do not rewrite things you were not asked to; commit-sized steps; keep it running.

## 3. Code-comprehension rounds
You are shown a function or small module and asked: *what does it do? what is its complexity? where is the bug? what breaks under concurrency? how would you test or improve it?* Method: read the signature and callers first; trace one small example by hand; list invariants; look for off-by-ones, null handling, resource leaks, shared mutable state, wrong `equals/hashCode`, N+1 queries (Day 49's file); say your reasoning aloud, including doubts.

## 4. Pair programming
Think aloud; ask before assuming; **take hints gracefully** (they are the point); keep typing and testing — silence is the failure mode; name things well; say "I'd normally write a test here" and then do. The interviewer scores collaboration as much as the final code.

## 5. Online assessments and take-homes
- **Platforms** (HackerRank, CodeSignal, Codility, CoderPad): learn the editor *beforehand* — usually no IDE help; how input is read (a `Scanner`/`BufferedReader` template you have typed before); how tests report; whether the language runtime is the version you expect (Java 21?).
- **Time-box** each question; first make it correct on the samples, then optimize; partial credit usually counts; run your own edge cases (empty, one element, duplicates, max size).
- **Take-home:** read the brief twice; scope tightly; a clear `README` (how to run, design choices, trade-offs, what is missing); tests; a clean commit history; respect the stated time limit (2–4 h).

## 6. Practice protocol (use what you already built)
1. Treat `todo-api` as "someone else's code": after two weeks away, add *labels*, *a due-date filter*, *soft delete* — each with migration, DTO, validation, service rule and tests, timed at 45 minutes.
2. Have a friend (or an AI tool, with your Day 140 rules) **plant 3 bugs** in a copy; find them in 30 minutes using only the debugger, logs and tests.
3. Pick a small open-source Spring Boot sample (for example, Spring PetClinic): in 45 minutes, find where a given validation happens and add a new field end to end.
4. Do one timed OA on a platform using only its built-in editor.
Record yourself once; review the silences.

## Checklist
- [ ] Can recite the 7-step existing-codebase loop.
- [ ] Can say what you would read first in an unfamiliar repo and why you run the tests first.
- [ ] Can describe what a pair-programming interviewer looks for.
- [ ] Practice 1 and 2 completed once. Continue with Day 108's regular plan.
