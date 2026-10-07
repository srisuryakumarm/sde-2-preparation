# Gap Fill: Day 105B — Rippling and Walmart Global Tech Research

**Gap addressed (#8):** Google, Databricks, Stripe, Uber, and Atlassian each got bespoke round-format research woven directly into the LLD/HLD weeks — specific things real candidates got dinged for, specific round names, specific framing advice. Rippling and Walmart Global Tech got folded into the generic company list without that same treatment. This file closes that gap, researched fresh rather than assumed, and gives it before applications launch on Day 118.

**How this slots in:** a standalone research/reading file, placed alongside `Day_105A` (the first DSA mock, also inserted right after Day 105) — no fixed order between the two, do either first. **Deadline: complete before Day 118, when applications go out to all 7 target companies.** Where useful, a note is added on which existing mock (Week 16–19) is the best vehicle for rehearsing each company's specific lens.

*A caveat worth stating upfront: Rippling is a fast-growing, comparatively young company, and its interview process has visibly evolved even within candidate reports from the last year or two — treat the specifics below as the current best read, not a fixed target, and it's worth a quick re-check of recent Glassdoor/Blind posts closer to Day 118.*

---

## Rippling

### Process shape
Most candidate reports describe 4–5 stages: a recruiter screen, a technical screening round, a hiring-manager round, and a multi-part onsite/virtual-onsite loop (typically 3–4 further interviews). The full process is commonly reported to run 2–6 weeks depending on scheduling. One aggregator's breakdown of a large sample of recent question reports found **System Design as the single largest category** — ahead of Coding & Algorithms, Software Engineering Fundamentals, and Behavioral combined — which is a genuinely different weighting than a typical DSA-heavy loop and worth internalizing before assuming this is "just another LeetCode company."

### What the technical screen actually looks like
Reports describe it as hands-on and often product-oriented rather than a pure algorithm puzzle — building something practical under time pressure (a simple HTTP server, a REST API, an LRU cache built from scratch) rather than only solving an isolated LeetCode-style problem. One detailed guide describes a 90-minute session split into two parts. Expect follow-ups specifically on runtime, memory, and implementation trade-offs, not just "does it work."

### The hiring-manager round is a real gate, not a formality
Multiple sources flag this round — sitting between the technical screen and the onsite loop — as a point where candidates who clear the technical bar still get screened out, evaluated explicitly against Rippling's stated values: ownership, efficiency, communication, and customer-centricity. Treat this round with the same rehearsal seriousness as a technical round, not as easy small talk.

### A live, current wrinkle: AI-tooling policy varies by interviewer and round
This is genuinely inconsistent across recent reports — some candidates were explicitly told AI coding tools were **not** allowed and to ask before looking anything up; others describe an onsite round framed around live AI-assisted coding, and one report specifically advises narrating architectural reasoning continuously "even if you are utilizing AI coding tools." The honest read: don't assume either way going in — ask the recruiter directly what tooling is and isn't permitted for each specific round, since this is one of the areas where getting caught off guard costs the most.

### System design flavor
A reported real question: "design a hotel booking system" — asked by an engineering manager, notably close in shape to the Hotel Booking LLD already built in Week 17 (Day 119). Booking/reservation-style systems (their overlap with concurrency, inventory, and scheduling) show up repeatedly across the sample.

### Where to rehearse this in the existing plan
- Use **Mock Interview #4 or #5** (Week 17, BookMyShow or Hotel Booking) specifically through Rippling's lens: emphasize the system-design defense and trade-off discussion over pure correctness, since that's where Rippling's own weighting sits.
- Before the **hiring-manager-round-style** questions come up in a real loop, make sure at least one STAR story (Week 21) is explicitly rehearsed against "ownership, efficiency, communication, customer-centricity" as its own framing pass — similar to how Google/Databricks/Atlassian each got a dedicated framework-mapping exercise in Week 21, Rippling's four values deserve the same five-minute mapping check.

---

## Walmart Global Tech

### Process shape
A consistently reported pattern across many independent sources: an **Online Assessment first** (either a HackerRank/proprietary-platform test — commonly ~25 CS-fundamentals MCQs plus 2 LeetCode-medium DSA problems — or a **Karat**-administered assessment, both appear across reports), followed by **2–3 technical rounds**, then a **hiring-manager round**. India-specific reports (multiple from Bengaluru) describe the coding platform as **HirePro** in several cases; US-based reports more often mention Karat.

### What the technical rounds actually cover
Reports converge on a consistent mix within each ~1-hour round: **2 DSA problems at LeetCode-medium difficulty**, generally solved by writing directly into a plain notepad/screen-share with **no IDE support** (closer to Google's execution-off format than to Microsoft's execution-on one), plus a real, recurring pattern of **OOP concept questions and one live SQL query** (a second-highest-salary-style query shows up more than once in independent reports) tacked onto the end of a DSA round rather than as a separate round. Java-specific candidates specifically report being probed on Java concepts even after solid general DSA prep — worth treating Java fundamentals (already well covered in Weeks 1–3 of this plan) as genuinely testable material here, not background color.

### System design / HLD round
Shows up at the SDE2/Senior level specifically, framed around familiar retail/e-commerce-adjacent domains — one detailed recent report describes "design an e-commerce application similar to Amazon," broken into catalog/search, cart/checkout, order lifecycle with idempotency, and inventory consistency under scale — a close match to the platform's own Order/Payment/Product module boundaries already built in `scalable-ecommerce-platform`. The interviewer in that report was explicitly more interested in *how the candidate thought through the breakdown* than in exact API shapes — a "walk me through your reasoning" register, not a "produce the perfect answer" one.

### A practical, India-specific data point worth knowing
More than one recent report describes reaching a positive outcome on every technical round and then the process stalling or the offer not meeting expectations specifically on **compensation relative to current CTC** — not on interview performance. This is a real, reported failure mode distinct from the technical prep this whole plan builds, and it's exactly the reason `Day_146A_Offer_Negotiation_Practice.md` exists as a dedicated gap-fill in its own right — worth reading that file with Walmart specifically in mind, given how directly this pattern shows up in real reports.

### Where to rehearse this in the existing plan
- Walmart's DSA rounds reward speed and clean notepad-style coding without IDE support — the **Day 105A / Day 132A** DSA mocks in `Day_105_to_132_DSA_Coding_Mocks.md` already include a no-tooling notepad variant; when running whichever one lands closest to a real Walmart screen, default to that variant specifically.
- The "design an e-commerce system like Amazon" prompt is close enough to the platform's own domain that walking through `docs/architecture.md` (finalized Day 141) *as if it were a live answer to that exact prompt* is close to free, high-value rehearsal — the architecture is already built and understood, it just needs to be reframed as a live answer to a slightly more open-ended prompt than "explain your own project."
- Practice the SQL-query-as-a-follow-up pattern specifically: after solving a DSA problem in a mock session, have the accountability partner tack on one live SQL question (the Week 15 SQL track already has strong material for this) rather than only ever testing SQL in isolation.

---

## Daily Deliverable
- [ ] Rippling's system-design-weighted format and hiring-manager-round stakes internalized; one STAR story explicitly mapped against Rippling's four named values.
- [ ] Walmart Global Tech's OA→technical(DSA+OOP+SQL)→HLD→hiring-manager shape internalized; `docs/architecture.md` rehearsed once as a live answer to an open-ended "design an e-commerce system" prompt.
- [ ] Both companies' research complete before Day 118 applications launch.
