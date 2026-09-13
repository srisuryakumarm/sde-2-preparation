# Micro-Primer: Why Flyway — Needed Before Day 40

**First needed:** Day 40's project task, which switches `todo-api` from `ddl-auto: update` to Flyway-managed migrations — without the theory block explaining why that switch is worth making.

Up through Day 36, `ddl-auto: update` has let Hibernate automatically create and adjust your database tables to match your `@Entity` classes. Convenient for early development — but risky in anything real: it can make destructive changes silently (dropping a column it decides is now unused), keeps no record of what changed or when, and can't be reliably replayed in the same order across environments (your laptop, staging, production).

**Flyway** fixes this by making every schema change an explicit, versioned, plain SQL file — `V1__create_tasks_table.sql`, `V2__add_priority_column.sql`, and so on — run in order, exactly once each, with every applied version tracked in a `flyway_schema_history` table it creates for you. It's the same discipline Git already gives your source code, applied to your database's structure instead.

This is exactly why Day 40 has you switch `ddl-auto` from `update` to `validate`: Hibernate now only *checks* that your entities match the schema Flyway already built, instead of being trusted to change that schema itself.

**Deferred:** nothing further — Day 40's own project task is the complete, hands-on treatment. This just supplies the "why bother" the task itself doesn't state.

### Checklist
- [ ] Can explain, in one sentence, what problem Flyway solves that `ddl-auto: update` doesn't.
- [ ] Ready for Day 40.
