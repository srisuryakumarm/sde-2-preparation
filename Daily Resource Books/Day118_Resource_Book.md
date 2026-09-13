# Day 118 Resource Book — LLD #9: Food Delivery System, and the System Design Framework Preview

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 117 Resource Book](Day117_Resource_Book.md)
**Next ▶:** [Day 119 Resource Book](Day119_Resource_Book.md)
**Companion to:** Day 118 of `Week_17_Revised.md`

---

## Recap

Day 117 closed BookMyShow's concurrency problem two ways and proved both fixes deterministically, then pushed on justifying the choice between them with a real traffic-pattern argument, not a coin flip. Today reuses Strategy a **second** time — Splitwise (Day 115) was its first full system-level application; today confirms the pattern generalizes rather than having been a one-off, and does it with a meaningfully different flavor of Strategy implementation worth naming precisely. Today also opens a new arc entirely: a first look at the System Design framework Week 18 will spend two full weeks on — **preview depth only**, not the real teaching.

## Learning Objectives

By the end of today, without notes:

1. Implement Food Delivery's partner-matching as Strategy, citing Day 107 and Day 115 directly.
2. Explain precisely why today's strategies being stateless/parameterless doesn't make this "less Strategy" than Splitwise's constructor-configured strategies — and name the actual test (who chooses, Day 109) that settles it, rather than leaning on the constructor-config heuristic.
3. Demonstrate runtime strategy-swapping, a capability Splitwise's fixed-per-`Expense` usage never exercised.
4. Name the System Design framework's five steps and what each is for, at preview depth — without pretending to have done Week 18's actual teaching yet.
5. Solve Edit Distance (LC 72) cold, narrating the three-way DP transition from memory.

## Concept Dependency Map

```
Week 16 Day 107: Strategy pattern taught
Week 16 Day 109: State vs Strategy — the actual test is WHO chooses/drives
                 the swap, not whether the implementation carries constructor
                 config (that's a common tell, not the definition)
Week 17 Day 115: Strategy's first full system application — Splitwise, split
                 types, mostly constructor-configured (ExactSplit(amounts))
        │
        ▼
Today: Food Delivery System
  └─ Strategy, SECOND application — partner matching
        ├─ NearestPartnerStrategy / HighestRatedPartnerStrategy — BOTH
        │  stateless, no constructor config — unlike Day 115's strategies
        ├─ Still genuinely Strategy: the CLIENT chooses which one runs —
        │  the actual test from Day 109, unaffected by the config difference
        └─ Runtime swap demonstrated — a capability unused in Splitwise
        │
        ▼
System Design Framework — PREVIEW ONLY (full depth: Week 18, Day 120+)
  1. Requirements  2. Estimation  3. HLD  4. Detailed Design  5. Bottlenecks
        │
        ▼
DSA Revision: Edit Distance (LC 72)
  (needs: 2D String DP — Wk13; the LCS mechanism it generalizes)
```

---

# Part 1 — Strategy, Second Application

**Prerequisites confirmed:** Strategy's mechanism (Day 107); its first full application (Day 115); the State-vs-Strategy test (Day 109).

## The design

```java
public class Restaurant {
    private final String restaurantId;
    private final double latitude;
    private final double longitude;
    public Restaurant(String restaurantId, double latitude, double longitude) {
        this.restaurantId = restaurantId; this.latitude = latitude; this.longitude = longitude;
    }
    public double getLatitude() { return latitude; }
    public double getLongitude() { return longitude; }
}

public class DeliveryPartner {
    private final String partnerId;
    private final double latitude;
    private final double longitude;
    private final double rating;
    public DeliveryPartner(String partnerId, double latitude, double longitude, double rating) {
        this.partnerId = partnerId; this.latitude = latitude; this.longitude = longitude; this.rating = rating;
    }
    public double getLatitude() { return latitude; }
    public double getLongitude() { return longitude; }
    public double getRating() { return rating; }
    public String getPartnerId() { return partnerId; }
}

public interface PartnerMatchingStrategy {
    DeliveryPartner findPartner(Restaurant restaurant, List<DeliveryPartner> availablePartners);
}
```

```java
public class NearestPartnerStrategy implements PartnerMatchingStrategy {
    @Override
    public DeliveryPartner findPartner(Restaurant restaurant, List<DeliveryPartner> availablePartners) {
        DeliveryPartner nearest = null;
        double minDistance = Double.MAX_VALUE;
        for (DeliveryPartner partner : availablePartners) {
            double distance = distance(restaurant.getLatitude(), restaurant.getLongitude(),
                                        partner.getLatitude(), partner.getLongitude());
            if (distance < minDistance) {
                minDistance = distance;
                nearest = partner;
            }
        }
        return nearest;
    }

    private double distance(double lat1, double lon1, double lat2, double lon2) {
        double dLat = lat1 - lat2;
        double dLon = lon1 - lon2;
        return Math.sqrt(dLat * dLat + dLon * dLon);   // simplified — real systems use Haversine or actual road-network ETA
    }
}

public class HighestRatedPartnerStrategy implements PartnerMatchingStrategy {
    @Override
    public DeliveryPartner findPartner(Restaurant restaurant, List<DeliveryPartner> availablePartners) {
        DeliveryPartner best = null;
        double maxRating = -1;
        for (DeliveryPartner partner : availablePartners) {
            if (partner.getRating() > maxRating) {
                maxRating = partner.getRating();
                best = partner;
            }
        }
        return best;
    }
}
```

```java
public class DeliveryAssignmentService {
    private PartnerMatchingStrategy strategy;

    public DeliveryAssignmentService(PartnerMatchingStrategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(PartnerMatchingStrategy strategy) {
        this.strategy = strategy;
    }

    public DeliveryPartner assign(Restaurant restaurant, List<DeliveryPartner> availablePartners) {
        return strategy.findPartner(restaurant, availablePartners);
    }
}
```

## Why this is still genuinely Strategy, despite looking different from Day 115

**🔑 Key Takeaway — the config-carrying heuristic is a tell, not the definition.** Day 115's strategies each carried their own constructor data (`ExactSplitStrategy(exactAmounts)`, `PercentageSplitStrategy(percentages)`). **Neither `NearestPartnerStrategy` nor `HighestRatedPartnerStrategy` carries any constructor configuration at all** — both are stateless, parameterless, and could even be reused as singletons, which sounds exactly like Day 109's description of *State* implementations, not Strategy's. That's worth noticing rather than glossing over.

The actual test, established Day 109, was never "does it carry constructor config" — that was flagged explicitly as a *secondary, common* signal, not the definition. The real test is **who chooses, and who drives the swap.** Here, whoever constructs `DeliveryAssignmentService` — the business logic, an ops dashboard, an A/B test configuration — explicitly decides `new NearestPartnerStrategy()` versus `new HighestRatedPartnerStrategy()`. The *client* is choosing, exactly as Strategy's definition requires; `DeliveryAssignmentService` itself never inspects which one it's holding or decides to swap on its own initiative (which is what would make it State instead). **Two structurally different-looking Strategy implementations, one config-carrying and one not, are both still Strategy for the identical underlying reason.**

## What's demonstrated here that Splitwise never exercised

Every `Expense` in Splitwise picked one `SplitStrategy` at construction and kept it for that expense's entire lifetime — never swapped. `DeliveryAssignmentService.setStrategy()` demonstrates the other half of Strategy's value: **swapping at runtime**, without touching any calling code. A promotional period favoring speed over cost, or an A/B test comparing the two matching approaches, becomes a single `setStrategy()` call — no changes anywhere else in the system, which is Open/Closed (Week 1, Day 6) showing up again, from a different angle than Chain of Responsibility demonstrated it on Day 113.

**⚠️ Common Mistake:** treating "no constructor config" as a reason to reach for an `enum` and a `switch` instead of Strategy ("it's just two stateless algorithms, why bother with an interface"). The switch would work today, with exactly two options — and become exactly the kind of edit-existing-code problem Open/Closed warns about the moment a third matching approach (say, `LowestETAStrategy`, factoring in current partner load) needs adding.

---

# Part 2 — The System Design Framework, Previewed

**This section is deliberately shallow.** Full depth — a real worked example, each step earning its own dedicated teaching — starts Week 18, Day 120 (URL Shortener). Today's job is only to have the five-step shape in hand as a mental map, not to have actually learned it yet.

The shape parallels the LLD framework already reflexive by now (Day 106: clarify → objects → relationships → patterns → code) — both move from clarification, through top-down structure, down to something concrete, just at a different unit of design (services and infrastructure, not classes):

1. **Requirements** — functional (what the system must do) and non-functional (scale, latency, consistency needs) — the system-level analogue of clarifying an LLD problem's scope before touching any class design.
2. **Estimation** — back-of-envelope math: expected requests per second, storage growth over time, bandwidth. This step exists because a system built for a thousand users and one built for a hundred million are different systems, and guessing wrong here undermines every decision made afterward.
3. **High-Level Design (HLD)** — the major services and how they connect: clients, load balancers, application servers, databases, caches, queues. The system-level analogue of an LLD system's core objects and relationships, one level up.
4. **Detailed Design** — drilling into one or two of the HLD's components in real depth. This is where LLD-style thinking gets reused directly at a larger scale — and where several individual techniques already taught piecemeal earlier in this series (Consistent Hashing, replication models, the CAP theorem — all Week 9) stop being isolated facts and become the actual working material of a step.
5. **Bottlenecks and Trade-offs** — naming the design's weak points (single points of failure, scaling ceilings) and how to address them.

**🔗 Forward reference, not a teaching point today:** step 4 is exactly where this week's concurrency work (Day 117) and Splitwise's algorithm-hiding-in-plain-sight lesson (Day 115) will resurface — a high-level design's weak points are very often exactly the kind of check-then-act race or greedy-vs-optimal question this week spent two full days on, just at a different scale. Worth having that connection ready when Week 18 gets there; not worth building out today.

---

# DSA Revision Block (1 hr)

## Edit Distance (LeetCode 72, Medium) — Pattern: 2D String DP

**Originally taught:** Week 13 (String DP), as a direct extension of Longest Common Subsequence's 2D table mechanism.

**Statement:** given two strings, return the minimum number of operations (insert, delete, replace a single character) to convert one into the other.

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];

    for (int i = 0; i <= m; i++) dp[i][0] = i;   // delete all i characters
    for (int j = 0; j <= n; j++) dp[0][j] = j;   // insert all j characters

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];             // characters already match — no operation needed
            } else {
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],   // replace
                                Math.min(dp[i - 1][j],       // delete from word1
                                         dp[i][j - 1]));     // insert into word1
            }
        }
    }
    return dp[m][n];
}
```

**The mechanism:** `dp[i][j]` = minimum operations to convert the first `i` characters of `word1` into the first `j` characters of `word2`. The base cases are the "convert to/from empty string" costs — converting `i` characters to nothing costs `i` deletions; converting nothing into `j` characters costs `j` insertions. When the current characters match, no operation is spent — the answer is exactly whatever it cost to align everything *before* these two characters (`dp[i-1][j-1]`). When they don't match, one operation is spent, and the three options are the three ways a mismatch can be resolved: **replace** the character (`dp[i-1][j-1]`, both pointers advance), **delete** from `word1` (`dp[i-1][j]`, only `word1`'s pointer advances), or **insert** into `word1` to match `word2` (`dp[i][j-1]`, only `word2`'s pointer advances) — take whichever of the three costs least.

**🔑 Key Takeaway — this is LCS's exact table shape, one more operation added.** LCS (Week 13) asked "how much can be kept without any edits at all"; Edit Distance asks "what's the minimum cost to make everything match," which needs the *insert* and *delete* options LCS never had to consider, alongside the *replace* option LCS's problem statement doesn't allow. Recognizing this as "LCS's table, extended" rather than an unrelated new problem is worth saying out loud.

**Complexity: O(m·n)** time and space — one table cell per pair of prefix lengths, O(1) work per cell. *(Extension, not needed today: space can be reduced to O(min(m,n)) by keeping only the current and previous row, since each cell only ever reads the row directly above and the current row's immediately preceding cell.)*

**Edge cases:** one string empty (immediately returns the other string's length — every character must be inserted or deleted, no matching possible); identical strings (`dp[m][n] = 0`, no operations needed, verified directly by the "characters match" branch firing every step along the diagonal); completely disjoint character sets (every cell forced into the "no match" branch — degenerates cleanly to `max(m,n)` when one string is a prefix-free scramble of the other, though the general formula handles this without needing a special case).

**💡 Interview Insight:** if asked to reconstruct the actual sequence of edits (not just the count), the same parent-pointer-tracking idea from LIS's O(n log n) approach (Day 117) applies here too — track which of the three options was chosen at each cell, then walk back from `dp[m][n]` to `dp[0][0]`.

---

# Project Block Guide

**Repository:** `lld-java/food-delivery/`. Build the classes above. **Definition of done:** both strategies correctly select different partners on a scenario where "nearest" and "highest-rated" disagree (construct a test case where the closest partner has a lower rating than a farther one — verify each strategy picks the one it should); `setStrategy()` demonstrated live, swapping mid-run with no changes to any calling code. Pushed.

**System Design preview:** no coding exercise — read Part 2 once, closely enough to name the five steps and what each is for without notes, and stop there.

# Career Block Guide

**Applications launch today** — this is the milestone the past week and a half of networking (Days 113–117) was deliberately front-loaded ahead of. **A practical order of operations:**

1. **Warm connections first.** Anyone from this week's targeted outreach (Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech) who responded is worth a direct message asking about a referral *before* applying cold through the portal — a referral changes how an application gets read, and asking after applying cold looks like an afterthought.
2. **Tailor the resume's top third per company**, not the whole document — the summary/headline and the most recent role's bullet points are what a recruiter's first 30-second pass actually reads closely; matching them to the specific role's language (without fabricating experience) meaningfully helps at that stage.
3. **Apply to every company on today's list**, even ones with a colder connection — application volume compounds, and today exists specifically to get all seven moving in parallel rather than trickling out one at a time.
4. **Log every application** — company, role, date, referral status, portal used — in a single tracker; five to seven days from now, keeping straight who to follow up with (and when a follow-up stops being premature) needs this written down, not remembered.

---

# Day 118 — Interview Questions

**Q1. Why is `NearestPartnerStrategy` still genuinely Strategy despite carrying no constructor configuration, unlike Splitwise's split strategies?** The definitional test (Day 109) is who chooses which implementation runs — the client, here whoever constructs `DeliveryAssignmentService` — not whether the implementation happens to carry constructor data. Constructor config is a common tell, not the definition; both config-carrying and stateless implementations can be equally valid Strategy usages.

**Q2. What capability does `setStrategy()` demonstrate that Splitwise's usage never did?** Runtime swapping — Splitwise fixed one strategy per `Expense` for that expense's entire lifetime, while Food Delivery can change its matching approach mid-run with zero changes to any calling code.

**Q3. Why would replacing this Strategy design with an `enum` and a `switch` be a mistake, even though it would work with exactly two options today?** It would violate Open/Closed the moment a third matching approach is needed — adding it would mean editing the existing switch statement, exactly the kind of modification-of-working-code Strategy (and Chain of Responsibility, Day 113) exist to avoid.

**Q4. Name the System Design framework's five steps, in order.** Requirements, Estimation, High-Level Design, Detailed Design, Bottlenecks and Trade-offs.

**Q5. What is the "Estimation" step actually for?** Establishing the real scale being designed for — expected request volume, storage growth, bandwidth — since a system built for a thousand users and one built for a hundred million require genuinely different designs, and every later decision depends on getting this roughly right first.

**Q6. In Edit Distance, why does the "characters match" case cost zero additional operations?** Because the cost of aligning everything before these two matching characters is already captured in `dp[i-1][j-1]` — a matching character requires no insert, delete, or replace, so the answer simply carries forward unchanged.

**Q7. What are the three options considered when characters don't match, and what does each correspond to?** Replace (`dp[i-1][j-1]`, both prefixes advance together), delete from `word1` (`dp[i-1][j]`, only `word1`'s prefix advances), and insert into `word1` (`dp[i][j-1]`, only `word2`'s prefix advances) — the minimum of the three, plus one operation, is the answer for that cell.

**Q8. How does Edit Distance relate to Longest Common Subsequence?** Same 2D table shape and base structure — LCS asks how much can be kept with zero edits allowed; Edit Distance asks the minimum cost to force a full match, which needs insert and delete options LCS doesn't have, plus a replace option LCS's problem statement doesn't permit at all.

---

# Daily Deliverable Check

- [ ] Food Delivery LLD complete — Strategy for partner matching, both implementations, `setStrategy()` demonstrated at runtime, pushed to `lld-java/food-delivery/`.
- [ ] Can explain, unprompted, why stateless Strategy implementations are still genuinely Strategy — the who-chooses test, not the constructor-config heuristic.
- [ ] System Design framework's five steps nameable without notes, at preview depth — no expectation of deeper fluency yet.
- [ ] All seven applications submitted (Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech); tracker started.
- [ ] Edit Distance (LC 72) solved cold; the three-way transition (replace/delete/insert) explainable from memory, tied explicitly back to LCS's table shape.

---

## What Tomorrow Assumes You Already Know Cold

Day 119 closes the LLD phase with Hotel Booking, and explicitly asks for a written comparison against BookMyShow's concurrency model — which means today's Strategy work isn't what tomorrow leans on directly, but this week's full run of "state the real trade-off, concretely, not abstractly" (Day 117's pessimistic-vs-optimistic answer; today's stateless-vs-configured Strategy nuance) is exactly the muscle Day 119's sync-vs-async comparison is going to require, one more time, before Week 17 closes.

**Next:** [Day 119 Resource Book](Day119_Resource_Book.md) — Hotel Booking System, Mock Interview #5, and Week 17 Consolidation.
