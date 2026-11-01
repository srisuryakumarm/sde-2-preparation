# Day 18 — Missing Primer: Iterators, Fail-Fast Behaviour, and the Rest of the Collections Toolkit

**Do this on Day 18 (≈ 25 min, right after the Exception Handling theory block), then continue with Day 18's regular plan.**
**Needed by:** Day 29–39 (`ConcurrentHashMap` and friends are "fail-safe" — you need the word it is contrasted with), Day 39 (LRU Cache — the `LinkedHashMap` shortcut is the usual interviewer follow-up), Days 54–58 (`PriorityQueue` is a heap — its iteration order surprises people), Day 114 (`TreeSet.higher/lower` in the Elevator), and the Java question "why do I get `ConcurrentModificationException`, and what is fail-fast vs fail-safe?"
**Why it is missing:** Day 12 teaches `ArrayList`/`LinkedList`/`ArrayDeque`, Day 15 `HashMap`, Day 17 names `TreeMap`/`PriorityQueue` — but `ConcurrentModificationException`, "fail-fast", `Iterator.remove()` and `LinkedHashMap` appear in none of the 148 books. `removeIf` and the immutable-view traps (`List.of` vs `unmodifiableList` vs `Arrays.asList`) are never contrasted.
**Already in place:** `ArrayList`/`LinkedList`/`ArrayDeque` internals (Day 12), `HashMap` (Day 15), generics (Day 16), `Comparable`/`Comparator` (Day 17), exceptions (today's theory block — do this *after* it).

---

## 1. What a for-each loop really is
`for (Integer n : list) { … }` is compiled into an **iterator loop**:
```java
for (Iterator<Integer> it = list.iterator(); it.hasNext(); ) {
    Integer n = it.next();
    …
}
```
Anything that implements `Iterable<T>` (one method: `Iterator<T> iterator()`) works in a for-each. An `Iterator` has `hasNext()`, `next()` and an optional `remove()` that deletes **the element `next()` just returned**. (Arrays are the exception: the compiler uses a plain index loop.)

## 2. Fail-fast: the exception you will meet within a week
Each `ArrayList`/`HashMap` keeps a counter, `modCount`, that goes up on every **structural** change (add, remove, resize — not `set`). An iterator remembers the value it started with and checks it on every `next()`. If they differ, it throws `ConcurrentModificationException` (CME).

```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4));
for (Integer n : nums) {
    if (n == 2) nums.remove(n);        // structural change behind the iterator's back → CME on the next next()
}
```
Three correct fixes (all verified):
```java
for (Iterator<Integer> it = nums.iterator(); it.hasNext(); )
    if (it.next() % 2 == 0) it.remove();           // 1. the iterator removes (updates its own bookkeeping)

nums.removeIf(n -> n % 2 == 0);                    // 2. the cleanest — one call, one pass

for (int i = nums.size() - 1; i >= 0; i--)         // 3. walk backwards by index
    if (nums.get(i) % 2 == 0) nums.remove(i);
```
Three things interviewers like you to know:
- **The quirk:** removing the *second-to-last* element inside a for-each does **not** throw (after the removal `hasNext()` is already false, so `next()` is never called). Fail-fast is a *best-effort bug detector*, not a guarantee — never write code that relies on it.
- **`remove(int)` vs `remove(Object)`:** on a `List<Integer>`, `list.remove(2)` removes the element at **index** 2; `list.remove(Integer.valueOf(2))` removes the **value** 2. (In the loop above, `n` is an `Integer`, so `remove(n)` is the by-value overload.)
- **Not only threads:** CME has nothing to do with multithreading in the single-threaded case above. With threads the same check happens, but it is not thread-safe — it only *sometimes* notices.

## 3. Fail-safe (a misleading but common name)
The concurrent collections do not throw CME; their iterators are built differently:

| Collection | What the iterator sees | Cost |
|---|---|---|
| `ConcurrentHashMap` | **Weakly consistent:** safe to iterate while others modify; may or may not reflect changes made after it was created | none extra |
| `CopyOnWriteArrayList` / `CopyOnWriteArraySet` | A **snapshot** taken when the iterator was created; never reflects later changes | every write copies the whole array — O(n) — fine for rarely-changed listener lists, terrible for hot lists |

"Fail-safe" is the usual interview wording; the JDK docs say "weakly consistent" or "snapshot".

## 4. Views vs copies — four ways to get a list you cannot break
```java
List<String> backing = new ArrayList<>(List.of("x", "y"));
List<String> view = Collections.unmodifiableList(backing);   // read-only VIEW: sees later changes to backing
List<String> copy = List.copyOf(backing);                    // immutable COPY: frozen
backing.add("z");     // view = [x, y, z]   copy = [x, y]
```
| Expression | `add`/`remove` | `set` | `null` elements | Notes |
|---|---|---|---|---|
| `List.of(…)` / `List.copyOf(…)` | ✗ `UnsupportedOperationException` | ✗ | ✗ `NullPointerException` | truly immutable |
| `Collections.unmodifiableList(l)` | ✗ | ✗ | ✓ | a **view** — whoever holds `l` can still change it |
| `Arrays.asList(a, b, c)` | ✗ | ✓ (writes through to the array) | ✓ | fixed-size view of an array |
| `new ArrayList<>(other)` | ✓ | ✓ | ✓ | independent copy |

Rule: **return copies or `List.copyOf` from methods that expose internal state** (Day 5's encapsulation, applied to collections).

## 5. The rest of the toolkit in one table
| Type | Remember | Typical use |
|---|---|---|
| `HashSet` | a `HashMap` whose keys are your elements (values are a dummy) | membership, de-duplication |
| `LinkedHashMap` / `LinkedHashSet` | `HashMap` + a linked list through the entries: **insertion order**, or **access order** if you ask | LRU cache, ordered output |
| `TreeMap` / `TreeSet` | red-black tree (self-balancing BST), O(log n); sorted by `Comparable`/`Comparator`; `floorKey`, `ceilingKey`, `lowerKey`, `higherKey`, `headMap`, `tailMap` | "the next larger/smaller", range queries, ordered sets |
| `PriorityQueue` | a binary **heap stored in an array**: `offer`/`poll` O(log n), `peek` O(1), `remove(Object)` O(n). **`toString()` and iteration are NOT sorted** — only `poll()` is | top-K, scheduling |
| `ArrayDeque` | circular array; use it instead of `Stack` (legacy, synchronized) and usually instead of `LinkedList` | stack, queue, deque |

Verified on a real run: `new PriorityQueue<>(List.of(5,1,4,2,3))` prints `[1, 2, 4, 5, 3]`, but polling gives `1 2 3 4 5`. And `treeMap.floorKey(25)` on keys {10,20,30} is `20`, `ceilingKey(25)` is `30`.

## 6. LRU cache in the JDK's own words (the 8-line version)
```java
class LruCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    LruCache(int capacity) {
        super(16, 0.75f, true);                 // true = ACCESS order (get/put move the entry to the end)
        this.capacity = capacity;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;               // called after every put; true → evict the eldest
    }
}
```
Put `a`, `b`, `get(a)`, put `c` with capacity 2 → keys are `[a, c]` (`b` was least recently used). In an interview, **build the doubly-linked-list + `HashMap` version by hand (Day 39) and mention this one as the library shortcut** — and that it is not thread-safe.

## 7. Exercise (≈ 15 min, repo `java-fundamentals`)
1. Reproduce the CME with the loop in §2; then fix it all three ways. Reproduce the "second-to-last" quirk and write one sentence on why it happens.
2. Build `view`, `copy` and `Arrays.asList` from §4; try `add` and `set` on each and record which throw.
3. Copy `LruCache`; write a test: capacity 2, `put a`, `put b`, `get a`, `put c` → assert keys are `[a, c]`.
4. Print a `PriorityQueue` built from `[5,1,4,2,3]` and then poll it into a list. Explain the difference in one sentence.

## Checklist
- [ ] Can explain what `modCount` is and why removing inside a for-each throws (and the second-to-last quirk).
- [ ] Can fix a removal-while-iterating bug three ways and pick `removeIf`.
- [ ] Can say fail-fast vs weakly consistent vs snapshot, and give one collection for each.
- [ ] Can say `List.of` vs `unmodifiableList` vs `Arrays.asList` vs a copy.
- [ ] Can write the `LinkedHashMap` LRU and say why you would still hand-build it in an interview. Continue with Day 18's regular plan.
