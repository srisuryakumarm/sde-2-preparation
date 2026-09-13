# Micro-Primer: CI/CD & GitHub Actions — Needed Before Day 139

**First needed:** Day 139, the only day GitHub Actions appears anywhere in the plan — which jumps straight to a full pipeline (checkout, JDK setup, `mvn test`, JaCoCo coverage gates, Docker build/push, Helm chart validation) with no prior explanation of what a workflow file even looks like.

**CI (Continuous Integration)** means automatically building and testing your code every time you push, so problems surface immediately — not later, and not for someone else to discover.

**CD (Continuous Delivery/Deployment)** means automatically packaging, and optionally deploying, code that passes CI, without manual steps.

**GitHub Actions** is GitHub's built-in automation system: you commit a YAML file under `.github/workflows/`, and GitHub runs it automatically based on triggers you define. The minimum anatomy:

```yaml
on: [push]                    # what triggers this workflow

jobs:
  build:                      # a named job
    runs-on: ubuntu-latest    # runs on a fresh virtual machine
    steps:                    # an ordered list of actions
      - uses: actions/checkout@v4         # step 1: get the code
      - uses: actions/setup-java@v4       # step 2: install a JDK
        with:
          java-version: '17'
      - run: mvn test                     # step 3: run a shell command
```

- `on:` — what triggers the workflow (`push`, `pull_request`, etc.).
- `jobs:` — one or more named jobs; each gets its own fresh machine.
- `steps:` — an ordered list within a job; each is either a reusable `uses:` action or a raw `run:` shell command.

That's the whole shape. Day 139's real pipeline is just more steps stacked in this same structure — a coverage check, a Docker build, a Helm validation — nothing conceptually new beyond this.

**Deferred:** coverage-gate enforcement specifics (JaCoCo), Docker image build/push steps, and Helm chart validation itself — all Day 139's own full task.

### Checklist
- [ ] Can explain the difference between CI and CD in one sentence each.
- [ ] Can read a basic GitHub Actions workflow and identify its trigger, jobs, and steps.
- [ ] Ready for Day 139.
