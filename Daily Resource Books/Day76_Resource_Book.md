# Day 76 — Union-Find: String Grouping, and Observability with Prometheus

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 75 Resource Book](Day75_Resource_Book.md)
**Next ▶:** [Day 77 Resource Book](Day77_Resource_Book.md)
**Companion to:** Day 76 of `Week_11_Revised.md`

---

## Recap

Yesterday's two problems unioned *rows and columns* and processed equations in two disciplined passes — both cases where the thing being unioned wasn't the raw input elements themselves, but an abstraction chosen to make the connectivity structure visible. Today continues that same instinct one step further: unioning **index positions** to justify a full character rearrangement, and unioning **emails** (not accounts) to correctly merge identities that share no direct account entry but do share an intermediate email.

---

## Learning Objectives

By the end of today, without notes:

1. Prove — not just state — why a connected component of swappable indices can be rearranged into *any* permutation, not merely one specific swap sequence.
2. Solve Smallest String With Swaps by sorting each component's characters independently and placing them at that component's sorted indices.
3. Explain why Accounts Merge unions *emails*, not account entries directly, and why that choice is what correctly merges two accounts sharing no email in common but each sharing one with a third.
4. Explain Prometheus's pull-based model, contrast it with a push-based alternative, and state one genuine trade-off in each direction — not just "pull is better."
5. Explain what Micrometer actually is (a facade, not a monitoring system itself) and how it connects to Actuator's `/actuator/prometheus` endpoint.

---

## Concept Dependency Map

```
Day 74-75: UnionFind class, and the recurring instinct
           "union the abstraction, not the raw input" —
           rows/columns (Day 75), equation variables (Day 75)
        │
        ├─▶ Smallest String With Swaps (LC 1202)
        │   union INDEX POSITIONS; proof that a connected
        │   component admits any permutation
        │
        └─▶ Accounts Merge (LC 721)
            union EMAILS, not accounts; grouping-by-root
            here is the same shape as Day 5's grouping-by-
            canonical-key HashMap pattern, different key type

Independent theory track:
Day 74-75: AWS Networking, IAM, S3/EBS
        │
        ▼
TODAY: Observability — Prometheus (pull-based scraping),
       Micrometer (metrics facade), /actuator/prometheus
```

---

# Part 1 — Union-Find, Continued

**Prerequisites, confirmed:** `UnionFind` (Day 74) ✅; the "union the abstraction, not the raw elements" pattern from yesterday (rows/columns, equation variables) ✅; `HashMap`-based grouping by a computed key (Day 5) ✅.

## Problem 5: Smallest String With Swaps (LeetCode 1202, Medium) — Pattern: Union-Find

**Statement:** given a string `s` and a list of index pairs `pairs`, where each pair `[a, b]` means the characters at indices `a` and `b` can be swapped any number of times, return the lexicographically smallest string achievable.

**The reduction:** union every index pair. Within each resulting connected component of indices, the character currently sitting at *any* one of those positions can, through some sequence of allowed swaps, end up at *any other* position in the same component.

### Why a Connected Component Admits Any Permutation — Proven, Not Asserted

Take two positions `i` and `j` in the same connected component. Because the component is connected, there's a path of allowed swaps `i — k₁ — k₂ — ... — j`. Performing that path's swaps in sequence — `swap(i,k₁)`, then `swap(k₁,k₂)`, ..., then `swap(...,j)` — moves the character originally at `i` all the way to `j`, shifting each intermediate position's character one step along the path in the process. Every individual swap used is one of the explicitly allowed pairs, so the whole maneuver only ever uses legal moves.

Repeating this "move one character along a path" operation, one character at a time — always moving whichever position currently holds the character that belongs at the next target position — is exactly a selection-sort-style process, and it only ever uses legal swaps at each step. Since this construction can realize the fully **sorted** arrangement of a component's characters specifically, and the same path-based reasoning works to reach *any* target arrangement, not sorted order alone, the component supports arbitrary rearrangement. (This is a specific case of a more general, well-established fact: the swaps corresponding to the edges of a connected graph generate every possible rearrangement of that graph's vertices — connectivity alone is both necessary and sufficient.)

Since arbitrary rearrangement is achievable, the lexicographically smallest achievable string is simply: within each component, sort the characters and place them back at the component's indices in **sorted index order** — smallest character to smallest index, and so on.

```java
public static String smallestStringWithSwaps(String s, List<List<Integer>> pairs) {
    int n = s.length();
    UnionFind uf = new UnionFind(n);

    for (List<Integer> pair : pairs) {
        uf.union(pair.get(0), pair.get(1));
    }

    Map<Integer, List<Integer>> components = new HashMap<>();
    for (int i = 0; i < n; i++) {
        components.computeIfAbsent(uf.find(i), k -> new ArrayList<>()).add(i);
    }

    char[] result = s.toCharArray();
    for (List<Integer> indices : components.values()) {
        List<Character> chars = new ArrayList<>();
        for (int idx : indices) chars.add(s.charAt(idx));
        Collections.sort(chars);
        Collections.sort(indices);
        for (int i = 0; i < indices.size(); i++) {
            result[indices.get(i)] = chars.get(i);
        }
    }

    return new String(result);
}
```

### Worked Trace

`s = "cba"`, `pairs = [[0,1],[1,2]]` — note this is the **same chaining shape** as yesterday's `a==b, b==c` transitive closure, just moving characters instead of enforcing equality.

`union(0,1)`: different roots, merge. `union(1,2)`: `find(1)` resolves through the just-set parent pointer to the same root as `0`; `find(2)=2`, different root, merge — all three indices end up under one root.

Single component `{0,1,2}`. Characters at those indices: `s[0]='c', s[1]='b', s[2]='a'` → sorted: `['a','b','c']`. Sorted indices: `[0,1,2]`. Placement: `result[0]='a', result[1]='b', result[2]='c'`.

**Final string: `"abc"`** — matches the known correct result for this exact classic input.

**Complexity:** Time O(n log n × α(n)) — unions and grouping cost O((n + pairs) × α(n)); sorting each component's characters costs, summed across all components, at most O(n log n) total (since `Σ kᵢ log kᵢ ≤ n log n` whenever `Σ kᵢ = n`). Space O(n) for the components map and result array.

**Edge cases:** a position that appears in no pair at all is its own singleton component — trivially "sorted" already, since there's nothing else to compare it against. Every index connected in one giant component — the entire string becomes fully sortable, the maximum possible improvement.

**⚠️ Common Mistake:** forgetting to sort the **indices** within a component before placing sorted characters back — the characters must be matched to indices in ascending index order specifically, not insertion order (which depends on iteration order over the `HashMap`, and isn't guaranteed to be ascending).

**💡 Interview Insight:** the proof of "any permutation is achievable" is worth stating out loud in condensed form — "connectivity through allowed swaps is enough to guarantee full rearrangement, so I just need to identify components, then greedily sort within each." Skipping straight to "sort within components" without that justification leaves the interviewer to wonder whether the leap was understood or just recalled.

---

## Problem 6: Accounts Merge (LeetCode 721, Medium) — Pattern: Union-Find + HashMap

**Statement:** each account is `[name, email₁, email₂, ...]`. Two accounts belong to the same person if they share **at least one email** — regardless of whether their names match (multiple different people can share a name; that's not what determines a merge). Merge every account belonging to the same person into one, with all emails deduplicated and sorted, name prepended.

**Why union *emails*, not account entries directly — this is the decision that makes the algorithm correct, not just convenient:** two accounts might share no email directly with each other, yet both share an email with a *third* account — meaning all three actually belong to the same person, transitively. Unioning account *indices* pairwise would miss that chain unless every pairwise overlap were checked explicitly (back to an O(n²) comparison). Unioning **emails** as the fundamental unit, with every email inside a single account entry unioned together, lets the *transitive* merging fall out for free — exactly the same transitive-closure benefit Union-Find has provided all week.

```java
public static List<List<String>> accountsMerge(List<List<String>> accounts) {
    Map<String, Integer> emailToId = new HashMap<>();
    Map<String, String> emailToName = new HashMap<>();
    int id = 0;

    for (List<String> account : accounts) {
        String name = account.get(0);
        for (int i = 1; i < account.size(); i++) {
            String email = account.get(i);
            emailToId.putIfAbsent(email, id++);
            emailToName.put(email, name);
        }
    }

    UnionFind uf = new UnionFind(id);
    for (List<String> account : accounts) {
        int firstEmailId = emailToId.get(account.get(1));
        for (int i = 2; i < account.size(); i++) {
            uf.union(firstEmailId, emailToId.get(account.get(i)));
        }
    }

    Map<Integer, List<String>> rootToEmails = new HashMap<>();
    for (String email : emailToId.keySet()) {
        int root = uf.find(emailToId.get(email));
        rootToEmails.computeIfAbsent(root, k -> new ArrayList<>()).add(email);
    }

    List<List<String>> result = new ArrayList<>();
    for (List<String> emails : rootToEmails.values()) {
        Collections.sort(emails);
        List<String> merged = new ArrayList<>();
        merged.add(emailToName.get(emails.get(0)));
        merged.addAll(emails);
        result.add(merged);
    }
    return result;
}
```

**🔗 Direct connection:** the final "group by root, then build one output entry per group" step is structurally identical to Day 5's canonical-key grouping pattern (grouping anagrams by a sorted-character key) — a `HashMap<Integer, List<String>>` keyed by Union-Find root plays exactly the same role a `HashMap<String, List<String>>` keyed by canonical string played there. Same shape, different key source.

### Worked Trace

```
["John", "johnsmith@mail.com", "john_newyork@mail.com"]
["John", "johnsmith@mail.com", "john00@mail.com"]
["Mary", "mary@mail.com"]
["John", "johnnybravo@mail.com"]
```

ID assignment: `johnsmith→0, john_newyork→1, john00→2, mary→3, johnnybravo→4`.

Unions: account 1 → `union(0,1)`. Account 2 → `union(0,2)` (now `0,1,2` all one component, since `1` was already unioned to `0`). Accounts 3 and 4 have only one email each — no union call happens for either (the inner loop never executes), so they each remain their own singleton component.

Grouping by root: `{johnsmith, john_newyork, john00}` (one component — all three route to the same root through the chain of unions above), `{mary}`, `{johnnybravo}`.

Sorted output for the first group: comparing `"john00@..."`, `"john_newyork@..."`, `"johnsmith@..."` character by character, the first point of difference is `'0'` vs. `'_'` vs. `'s'` — and since `'0' < '_' < 's'` in character ordering, the sorted result is exactly `["john00@mail.com", "john_newyork@mail.com", "johnsmith@mail.com"]`, prefixed with `"John"`. Matches the known correct output exactly.

**Complexity:** Time O(NK × α(NK) + NK log(NK)) — `N` accounts, average `K` emails each; building the ID maps and unions costs O(NK) work at near-constant amortized cost each; sorting each output group's emails costs, summed across all groups, O(NK log(NK)) in the worst case (all emails in one group). Space O(NK) for the maps and grouped output.

**Edge cases:** an account with a single email and no overlap with anything else — its own singleton group, output unchanged apart from the sort (trivial on one element). Two accounts under the same *name* but sharing no email at all — correctly kept as **separate** people, since the merge condition is purely email-based; the name is never used to decide connectivity, only to label the final merged group.

**⚠️ Common Mistake:** unioning by *name* instead of (or in addition to) email, which would incorrectly merge two different people who simply share a common name — the problem statement is explicit that name never determines identity, only shared emails do.

**💡 Interview Insight:** naming the transitive-chain scenario explicitly — "account A and account C might share no email directly, but if both share one with account B, all three are the same person" — before writing any code demonstrates the actual reduction was understood, not just pattern-matched from "there are groups, so Union-Find."

---

**Note on extra practice: none added today.** Yesterday carried the week's one deferred extra (Number of Operations to Make Network Connected); today's two required problems are each substantial enough, paired with a genuinely new theory thread, that padding today's schedule further would work against the realistic pacing this series has held to since Day 71.

---

# Part 2 — Observability: Prometheus and Micrometer

### Prerequisites (confirmed)

None from the AWS thread specifically — this opens a new theory sub-thread (production observability) that will continue tomorrow with Grafana.

## Prometheus: Pull-Based Metrics Collection

**What it is:** Prometheus is a time-series metrics database paired with a scraper. On a fixed schedule (commonly every 15 seconds), Prometheus itself reaches out and performs an HTTP GET against a `/metrics`-style endpoint exposed by each monitored service, reading whatever the service's current metric values are at that moment.

**"Pull-based" — the alternative it's deliberately not:** a push-based system (e.g., StatsD) has each service actively send metric updates outward to a central collector, rather than waiting to be asked. Prometheus's pull model inverts that.

**A genuine trade-off in each direction, not just "pull is better":**
- **Pull's advantage:** a failed or missed scrape is *itself* meaningful information — "this target didn't respond" is a direct, centrally-visible signal that something is wrong with the service, with no extra instrumentation needed to detect it. It also means services don't need to know anything about where to send data, or handle retries/backpressure if a collector is temporarily unavailable — Prometheus controls the schedule and absorbs that complexity centrally.
- **Push's advantage, and where pull genuinely struggles:** a short-lived batch job that starts, runs for a few seconds, and exits may never live long enough to be caught by a periodic scrape at all. Prometheus's own answer to this specific gap is a **Pushgateway** — an intermediary that short-lived jobs push their final metrics *to*, which Prometheus then scrapes from as if it were an ordinary long-running target. This is a genuine exception carved into an otherwise pull-based system, not a contradiction of it.

## Micrometer: A Facade, Not a Monitoring System

**What it is:** Micrometer is a vendor-neutral **instrumentation API** for JVM applications — you write counters, gauges, and timers once against Micrometer's interface, and plug in different **registries** as the actual backend (a Prometheus registry, a Datadog registry, a CloudWatch registry, and others) without touching the instrumentation code itself when the backend changes.

**⚠️ Common Mistake:** describing Micrometer itself as "a monitoring tool." It collects and exposes metrics through a vendor-neutral API — it doesn't store, query, or visualize anything on its own; that's the registry's (Prometheus's) job on one side, and Grafana's (tomorrow) on the other.

## Spring Boot Actuator's `/actuator/prometheus` Endpoint

Once the `micrometer-registry-prometheus` dependency is on the classpath, Spring Boot Actuator automatically wires Micrometer's collected metrics into a `/actuator/prometheus` endpoint, formatted in Prometheus's specific plain-text exposition format — one line per metric series, e.g. `http_server_requests_seconds_count{method="GET",uri="/orders",status="200"} 42.0`. Prometheus is then configured to scrape exactly this endpoint on its normal schedule, and from that point on, every metric emitted by the application flows into Prometheus without any further application-side work.

**🔑 Key Takeaway:** the whole chain is deliberately decoupled at each seam — the application only knows Micrometer's API; Micrometer only knows how to format for whichever registry is configured; Prometheus only knows to scrape a URL on a schedule. No layer needs to know anything about the layers beyond its immediate neighbor.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`.

**Task:** add the Actuator and Micrometer Prometheus registry dependencies, and expose `/actuator/prometheus` across the platform's modules.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus, health
  metrics:
    tags:
      application: ${spring.application.name}
```

The `application` tag on every metric is worth including deliberately: with multiple modules (Order, Payment, Inventory, Gateway) all exposing `/actuator/prometheus`, this tag is what lets Prometheus (and, tomorrow, Grafana) distinguish which module a given metric series actually came from once they're all being scraped into the same time-series database.

**Definition of done:** the endpoint returns real metrics in Prometheus's text format for each module, each one correctly tagged with its own `application` name.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** no specific outreach today — a deliberately lighter career load, appropriate given today's heavier DSA + project combination.

---

## Day 76 — Interview Questions

**Q1. Prove that a connected component of swappable indices admits any permutation, not just state it.**
*A:* For any two positions in the component, a path of allowed swaps connects them; performing that path's swaps in sequence moves a character all the way from one end to the other. Repeating this, one character at a time, realizes any target arrangement — connectivity through allowed swaps is what guarantees full rearrangement.

**Q2. Why does Smallest String With Swaps sort characters within each component independently, rather than sorting the whole string at once?**
*A:* Only characters within the same connected component can ever reach each other's positions — characters in different components are never interchangeable, so each component's achievable arrangements are entirely independent of every other component's.

**Q3. Why does Accounts Merge union emails, not account entries?**
*A:* Two accounts might share no email directly but both share one with a third account, meaning all three belong to the same person transitively. Unioning emails as the fundamental unit captures that chain automatically; unioning accounts pairwise would require checking every pair's overlap explicitly.

**Q4. Why is name never used to determine which accounts merge?**
*A:* The problem defines identity purely by shared email — two different people can share a name, and merging on name would incorrectly combine them. Name is only used to label the final merged group, never to decide connectivity.

**Q5. What does "pull-based" mean for Prometheus, precisely?**
*A:* Prometheus itself initiates an HTTP request against each monitored service's metrics endpoint on a fixed schedule, rather than services pushing their own metric updates outward to a central collector.

**Q6. Name a genuine advantage of push-based metrics collection over pull, and Prometheus's own answer to that gap.**
*A:* A short-lived batch job may not exist long enough to be caught by a periodic scrape. Prometheus's Pushgateway lets such jobs push their final metrics to an intermediary, which Prometheus then scrapes from normally.

**Q7. Is Micrometer a monitoring system? If not, what is it?**
*A:* No — it's a vendor-neutral instrumentation API. It doesn't store, query, or visualize metrics itself; it lets application code emit counters/gauges/timers once and plug in different backend registries without changing that instrumentation code.

**Q8. What format does `/actuator/prometheus` expose metrics in, and why does that matter?**
*A:* Prometheus's plain-text exposition format — one line per metric series with labels and a value. It matters because it's exactly what Prometheus's scraper expects; no translation layer is needed between the application and the metrics database.

**Q9. Why tag metrics with the application/module name in a multi-module platform?**
*A:* Once multiple modules all expose `/actuator/prometheus` and get scraped into the same Prometheus instance, the tag is what distinguishes which module a given metric series came from — without it, identical metric names from different modules would be indistinguishable.

**Q10. What's the shared reasoning connecting today's two DSA problems?**
*A:* Both union an abstraction chosen to make connectivity visible — index positions (not raw characters) in one, emails (not account entries) in the other — and both then group by resulting root to derive the final answer, the same "union the right thing, then group by root" shape from two different problem surfaces.

---

## Daily Deliverable Check

- [ ] Smallest String With Swaps and Accounts Merge solved, pushed — Union-Find ladder now at 6/7 required.
- [ ] Can prove (not just state) why a connected component supports arbitrary rearrangement.
- [ ] Can explain why Accounts Merge unions emails rather than accounts, with the transitive-chain scenario as justification.
- [ ] `/actuator/prometheus` live with real, correctly-tagged metrics across all platform modules.

---

## What Tomorrow Assumes You Already Know Cold

Day 77 closes Union-Find with one final problem, then pivots to Minimum Spanning Trees — which reuses the `UnionFind` class directly as a cycle-detection subroutine inside a new algorithm (Kruskal's), rather than teaching a new structure from scratch. Today's fluency with the class as a settled, unremarkable tool is exactly what tomorrow assumes; if `find`/`union` still need conscious thought to call correctly, that's worth a quick reread of Day 74 before continuing. Tomorrow's Prometheus→Grafana connection also assumes today's exposition-format detail is solid, since Grafana queries exactly what Prometheus scraped from here.

**Next:** [Day 77 Resource Book](Day77_Resource_Book.md) — Union-Find Capstone, Minimum Spanning Trees, and Week 11 Consolidation.
