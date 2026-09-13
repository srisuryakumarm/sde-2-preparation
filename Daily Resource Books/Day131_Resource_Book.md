# Day 131 — HLD #13: Distributed Job Scheduler, and HLD Mock #4

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 130 Resource Book](Day130_Resource_Book.md)
**Next ▶:** [Day 132 Resource Book](Day132_Resource_Book.md)
**Companion to:** Day 131 of `Week_19_Revised.md`

---

## Recap

Yesterday closed the loop on idempotency and distributed locks, naming — for the first time in full — the general mechanism behind four earlier informal appearances. Today's system needs both of yesterday's mechanisms again, one layer down the stack: a job scheduler's core guarantee, **at-least-once execution**, is the exact same "might run twice under failure" shape Day 130 solved for payments, now applied to background jobs instead of API requests. Today also runs a mock that reuses yesterday's material directly — HLD Mock #4, subject Payment System, deliberately spending the full time pushing into race conditions rather than stopping at a first correct answer.

---

## Learning Objectives

By the end of today, without notes:

1. Explain SQS Visibility Timeout as a TTL-based lease, and name precisely which of yesterday's mechanisms it's structurally identical to.
2. State why "at-least-once" is the realistic guarantee distributed schedulers offer, and why "effectively-once" — not true exactly-once — is what idempotent handling actually buys you.
3. Explain Quartz's misfire-handling problem and name at least two different, situationally-correct policies for it.
4. Defend a payment design under sustained "what if these two things race" pressure, going past a first correct-sounding answer to a genuinely complete one.

---

## Concept Dependency Map

```
Day 121: Redis + Lua atomicity (the pattern reused a fifth time today)
Day 130: Idempotency keys; Distributed locks (TTL-based leases); the double-entry
         ledger's self-checking property
        │
        ▼
Today — Distributed Job Scheduler
  ├─ SQS Visibility Timeout (NEW — a TTL lease, same shape as Day 130's lock)
  ├─ Redis Sorted Set as a delayed queue (APPLIED — score = execution time;
  │     needs: Sorted Sets' O(log n) contract, named Day 123, mechanism
  │     still deferred to tomorrow, Day 132)
  ├─ Quartz Scheduler: cron + misfire handling (NEW)
  └─▶ At-least-once vs. exactly-once execution (NEW — generalizes Day 130's
        "at-least-once delivery + idempotent processing" directly)

  Separately, today: HLD Mock #4 — Payment System (Day 130), race-condition
  pressure format; and a DSA cold-revision check (Union-Find, DP)
```

---

## Part 1 — Requirements (Step 1)

A scheduler must run a job at (or near) its intended time, even across worker crashes, deployments, and restarts — "the schedule said 2am" has to survive the scheduler itself occasionally being unavailable at 2am. **Non-functional:** jobs must not be silently dropped (a missed run is a real, visible problem); occasionally running a job **more than once** under failure is an acceptable trade, **provided job handlers are written to tolerate it** — a requirement that shapes everything below.

---

## Part 2 — Delayed Message Queues

### SQS Visibility Timeout

**Definition:** when a consumer receives a message from the queue, it doesn't disappear immediately — it becomes **invisible** to other consumers for a configured window (the visibility timeout). If the consumer finishes and explicitly deletes the message within that window, it's gone permanently. If it doesn't — a crash, a timeout, anything — the message becomes visible again once the window elapses, and some consumer (the same one or a different one) will pick it up and try again.

> 🔑 **Key Takeaway:** mechanically, this is **yesterday's TTL-based distributed lock**, applied to a queue message instead of a payment order. A visibility timeout *is* a lease with an expiry — claim it, do the work, release it (delete) before the lease runs out, or someone else reclaims it.

**Why this is what makes "at-least-once" true rather than "at-most-once" by accident:** a naive queue that deletes a message the moment it's *received* (not when it's *completed*) would silently lose that message forever if the consumer crashes mid-processing. Visibility timeout exists specifically to prevent that — the message is never removed from the system of record until the work is actually confirmed done.

> ⚠️ **Common Mistake:** setting the visibility timeout shorter than realistic processing time. If a job genuinely takes 90 seconds but the timeout is 30, the message becomes visible again — and gets picked up by a second worker — **while the first worker is still legitimately working on it.** This is a duplicate-processing bug caused purely by a misconfigured timeout, not a flaw in the mechanism, and it's exactly why idempotent handling (below) isn't optional hardening — it's load-bearing.

### Redis Sorted Set as a Delayed Queue

**Mechanism:** store each pending job as a member of a Sorted Set, scored by its intended execution timestamp. A poller repeatedly asks "give me everything due right now" (`ZRANGEBYSCORE key -inf now`) and atomically claims (removes) whatever comes back.

> 💡 **Interview Insight:** SQS gives visibility-timeout-style leasing as a **built-in queue feature**. A hand-rolled Redis Sorted Set gives you none of that automatically — claiming a job here means *you* build the same leasing discipline yourself, out of Day 130's own SETNX-with-TTL pattern, one layer down. Naming this contrast unprompted — managed primitive vs. hand-assembled equivalent — is a genuinely strong thing to say in an interview, not just a fact to know.

> 🔗 **Forward reference:** `ZRANGEBYSCORE`'s O(log n + m) cost rests on the Skip List backing Sorted Sets — named on Day 123, used operationally on Day 128, used again here, and finally explained mechanically tomorrow, Day 132.

---

## Part 3 — Quartz Scheduler: Cron and Misfire Handling

### Cron, precisely (worth being exact about, not approximate)

Quartz's cron format uses **six fields**, seconds first — `second minute hour day-of-month month day-of-week` — and requires `?` (meaning "no specific value") in exactly one of day-of-month or day-of-week, since specifying both simultaneously is contradictory. `0 0 2 * * ?` means "at second 0, minute 0, hour 2, every day-of-month, every month, no specific day-of-week" — every day at 2:00:00 AM. This differs from plain Unix cron's five-field format (no seconds field, and no `?` wildcard) — worth being precise about, since conflating the two formats is an easy, exactly-the-kind-of-detail-worth-getting-backwards mistake.

### Misfire Handling — the actually new concept

**Definition:** a misfire is a scheduled trigger's fire time passing **without the job running** — most commonly because the scheduler process itself was down at that moment.

**The real decision, not an afterthought:** what should happen when the scheduler comes back up and notices a fire time was missed?

| Policy | Behavior | Correct for... |
|---|---|---|
| Fire now, once | Run immediately on recovery, then resume the normal schedule | Jobs where "late is fine, but only once" holds — tonight's reconciliation job is exactly this shape |
| Ignore the misfire | Skip straight to the *next* normally-scheduled time | Jobs tied to a specific moment's meaning (a "2am snapshot" run at 9am no longer represents a 2am snapshot at all) |
| Fire once per missed occurrence | Catch up on every missed run individually | Rarely correct — a scheduler down for a day could trigger a flood of catch-up executions |

> ⚠️ **Common Mistake:** not choosing a misfire policy deliberately, and inheriting whatever the library's default happens to be — a job whose semantics need "ignore and wait for next time" silently running under a "fire once per missed occurrence" default could genuinely do real harm (imagine that flood applied to, say, a job that sends a customer notification).

---

## Part 4 — At-Least-Once vs. Exactly-Once Execution

**At-least-once:** the system guarantees a job is attempted at least once, but may attempt it more than once under specific failure conditions — concretely, a worker claims a job, crashes mid-execution *after* the visibility timeout's window starts but *before* marking it done; the job becomes visible again and another worker (or the same one, restarted) picks it up and runs it a second time.

**Exactly-once, honestly:** a true end-to-end guarantee that a distributed job runs precisely once is genuinely hard — it would require some form of distributed consensus between "did this actually complete" and "will nobody else ever attempt it again," which is expensive and complex to build correctly. Real systems don't usually build that. They build **at-least-once delivery plus idempotent processing** instead, which produces an outcome *indistinguishable from true exactly-once*, viewed from outside the system, even though the delivery mechanism genuinely may retry underneath. This combination is sometimes called "effectively-once" — worth using that precise term rather than claiming a system is "exactly-once" when what's actually true is this combination.

> 🔗 **This is yesterday's exact mechanism, one layer down.** Day 130's payment system needed idempotency because *client retries* could duplicate a request; today's scheduler needs it because *worker crashes under a visibility timeout* can duplicate a job execution. Different trigger, same underlying fix — a job handler that is not idempotent is a genuine correctness bug under this model, not a hypothetical.

---

## Coding Exercise — Atomic Polling Lua Script

```lua
-- KEYS[1] = sorted-set key holding delayed jobs (score = execution timestamp)
-- ARGV[1] = current timestamp (now)
-- ARGV[2] = max jobs to claim this poll

local now   = tonumber(ARGV[1])
local limit = tonumber(ARGV[2])

local dueJobs = redis.call('ZRANGEBYSCORE', KEYS[1], '-inf', now, 'LIMIT', 0, limit)

if #dueJobs > 0 then
    redis.call('ZREM', KEYS[1], unpack(dueJobs))
end

return dueJobs
```

**Why this must run as one atomic Lua script, not two separate Redis calls:** a plain `ZRANGEBYSCORE` followed by a separate `ZREM`, issued as two commands, leaves a window where a second poller could read the *same* due jobs before the first poller removes them — both would then claim and process the same jobs. Wrapping the read-and-remove in one Lua script closes that window entirely, since Redis guarantees a Lua script executes without any other command interleaving.

> 🔑 **Key Takeaway:** this is the **fifth** distinct place this series has hit "read, then act on what you read, atomically or not at all" and reached for the same fix — Day 121 (rate limiting), Day 122 (dedup), Day 126 (seat-hold), Day 130 (idempotency claim and safe lock release), and now this. Recognizing the shape on sight, unprompted, is the actual skill; the specific Lua incantation is just its expression.

---

## Part 5 — HLD Mock #4: Payment System, Race-Condition Pressure

**Format, exactly as specified:** 45 minutes, subject is **Payment System** (Day 130) — the system with the most available follow-up depth of anything covered so far. Today's mock deliberately spends the *entire* time pushing into "what if two of these requests race" scenarios, rather than accepting a first correct-sounding answer and moving on.

**A worked run-through — read as a model, then run your own live:**

> **[Interviewer]:** "Walk me through your idempotency design, briefly, then let's stress it."
>
> **[You]:** *(brief recap of the Day 130 flow — check store, claim atomically, charge, ledger-write, release lock)*
>
> **[Interviewer, push 1]:** "Two identical retries of the same idempotency key arrive at literally the same millisecond, on two different application server instances. What happens?"
>
> **[You]:** "The claim step is a single atomic `SETNX` against the idempotency key itself — only one of those two calls can possibly succeed, because it's one Redis operation, not a check followed by a separate write. The instance that loses sees `claimed = false` and returns 'still processing' rather than proceeding."
>
> **[Interviewer, push 2]:** "The instance that won the claim charges the gateway successfully, but crashes before it writes the `completed` record. What happens on the next retry?"
>
> **[You]:** "This is the real gap, and I want to be honest about it rather than pretend the design so far closes it: our own store still shows `in-progress` or nothing, even though the charge genuinely went through externally. A naive retry would re-charge. The real mitigation is that the payment gateway itself should also be given the same idempotency key as a parameter — most real gateways, including Stripe, support exactly this — so the *gateway* becomes a second, independent source of truth for 'did this already happen,' not just our own store. That's a deliberate second layer, not a nice-to-have."
>
> **[Interviewer, push 3]:** "The distributed lock expires while the gateway call is still legitimately in flight, because the gateway is having a slow day. What now?"
>
> **[You]:** "That's yesterday's TTL-sizing trade-off showing up concretely — too short a TTL risks exactly this. Two real fixes: size the TTL generously against the gateway's actual observed p99 latency, with alerting if calls start approaching that ceiling; or use a heartbeat/lock-extension pattern, where the holder periodically renews the TTL while genuinely still working, instead of gambling on one fixed timeout set at acquire time."

### Debrief checklist

- [ ] Did at least one answer honestly name a real gap in the design rather than defending it as airtight?
- [ ] Was every "what if X and Y race" answer grounded in a specific mechanism (an atomic operation, a second source of truth, a TTL policy) rather than a reassurance ("that shouldn't happen")?
- [ ] Did the conversation go past the *first* correct-sounding answer at least twice, matching today's specific format requirement?
- [ ] Debrief with your partner: which race scenario was hardest to answer cleanly, and is that the one worth reviewing again before the real interview?

---

## Project Block Guide (1 hr)

**Repository:** `scalable-ecommerce-platform`. **Task:** add a `@Scheduled` background task simulating a nightly reconciliation job — verifying every Order maps to a successful Payment, flagging mismatches.

```java
@Component
public class ReconciliationJob {

    private static final Logger log = LoggerFactory.getLogger(ReconciliationJob.class);
    private final OrderRepository orderRepo;
    private final PaymentRepository paymentRepo;

    public ReconciliationJob(OrderRepository orderRepo, PaymentRepository paymentRepo) {
        this.orderRepo = orderRepo;
        this.paymentRepo = paymentRepo;
    }

    @Scheduled(cron = "0 0 2 * * ?")   // Quartz-style cron: every day at 2:00:00 AM
    public void reconcile() {
        List<Order> paidOrders = orderRepo.findByStatus(OrderStatus.PAID);
        List<String> mismatches = new ArrayList<>();

        for (Order order : paidOrders) {
            Optional<Payment> payment = paymentRepo.findByOrderId(order.getId());
            if (payment.isEmpty() || payment.get().getStatus() != PaymentStatus.SUCCESS) {
                mismatches.add("Order " + order.getId() + " marked PAID with no matching successful Payment");
            }
        }

        if (!mismatches.isEmpty()) {
            log.error("Reconciliation found {} mismatch(es): {}", mismatches.size(), mismatches);
        } else {
            log.info("Reconciliation clean — {} paid orders verified", paidOrders.size());
        }
    }
}
```

> ⚠️ **Worth being precise about, not conflating:** Spring's `@Scheduled(cron = ...)` uses Spring's own cron parser, which follows the same six-field, seconds-first format taught above — but it does **not** expose Quartz's actual misfire-policy API (`MISFIRE_INSTRUCTION_*`). Using `@Scheduled` cron syntax gives you Quartz-*style* scheduling syntax; it does not, by itself, give you Quartz's configurable misfire handling. Getting genuine misfire-policy control means wiring actual Quartz beans instead of the simpler annotation. Today's theory teaches the *concept* (what a misfire is, and that the policy is a real decision); the project's actual code demonstrates cron syntax, not full Quartz misfire configuration — a distinction worth stating plainly rather than blurring.

**Definition of done:** the scheduled task runs successfully and correctly flags a deliberately-introduced mismatch in test data (an order marked `PAID` with no corresponding successful `Payment` row) — verified by asserting the mismatch list is non-empty for that specific seeded case, and empty for a clean data set.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** review the application tracker across all 7 target companies — status, next steps, and any prep gap a specific upcoming round exposes.

**Uber-specific DSA cold-revision check** — Uber's own stated advice is to "practice DSU (Union-Find), DP, and classic CS problems, be ruthless about time complexity." Rather than leave that as a vague intention, here are two specific, already-taught problems worth redoing cold right now, before it's the week of the actual interview:

**Union-Find — LC 721, Accounts Merge** *(first taught Week 11, Day 76 — Union-Find + HashMap Grouping)*. Union accounts sharing any email in common, using emails (not account indices) as the elements being unioned; group all emails by their final root, attach each group back to its original account name. Time O(n·α(n)·k) for n accounts averaging k emails each with near-constant-time Union-Find operations; Space O(n·k). If this doesn't come back within a minute or two of reading the prompt, it's worth actually re-solving from scratch today, not just reading the recap.

**Dynamic Programming — LC 1143, Longest Common Subsequence** *(first taught Week 13, Day 88 — 2D String DP)*. `dp[i][j]` = LCS length of the first i characters of one string and the first j of the other; if characters match, `dp[i][j] = dp[i-1][j-1] + 1`; otherwise `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`. Time and Space O(m·n). Worth noting directly: this exact recurrence is what **Edit Distance** (LC 72, revised cold back in Week 17) generalizes — if LCS feels rusty, Edit Distance is evidence the underlying 2D-DP instinct is probably fine and this is just a specific-recurrence refresh, not a pattern-recognition gap.

---

## Day 131 — Interview Questions

**Q1. What is SQS Visibility Timeout, mechanically?** When a consumer receives a message, it becomes invisible to other consumers for a set window; if not explicitly deleted within that window, it becomes visible again for reclaiming — a lease with an expiry.

**Q2. What earlier concept in this series is Visibility Timeout structurally identical to?** Day 130's TTL-based distributed lock — same lease-with-expiry shape, applied to a queue message instead of a payment resource.

**Q3. Why does deleting a message on receive (rather than on completion) break the at-least-once guarantee?** A consumer that crashes mid-processing after the message was already deleted would silently lose it forever — at-least-once specifically requires the message to survive until completion is *confirmed*, not merely attempted.

**Q4. What's a concrete real bug caused by setting the visibility timeout too short?** A job that genuinely takes longer than the timeout gets reclaimed and run again by a second worker while the first is still legitimately processing it — duplicate execution caused purely by misconfiguration, not a flaw in the mechanism.

**Q5. Why does a hand-rolled Redis Sorted Set queue need extra work that SQS gives for free?** SQS's visibility timeout is a built-in queue feature; a Sorted Set poller has to build that same leasing discipline itself, typically using the exact SETNX-with-TTL pattern from Day 130.

**Q6. What's a misfire, and name two different correct policies for handling one.** A scheduled fire time passing without the job running (usually because the scheduler was down). "Fire now, once" suits jobs where late-but-once is fine; "ignore and wait for the next scheduled time" suits jobs tied to a specific moment's meaning that a late run can't actually represent.

**Q7. Why is Quartz's cron format different from plain Unix cron, precisely?** Quartz uses six fields with seconds first, and requires a `?` wildcard in exactly one of day-of-month or day-of-week since specifying both is contradictory; plain Unix cron has five fields and no `?`.

**Q8. Why do distributed schedulers commonly guarantee at-least-once rather than true exactly-once?** True exactly-once would require distributed consensus over "did this complete" and "will nothing else attempt it" — expensive and complex. It's cheaper to accept occasional duplicate delivery and require the job handler to be idempotent instead.

**Q9. What does "effectively-once" mean, and how does it differ from true exactly-once?** At-least-once delivery combined with idempotent processing, producing an outcome indistinguishable from exactly-once *from the outside*, even though the underlying mechanism genuinely may retry.

**Q10. Why must the polling Lua script combine the range-read and the removal into one atomic operation?** Two separate commands leave a window where a second poller could read the same due jobs before the first removes them, causing both to claim and process the same job.

**Q11. This is the fifth appearance of the same underlying shape in this series — name at least three of the other four.** Day 121 (rate limiting), Day 122 (notification dedup), Day 126 (seat-hold), Day 130 (idempotency claim and safe lock release).

**Q12. In today's mock, what's the real gap in a naive idempotency design that push 2 exposed?** A charge that succeeds at the gateway but crashes before the local `completed` record is written leaves the system's own store out of sync with what actually happened externally — closed by also passing the idempotency key to the gateway itself as a second source of truth.

**Q13. Why does `@Scheduled(cron = ...)` in Spring not give you Quartz's actual misfire-policy configuration?** `@Scheduled` uses Spring's own cron parser for scheduling syntax only; Quartz's configurable `MISFIRE_INSTRUCTION_*` policies require wiring actual Quartz beans, not just the annotation.

**Q14. Why is LC 1143 (Longest Common Subsequence) a reasonable single problem to represent "is my DP still sharp," rather than an arbitrary pick?** It's the foundational 2D-string-DP recurrence that Edit Distance directly generalizes — a clean, minimal test of the exact recognition skill a harder, already-revised problem builds on.

---

## Daily Deliverable Check

- [ ] Can explain at-least-once execution and why it demands idempotent job handlers, without notes.
- [ ] Nightly reconciliation `@Scheduled` task live and correctly flagging a deliberately-introduced mismatch.
- [ ] HLD Mock #4 completed and debriefed, with at least two "why not stop here" pushes genuinely worked through.
- [ ] LC 721 and LC 1143 both re-attempted cold today, not just read.

---

## What Tomorrow Assumes You Already Know Cold

Day 132 assumes Sorted Sets' O(log n) contract — used operationally today for the third time (Day 123 named it, Day 128 used it for Geo, today for delayed-job polling) without its mechanism ever being explained — is genuinely ready to finally be opened up tomorrow. It also assumes today's "at-least-once + idempotent processing = effectively-once" framing doesn't need re-deriving, since tomorrow's leaderboard system will briefly touch update semantics of its own and will refer back to today's language rather than re-explain it.
