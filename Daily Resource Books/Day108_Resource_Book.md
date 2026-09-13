# Day 108 — LLD #1: Tic-Tac-Toe (Warm-Up), and Coupling, Cohesion, and the Law of Demeter

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 107 Resource Book](Day107_Resource_Book.md)
**Next ▶:** [Day 109 Resource Book](Day109_Resource_Book.md)
**Companion to:** Day 108 of `Week_16_Revised.md`

---

## Recap

Days 106–107 taught ten patterns and the 5-step framework in isolation, one pattern at a time, against small illustrative examples. Today is the first time the framework runs **end to end, against one real system, live** — every step from "clarify requirements" through "code the core," in order, on Tic-Tac-Toe specifically because it's small enough that the *process* is the actual lesson, not the system's own complexity. Two new, genuinely new-today concepts — Coupling, Cohesion, and the Law of Demeter — get applied retroactively to whatever gets built, which only works because there's now a real, freshly-written design to point at rather than a hypothetical one.

On the DSA side, today's single revision problem, Combination Sum, reaches back to **Week 10, Day 64** — Backtracking with element reuse, the second of five spaced-repetition checks this week.

---

## Learning Objectives

By the end of today, without notes:

1. Walk the 5-step LLD framework against a real system from a blank page to working code, narrating each step's decision out loud.
2. Explain why Tic-Tac-Toe correctly needs **zero** Structural or Behavioral patterns from the last two days, and recognize that judgment as a skill, not a gap.
3. Define Coupling, Cohesion, and the Law of Demeter, and find at least one genuine example of each in code you personally just wrote.
4. Solve Combination Sum (LC 39) cold, without hints, and state precisely why it must reuse the current index rather than advancing past it.

---

## Concept Dependency Map

```
Day 106: 5-step LLD framework · Structural patterns
Day 107: Behavioral patterns
Week 1 Day 5: OOP Four Pillars (encapsulation — needed for today's Cell/Board design)
Week 10 Day 64: Backtracking with reuse (today's revision problem)
        │
        ▼
Today: Framework applied end-to-end, live — Tic-Tac-Toe
  Step 1: Clarify requirements
  Step 2: Core objects — Mark, Cell, Player, Board, Game
  Step 3: Relationships — composition throughout; class diagram
  Step 4: Patterns — deliberately none; the judgment call itself is the lesson
  Step 5: Code the core — full, playable implementation
        │
        ▼
Coupling / Cohesion / Law of Demeter (NEW — needs: encapsulation, Week 1 Day 5)
        │
        └──▶ Applied retroactively to the Tic-Tac-Toe code just written —
             including one genuine Law-of-Demeter violation found and fixed live
        │
        ▼
🔗 Forward: Day 109's Vending Machine is the first system where a pattern
   (State) IS the point — a direct contrast with today's "no pattern needed."
```

---

# Part 1 — DSA Revision Block (1 hr)

## Revision — Combination Sum (LeetCode 39, Medium)

**🔗 Originally taught in full depth:** Week 10, Day 64 — Backtracking, duplicate-skip translated to forward-index form; this problem specifically established reuse via a non-advancing recursive index.

**Statement:** given an array of **distinct** positive integers `candidates` and a target, return all unique combinations where the chosen numbers sum to target. The same number may be chosen from `candidates` an unlimited number of times.

**Attempt cold before reading on.**

### Recap: The Approach

Standard backtracking, tracking a running `remaining` target and a `start` index — but with one deliberate difference from a plain subset/permutation backtrack: **the recursive call reuses `start` itself (`i`, not `i + 1`) rather than advancing past the chosen index**, since a candidate can be picked again. `start` still prevents re-visiting *earlier* candidates (which would generate the same combination in a different order — e.g. `[2,3]` and `[3,2]` counted as duplicates), it just no longer forbids re-picking the *current* one.

```java
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] candidates, int remaining, int start,
                        List<Integer> current, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(current));   // COPY — current keeps mutating after this
        return;
    }
    if (remaining < 0) {
        return;   // prune — this branch overshot, nothing further down it can work
    }
    for (int i = start; i < candidates.length; i++) {
        current.add(candidates[i]);
        backtrack(candidates, remaining - candidates[i], i, current, result);   // i, NOT i+1: allows reuse
        current.remove(current.size() - 1);   // backtrack: undo the choice before trying the next
    }
}
```

### Fresh Trace — A Different Example Than Day 64 Used

`candidates = [2, 3, 6, 7]`, `target = 7`. A representative slice of the recursion, showing the mechanism (not every pruned leaf):

```
backtrack(remaining=7, start=0, current=[])
├─ choose 2: backtrack(remaining=5, start=0, current=[2])
│  ├─ choose 2: backtrack(remaining=3, start=0, current=[2,2])
│  │  ├─ choose 2: remaining=1 → all further choices overshoot → pruned
│  │  ├─ choose 3: remaining=0 → ADD [2,2,3]
│  │  ├─ choose 6, 7: remaining < 0 → pruned
│  ├─ choose 3: backtrack(remaining=2, start=1, current=[2,3])
│  │  └─ choose 3,6,7: all overshoot remaining=2 → pruned
│  ├─ choose 6, 7: remaining < 0 → pruned
├─ choose 3: backtrack(remaining=4, start=1, current=[3]) → every extension overshoots → no additions
├─ choose 6: backtrack(remaining=1, start=2, current=[6]) → every extension overshoots → no additions
├─ choose 7: remaining=0 → ADD [7]

Final result: [[2,2,3], [7]]
```

Both entries check out directly: `2+2+3=7`, `7=7`.

**Complexity:** derived, not asserted. Let `N = candidates.length`, `T = target`, `M = min(candidates)`. Every recursive call either adds a result, prunes (`remaining < 0`), or branches into at most `N` children. Along any single path, `remaining` decreases by at least `M` per step (choosing the smallest reusable candidate repeatedly is the deepest possible path), so depth is bounded by `⌈T/M⌉`. With branching factor at most `N` and depth at most `T/M + 1`, the recursion tree has at most **O(N^(T/M + 1))** nodes — a loose upper bound (real runs prune far more aggressively than this suggests, since most branches overshoot long before reaching that depth), but one that's actually derivable from the recursion's own shape rather than asserted from memory. Space is `O(T/M)` for the recursion stack and the `current` list, excluding the space needed to store the output itself.

### Common Mistakes Checklist

- [ ] Using `i + 1` instead of `i` in the recursive call — this forbids reusing the same candidate, which is the rule for the *different*, easily-confused problem Combination Sum II (duplicate candidates allowed in the input, but each used at most once). Getting these two backwards is the single most common way this problem goes wrong.
- [ ] Omitting the `remaining < 0` prune. **This is a correctness bug, not just a performance one:** without it, the branch that keeps re-choosing the same smallest reusable candidate never lands exactly on `remaining == 0` and never hits a base case on its own — it recurses indefinitely down that path, risking a `StackOverflowError` rather than merely doing extra work.
- [ ] Adding `current` directly to `result` instead of `new ArrayList<>(current)` — `current` is the *same* mutable list object being repeatedly modified throughout the whole backtrack; every stored reference would end up reflecting whatever `current` looks like once the entire recursion finishes, not what it looked like at the moment of the "add."

**If any of this needed re-deriving rather than confirming:** Week 10, Day 64 has the full original treatment, including how this connects to Day 61's opening Backtracking mechanics.

---

# Part 2 — LLD #1: Tic-Tac-Toe, Framework Applied End to End

### Step 1 — Clarify Requirements

Real questions, asked and answered before any code exists:

- **Board size:** standard 3×3, or should the design generalize to N×N? *Decision: build for 3×3 — this is deliberately the warm-up system — but design the win-check so that generalizing to N×N later costs a constant change, not a rewrite. Stated as a scoping decision, not skipped.*
- **Players:** exactly two human players, alternating turns — no AI opponent, no more than two players. *Confirmed in scope.*
- **Persistence / scoring across multiple games:** out of scope for today — single game, single result, no running score. *Explicitly confirmed out of scope, not silently assumed.*
- **Win conditions:** three in a row, any row, column, or either diagonal. Standard.

### Step 2 — Identify Core Objects

The plan names four: `Board`, `Player`, `Cell`, `Game`. Plus one small addition worth making explicit rather than leaving as a bare `String` or `char`: a `Mark` enum (`EMPTY`, `X`, `O`) — a fixed, closed, three-value set is exactly what an enum is for.

**Why `Cell` earns its own class rather than the board just being `Mark[][]` directly:** with only today's requirements, a raw `Mark[][]` would technically work. `Cell` is a deliberate, small anticipatory choice — it gives future extensions (marking which cell completed the winning line, per-cell metadata) a place to live without restructuring `Board` later. Worth being honest about this trade-off rather than presenting `Cell` as strictly necessary: it's a class that costs very little today and buys optionality later, which is a defensible reason to introduce it, not an obviously-forced one.

### Step 3 — Define Relationships, and the Class Diagram

Composition throughout — none of these classes make sense detached from their owner, which is exactly the "part cannot outlive the whole" test Day 110's dedicated Composition-over-Inheritance day formalizes properly.

```
Game
 ├─ Board board                  (composition — a Game owns exactly one Board)
 ├─ List<Player> players         (composition — exactly two, for today's scope)
 ├─ int currentPlayerIndex
 └─ playMove(row, col) : GameStatus

Board
 ├─ Cell[][] grid                (3x3, composition)
 ├─ placeMark(row, col, Mark) : boolean
 ├─ isWinningMove(row, col) : boolean
 └─ isFull() : boolean

Cell
 └─ Mark mark

Player
 ├─ String name
 └─ Mark mark

Mark (enum)
 └─ EMPTY, X, O

GameStatus (enum, on Game)
 └─ CONTINUE, WIN, DRAW
```

### Step 4 — Apply Patterns Deliberately (Here: Deliberately None)

**Worth stating directly rather than skipping past:** none of the last two days' ten patterns are actually called for here, and recognizing that is the point of this step, not a failure to find one. There's no incompatible interface to bridge (no Adapter), no combinable optional behaviors (no Decorator), no complex multi-class subsystem to simplify (no Facade), no cross-cutting access concern (no Proxy), no recursive whole-part structure (no Composite — a 3×3 grid isn't a tree). Forcing a pattern in anyway — wrapping `Board` in a `Decorator` nobody asked for, or adding an `Observer` for state changes nothing is actually watching — would be exactly Day 106's flagged Common Mistake: over-engineering with unwarranted patterns, indirection added with no requirement behind it. **The judgment to recognize "no pattern is warranted here" is a real, checkable skill this framework is building, not a gap in its coverage.**

### Step 5 — Code the Core

```java
public enum Mark {
    EMPTY, X, O
}

public class Cell {
    private Mark mark = Mark.EMPTY;

    public Mark getMark() {
        return mark;
    }

    public void setMark(Mark mark) {
        this.mark = mark;
    }

    public boolean isEmpty() {
        return mark == Mark.EMPTY;
    }
}

public class Player {
    private final String name;
    private final Mark mark;

    public Player(String name, Mark mark) {
        this.name = name;
        this.mark = mark;
    }

    public String getName() { return name; }
    public Mark getMark() { return mark; }
}
```

```java
public class Board {
    private static final int SIZE = 3;
    private final Cell[][] grid;

    public Board() {
        grid = new Cell[SIZE][SIZE];
        for (int r = 0; r < SIZE; r++) {
            for (int c = 0; c < SIZE; c++) {
                grid[r][c] = new Cell();
            }
        }
    }

    public boolean placeMark(int row, int col, Mark mark) {
        if (row < 0 || row >= SIZE || col < 0 || col >= SIZE) {
            throw new IllegalArgumentException("Position out of bounds: (" + row + ", " + col + ")");
        }
        if (!grid[row][col].isEmpty()) {
            return false;   // occupied — move rejected, caller re-prompts
        }
        grid[row][col].setMark(mark);
        return true;
    }

    // Checks only the row, column, and diagonal(s) THROUGH (row, col) — not a full-board rescan.
    public boolean isWinningMove(int row, int col) {
        Mark mark = grid[row][col].getMark();
        if (mark == Mark.EMPTY) return false;

        boolean rowWin = true, colWin = true;
        for (int i = 0; i < SIZE; i++) {
            if (grid[row][i].getMark() != mark) rowWin = false;
            if (grid[i][col].getMark() != mark) colWin = false;
        }
        if (rowWin || colWin) return true;

        if (row == col) {
            boolean diagWin = true;
            for (int i = 0; i < SIZE; i++) {
                if (grid[i][i].getMark() != mark) { diagWin = false; break; }
            }
            if (diagWin) return true;
        }

        if (row + col == SIZE - 1) {
            boolean antiDiagWin = true;
            for (int i = 0; i < SIZE; i++) {
                if (grid[i][SIZE - 1 - i].getMark() != mark) { antiDiagWin = false; break; }
            }
            if (antiDiagWin) return true;
        }

        return false;
    }

    public boolean isFull() {
        for (Cell[] row : grid) {
            for (Cell cell : row) {
                if (cell.isEmpty()) return false;
            }
        }
        return true;
    }

    public void print() {
        for (int r = 0; r < SIZE; r++) {
            StringBuilder sb = new StringBuilder();
            for (int c = 0; c < SIZE; c++) {
                sb.append(grid[r][c].getMark() == Mark.EMPTY ? "." : grid[r][c].getMark());
                if (c < SIZE - 1) sb.append(" | ");
            }
            System.out.println(sb);
        }
    }
}
```

```java
public class Game {
    public enum GameStatus { CONTINUE, WIN, DRAW }

    private final Board board = new Board();
    private final List<Player> players;
    private int currentPlayerIndex = 0;

    public Game(Player player1, Player player2) {
        this.players = List.of(player1, player2);
    }

    public GameStatus playMove(int row, int col) {
        Player currentPlayer = players.get(currentPlayerIndex);
        boolean placed = board.placeMark(row, col, currentPlayer.getMark());
        if (!placed) {
            System.out.println("Cell already occupied — try again.");
            return GameStatus.CONTINUE;   // same player's turn again; index does not advance
        }

        if (board.isWinningMove(row, col)) {
            return GameStatus.WIN;   // currentPlayerIndex still points at the WINNER
        }
        if (board.isFull()) {
            return GameStatus.DRAW;
        }

        currentPlayerIndex = (currentPlayerIndex + 1) % players.size();
        return GameStatus.CONTINUE;
    }

    public Player getCurrentPlayer() {
        return players.get(currentPlayerIndex);
    }

    public void printBoard() {
        board.print();
    }
}
```

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    Game game = new Game(new Player("Player 1", Mark.X), new Player("Player 2", Mark.O));

    Game.GameStatus status = Game.GameStatus.CONTINUE;
    while (status == Game.GameStatus.CONTINUE) {
        game.printBoard();
        Player current = game.getCurrentPlayer();
        System.out.println(current.getName() + " (" + current.getMark() + "), enter row and col (0-2), space-separated:");
        int row = scanner.nextInt();
        int col = scanner.nextInt();
        status = game.playMove(row, col);
    }

    game.printBoard();
    System.out.println(status == Game.GameStatus.WIN
            ? game.getCurrentPlayer().getName() + " wins!"
            : "It's a draw!");
}
```

**Worked trace — win detection firing correctly:** X plays (0,0), O plays (1,0), X plays (1,1), O plays (2,0) — O now has the entire left column filled. `board.placeMark(2, 0, O)` succeeds; `board.isWinningMove(2, 0)` checks: `row=2`'s cells are `[O, ., .]` (not all O — `rowWin=false`); `col=0`'s cells are `[O, O, O]` (all O — `colWin=true`) → returns `true` immediately. `Game.playMove` returns `WIN` with `currentPlayerIndex` still pointing at Player 2 (O) — correct.

**Complexity:** `placeMark` and `isWinningMove` are both `O(N)` for an `N × N` board — checking one row, one column, and at most two diagonals through a single cell, not the whole board (an `O(N²)` full rescan would also be correct, just strictly worse — this is the smarter version, and worth being able to state *why* it's smarter, not just that it is). `isFull` is `O(N²)` since it genuinely must inspect every cell. For the required 3×3, every one of these is small enough to be irrelevant in practice — stated precisely anyway, since "irrelevant at N=3" and "irrelevant in general" are different claims, and only the first one is actually true here.

---

# Part 3 — Coupling, Cohesion, and the Law of Demeter

Three related but distinct lenses for evaluating a design *after* it exists — genuinely new today, and, deliberately, applied retroactively to the Tic-Tac-Toe code just written rather than taught against a fresh example, since a design that's fresh in your own head is a far better teaching surface than an abstract one.

## Coupling

**Definition:** the degree to which one class depends on — or needs to know about — another class's internal details. **Low coupling** (good): classes can change independently, as long as their public contracts hold. **High coupling** (bad): a change inside one class forces changes in others that merely use it.

**Applied retroactively:** does `Game` know anything about `Board`'s *internal* representation? Checking the code above — `Game.playMove()` calls `board.placeMark(...)`, `board.isWinningMove(...)`, `board.isFull()` — all public methods, and `Game` never once reaches into `Board`'s `Cell[][] grid` field directly. This is genuinely **low coupling**: if `Board`'s internal representation changed tomorrow — a flat `Mark[]` array instead of `Cell[][]`, say — `Game` wouldn't need a single line changed, as long as `placeMark`/`isWinningMove`/`isFull` kept their existing contracts.

## Cohesion

**Definition:** the degree to which a single class's responsibilities are focused on one purpose. **High cohesion** (good): a class does one thing, and everything in it serves that one thing. **Low cohesion** (bad): a class accumulates unrelated responsibilities — a "god class."

**Applied retroactively, with an honest critique:** `Board` handles placing marks, checking wins, checking fullness — all clearly "managing board state," high cohesion so far. But it *also* handles `print()` — rendering itself to the console. Is that really `Board`'s job, or a presentation concern that's been folded in? **A fair critique, worth stating plainly:** in a fuller system, a separate `BoardRenderer` (or a general view/presentation layer) would be the more cohesive design — `Board`'s purpose would stay purely "manage state," and "how to display state" would live elsewhere entirely, swappable independently (a console renderer today, a GUI renderer later, neither touching `Board`). For a small, throwaway console warm-up, folding `print()` into `Board` is a reasonable, deliberate simplification — but it's a real trade-off being made, not a cohesion-neutral choice, and it's worth being able to name that distinction explicitly rather than treating today's shortcut as the "correct" design in every context.

## The Law of Demeter ("Principle of Least Knowledge")

**Definition:** a method should only call methods on: itself, its own parameters, objects it creates directly, or its own direct fields — **not** on objects returned by calling a method on some *other* object first. Chaining through multiple objects to reach a distant one (`a.getB().getC().doSomething()`) is informally called a "train wreck," and is exactly what this principle flags.

**Why it matters, mechanically:** a chain like `a.getB().getC().doSomething()` means the calling code now depends on `A` having a `B`, *and* `B` having a `C`, *and* `C` having `doSomething()` — three separate things that can each independently break the caller if any one of them changes, none of which the caller should have needed to know about at all. Compare to `a.doSomething()`, where `A` internally handles reaching through `B` and `C` itself — only `A`'s public contract can break the caller now, not `B`'s or `C`'s internal structure.

**Classic illustrative example, before returning to Tic-Tac-Toe:**

```java
// VIOLATION — reaches through Customer into Wallet, three hops from the caller
order.getCustomer().getWallet().deductBalance(amount);

// COMPLIANT — Order asks Customer to pay; Customer handles ITS OWN wallet internally
order.getCustomer().pay(amount);
```

**A genuine violation, found in the code just written above, live:** `main()`'s original version called `game.getBoard().print()` — reaching *through* `Game` into the `Board` it returned, to call a method on that returned object. That's a real train wreck by the letter of the definition: `main()` now depends on `Game` having a `Board`, *and* on `Board` having a `print()` method — two facts `main()` shouldn't need to know. **This is exactly why the `Game` class above defines `printBoard()`, delegating internally to `board.print()`, and why `main()` calls `game.printBoard()` instead of chaining through** — the fix is already reflected in Part 2's code, not left as an exercise; catching it here is the "retroactive" application actually happening, not just being described.

**A worthwhile nuance, not just a rule applied blindly:** what about `game.getCurrentPlayer().getName()`, at the very end of `main()`? That's *also* technically a two-hop chain. **Is it also a violation?** The Law of Demeter is conventionally relaxed for simple, immutable data-holder objects (sometimes called DTOs) precisely because they have no meaningful "internals" to leak — `Player` has no behavior or invariants a caller could accidentally violate by reading `getName()` off of it, unlike `Board`, which owns real state and real rules about how that state may change. Applying the rule with equal strictness to every single method chain, regardless of what's on the other end, produces code buried in trivial wrapper methods that add indirection without preventing any real coupling — worth naming as a genuine trade-off of *over*-applying the principle, not just a reason to apply it correctly.

> 🔑 **Key Takeaway:** these three lenses aren't independent checklist items — they interact. Low coupling and high cohesion tend to reinforce each other (a focused class with a clean public contract is naturally easier to depend on loosely); the Law of Demeter is really coupling's *concrete, checkable* symptom — a train-wreck chain is coupling made visible in the actual call syntax, not just an abstract property you have to reason your way to.

---

# Project Block Guide (3.5 hrs)

**Repository:** `lld-java`. **Module:** new — `tic-tac-toe/`.

**Task:** the full implementation from Part 2 above — `Mark`, `Cell`, `Player`, `Board`, `Game`, and the `main`-method game loop.

**Definition of done:**
- Playable end-to-end via the `main`-method loop shown above.
- Win detection verified correct for all three line shapes — play at least one game through to a row or column win, and one through to a diagonal win, plus one full game to a draw.
- The class diagram from Step 3 (the ASCII version above, or your own redrawn version) committed alongside the code, e.g. as `DESIGN.md` in the same module.
- Pushed to `lld-java/tic-tac-toe/`.

---

# Career Block Guide (1 hr)

**LinkedIn engagement:** 20 minutes, same as always.

**Networking — begin the targeted ramp.** From today forward, work through this list deliberately, roughly 15–20 connection requests per company, spread across the coming weeks rather than all at once: **Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech.** Every note stays under 300 characters and references something specific and real — a recent post, a shared connection, a specific team — never a generic "I'd love to connect." A templated-sounding note gets ignored at a measurably higher rate than one that shows five seconds of actual looking.

---

# Day 108 — Interview Questions

**Q1. Why does Tic-Tac-Toe's Step 4 correctly conclude "no pattern needed," and why is that conclusion itself worth stating out loud in an interview?** None of the ten patterns from the last two days solve a problem this system actually has — no incompatible interface, no combinable behaviors, no complex subsystem, no access-control need, no recursive whole-part structure. Stating this explicitly signals the same judgment Day 106 flagged as a common mistake in reverse: recognizing when *not* to apply a pattern is as much a demonstrated skill as applying one correctly.

**Q2. Why does `Board.isWinningMove` check only the row/column/diagonal through the just-placed cell, instead of rescanning the whole board?** It's strictly less work for an equally correct result — a move can only possibly create a win along a line that passes through the cell that just changed; every other line's status is unaffected by this move and doesn't need re-checking.

**Q3. What's the concrete difference between low coupling and high cohesion — they're often confused?** Coupling is about relationships *between* classes — how much one depends on another's internals. Cohesion is about *within* a single class — how focused its own responsibilities are. A class can be low-coupled to everything else and still be low-cohesion internally (a "god class" with a clean external interface hiding an unfocused jumble inside), so they're genuinely separate axes.

**Q4. Why is `game.getBoard().print()` a Law of Demeter violation, and what's the fix?** It chains through `Game` to reach a method on the `Board` object it returns — the caller ends up depending on both `Game` having a `Board` and `Board` having `print()`. The fix is `Game` exposing its own `printBoard()` that delegates internally, so callers only ever depend on `Game`'s contract.

**Q5. Why is `game.getCurrentPlayer().getName()` conventionally *not* treated as a violation, even though it's also a two-hop chain?** The Law of Demeter is conventionally relaxed for simple, immutable data-holder objects with no real internal behavior or invariants to protect — `Player` has nothing a caller could break by reading a field off it, unlike a stateful object like `Board`.

**Q6. In Combination Sum, why does the recursive call pass `i` rather than `i + 1`?** The problem allows reusing the same candidate an unlimited number of times; passing `i` keeps that same index eligible again on the next recursive level, while still using `start` to prevent revisiting *earlier* indices, which is what stops duplicate combinations in a different order from being generated separately.

**Q7. Derive Combination Sum's worst-case time complexity rather than stating it from memory.** With `N` candidates, target `T`, and minimum candidate value `M`: each call branches into at most `N` children, and `remaining` shrinks by at least `M` per level, bounding depth at `⌈T/M⌉`. Branching factor `N` to depth `T/M + 1` gives a loose upper bound of `O(N^(T/M + 1))` total nodes in the recursion tree.

**Q8. Why is omitting Combination Sum's `remaining < 0` prune a correctness bug, not just a slowdown?** Without it, the branch that keeps re-choosing the same smallest reusable candidate never lands exactly on `remaining == 0` and has no other way to stop — it recurses indefinitely down that path, risking a `StackOverflowError` rather than merely doing avoidable extra work.

---

## Daily Deliverable Check

- [ ] Combination Sum (LC 39) solved cold, without hints.
- [ ] Tic-Tac-Toe complete and playable via the `main` method loop; win and draw detection both correct.
- [ ] Class diagram committed alongside the code.
- [ ] At least one genuine Coupling, Cohesion, or Law of Demeter observation found in your own Tic-Tac-Toe code (the walkthrough above models this — your own pass may surface the same ones, or different ones, either is fine as long as it's a real observation, not a restated example).
- [ ] Connection requests sent toward this week's target-company list.

---

## What Tomorrow Assumes You Already Know Cold

Day 109 assumes the full framework-applied-live experience from today is fresh, since tomorrow repeats it against a genuinely harder system (Vending Machine) without re-explaining what each step is for. It also assumes State (Day 107) is fully reflexive, not just recognized when pointed at — tomorrow is where State gets its real system-level test, and the theory block explicitly compares it against Strategy, which only makes sense if both are already solid rather than being re-derived from scratch.
