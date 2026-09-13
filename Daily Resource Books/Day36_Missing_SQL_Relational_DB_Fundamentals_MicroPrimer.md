# Micro-Primer: SQL / Relational Database Fundamentals — Needed Before Day 36

**First needed:** Day 36, where you configure PostgreSQL and define a `Task` `@Entity` with `@Id` — four days before Day 40's "SQL Fundamentals" theory block actually explains what any of that means in relational-database terms.

A **relational database** stores data in **tables**. Think of a table like a spreadsheet: each **row** is one record, each **column** is a named field every record has (e.g., a `tasks` table might have columns `id`, `title`, `status`).

A **primary key** is a column (or combination of columns) guaranteed unique across every row — how you unambiguously refer to *this exact record* and no other. This is exactly what `@Id` marks on Day 36's `Task` entity: it tells Hibernate which field is the primary key.

**PostgreSQL** is one specific relational database system (others you'll hear of: MySQL, SQL Server) — a running program you connect to over the network using a **connection string**: host, port, database name, username, password. That's exactly what Day 36's `application.yml` PostgreSQL configuration supplies.

**SQL** (Structured Query Language) is the language you write to ask a relational database questions or make changes — `SELECT`, `INSERT`, `UPDATE`, `DELETE`. You won't write raw SQL yet on Day 36 (Spring Data JPA generates it for you from your repository interface), but it's worth knowing that's what's happening underneath.

When Day 36 says "Hibernate creates the `tasks` table": based on your `@Entity`-annotated `Task` class, Hibernate generates and runs the SQL needed to create a matching table, with one column per field.

**Deferred:** how to combine data across multiple tables (`JOIN`s), how to structure tables to avoid redundant data (normalization), how lookups get sped up (indexes), and how tables reference each other (foreign keys) — all of this is Day 40's "SQL Fundamentals" block, in full.

### Checklist
- [ ] Can explain what a table, row, column, and primary key are.
- [ ] Can explain what a connection string is for.
- [ ] Ready for Day 36.
