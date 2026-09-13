# Micro-Primer: YAML Syntax — Needed Before Day 36

**First needed:** Day 36, editing `application.yml` to configure PostgreSQL. Used again constantly after: Day 51 profiles, Day 96 Kubernetes manifests, Day 135 Helm values, Day 139 GitHub Actions workflows.

**YAML** ("YAML Ain't Markup Language") is a human-readable format for structured configuration data. Every `.yml`/`.yaml` file you touch for the rest of this plan uses exactly the two rules below — nothing more is needed.

**Rule 1 — indentation defines nesting.** Structure is shown by spaces (never tabs), not brackets or braces. A key indented one level further is nested inside the key above it:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/tododb
    username: postgres
    password: secret
```

This means: under `spring`, there's a `datasource` section, which has `url`, `username`, and `password` keys — equivalent to a nested object in JSON, just without the `{ }`.

**Rule 2 — a dash starts a list item.**

```yaml
profiles:
  active:
    - dev
```

That's genuinely the whole syntax you need. Every later YAML file in this plan — Kubernetes manifests, Helm values, GitHub Actions workflows — is just these same two rules describing different things.

**Deferred:** nothing further.

### Checklist
- [ ] Can read a nested `application.yml` and correctly identify what's nested under what.
- [ ] Ready for Day 36.
