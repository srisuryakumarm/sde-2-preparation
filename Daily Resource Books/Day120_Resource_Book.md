# Day 120 Resource Book — System Design Begins: the 5-Step Framework, Applied to a URL Shortener

**Series:** SDE-2 Interview Prep Resource Books · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 119](./Day119_Resource_Book.md) · **Next ▶:** [Day 121](./Day121_Resource_Book.md)
**Companion to:** Day 120 of `Week_18_Revised.md`

---

## Recap: what today is

Day 119 closed the LLD phase at ten complete systems and five mocks deep. Along the way, Day 118 showed you the *shape* of the System Design framework — five step names, no depth, deliberately — while building Food Delivery. Today is where that shape gets filled in for real, for the first time, against a genuinely new kind of problem: not "design a class hierarchy for one process," but "design a system that runs across many machines, serving requests you can't predict the volume of in advance."

This is a different interview format from everything in Weeks 16–17, and it's worth being explicit about that difference before going further, because both frameworks happen to have five steps and it would be easy to quietly conflate them:

| | LLD Framework (Day 106) | System Design / HLD Framework (today) |
|---|---|---|
| Steps | Clarify requirements → identify core objects → define relationships/class diagram → apply patterns deliberately → code the core | Requirements → Estimation → High-Level Design → Detailed Design → Bottlenecks |
| Question being answered | "How do I model this domain as classes, inside one process?" | "How do I get this system to actually run, correctly, at a given scale, across multiple machines?" |
| Output | Working, compilable code | An architecture — boxes, arrows, and the reasoning connecting them; code only for the one piece you go deep on |
| Failure mode it's built to catch | A design with tangled responsibilities or a missing pattern | A design that works for 100 users and falls over at 100 million |

Same discipline — a fixed sequence, said out loud, every time — aimed at a different axis of the problem. You'll use both, and today's project even leans on something from the LLD phase directly: the row-locking mechanism BookMyShow needed (Week 17, Day 117) reappears below, repurposed for a completely different problem.

## Learning Objectives

By the end of today, without notes:

1. State all 5 steps of the System Design framework, in order, and explain what each step is actually *for* — not just its name.
2. Perform back-of-envelope estimation live for a given set of inputs (traffic, storage), showing the arithmetic, not just citing a memorized final number.
3. Explain what Base62 encoding is, implement an encoder and decoder from scratch, and justify why 62 symbols specifically.
4. Explain, with a concrete mechanism, how a Key Generation Service avoids handing the same short code to two different requests when multiple instances run concurrently.
5. Justify the choice between a 301 and a 302 redirect for a URL shortener's redirect path — a small, frequently-asked detail with a real, non-obvious answer.

## Concept Dependency Map

```
Prerequisites, confirmed already covered:
  Sharding Strategies ................. Week 12, Day 78
  Replication Models ................... Week 9, Day 61
  SELECT ... FOR UPDATE row locking .... Week 17, Day 117 (BookMyShow)
  REST APIs / HTTP basics .............. Week 5, Day 34 · Week 10, Day 72
  Java: StringBuilder, char arithmetic,
        integer division & modulo ...... Week 1
        │
        ▼
NEW today: the System Design (HLD) 5-Step Framework
(a different framework from the LLD 5-Step Framework, Day 106 — contrasted above)
        │
        ├── Step 1: Requirements ................ functional vs. non-functional, scoped out loud
        ├── Step 2: Estimation ................... back-of-envelope math, done live
        │        │
        │        ▼
        ├── Step 3: High-Level Design ............ needs Base62 (below) + Sharding (cited)
        │        │
        │        ▼
        │   Base62 Encoding (NEW) — a positional numeral system
        │        │
        │        ▼
        ├── Step 4: Detailed Design .............. the Key Generation Service's concurrency problem
        │        │                                  needs: FOR UPDATE (cited) + SKIP LOCKED (NEW, one keyword)
        │        ▼
        └── Step 5: Bottlenecks .................. needs Sharding + Replication (cited);
                                                     forward-references Day 123 (full caching depth)
        │
        ▼
Applied end-to-end: URL Shortener
```

---

## Part 1 — The System Design Interview Is a Different Question

Every LLD system in Weeks 16–17 ran on one machine, in one process. `ParkingLot`, `BookMyShow`, `ATM` — however sophisticated the object model got, "how many machines is this running on" was never the question. Concurrency showed up (Day 111, Day 117), but as *threads inside one process* contending for shared state — a fundamentally different problem from *machines across a network* contending for shared state, which is what today begins.

System Design interviews test a specific, narrower thing than "can you architect software" in general: **can you reason about scale, explicitly, in a structured way, out loud, under time pressure.** That's why the framework exists — not as bureaucracy, but because the single most common failure mode in these interviews isn't a bad final design, it's an unstructured one: a candidate jumps straight to "we'll use a database and a cache" without ever stating what they're actually building for, and the interviewer has no way to tell whether that jump was informed judgment or a guess.

### The 5 Steps, Named

1. **Requirements** — what does this system actually need to do, and what's explicitly out of scope? Functional (features) and non-functional (scale, latency, consistency expectations).
2. **Estimation** — traffic, storage, bandwidth, done as visible arithmetic, not asserted.
3. **High-Level Design** — the boxes and arrows. The major components and how a request flows through them.
4. **Detailed Design** — pick the one or two pieces that matter most and go a level deeper. Not everything gets this treatment; the whole system in full depth doesn't fit in 45 minutes, and trying signals you can't prioritize.
5. **Bottlenecks & Scale** — where does this design break first as load grows, and what specifically fixes it?

> 🔑 **Key Takeaway:** every step is worth narrating even when it feels obvious. Saying "Step 1, requirements" out loud before listing them, every single time, is what turns this into a reflex instead of something that quietly gets skipped the moment time pressure hits — which is exactly the failure mode Day 122's first mock is designed to catch.

---

## Part 2 — Step 1: Requirements

**Functional requirements** — what the system does, from a user's perspective:
- Given a long URL, produce a short one.
- Given a short URL, redirect to the original long URL.
- *Optional, scope explicitly:* custom aliases (`bit.ly/my-brand`), click analytics.

**Non-functional requirements** — the qualities the system needs, which shape *how* you build it more than *what* you build:
- **Availability over strict consistency.** If the redirect service is briefly serving a URL that was created 2 seconds ago from a slightly stale replica, that's a non-event. If the redirect service is *down*, every shared link in the world breaks simultaneously. This is a real, concrete CAP-flavored decision (Week 9, Day 59) — for this system, lean AP, not CP — and it directly shapes the caching and replication choices in Step 5.
- **Low latency on the redirect path specifically.** A user clicking a shortened link is mid-action; every extra hundred milliseconds is felt directly. The *creation* path (shortening a URL) has much more slack — nobody times how fast the "shorten" button responds the way they'd notice a slow redirect.
- **The system must not hand out the same short code to two different long URLs.** This single non-functional requirement is what makes Step 4 non-trivial, and it's worth stating explicitly here rather than discovering it's a real constraint mid-design.

> 💡 **Interview Insight:** Stating "should this optimize for read latency or write latency?" out loud, and then answering your own question with a reason, is exactly the kind of visible judgment this step exists to surface. For a URL shortener specifically, reads dominate writes by a wide margin (formalized in Step 2 below) — that ratio is *the* fact that should shape almost every subsequent decision, so naming it here, not just computing it later, is worth doing.

---

## Part 3 — Step 2: Estimation

**Why this step exists, stated plainly:** the actual numeric answer matters far less than watching a candidate move from "100 million new URLs a day" to "roughly 1,200 writes a second" without a calculator, narrating each conversion. That fluency is what's being evaluated — getting within 20% of a "correct" figure is fine; freezing, or silently pulling a number from nowhere, is the actual failure mode.

**Given:** 100M new URLs created per day.

**Writes per second:**

$$\frac{100{,}000{,}000 \text{ URLs}}{86{,}400 \text{ seconds/day}} \approx 1{,}157 \text{ writes/sec}$$

(86,400 = 24 × 60 × 60 — worth having this number memorized cold, since re-deriving it live burns time better spent elsewhere.) Rounding up for a clean planning number: **~1,200 writes/sec.**

**Reads per second:** a URL shortener's whole value proposition is that a link gets clicked many times after being created once — reads outnumber writes heavily, commonly cited at 10:1 or higher for this exact system.

$$1{,}200 \times 10 \approx 12{,}000 \text{ reads/sec}$$

**Storage, worked fully:**
- Assume ~500 bytes per record (short code + long URL + metadata — a deliberately generous estimate, not a tight one; over-estimating storage is the safer direction to round in when the actual schema isn't finalized yet).
- Per day: $100{,}000{,}000 \times 500 \text{ bytes} = 50{,}000{,}000{,}000 \text{ bytes} = 50\text{GB/day}$
- Per year: $50\text{GB} \times 365 \approx 18.25\text{TB/year}$
- Over 5 years: $18.25\text{TB} \times 5 \approx 91.25\text{TB} \approx \textbf{90TB}$

**Worth checking, since it's a number Step 3 leans on directly:** over 5 years, total URLs created ≈ $100M \times 365 \times 5 = 182.5$ billion. Section 4 below shows the 7-character Base62 keyspace holds ≈3.52 trillion codes — so this system would use roughly $182.5B / 3.52T \approx 5.2\%$ of the available keyspace in 5 years. That's the arithmetic actually underneath the plan's "comfortably enough" — worth having ready if an interviewer asks *why* 7 characters specifically, rather than 6 or 8.

> ⚠️ **Common Mistake:** treating this step as a box to check quickly before "the real design." Interviewers frequently anchor Step 5 questions directly on Step 2's numbers ("you said 12,000 reads/sec — walk me through what happens to your design if that's actually 120,000") — skipping past estimation without genuinely internalizing the numbers leaves nothing to reason from later.

---

## Part 4 — Base62 Encoding (New Concept)

**Prerequisites, confirmed:** integer division and modulo, `StringBuilder`, and `char` arithmetic — all Week 1 material, fully assumed cold at this point in the series.

**What it is:** Base62 is a positional numeral system, exactly the same *kind* of thing decimal (base 10) or hexadecimal (base 16) is — just with 62 symbols instead of 10 or 16. The alphabet used here is `0–9`, `A–Z`, `a–z` (10 + 26 + 26 = 62 symbols), which gives every code two properties a URL shortener specifically wants: it's **URL-safe** (no characters needing escaping in a URL path) and it's **case-sensitive**, which is exactly what buys the compactness — a 6-character Base62 string can represent far more distinct values than a 6-character *decimal* string, because each position has 62 possible symbols instead of 10.

**The mechanism — encoding:** exactly the long-division-by-the-base process you'd use to convert decimal to any other base by hand, just automated:

```java
public class Base62Codec {
    private static final String ALPHABET =
        "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
    private static final int BASE = 62;   // ALPHABET.length() — asserted below, not just assumed

    public static String encode(long value) {
        if (value == 0) return String.valueOf(ALPHABET.charAt(0));
        StringBuilder sb = new StringBuilder();
        long n = value;
        while (n > 0) {
            int remainder = (int) (n % BASE);
            sb.append(ALPHABET.charAt(remainder));
            n /= BASE;
        }
        return sb.reverse().toString();   // digits were produced least-significant-first
    }

    public static long decode(String code) {
        long result = 0;
        for (int i = 0; i < code.length(); i++) {
            int digitValue = ALPHABET.indexOf(code.charAt(i));
            if (digitValue == -1) {
                throw new IllegalArgumentException("Invalid Base62 character: " + code.charAt(i));
            }
            result = result * BASE + digitValue;
        }
        return result;
    }
}
```

**Trace, to confirm correctness — encode(125):**

| Step | `n` | `n % 62` | char appended | `n / 62` |
|---|---|---|---|---|
| 1 | 125 | 1 | `'1'` | 2 |
| 2 | 2 | 2 | `'2'` | 0 (loop ends) |

`sb` built as `"12"`; reversed → **`"21"`**.

**Decode("21"), to confirm the round trip:**

| `i` | char | `indexOf` | `result = result × 62 + digitValue` |
|---|---|---|---|
| 0 | `'2'` | 2 | `0 × 62 + 2 = 2` |
| 1 | `'1'` | 1 | `2 × 62 + 1 = 125` |

`decode(encode(125))` → `125`. Correct.

**Why the reversal step is necessary, not cosmetic:** the divide-and-mod loop naturally produces digits from *least* significant to *most* significant (the same reason hand-converting decimal to binary produces bits backward), so the accumulated string is built in reverse order and must be flipped once at the end — forgetting the `.reverse()` call is a genuine, easy-to-make bug that still "looks like" a valid Base62 string, just the wrong one, which makes it a bad kind of wrong (silent, not a crash).

**Complexity:** encoding — `O(log₆₂ n)` iterations, since each iteration divides `n` by 62 (equivalently, output length is proportional to the number of base-62 digits needed); `O(1)` extra space beyond the output itself. Decoding — `O(k)` where `k` is the code's length, one constant-time `indexOf` per character (the alphabet is fixed at 62 characters, so this is a bounded linear scan, not a growing one).

**Edge cases:** `value = 0` (handled explicitly — the general loop's `while (n > 0)` would otherwise produce an empty string); negative values (not handled above — a KGS never encodes a negative value, so this is a deliberately scoped omission, worth stating out loud rather than silently ignoring); a `decode` call given a character outside the 62-symbol alphabet (throws, rather than silently producing a wrong number — validate untrusted input at the boundary, a habit worth carrying from the very first exception-handling material, Week 4).

> 💡 **Interview Insight:** a subtlety worth having ready — the *coding exercise* (encode/decode) and the *system design* (how the KGS actually uses Base62) are related but distinct questions. Encoding a simple auto-incrementing counter directly (`encode(1)`, `encode(2)`, `encode(3)`, …) is what the algorithm above naturally suggests, but it's a **real design mistake** if shipped as-is: sequential, predictable short codes let anyone enumerate every URL ever shortened by requesting `code`, `code+1`, `code+2`, … — a genuine information-leak / scraping risk, not a theoretical one. That's precisely *why* the plan's Key Generation Service pre-generates codes rather than encoding a counter live — Part 6 below covers exactly what it generates instead.

---

## Part 5 — Step 3: High-Level Design

With Requirements, Estimation, and Base62 in hand, the architecture:

```
                     ┌─────────────────┐
  Client ───write──▶ │   API Server    │
                     └────────┬────────┘
                              │ requests a code
                              ▼
                     ┌─────────────────┐        ┌──────────────────┐
                     │  Key Generation  │◀──────▶│  Pre-generated    │
                     │     Service      │        │  code pool (DB)   │  (Part 6)
                     └────────┬────────┘        └──────────────────┘
                              │ writes {code → longURL}
                              ▼
                     ┌─────────────────┐
                     │  URL Mapping DB  │  (sharded — Part 7)
                     └────────┬────────┘
                              ▲
                     ┌────────┴────────┐
  Client ───read────▶│  Cache (hot)    │  (light touch here; full depth Day 123)
   (redirect)        └─────────────────┘
```

- **Key Generation Service (KGS):** the component whose entire job is producing unique short codes *without* checking the mapping table for collisions on every single write — Part 6 explains exactly how.
- **A cache sits in front of the URL Mapping DB**, specifically because Step 2 established reads outnumber writes roughly 10:1 — the redirect path (the one users feel latency on directly, per Step 1) should almost never touch the database on a hot link. Full depth on *how* this cache is kept correct — the Cache-Aside pattern, Thundering Herd — is Day 123's job; today's design intentionally states *that* a cache belongs here and *why*, without re-deriving cache-invalidation mechanics that get their own full day shortly.
- **The URL Mapping DB is sharded**, addressed in Step 5.

---

## Part 6 — Step 4: Detailed Design — the KGS's Concurrency Problem

This is the one piece of today's design worth going a full level deeper on, per the framework's own instruction to pick what matters most rather than treating every box equally.

**The problem, stated precisely:** if the Key Generation Service runs as multiple instances (it has to, to handle ~1,200 writes/sec without becoming a single point of failure), what stops two instances from independently handing out the exact same short code to two different long URLs at the same moment?

**The design, building directly on what's already taught:** an offline batch process pre-generates a large pool of **random** 7-character Base62 strings (random — not sequential, exactly for the enumeration reason flagged in Part 4) and inserts them into a table:

```sql
CREATE TABLE available_codes (
    code VARCHAR(7) PRIMARY KEY
);
```

When a KGS instance needs to hand out a code, it must **atomically claim exactly one row that no other instance is simultaneously claiming.** This is the same category of problem BookMyShow's seat-booking solved with `SELECT ... FOR UPDATE` (Week 17, Day 117) — two concurrent transactions must not both believe they've claimed the same resource — with one genuine refinement needed here that BookMyShow's version didn't: BookMyShow's query locked one *specific* seat by ID; here, any available row will do, and blocking one instance until another finishes is wasted latency when a different, equally-valid row was sitting right there, unlocked.

```sql
BEGIN;
SELECT code FROM available_codes
  LIMIT 1
  FOR UPDATE SKIP LOCKED;   -- NEW: skip rows a concurrent transaction already has locked,
                            -- rather than waiting on them
DELETE FROM available_codes WHERE code = :claimed_code;
-- application layer now inserts {claimed_code -> longUrl} into the URL Mapping DB
COMMIT;
```

**What `SKIP LOCKED` adds, precisely, on top of the `FOR UPDATE` you already know:** plain `FOR UPDATE` (Day 117) makes a second transaction *wait* for a lock on the same row before proceeding — correct there, since BookMyShow's two concurrent bookings genuinely were contending for the identical seat, and one of them *should* fail or wait. Here, two KGS instances are **not** trying to claim the same row — any unclaimed row satisfies either request equally well — so making one instance wait on a lock held by the other wastes latency for no correctness benefit. `SKIP LOCKED` tells Postgres "if the first available row is already locked by someone else, skip it and give me the next one instead" — turning what would otherwise be unnecessary blocking into two instances proceeding in parallel, each getting a *different* row, with zero risk of a duplicate.

**The trade-off against the nearest alternative, stated explicitly:** a **static partition** scheme — instance 1 owns codes starting with `'0'`–`'F'`, instance 2 owns `'G'`–`'V'`, and so on — avoids touching a shared database row at all for each claim, which is faster per-claim. Its cost: rebalancing when an instance is added or removed requires re-partitioning the *unclaimed* pool, a genuinely harder coordination problem than the DB-atomic-claim approach, which scales instance count up or down with zero reconfiguration, since every instance just queries the same shared pool. For ~1,200 writes/sec total, the DB-atomic-claim approach's per-claim cost is not the bottleneck — the operational simplicity wins here; at a much higher write volume, that trade-off could flip, which is exactly the kind of "why not X instead" follow-up worth being ready to reason through live, not just recite an answer to.

> 🔑 **Key Takeaway:** Detailed Design isn't "explain everything in the diagram more" — it's picking the one place where a shallow answer would actually be wrong, and proving you can go deep on exactly that piece. Here, that's uniqueness under concurrency; the cache and the database schema, by contrast, are correctly left at high-level-design depth today, because nothing about them is non-obvious yet.

---

## Part 7 — Step 5: Bottlenecks & Scale

- **Redirect latency (the path users feel directly, per Step 1):** cache aggressively in front of the URL Mapping DB — a hot link should be served from cache on nearly every read. Full mechanics: Day 123.
- **Write throughput / single-node capacity:** once the URL Mapping DB exceeds one node's comfortable write capacity, **shard by short-code hash** — this is a direct, concrete application of Sharding Strategies (Week 12, Day 78), specifically the *hash-based* variant: since a lookup is always "give me the long URL for exactly this short code," there's no range query to preserve, so hash-based sharding's even-load property is a strict win here with none of its usual "expensive range queries" downside. Recall from Day 78 that naive `hash(code) % N` remaps almost every key when `N` changes — production hash-based sharding is virtually always **consistent-hash-based** in practice (Week 9, Day 60) for exactly that reason; nothing new to derive here, just applying two already-taught mechanisms to a concrete system.
- **Read availability:** **replicate** the URL Mapping DB (Week 9, Day 61) — given Step 1 explicitly chose availability over strict consistency for this system, asynchronous replication is the right default here specifically: a redirect served from a replica that's a few hundred milliseconds behind the leader is imperceptible to a user, and async replication buys better write latency and throughput than sync would, with no real cost given this system's own stated priorities.

---

## Part 8 — Bonus: 301 vs. 302 Redirects

A small, concrete detail that comes up often enough in this exact system to be worth having ready, and that the plan doesn't spell out but a tier-1 candidate is expected to know:

| | HTTP 301 (Permanent) | HTTP 302 (Found / Temporary) |
|---|---|---|
| Browser caches the redirect? | Yes — subsequent clicks go straight to the long URL, **never hitting your server again** | No — every click round-trips through your server |
| Effect on click analytics | Broken after the first click per browser — you lose visibility into repeat clicks | Accurate — every click is observable |
| Effect on server load | Lower, since the browser bypasses you after the first hit | Higher, but by design |

**The answer worth defending:** a URL shortener that offers analytics (Step 1's optional feature) should use **302**, deliberately — the whole point of tracking clicks is defeated if the browser silently stops asking after the first one. This is a case where the "obviously more efficient" choice (301, fewer round trips) is the *wrong* one once the actual requirement (accurate click counts) is stated — a good example of Step 1 constraining a decision made three steps later.

---

## Project Block

**Repository:** `java-fundamentals`. **Task:** the `Base62Codec` above (or your own equivalent). **Definition of done:** encodes and decodes correctly, round-trip, for a range of test integers — explicitly include `0`, a single-digit result, a value that needs the full 7 characters, and at least one value near `Long.MAX_VALUE` to confirm no overflow surprises.

## Career Block

**Blog Post 1 — "System Design: The 5-Step Framework, Applied to a URL Shortener."** A first technical blog post is a different exercise from a LinkedIn post — longer-form, more room to actually teach. A structure that covers what today built without needing to invent anything new:

1. Open with the framework's 5 names — this post's whole spine.
2. One paragraph per step, using today's URL shortener as the running example — Requirements' AP-over-CP call, Estimation's actual arithmetic (readers respond well to seeing real numbers, not just claims), the KGS's `SKIP LOCKED` mechanism as the "detailed design" centerpiece.
3. Close with the 301-vs-302 detail as a concrete, memorable takeaway — the kind of specific fact that makes a post feel like it taught something, not just summarized a framework.

Publish, then share to LinkedIn per the daily deliverable.

**Worth knowing as you start this phase:** Google's L4 loop specifically does not include a system design round — L4 engineers aren't expected to design systems there, so none of the next two weeks is tested at that level, specifically. That doesn't make this phase's depth wasted for a Google-track candidate: Stripe, Rippling, Databricks, Atlassian, and Uber all test system design directly, and how you describe your own project's architecture — the `scalable-ecommerce-platform` capstone itself — matters in *every* one of these conversations, Google included, when past work comes up.

## Daily Deliverable Check

- [ ] Can walk through the URL Shortener design end to end, naming each of the 5 steps explicitly, including the estimation math, without notes.
- [ ] Can state why 62 symbols and why 7 characters, with the supporting arithmetic (62⁷ ≈ 3.52 trillion; ≈5.2% of that keyspace used over 5 years at this traffic).
- [ ] `Base62Codec` encoder/decoder pushed, round-trip tested including 0 and a near-`Long.MAX_VALUE` case.
- [ ] Can explain how `SELECT ... FOR UPDATE SKIP LOCKED` prevents duplicate code handout, and how it differs from the plain `FOR UPDATE` already used for BookMyShow.
- [ ] Blog Post 1 published and shared to LinkedIn.

---

## Day 120 — Interview Questions

---

**1. Name the 5 steps of the System Design framework, in order, and state in one sentence what each is for.**

*Answer:* Requirements (what to build and what's out of scope), Estimation (traffic/storage math, done live), High-Level Design (the major components and how a request flows through them), Detailed Design (go one level deeper on the one piece that matters most), Bottlenecks & Scale (where does this break first, and what fixes it).

---

**2. How does this framework differ from the LLD 5-step framework from Day 106, beyond both happening to have 5 steps?**

*Answer:* The LLD framework answers "how do I model this as classes inside one process" and produces working code. The System Design framework answers "how does this system run correctly across multiple machines at a given scale" and produces an architecture — code only for whichever single piece gets the Detailed Design treatment.

---

**3. What is a System Design interviewer actually evaluating during the Estimation step — the final number, or something else?**

*Answer:* The process — fluently converting a daily figure into a per-second rate, narrating the arithmetic live. Landing within roughly 20% of a defensible number is fine; freezing or asserting a number without showing the derivation is the real failure mode.

---

**4. Walk through the writes/sec calculation for 100M new URLs/day.**

*Answer:* 100,000,000 ÷ 86,400 seconds/day ≈ 1,157 writes/sec, rounded to ~1,200 for planning purposes. 86,400 = 24 × 60 × 60.

---

**5. Why do reads outnumber writes for a URL shortener, and roughly by how much?**

*Answer:* Each URL is created once but clicked many times afterward — commonly modeled at a 10:1 read:write ratio or higher, giving roughly 12,000 reads/sec against ~1,200 writes/sec here. That ratio is why the design leans so heavily on caching the redirect path specifically.

---

**6. What is Base62 encoding, and why 62 symbols rather than 10 or 16?**

*Answer:* A positional numeral system using 62 symbols (0–9, A–Z, a–z) instead of decimal's 10 or hex's 16. More symbols per position means shorter strings represent the same range of values, and the specific 62-symbol alphabet is URL-safe and case-sensitive, both properties a short link wants.

---

**7. [Trace] Hand-encode the integer 125 in Base62 and show each step.**

*Answer:* 125 % 62 = 1, append `'1'`, n becomes 125/62 = 2. Then 2 % 62 = 2, append `'2'`, n becomes 0, loop ends. Built string `"12"`, reversed to `"21"`. Decoding `"21"` gives 2×62+1 = 125, confirming the round trip.

---

**8. Why does the reversal step in `encode()` matter — what breaks without it?**

*Answer:* The div/mod loop naturally produces digits least-significant-first (the same reason manual decimal-to-binary conversion reads backward). Skipping the reversal doesn't crash — it silently produces a different, wrong-but-valid-looking Base62 string, which is a worse kind of bug than one that throws.

---

**9. Why shouldn't the Key Generation Service just Base62-encode a simple auto-incrementing counter?**

*Answer:* Sequential codes are predictable — anyone can enumerate every shortened URL in the system by requesting `code`, `code+1`, `code+2`, and so on, a real information-leak and scraping risk. The KGS pre-generates *random* codes specifically to avoid this.

---

**10. How does `SELECT ... FOR UPDATE SKIP LOCKED` prevent two KGS instances from handing out the same code, and how does it differ from the plain `FOR UPDATE` used for BookMyShow (Week 17, Day 117)?**

*Answer:* It locks and returns one currently-unlocked row, skipping any row a concurrent transaction has already locked, rather than blocking on it. BookMyShow's `FOR UPDATE` needed two transactions contending for one *specific* seat to genuinely wait on each other; here, any unclaimed code works equally well for either instance, so `SKIP LOCKED` lets both proceed in parallel against different rows instead of one waiting unnecessarily.

---

**11. What's the trade-off between the DB-atomic-claim approach and statically partitioning the code pool by instance?**

*Answer:* Static partitioning avoids touching a shared database row per claim, which is faster per-operation, but rebalancing the unclaimed pool when an instance is added or removed is a genuinely harder coordination problem. DB-atomic-claim scales instance count up or down with zero reconfiguration, at the cost of a shared-row operation per claim — the right choice at this system's actual write volume (~1,200/sec).

---

**12. Why does a cache belong in front of the URL Mapping DB specifically for this system?**

*Answer:* Step 2 established reads outnumber writes roughly 10:1, and Step 1 established the redirect path is the latency users feel directly — caching hot redirects keeps the vast majority of read traffic off the database entirely.

---

**13. Why hash-based sharding rather than range-based for the URL Mapping table specifically (cite the relevant prior material)?**

*Answer:* Range-based sharding (Week 12, Day 78) is worth its cost only when range queries need to stay cheap — this system never runs a range query, only exact-match lookups by short code, so hash-based sharding's even-load property is a pure win with none of its usual downside. In production this is virtually always consistent-hash-based (Week 9, Day 60), specifically to avoid a near-total remap when the shard count changes.

---

**14. A URL shortener wants to offer click analytics. Should redirects use HTTP 301 or 302, and why?**

*Answer:* 302 — a 301 gets cached by the browser after the first click, so every subsequent click bypasses the server entirely and analytics silently stop counting them. 302 forces every click to round-trip through the server, which is required for accurate counts even though it costs more server load.

---

**15. At 100M new URLs/day for 5 years, roughly what fraction of the 7-character Base62 keyspace gets consumed?**

*Answer:* Total URLs over 5 years ≈ 100M × 365 × 5 = 182.5 billion. The keyspace is 62⁷ ≈ 3.52 trillion. 182.5B ÷ 3.52T ≈ 5.2% — comfortably within capacity, which is the concrete arithmetic behind choosing 7 characters rather than 6.

---

## What Tomorrow Assumes You Already Know Cold

Day 121 assumes the 5-step framework is now something you reach for by reflex, not something to re-derive — it narrates through all five steps again, faster, with less scaffolding. It also assumes Token Bucket (Week 10, Day 68) is solid enough to be recapped in one paragraph rather than re-taught, since tomorrow's real content is the *other three* rate-limiting algorithms and what changes once the rate limiter itself needs to run on more than one machine — the same multi-instance-coordination shape today's Key Generation Service just solved, applied to a different problem.

**Next:** [Day 121 Resource Book](./Day121_Resource_Book.md) — Rate Limiter, Formalized.
