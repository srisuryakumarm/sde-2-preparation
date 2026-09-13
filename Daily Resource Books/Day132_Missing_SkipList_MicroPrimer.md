# Micro-Primer: Skip Lists — Needed Before Day 132

**First needed:** Day 132, which asks you to explain "why Skip Lists beat B-trees for this specific workload" — a question that only makes sense if you already know what a Skip List is. It never appears anywhere in the 105-day DSA curriculum.

A **Skip List** is a probabilistic data structure: several "levels" of linked lists stacked on top of one normal sorted linked list, where each higher level skips over more elements than the one below it — like an express lane that only stops every 4th or 8th node, letting you cover long distances quickly before dropping down to a slower, more precise lane near your actual target.

When an element is inserted, it gets randomly promoted to higher levels with some fixed probability (commonly 50%). That randomness is what keeps the structure roughly balanced on average, without needing the strict, deterministic rebalancing rules a tree structure (like a Red-Black tree) requires.

This gives **expected O(log n)** search/insert/delete — competitive with a balanced tree, but with simpler, more lock-friendly implementation logic. That's exactly why Redis uses a Skip List underneath its Sorted Set type, and why it handles Day 132's constant real-time score updates more gracefully than a B-tree index would: a B-tree's rebalancing under frequent writes is comparatively more disruptive to concurrent readers.

**Deferred:** nothing further. This isn't part of the DSA problem curriculum, and no later day asks you to implement one — this is "know what it is and why it's fast," matching the same deliberately light depth the plan already gives Segment Trees.

### Checklist
- [ ] Can describe, in a sentence or two, how a Skip List's levels work.
- [ ] Can explain why it competes with a balanced tree on complexity, without being one.
- [ ] Ready for Day 132.
