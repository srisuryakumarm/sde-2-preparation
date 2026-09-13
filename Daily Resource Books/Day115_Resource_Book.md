# Day 115 Resource Book — LLD #7: Splitwise — Strategy, Applied For Real, and a Debt-Settlement Algorithm

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 114 Resource Book](Day114_Resource_Book.md)
**Next ▶:** [Day 116 Resource Book](Day116_Resource_Book.md)
**Companion to:** Day 115 of `Week_17_Revised.md`

---

## Recap

Day 114 reused State a third time and introduced SCAN/LOOK dispatch, proven — not just claimed — to beat naive FIFO by more than 2x on a worked example. Today reuses a different pattern for the first time in this series' actual system-building: **Strategy**, taught conceptually on Day 107 (the `PercentageDiscount(0.10)` example, and directly contrasted against State the same week) but never yet chosen as the pattern actually implementing a full LLD system. Today it is. Today also opens the theory block's central claim for this stretch of the series — that a "simple" LLD problem can be hiding a real algorithm underneath — with Splitwise's debt-settlement logic as the first proof of it.

## Learning Objectives

By the end of today, without notes:

1. Implement Splitwise's three split types (Equal, Exact, Percentage) as Strategy implementations, citing Day 107's mechanism directly and explaining precisely why this is a genuine, not superficial, application of the pattern.
2. Explain the difference between a pairwise debt ledger and net balances, and why settlement needs the latter.
3. Implement the dual-heap greedy settlement algorithm, and **prove** — via an exchange-style counting argument, not an assertion — that it never needs more than (n − 1) transactions for n people with nonzero balance.
4. State honestly where that guarantee stops being the *true theoretical minimum*, and why finding the true minimum is a fundamentally harder problem (tied to LeetCode 465, NP-hard in general) that a greedy heap approach was never going to solve.
5. Solve Generate Parentheses (LC 22) cold, narrating the backtracking constraint from memory.

## Concept Dependency Map

```
Week 16 Day 107: Strategy pattern taught (PercentageDiscount(0.10) — client
                 chooses the algorithm, implementations carry their own
                 constructor-configured data)
Week 8  Day 54:  Heap mechanism (array-backed complete binary tree, sift-up/down)
Week 8  Day 55:  Custom Comparator (anonymous inner class) for heap ordering
Week 9  Day 58:  Two Heaps — max-heap of smaller half + min-heap of larger half,
                 BALANCED for O(1) median access (a different heap role — noted
                 explicitly below, not the same technique reused)
Week 4  Day 23:  Greedy algorithms — exchange arguments, not assertions
Week 12 Day 84:  Coin Change (LC 322) — DP, because that greedy failed
        │
        ▼
Today: Splitwise
  ├─ Strategy — FIRST full system-level application (not a toy example)
  │     EqualSplitStrategy / ExactSplitStrategy / PercentageSplitStrategy
  │
  ├─ BalanceSheet — pairwise ledger, netted into per-person balances
  │
  └─ Dual-Heap Greedy Settlement (NEW heap role — 7th distinct role this
     series has built, after: bounded max-heap of size k, running-median
     two-heaps, task-scheduling cooldown heap, etc.)
        ├─ Two INDEPENDENT max-heaps (creditors, debtors) — not one balanced
        │  pair splitting a single dataset, unlike Day 58's Two Heaps
        ├─ Proof: ≤ (n−1) transactions, always
        └─ Honest limit: NOT always the true theoretical minimum — that's
           LC 465, Optimal Account Balancing, NP-hard, out of scope today
        │
        ▼
DSA Revision: Generate Parentheses (LC 22)
  (needs: Backtracking mechanism — Wk9 D61; StringBuilder — Wk2)
```

---

# Part 1 — Strategy, Applied For Real

**Prerequisites confirmed:** Strategy's mechanism — Day 107. **What's genuinely new today:** not the pattern itself, but building an entire system around it as the primary design decision, rather than a supporting example.

**Requirements clarified first:** an expense has a payer, a total amount, a set of participants, and a way of dividing the amount among them. That "way of dividing" is exactly what varies independently of everything else about an expense — precisely Strategy's signature symptom (Day 107: *the client picks which algorithm runs*).

```java
public interface SplitStrategy {
    Map<String, Double> calculateSplit(double totalAmount, List<String> participants);
}
```

```java
public class EqualSplitStrategy implements SplitStrategy {
    @Override
    public Map<String, Double> calculateSplit(double totalAmount, List<String> participants) {
        Map<String, Double> shares = new HashMap<>();
        double perPerson = totalAmount / participants.size();
        for (String person : participants) {
            shares.put(person, perPerson);
        }
        return shares;
    }
}

public class ExactSplitStrategy implements SplitStrategy {
    private final Map<String, Double> exactAmounts;   // constructor-configured — same shape as Day 107's PercentageDiscount(0.10)

    public ExactSplitStrategy(Map<String, Double> exactAmounts) {
        this.exactAmounts = exactAmounts;
    }

    @Override
    public Map<String, Double> calculateSplit(double totalAmount, List<String> participants) {
        double sum = 0.0;
        for (double amount : exactAmounts.values()) {
            sum += amount;
        }
        if (Math.abs(sum - totalAmount) > 0.01) {
            throw new IllegalArgumentException("Exact amounts must sum to the total.");
        }
        return new HashMap<>(exactAmounts);
    }
}

public class PercentageSplitStrategy implements SplitStrategy {
    private final Map<String, Double> percentages;   // constructor-configured

    public PercentageSplitStrategy(Map<String, Double> percentages) {
        this.percentages = percentages;
    }

    @Override
    public Map<String, Double> calculateSplit(double totalAmount, List<String> participants) {
        double sum = 0.0;
        for (double pct : percentages.values()) {
            sum += pct;
        }
        if (Math.abs(sum - 100.0) > 0.01) {
            throw new IllegalArgumentException("Percentages must sum to 100.");
        }
        Map<String, Double> shares = new HashMap<>();
        for (Map.Entry<String, Double> entry : percentages.entrySet()) {
            shares.put(entry.getKey(), totalAmount * entry.getValue() / 100.0);
        }
        return shares;
    }
}
```

```java
public class Expense {
    private final String paidBy;
    private final double totalAmount;
    private final List<String> participants;
    private final SplitStrategy splitStrategy;

    public Expense(String paidBy, double totalAmount, List<String> participants, SplitStrategy splitStrategy) {
        this.paidBy = paidBy;
        this.totalAmount = totalAmount;
        this.participants = participants;
        this.splitStrategy = splitStrategy;
    }

    public Map<String, Double> getShares() { return splitStrategy.calculateSplit(totalAmount, participants); }
    public String getPaidBy() { return paidBy; }
}
```

**🔑 Key Takeaway — why this is a genuine application, not a superficial one.** The tell from Day 107 was: does the *client* choose the algorithm, and do implementations carry their own configuration? Both hold here exactly: whoever creates an `Expense` decides `new EqualSplitStrategy()` versus `new ExactSplitStrategy(amounts)` versus `new PercentageSplitStrategy(percentages)`, and the latter two carry meaningfully different constructor data, not just a label. `Expense` itself never inspects which concrete strategy it holds — it calls `calculateSplit` and nothing else, exactly the same shape as Day 107's discount example, just with real consequences (money) attached instead of a toy price.

**⚠️ Common Mistake — validating inside the wrong place.** The sum-check in `ExactSplitStrategy` and `PercentageSplitStrategy` belongs *inside* each strategy, not in `Expense` or some outer validator — each strategy alone knows what "valid input" means for its own algorithm (exact amounts summing to the total; percentages summing to 100). Pushing that check upward would mean `Expense` needs to know about every strategy's internal rules, which defeats the entire point of the interface boundary.

---

# Part 2 — From Pairwise Ledger to Net Balances

An expense produces per-person *shares* — but Splitwise's real ledger needs to track who owes *whom*, not just how much each person's share was.

```java
public class BalanceSheet {
    // balances.get(debtor).get(creditor) = amount debtor owes creditor
    private final Map<String, Map<String, Double>> balances = new HashMap<>();

    public void addExpense(Expense expense) {
        String paidBy = expense.getPaidBy();
        for (Map.Entry<String, Double> entry : expense.getShares().entrySet()) {
            String person = entry.getKey();
            if (!person.equals(paidBy)) {
                recordDebt(person, paidBy, entry.getValue());
            }
        }
    }

    private void recordDebt(String debtor, String creditor, double amount) {
        if (!balances.containsKey(debtor)) {
            balances.put(debtor, new HashMap<String, Double>());
        }
        Map<String, Double> debtorBalances = balances.get(debtor);
        double current = debtorBalances.getOrDefault(creditor, 0.0);
        debtorBalances.put(creditor, current + amount);
    }

    public Map<String, Double> computeNetBalances() {
        Map<String, Double> net = new HashMap<>();
        for (Map.Entry<String, Map<String, Double>> debtorEntry : balances.entrySet()) {
            String debtor = debtorEntry.getKey();
            for (Map.Entry<String, Double> creditorEntry : debtorEntry.getValue().entrySet()) {
                String creditor = creditorEntry.getKey();
                double amount = creditorEntry.getValue();
                net.put(debtor, net.getOrDefault(debtor, 0.0) - amount);
                net.put(creditor, net.getOrDefault(creditor, 0.0) + amount);
            }
        }
        return net;
    }
}
```

**Why netting matters:** a pairwise ledger can have Bob owing Alice ₹1000 *and* Alice owing Bob ₹500 simultaneously (from two different expenses) — settling those as two separate transactions is strictly wasteful when a single ₹500 payment from Bob to Alice closes both out. Netting collapses every person down to one signed number — positive means owed money overall, negative means owes money overall — which is the only representation the settlement algorithm below actually needs.

---

# Part 3 — Dual-Heap Greedy Settlement (New)

**Prerequisites confirmed:** Heap mechanism (Week 8, Day 54); custom `Comparator` via anonymous inner class (Week 8, Day 55); Greedy algorithms and the exchange-argument discipline (Week 4, Day 23).

## Why this is a genuinely new heap role, not a repeat of Day 58

Week 9's Two Heaps technique (running median) splits **one** dataset into two balanced halves — a max-heap of the smaller half, a min-heap of the larger half — kept within one element of each other by size. What Splitwise needs is structurally different: **two entirely independent max-heaps**, one of creditors and one of debtors, with no balancing relationship between their sizes at all. Calling both "two heaps" would blur a real distinction — this is heap role **#7** in this series (after: bounded-size-k max-heap, running median, task-scheduling cooldown heap, and others already logged in the curriculum map), not a reapplication of #3.

```java
class Balance {
    String person;
    double amount;
    Balance(String person, double amount) { this.person = person; this.amount = amount; }
}

class Transaction {
    String from;
    String to;
    double amount;
    Transaction(String from, String to, double amount) { this.from = from; this.to = to; this.amount = amount; }
    @Override
    public String toString() {
        return from + " pays " + to + ": Rs." + String.format("%.2f", amount);
    }
}

public class SettlementCalculator {
    public List<Transaction> minimizeCashFlow(Map<String, Double> netBalances) {
        PriorityQueue<Balance> creditors = new PriorityQueue<>(new Comparator<Balance>() {
            @Override
            public int compare(Balance a, Balance b) { return Double.compare(b.amount, a.amount); }
        });
        PriorityQueue<Balance> debtors = new PriorityQueue<>(new Comparator<Balance>() {
            @Override
            public int compare(Balance a, Balance b) { return Double.compare(b.amount, a.amount); }
        });

        for (Map.Entry<String, Double> entry : netBalances.entrySet()) {
            double balance = entry.getValue();
            if (balance > 0.01) {
                creditors.offer(new Balance(entry.getKey(), balance));
            } else if (balance < -0.01) {
                debtors.offer(new Balance(entry.getKey(), -balance));   // store as a positive amount owed
            }
        }

        List<Transaction> transactions = new ArrayList<>();
        while (!creditors.isEmpty() && !debtors.isEmpty()) {
            Balance creditor = creditors.poll();
            Balance debtor = debtors.poll();
            double settleAmount = Math.min(creditor.amount, debtor.amount);
            transactions.add(new Transaction(debtor.person, creditor.person, settleAmount));

            double creditorRemaining = creditor.amount - settleAmount;
            double debtorRemaining = debtor.amount - settleAmount;
            if (creditorRemaining > 0.01) creditors.offer(new Balance(creditor.person, creditorRemaining));
            if (debtorRemaining > 0.01) debtors.offer(new Balance(debtor.person, debtorRemaining));
        }
        return transactions;
    }
}
```

**⚠️ Common Mistake — using `double` for money at all.** The `0.01` epsilon threshold above is a real symptom of a real problem: floating-point arithmetic accumulates rounding error across enough operations, which is exactly the kind of bug that shows up as "Splitwise says I owe ₹0.0000000003" in production. A real system uses integer minor-units (paise) or `BigDecimal`, never raw `double`, for money. `double` is used here only so the algorithm's actual logic isn't obscured by `BigDecimal`'s more verbose API — flag this explicitly if building this for real, or if an interviewer asks about production-readiness.

## Worked trace, netted from real expenses

Three friends, three expenses, split equally each time: Alice pays ₹3000 (hotel), Bob pays ₹1500 (dinner), Carol pays ₹900 (cab).

```
Hotel  (Alice paid ₹3000, ÷3 = ₹1000 each): Bob owes Alice 1000; Carol owes Alice 1000
Dinner (Bob paid ₹1500, ÷3 = ₹500 each):    Alice owes Bob 500; Carol owes Bob 500
Cab    (Carol paid ₹900, ÷3 = ₹300 each):   Alice owes Carol 300; Bob owes Carol 300

Net(Alice) = (owed: 1000+1000) − (owes: 500+300) = 2000 − 800 = +1200
Net(Bob)   = (owed: 500+500)  − (owes: 1000+300) = 1000 − 1300 = −300
Net(Carol) = (owed: 300+300)  − (owes: 1000+500) = 600 − 1500  = −900
Check: 1200 − 300 − 900 = 0 ✓

creditors = [Alice:1200], debtors = [Carol:900, Bob:300]   (max-first: Carol before Bob)

Round 1: creditor=Alice(1200), debtor=Carol(900). settle=min(1200,900)=900.
         Transaction: Carol pays Alice ₹900.
         Alice remaining 300 → pushed back. Carol remaining 0 → done.
Round 2: creditor=Alice(300), debtor=Bob(300). settle=300.
         Transaction: Bob pays Alice ₹300.
         Both remaining 0 → done.
```

**Result: 2 transactions** (Carol→Alice ₹900, Bob→Alice ₹300) settle all three people, versus up to 3 separate pairwise relationships (Alice–Bob, Alice–Carol, Bob–Carol) a naive per-pair settlement would need to consider. This also happens to be the true minimum for this example — provably, since no proper subset of {Alice:+1200, Bob:−300, Carol:−900} sums to zero (checked directly: 1200−300=900≠0, 1200−900=300≠0, −300−900=−1200≠0) — so 3 people with no separable sub-group forces exactly (n−1) = 2 as both the greedy result *and* the theoretical floor.

## Proof: the algorithm never needs more than (n − 1) transactions

**Claim:** for n people with nonzero net balance, the loop above terminates in at most (n − 1) transactions.

**Proof:** every iteration pops exactly one creditor and one debtor and settles `min(creditor.amount, debtor.amount)`. Whichever of the two has the smaller amount is **fully** settled by that transaction and is *not* pushed back — its balance reaches exactly zero. So every single transaction strictly reduces the count of people with a nonzero balance by **at least one** (by exactly two, on the rare occasion both amounts were equal). Starting from n people with nonzero balance and needing to reach zero: since all balances sum to zero overall, the very last two people remaining must exactly cancel each other (a single leftover nonzero balance would break the sum-to-zero invariant) — so the final transaction always retires exactly two people at once. That gives, at most, (n − 2) transactions retiring one person each, plus one final transaction retiring the last two: **(n − 2) + 1 = n − 1**, exactly matching the exchange-argument discipline Week 4 established — a locally-best choice justified by tracking what it provably guarantees, not just asserted to be good.

## The honest limit — where "minimizes transactions" needs a caveat

**What's proven above is a guaranteed ceiling of (n − 1), and it's genuinely tight** — achieved exactly whenever no proper subset of the balances happens to sum to zero, which is the common case (and true in the trace above). **That ceiling is what "minimizes settlement transactions" means in the practical, interview-relevant sense the plan is asking for, and it's a real, substantial improvement over a naive pairwise ledger.**

**💡 Interview Insight, worth having ready but not over-claiming unprompted:** the (n − 1) bound is *not* always the absolute theoretical minimum. If the balances happen to split into separable zero-summing sub-groups — say six people where one subset of three nets to zero independently of the other three — the true minimum could be lower than (n − 1), because each sub-group can settle internally without ever routing money through the other. Finding *that* true minimum in general is a fundamentally harder problem: it's LeetCode 465, **Optimal Account Balancing**, solved via backtracking/DFS over which balances to net against each other, and it's NP-hard in the general case — closely related to subset-sum. A greedy, polynomial-time heap approach was never going to solve an NP-hard problem exactly; it wasn't designed to, and no real production Splitwise-scale system bothers, because (n − 1) is already an excellent practical bound at typical group sizes. **If an interviewer pushes with "is this actually optimal?" — this is the precise, honest answer:** guaranteed ≤ (n − 1), tight in the common case, not guaranteed to be the global theoretical minimum in the general case, and finding that general minimum is a different, much harder problem.

**Complexity:** each iteration is **O(log n)** (two heap pops, up to two pushes), and at most n iterations run (bounded by the proof above) — **O(n log n)** total. **Space O(n)** for the two heaps.

**Edge cases:** a single person with a nonzero balance (impossible if balances actually sum to zero — would indicate a bug upstream, worth an explicit sanity-check assertion); everyone's balance already zero (both heaps start empty, loop never runs, zero transactions — correct); a two-person group (trivially one transaction, matching n − 1 = 1).

---

# DSA Revision Block (1 hr)

## Generate Parentheses (LeetCode 22, Medium) — Pattern: Backtracking, Counter-Gated

**Originally taught:** Week 9, Day 61.

**Statement:** given `n` pairs of parentheses, generate all combinations of well-formed parenthesis strings.

```java
public List<String> generateParenthesis(int n) {
    List<String> result = new ArrayList<>();
    backtrack(result, new StringBuilder(), 0, 0, n);
    return result;
}

private void backtrack(List<String> result, StringBuilder current, int open, int close, int max) {
    if (current.length() == max * 2) {
        result.add(current.toString());
        return;
    }
    if (open < max) {
        current.append('(');
        backtrack(result, current, open + 1, close, max);
        current.deleteCharAt(current.length() - 1);
    }
    if (close < open) {
        current.append(')');
        backtrack(result, current, open, close + 1, max);
        current.deleteCharAt(current.length() - 1);
    }
}
```

**The mechanism, recapped:** two counters gate the two possible moves at every step. `open < max` permits adding `(` — there's still an opening bracket left to place. `close < open` permits adding `)` — and only when strictly fewer closes than opens have been placed so far, which is exactly the invariant that guarantees every *prefix* of the string stays valid, not just the final result. Both branches append, recurse, then `deleteCharAt` — the backtracking "undo" step, restoring `current` to exactly what it was before that branch was tried, so the next branch starts from a clean slate.

**Why `close < open`, not `close <= open`:** if closes were ever allowed to equal or exceed opens, a `)` could appear with no matching `(` before it in the string built so far — an invalid prefix. This is the entire correctness argument in one inequality.

**Complexity:** the number of valid sequences generated is the nth Catalan number, and the standard accepted bound for this problem is **O(4^n / √n)** — each valid string also costs O(n) to build via `StringBuilder` and copy into the result list, but that factor is dominated by the Catalan growth rate itself. **Space O(n)** beyond the output, for the recursion depth and the `StringBuilder`.

**Edge cases:** `n = 0` (a single empty string is the only valid output — the base case fires immediately since `max * 2 = 0`); `n = 1` (exactly `"()"`, one valid sequence).

**💡 Interview Insight:** if asked to *count* rather than *generate* all valid sequences, the answer is the closed-form Catalan number, `C(n) = (2n)! / ((n+1)! · n!)` — worth citing directly rather than re-deriving the backtracking tree's size by hand, since counting doesn't require actually building any strings at all.

---

# Project Block Guide

**Repository:** `lld-java/splitwise/`. Build the classes above. **Definition of done:** all three split strategies produce correct shares (verify percentages/exact amounts against their own validation); the three-person hotel/dinner/cab example above settles in exactly 2 transactions, matching the worked trace precisely (Carol→Alice ₹900, Bob→Alice ₹300). Pushed.

# Career Block Guide

**LinkedIn Post 22 (30–40 min):** *"Minimizing Cash Flow: the Splitwise debt algorithm, explained."* Draft outline: open with the concrete problem (group expenses create a tangled web of who-owes-whom); show the netting step collapses that web to one number per person; walk through the dual-heap greedy match with the worked trace above (2 transactions, not 3); close with the honest caveat — this guarantees ≤(n−1), which is what real systems use, while naming (without over-explaining) that the absolute theoretical minimum is a harder, NP-hard problem. The caveat is what separates a post that sounds confident from one that's actually credible to an audience that includes engineers who'd know if you oversold it.

---

# Day 115 — Interview Questions

**Q1. What makes Splitwise's split logic a genuine Strategy application rather than a superficial one?** The client (whoever creates the `Expense`) chooses which concrete strategy to use, and two of the three strategies carry their own constructor-configured data (exact amounts, percentages) — the identical shape as Day 107's `PercentageDiscount(0.10)`, just with real financial consequences.

**Q2. Why does the sum-validation for exact amounts and percentages live inside each strategy rather than in `Expense`?** Each strategy alone knows what "valid input" means for its own algorithm. Pushing validation up into `Expense` would force it to understand every strategy's internal rules, defeating the interface boundary's purpose.

**Q3. Why net balances instead of settling the pairwise ledger directly?** A pairwise ledger can have redundant opposing debts (Bob owes Alice ₹1000 while Alice owes Bob ₹500 from a different expense) that settle more efficiently as one payment once netted — settling every pairwise entry separately wastes transactions.

**Q4. How is the dual-heap technique here different from Week 9's Two Heaps (running median)?** Two Heaps splits one dataset into two *balanced*, size-tracked halves for O(1) median access. This uses two entirely *independent* max-heaps — creditors and debtors — with no balancing relationship between their sizes at all. Structurally different roles that happen to share the word "two heaps."

**Q5. Prove the algorithm never needs more than (n − 1) transactions.** Every transaction settles `min(creditor, debtor)`, which fully zeroes out whichever side was smaller — so every transaction reduces the count of people with nonzero balance by at least one. Since all balances sum to zero, the last two remaining must exactly cancel, so the final transaction always retires two at once: at most (n − 2) single-retirement transactions plus one double-retirement transaction gives (n − 2) + 1 = n − 1.

**Q6. Is (n − 1) always the true theoretical minimum number of transactions?** No — when balances split into separable zero-summing sub-groups, the true minimum can be lower, since each sub-group settles independently. Finding that true minimum in general is LeetCode 465 (Optimal Account Balancing), NP-hard, solved via backtracking — a fundamentally different, harder problem than this greedy approach was built to solve.

**Q7. Why use integer minor-units or `BigDecimal` instead of `double` for money in a real system?** `double` accumulates floating-point rounding error across enough operations, which surfaces as impossible-looking residual balances in production. `double` is used here only to keep the algorithm's logic visible without `BigDecimal`'s verbosity.

**Q8. In Generate Parentheses, why does the close-paren branch check `close < open` rather than `close <= open`?** Allowing closes to equal or exceed opens would let a `)` appear before a matching `(` somewhere earlier in the string — an invalid prefix. `close < open` is the entire correctness guarantee for every prefix, not just the finished string.

**Q9. What does `deleteCharAt` accomplish in the backtracking recursion?** It's the "undo" step — after a branch has been fully explored, it restores `current` to exactly its state before that branch was tried, so the next branch (if any) starts clean rather than carrying over the previous attempt's characters.

**Q10. If asked to count valid parenthesis sequences instead of generating them, what's the fast answer?** The nth Catalan number, `C(n) = (2n)! / ((n+1)! · n!)` — a closed form, requiring no backtracking tree to actually be built.

---

# Daily Deliverable Check

- [ ] Splitwise LLD complete — Strategy for splits, `BalanceSheet` netting, dual-heap settlement, pushed to `lld-java/splitwise/`.
- [ ] Three-person worked example settles in exactly 2 transactions, matching the trace above.
- [ ] Can state, unprompted, both the proven (n − 1) guarantee *and* its honest limit (not always the true NP-hard-optimal minimum) — not just the first half.
- [ ] LinkedIn Post 22 drafted and published, including the caveat, not just the confident version.
- [ ] Generate Parentheses (LC 22) solved cold; the `close < open` correctness argument explainable from memory.

---

## What Tomorrow Assumes You Already Know Cold

Day 116 (BookMyShow, Part 1) needs nothing new from today directly — no new pattern, just the 5-step framework applied once more, this time to a system explicitly described as hiding a concurrency problem the same way today's system hid an algorithm problem underneath what looked like plain object design. That pattern-recognition instinct — "this looks like simple CRUD-shaped design, but there's a real algorithm/problem underneath" — is exactly what today was building, and Day 116 is the next place it gets tested.

**Next:** [Day 116 Resource Book](Day116_Resource_Book.md) — BookMyShow, Part 1: Design.
