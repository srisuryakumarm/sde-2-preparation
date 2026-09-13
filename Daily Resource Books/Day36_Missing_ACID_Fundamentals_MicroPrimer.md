# Micro-Primer: ACID — Needed Before Day 36

**First needed:** Day 36, your first real database work. Referenced again implicitly by `@Transactional` (Spring primer) and directly by Day 79's Two-Phase Commit vs. Saga discussion.

**ACID** is the standard framework for what a database transaction guarantees:

- **Atomicity** — a transaction either fully happens or fully doesn't; no partial completion. This is exactly what `@Transactional` (covered in the Spring primer before Day 34) gives you in practice: if anything inside a `@Transactional` method fails, everything it already did gets rolled back too.
- **Consistency** — a transaction can only move the database from one valid state to another; it can never leave data violating the rules you've defined (e.g., a required field left null).
- **Isolation** — concurrently running transactions don't see each other's uncommitted, in-progress changes; each behaves as if it ran alone.
- **Durability** — once a transaction commits successfully, it survives, even if the database crashes a moment later, because it's already been written to durable storage.

Worth knowing by name for two reasons: "what does ACID stand for" is a genuinely common, rote interview question, and Day 70's Saga pattern and Day 79's Two-Phase Commit are both, underneath, different strategies for trying to preserve these same four guarantees once a transaction has to span more than one service's database — where a single local transaction can't reach.

**Deferred:** nothing further — this is a complete, self-contained definition. No later day expands on ACID by name specifically, though Days 70 and 79 build on the underlying ideas without re-naming them.

### Checklist
- [ ] Can name and define all four ACID properties without notes.
- [ ] Ready for Day 36.
