# Day 112 — LLD #4: Library Management System, and Mock Interview #2

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 111 Resource Book](Day111_Resource_Book.md)
**Next ▶:** [Day 113 Resource Book](Day113_Resource_Book.md)
**Companion to:** Day 112 of `Week_16_Revised.md`

---

## Recap

The week's fourth and final new system. Parking Lot proved a design can correctly need no pattern, twice over (Day 108's Tic-Tac-Toe, and Parking Lot's own Step 4 on Day 110); Vending Machine proved State when it's genuinely earned. Today's Library Management System adds a different, equally real design skill: **cleanly separating two concerns that a rushed design would happily tangle together** — searching a catalog, and managing what happens to a physical copy once it's found. It's also a shorter day by design (Sunday) — a self-check instead of new theory, one system instead of a system-plus-concept pairing, and the week's second mock interview closing out the week's live-practice count at two.

---

## Learning Objectives

By the end of today, without notes:

1. Recite the 5-step LLD framework from memory, unprompted.
2. Design and implement a Library Management System where `Catalog` (search) has zero dependency on reservation/checkout concepts, and prove that separation concretely, not just assert it.
3. Explain why `BookItem`'s status lifecycle is a data-driven validity check rather than a full State-pattern implementation, and state the concrete difference from Vending Machine that justifies the lighter treatment.
4. Run and debrief Mock Interview #2, rehearsing the "now make it thread-safe" mid-interview escalation live.

---

## Concept Dependency Map

```
Day 106-111: full framework, all ten patterns, TDD, Coupling/Cohesion/LoD,
             Composition over Inheritance, concurrency granularity — all established
Day 109: State pattern, genuinely earned (Vending Machine) — today's contrast point
Day 110: data-driven rules over scattered conditionals (VehicleSize ordinals) —
         directly reused today for BookItem's transition table
        │
        ▼
Today: Library Management System
  Book (catalog entry) vs. BookItem (physical copy) — a NEW distinction,
  needs: nothing beyond basic class design already established
  Catalog (search only) ←── zero dependency ──── Library (search + circulation)
  BookItem lifecycle: data-driven transition table (Day 110's technique, reused,
  not State-pattern class-per-status — justified directly against Day 109)
        │
        ▼
Mock Interview #2 — Parking Lot + concurrency, live, with a forced
"now make it thread-safe" escalation partway through
        │
        ▼
🔗 Forward: Week 17 opens with ATM (Day 113) — ChainOfResponsibility joins the
   pattern catalog for the first time; today's separation-of-concerns instinct
   (Catalog vs. circulation) is exactly what ATM's own command/state layering needs.
```

---

# Part 1 — Self-Check: The 5-Step Framework, From Memory (15 min)

No re-explanation here — Day 106 already gave the full reasoning for each step, and by now this needs to be recall, not reading. Say each one out loud, in order, before checking below:

1. Clarify requirements — ?
2. Identify core objects — ?
3. Define relationships, sketch a class diagram — ?
4. Apply patterns deliberately — ?
5. Code the core — ?

**If any step needed more than a few seconds to name, or came out of order:** back to Day 106 before continuing — everything from here forward assumes this is fully automatic, the way three weeks of Two Pointers practice made that pattern automatic back in Week 1.

---

# Part 2 — LLD #4: Library Management System

### Steps 1–4, Briskly

**Requirements, clarified:** a library catalogs books searchable by title, author, and subject; each catalog entry (`Book`) may have multiple physical copies (`BookItem`), each independently trackable — a library owning 3 copies of the same ISBN needs each copy's own lifecycle, not one shared status for the title as a whole. Members can reserve an item, then check it out; direct checkout without a prior reservation is also allowed. A librarian can mark a copy lost. Fines, due dates, and multi-branch routing are explicitly out of scope today.

**Core objects:** `Book` (ISBN, title, author, subject — the catalog-level concept), `BookItem` (barcode, a reference to its `Book`, and its own status — the physical-copy-level concept), `Member`, `Reservation`, and two collaborating classes handling the two halves of the system: `Catalog` (search, and only search) and `Library` (composes a `Catalog`, and separately owns reservation/checkout/return).

**Why `Book` and `BookItem` are two classes, not one:** a `Book` represents *the title* — one row for "Clean Code" regardless of how many physical copies exist. A `BookItem` represents *one physical, borrowable copy* — if the library owns three copies, that's one `Book` and three `BookItem`s, each independently `AVAILABLE`, `LOANED`, or `LOST` at any given moment. Collapsing these into one class would force a choice that doesn't actually exist in reality: is a book with 2 of 3 copies checked out "loaned" or "available"? Both, depending on *which copy* — which is precisely why the status belongs on `BookItem`, not `Book`.

**Pattern, deliberately — and deliberately lighter than Day 109's State:** `BookItem`'s status lifecycle *could* be modeled as a full State pattern (a `BookItemState` interface with `AvailableState`, `ReservedState`, `LoanedState`, `LostState` classes). **It isn't, here, and that's a real design decision, not an oversight.** Day 109's Vending Machine states each responded *differently* to the *same* external triggers (`insertCoin` did genuinely different things in `NoCoinState` versus `HasCoinState` versus `SoldOutState`) — real, varying behavior per state. `BookItem`'s states don't vary behavior this way; the only thing that changes per status is **which transitions are valid from here**, which is a validity check, not divergent behavior. That's a data problem (Day 110's exact lesson: express the rule as data, not scattered code), not a behavioral one — so it's built as a transition table, below, not four extra classes.

### Step 5 — Code the Core

```java
public class Book {
    private final String isbn;
    private final String title;
    private final String author;
    private final String subject;

    public Book(String isbn, String title, String author, String subject) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
        this.subject = subject;
    }

    public String getIsbn() { return isbn; }
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    public String getSubject() { return subject; }
}

public enum BookItemStatus {
    AVAILABLE, RESERVED, LOANED, LOST
}

public class BookItem {
    private final String barcode;
    private final Book book;
    private BookItemStatus status = BookItemStatus.AVAILABLE;

    public BookItem(String barcode, Book book) {
        this.barcode = barcode;
        this.book = book;
    }

    public String getBarcode() { return barcode; }
    public Book getBook() { return book; }
    public BookItemStatus getStatus() { return status; }
    void setStatus(BookItemStatus status) { this.status = status; }   // package-private — only Library drives transitions
}

public class Member {
    private final String memberId;
    private final String name;

    public Member(String memberId, String name) {
        this.memberId = memberId;
        this.name = name;
    }

    public String getMemberId() { return memberId; }
    public String getName() { return name; }
}

public class Reservation {
    private final String reservationId;
    private final Member member;
    private final BookItem bookItem;
    private final LocalDateTime reservedAt;

    public Reservation(String reservationId, Member member, BookItem bookItem, LocalDateTime reservedAt) {
        this.reservationId = reservationId;
        this.member = member;
        this.bookItem = bookItem;
        this.reservedAt = reservedAt;
    }

    public String getReservationId() { return reservationId; }
    public Member getMember() { return member; }
    public BookItem getBookItem() { return bookItem; }
    public LocalDateTime getReservedAt() { return reservedAt; }
}
```

```java
// SEARCH ONLY. Notice everything this class does NOT reference: Member, Reservation,
// BookItemStatus transitions, checkout, or return — that omission IS the separation.
public class Catalog {
    private final Map<String, List<BookItem>> itemsByIsbn = new HashMap<>();
    private final Map<String, List<Book>> booksByTitle = new HashMap<>();
    private final Map<String, List<Book>> booksByAuthor = new HashMap<>();
    private final Map<String, List<Book>> booksBySubject = new HashMap<>();

    public void addBookItem(BookItem item) {
        Book book = item.getBook();

        List<BookItem> itemsForIsbn = itemsByIsbn.get(book.getIsbn());
        if (itemsForIsbn == null) {
            itemsForIsbn = new ArrayList<>();
            itemsByIsbn.put(book.getIsbn(), itemsForIsbn);
        }
        itemsForIsbn.add(item);

        if (itemsForIsbn.size() == 1) {   // first copy of this ISBN — index the Book itself, once
            indexBook(booksByTitle, book.getTitle(), book);
            indexBook(booksByAuthor, book.getAuthor(), book);
            indexBook(booksBySubject, book.getSubject(), book);
        }
    }

    private void indexBook(Map<String, List<Book>> index, String key, Book book) {
        List<Book> books = index.get(key);
        if (books == null) {
            books = new ArrayList<>();
            index.put(key, books);
        }
        books.add(book);
    }

    public List<Book> searchByTitle(String title) {
        List<Book> result = booksByTitle.get(title);
        return result != null ? result : new ArrayList<>();
    }

    public List<Book> searchByAuthor(String author) {
        List<Book> result = booksByAuthor.get(author);
        return result != null ? result : new ArrayList<>();
    }

    public List<Book> searchBySubject(String subject) {
        List<Book> result = booksBySubject.get(subject);
        return result != null ? result : new ArrayList<>();
    }

    public List<BookItem> getItemsForIsbn(String isbn) {
        List<BookItem> result = itemsByIsbn.get(isbn);
        return result != null ? result : new ArrayList<>();
    }
}
```

```java
public class Library {
    private final Catalog catalog = new Catalog();
    private final Map<String, Reservation> activeReservations = new HashMap<>();
    private int nextReservationId = 1;

    private static final Map<BookItemStatus, Set<BookItemStatus>> VALID_TRANSITIONS = buildTransitions();

    private static Map<BookItemStatus, Set<BookItemStatus>> buildTransitions() {
        Map<BookItemStatus, Set<BookItemStatus>> transitions = new HashMap<>();
        transitions.put(BookItemStatus.AVAILABLE, new HashSet<>(List.of(BookItemStatus.RESERVED, BookItemStatus.LOANED, BookItemStatus.LOST)));
        transitions.put(BookItemStatus.RESERVED, new HashSet<>(List.of(BookItemStatus.LOANED, BookItemStatus.AVAILABLE)));
        transitions.put(BookItemStatus.LOANED, new HashSet<>(List.of(BookItemStatus.AVAILABLE, BookItemStatus.LOST)));
        transitions.put(BookItemStatus.LOST, new HashSet<>());   // terminal — no valid transitions out
        return transitions;
    }

    private void transition(BookItem item, BookItemStatus newStatus) {
        if (!VALID_TRANSITIONS.get(item.getStatus()).contains(newStatus)) {
            throw new IllegalStateException("Cannot transition BookItem " + item.getBarcode()
                    + " from " + item.getStatus() + " to " + newStatus);
        }
        item.setStatus(newStatus);
    }

    public void addBookItem(BookItem item) {
        catalog.addBookItem(item);
    }

    // Pure pass-throughs — Library adds NOTHING to the search logic itself. This is
    // the "cleanly separated" requirement, made checkable: read these three lines
    // and confirm there is no reservation-aware filtering hiding inside them.
    public List<Book> searchByTitle(String title) { return catalog.searchByTitle(title); }
    public List<Book> searchByAuthor(String author) { return catalog.searchByAuthor(author); }
    public List<Book> searchBySubject(String subject) { return catalog.searchBySubject(subject); }

    // A query that genuinely NEEDS both concerns — composed here, at the Library
    // layer, rather than smearing status-awareness into Catalog to support it.
    public List<BookItem> searchAvailableCopiesByTitle(String title) {
        List<BookItem> availableItems = new ArrayList<>();
        for (Book book : catalog.searchByTitle(title)) {
            for (BookItem item : catalog.getItemsForIsbn(book.getIsbn())) {
                if (item.getStatus() == BookItemStatus.AVAILABLE) {
                    availableItems.add(item);
                }
            }
        }
        return availableItems;
    }

    public Reservation reserve(Member member, BookItem item) {
        transition(item, BookItemStatus.RESERVED);
        String reservationId = "R-" + (nextReservationId++);
        Reservation reservation = new Reservation(reservationId, member, item, LocalDateTime.now());
        activeReservations.put(reservationId, reservation);
        return reservation;
    }

    public void checkout(String reservationId) {
        Reservation reservation = activeReservations.get(reservationId);
        if (reservation == null) {
            throw new IllegalArgumentException("No such reservation: " + reservationId);
        }
        transition(reservation.getBookItem(), BookItemStatus.LOANED);
        activeReservations.remove(reservationId);
    }

    public void checkoutDirectly(BookItem item) {
        transition(item, BookItemStatus.LOANED);
    }

    public void returnItem(BookItem item) {
        transition(item, BookItemStatus.AVAILABLE);
    }

    public void reportLost(BookItem item) {
        transition(item, BookItemStatus.LOST);
    }
}
```

### Worked Trace — Lifecycle, Search, and the Separation, All Proven Together

```
Book: "Clean Code" (ISBN 978-1), copies BC-1 and BC-2, both start AVAILABLE.

searchAvailableCopiesByTitle("Clean Code")  →  [BC-1, BC-2]   (both available)

reserve(alice, BC-1)     → BC-1: AVAILABLE → RESERVED.        Reservation R-1 issued.
checkout("R-1")          → BC-1: RESERVED → LOANED.           R-1 removed from active reservations.
checkoutDirectly(BC-2)   → BC-2: AVAILABLE → LOANED.

checkoutDirectly(BC-1)   → transition(BC-1, LOANED) — VALID_TRANSITIONS.get(LOANED) = {AVAILABLE, LOST}
                            does not contain LOANED → throws IllegalStateException. Correctly rejected:
                            BC-1 is already out; a second checkout attempt on it is not a valid transition.

searchAvailableCopiesByTitle("Clean Code")  →  []              (both copies now loaned — proven, not assumed)
searchByTitle("Clean Code")                 →  [Book("Clean Code", ...)]   (STILL found — the Book itself
                                                 never left the catalog; only its copies' availability changed)

returnItem(BC-1)          → BC-1: LOANED → AVAILABLE.
searchAvailableCopiesByTitle("Clean Code")  →  [BC-1]           (back to reflecting reality immediately)
```

**What this trace actually proves, precisely:** the second-to-last block is the load-bearing one — `searchByTitle` still finds the book even when *every copy* is checked out, while `searchAvailableCopiesByTitle` correctly returns nothing. If `Catalog` had any reservation-awareness baked in, these two queries would risk becoming inconsistent with each other in ways that are hard to reason about; keeping them genuinely separate, composed only at the `Library` layer, is exactly what keeps both queries independently correct and independently testable.

**Complexity:** `searchByTitle`/`Author`/`Subject` are `O(1)` average-case hash lookups plus `O(k)` to return `k` matching books. `searchAvailableCopiesByTitle` is `O(k + m)`, where `k` is matching books and `m` is total copies across them — it must inspect every copy's status, since availability isn't itself indexed. `reserve`/`checkout`/`returnItem`/`reportLost` are all `O(1)` — a map lookup plus a constant-size transition-table check.

> ⚠️ **Common Mistake:** adding a `boolean isAvailable` convenience field directly onto `Book` "for speed," updated whenever any of its copies' statuses change. This reintroduces exactly the tangling this design exists to avoid — `Catalog`, which owns `Book`, would now need to know about `BookItemStatus` and be notified of every status change happening in `Library`'s circulation logic, exactly the dependency direction Step 1's separation was meant to prevent. `searchAvailableCopiesByTitle` computing availability on demand, from `BookItem`'s own source-of-truth status, is slightly more work per call and considerably cleaner to reason about — worth being able to name that trade-off directly if asked why it wasn't cached.

---

# Part 3 — Mock Interview #2

**Format:** 45 minutes, same accountability partner. Subject: **Parking Lot, including concurrency** — the interviewer should let the candidate design and code the single-threaded core first, then, **partway through, explicitly interrupt with "now make it thread-safe."** This isn't a curveball; it's rehearsal for exactly the escalation Day 111's career block flagged as a real, reported pattern in Rippling's and Uber's actual onsite loops.

**Suggested time allocation:**

- **~5 min — Requirements.**
- **~8 min — Objects and class diagram.**
- **~12 min — Single-threaded code.**
- **[Interviewer interrupts: "now make it thread-safe."]**
- **~15 min — Adapting live:** identifying the race, choosing a locking granularity, justifying the choice out loud.
- **~5 min — Wrap-up and feedback.**

**A genuinely useful piece of live-interview strategy, worth stating directly:** when the thread-safety push lands, **reaching for the simple, obviously-correct option first — a single lock around the whole operation — and stating it plainly, then explicitly proposing the finer-grained refinement and why it's better, often reads better than jumping straight to the most optimized version without narrating the path there.** An interviewer watching someone silently produce the "correct" fine-grained answer learns less about how that person reasons than watching them state the safe baseline, then improve on it out loud. This is worth actually practicing today, not just reading — narrate both steps in the mock.

**Self-debrief, afterward:**
- Did the thread-safety escalation get met with a clear, narrated baseline-then-refine, or a scramble?
- Was the specific race condition (check-then-act) named explicitly, the way Day 111 named it, or only vaguely gestured at ("we need to synchronize this")?
- Did the granularity choice get justified with a *reason* (what's actually shared, what isn't), or just asserted?

---

# Project Block Guide (2.5 hrs)

**Repository:** `lld-java`. **Module:** new — `library-management/`.

**Task:** the full implementation above — `Book`, `BookItemStatus`, `BookItem`, `Member`, `Reservation`, `Catalog`, `Library`.

**Definition of done:**
- `BookItem` lifecycle correct: `AVAILABLE → RESERVED/LOANED → AVAILABLE`, plus `→ LOST` from any non-terminal status, verified with unit tests covering at least one valid full cycle and at least one rejected invalid transition (mirroring the `checkoutDirectly(BC-1)`-while-already-loaned case traced above).
- `Catalog` provably has zero dependency on reservation logic — confirm by checking its imports/fields directly: no `Member`, no `Reservation`, no `BookItemStatus`-transition logic anywhere in the class.
- `searchAvailableCopiesByTitle` (or equivalent) demonstrates the two concerns composing correctly without merging.
- Pushed to `lld-java/library-management/`.

---

# Career Block Guide (1 hr)

**Weekly Industry Awareness Ritual.** Read today's TLDR Newsletter issue, and read one full engineering blog post from a target company — Rippling, Google, Databricks, Stripe, Uber, Atlassian, or a similar tier-1 engineering blog. The point of this ritual isn't information for its own sake; it's staying fluent in how real engineering organizations talk about the trade-offs this exact plan keeps practicing — reading a real post about, say, a company's own approach to locking granularity or state-machine design lands very differently after today than it would have five weeks ago.

**Weekly Scorecard — Week 16.**

| | |
|---|---|
| LLD systems built | 4 — Tic-Tac-Toe, Vending Machine, Parking Lot (single-threaded + concurrent), Library Management |
| Mock interviews completed & debriefed | 2 — Tic-Tac-Toe (Day 109), Parking Lot + concurrency (today) |
| Patterns covered, full depth | 10 — Adapter, Decorator, Facade, Proxy, Composite, Observer, Strategy, State, Command, Template Method |
| Process/design theory covered | 4 — TDD (Red-Green-Refactor), Coupling/Cohesion/Law of Demeter, Composition over Inheritance, lock-granularity trade-offs |
| DSA revision problems, cold-solved | 6 — Course Schedule, Coin Change, Combination Sum, Longest Substring Without Repeating Characters, Validate BST, Redundant Connection |
| Resume | Updated to reflect current platform architecture and LLD systems built |
| Networking | Targeted ramp begun across 7 companies; 2–3 references reconnected |
| Systems remaining | 6 — ATM, Elevator, Splitwise, BookMyShow, Food Delivery, Hotel Booking (Week 17) |

Applications begin next week, alongside the remaining six systems — worth having this table as the concrete "here's what three weeks of consistent work actually adds up to" artifact, not just a mental sense of progress.

---

# Day 112 — Interview Questions

**Q1. Why are `Book` and `BookItem` two separate classes rather than one?** `Book` represents the catalog-level title, shared across every physical copy; `BookItem` represents one specific, independently-trackable physical copy. A library with 3 copies of one ISBN needs 3 independent statuses, not one shared status for the title — collapsing them into one class would make that impossible to represent correctly.

**Q2. Why is `BookItem`'s lifecycle a transition table rather than a full State pattern, when Vending Machine used State directly?** Vending Machine's states responded *differently* to the *same* triggers — genuinely varying behavior per state. `BookItem`'s statuses don't vary behavior this way; the only thing that changes per status is which transitions are valid, which is a data/validity problem, not a behavioral one — so it's expressed as data (Day 110's technique), not as four additional classes.

**Q3. How does `Catalog` provably have zero dependency on reservation logic, rather than just being described that way?** Its fields and imports contain no reference to `Member`, `Reservation`, or `BookItemStatus`-transition logic at all — the claim is checkable directly by reading the class, not something that has to be taken on faith.

**Q4. `searchAvailableCopiesByTitle` needs both catalog search and status information. Where does that composition happen, and why there specifically?** In `Library`, not `Catalog` — `Library` calls `catalog.searchByTitle`, then separately filters by each `BookItem`'s status. Composing at `Library` keeps `Catalog` itself entirely unaware that availability filtering exists, preserving the one-directional dependency instead of teaching `Catalog` about reservation state to support one specific query.

**Q5. Why is caching a `boolean isAvailable` flag directly on `Book`, updated on every status change, a design mistake here — not just a minor inefficiency?** It would force `Catalog` (which owns `Book`) to become aware of `BookItemStatus` changes happening inside `Library`'s circulation logic — reintroducing exactly the dependency direction the separation was built to prevent, in exchange for an optimization the actual access pattern doesn't clearly need.

**Q6. In Mock Interview #2's escalation, why is narrating a coarse-but-correct lock first, then proposing a refinement, often a stronger live-interview move than jumping straight to the optimized answer?** It demonstrates the reasoning process, not just the destination — an interviewer watching someone silently produce a fine-grained answer learns little about how they think; watching them state a safe baseline and then justify improving on it shows the actual judgment being evaluated.

**Q7. What specifically should be named, out loud, when the "now make it thread-safe" escalation lands — not just "we need to synchronize this"?** The precise check-then-act race (which two steps are non-atomic, and how two threads could interleave through the gap), followed by a specific granularity choice with a stated reason — matching Day 111's own standard, not a vague gesture toward "adding some locking."

---

## Daily Deliverable Check

- [ ] 5-step LLD framework recited from memory, unprompted.
- [ ] Library Management System complete: lifecycle correct, catalog/circulation separation verified.
- [ ] Mock Interview #2 completed and debriefed, including the thread-safety escalation.
- [ ] Weekly Industry Awareness Ritual completed.
- [ ] Weekly Scorecard reviewed.

---

## What Tomorrow Assumes You Already Know Cold

Day 113 opens Week 17 and ATM — the first system to bring in Chain of Responsibility, a pattern not yet covered, alongside State (now well-established from Day 109's Vending Machine). It assumes all ten of this week's patterns, the 5-step framework, and — just as much — the judgment calls this week kept making explicit (when a pattern is earned versus forced, when a lighter data-driven rule beats a full pattern, how to separate two concerns cleanly) are fully load-bearing going forward, with no further recap of any of them.

---

# Week 16 Consolidation

**What actually got built:** four complete LLD systems (Tic-Tac-Toe, Vending Machine, Parking Lot single-threaded, Parking Lot concurrent), ten design patterns at full depth (five Structural: Adapter, Decorator, Facade, Proxy, Composite; five Behavioral: Observer, Strategy, State, Command, Template Method), and four process/design topics (TDD's Red-Green-Refactor, Coupling/Cohesion/Law of Demeter, Composition over Inheritance, lock-granularity trade-offs). Two mock interviews run and debriefed. Resume updated; networking ramp begun across seven target companies; 2–3 references reconnected.

**Planned vs. actual, adapted for a non-DSA week:** this week introduced no new DSA pattern and no plan-required LeetCode problems in the traditional sense, so the usual "required vs. extra practice" problem count doesn't directly apply — noted here explicitly rather than silently forcing DSA-phase bookkeeping onto content it wasn't built for (the same open question `00_Curriculum_Map.md` flagged ahead of this week, now resolved by how this week's actual content turned out). What *does* map cleanly: **6 of 6 planned DSA revision problems solved cold** (Course Schedule, Coin Change, Combination Sum, Longest Substring Without Repeating Characters, Validate BST, Redundant Connection — one each from Graph, DP, Backtracking, Sliding Window, Tree, and Union-Find), and **4 of 4 planned LLD systems delivered**, each meeting its stated definition of done. Zero "extra practice" LLD systems were added beyond the plan's four — Day 106 reasoned explicitly that the DSA-phase "add extra reps until recognition is reflexive" instruction doesn't have a clean analog for whole LLD systems the way it does for a thin LeetCode pattern, and that reasoning held for the rest of the week without needing revisiting.

**Short diagnostic — worth answering honestly before Week 17, not just nodding along to:**
- Can the 5-step framework be recited *and applied live* under time pressure, not just read and agreed with? (Today's self-check and two mocks are the actual evidence here, not a feeling of familiarity.)
- Given a new, unfamiliar requirement, is "does this need a pattern at all" a live question asked before reaching for one — or has pattern-application become a reflex applied regardless of whether it's earned?
- Can State versus Strategy versus "just a data-driven rule" (today's `BookItem` contrast) be told apart on a *new* example, not just the three examples this week already worked through?
- Can the concurrency reasoning from Day 111 be reproduced from scratch on a *different* system, the way Mock Interview #2 just asked for — or does it only come back clearly when Parking Lot specifically is the subject?

**What Week 17 assumes:** everything above, fully reflexive, with zero further recap — Week 17 opens directly into ATM's requirements on Day 113, introduces Chain of Responsibility as this series' eleventh pattern, and continues the same cold-DSA-revision rhythm (Heap, Trie, Backtracking, Graph, DP, String DP — one each across Days 113–118) against six more systems: ATM, Elevator, Splitwise, BookMyShow, Food Delivery, and Hotel Booking.
