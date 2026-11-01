# Day 74 — Missing Primer: The Linux Command Line, a Production Troubleshooting Toolkit, and What a Container Really Is

**Do this on Day 74 (≈ 50 min, before the Day 74 project block), then continue with Day 74's regular plan.**
**Needed by:** Day 75 (the JVM troubleshooting primer reads `top -H`, process CPU and memory and assumes you can use them), Days 95–99 (`kubectl exec`, `kubectl logs`, "exit code 137 = out of memory"), Days 134–137 (load tests, chaos experiments and bottleneck analysis are all read off CPU / memory / connection numbers), Day 141 (on-call and incident response), and the interview question **"the service is slow / the disk is full / CPU is at 100 % — walk me through what you do."**
**Why it is missing:** the only terminal teaching is Day 1's basics. `ps`, `top`, `df`, `lsof`, `ss`, `curl -w`, `dig`, pipelines such as `sort | uniq -c`, signals and the OOM killer appear in no book, and Day 71's OS primer teaches the *concepts* (processes, file descriptors, virtual memory) without the commands. **Your Mac is not Linux** — but production servers, Docker containers and Kubernetes pods all are.
**Already in place:** terminal basics and the IDE (Day 1), processes / threads / virtual memory / file descriptors / I/O models (Day 71's OS primer), TCP, DNS and HTTP (Days 71–72), Docker (Days 43–44), `todo-api` and a local Postgres.

---

## 0. Where to practise (a Mac is BSD-flavoured)
macOS ships BSD versions of many tools: no `/proc`, no `free`, no `ss`, no `journalctl`, and `ps`/`top` flags differ. **Practise inside a Linux container** — it costs nothing:
```bash
docker run --rm -it ubuntu:24.04 bash
apt-get update && apt-get install -y procps iproute2 curl dnsutils lsof     # ps/top/free, ss, curl, dig, lsof
```
(You can also `docker exec -it <your-postgres-container> bash` to look around a real service.) Docker Desktop runs a small Linux VM underneath — that is why containers on your Mac see Linux.

## 1. Files and text pipelines (the 80 % you will actually type)
| Task | Command |
|---|---|
| where am I / list / move | `pwd`, `ls -la`, `cd /var/log`, `cd -` |
| read | `cat f`, `less f` (`/text` to search, `q` quits), `head -n 20 f`, **`tail -f app.log`** (follow live) |
| search text | `grep -rn "ERROR" .`, `grep -i`, `grep -v` (invert), `grep -C 3` (context) |
| find files | `find . -name "*.log" -mtime -1` |
| count / sort | `wc -l`, `sort`, `uniq -c` |
| redirects and pipes | `>` overwrite, `>>` append, `2>&1` merge errors into output, `a \| b` feed one command to the next |
| permissions | `ls -l` shows `rwxr-xr-x`; `chmod 755 run.sh` (owner rwx, others r-x); `chmod 644 f` (rw-r--r--); `chown user:group f` |
**The classic one-liner** — "which 10 clients hit us most?": `awk '{print $1}' access.log | sort | uniq -c | sort -rn | head`. Read it left to right: take column 1, group identical lines, count, sort by count, top 10.

## 2. Processes and resources — "is the machine the problem?"
| Question | Command | What to read |
|---|---|---|
| what is running / who is using CPU | `ps aux --sort=-%cpu \| head`, `top` (`1` per-core, `M` sort by memory, `H` threads) | `%CPU`, `%MEM`, state |
| how loaded is it | `uptime` → **load average** (1, 5, 15 min) | compare with the **number of cores**: load ≈ cores = full; load ≫ cores = queueing |
| memory | `free -m` | look at **`available`**, not `free` (Linux uses spare RAM as cache) |
| disk space / who ate it | `df -h`, `du -sh /var/log/*` | a full disk breaks logging, databases and uploads |
| disk / CPU waiting | `vmstat 1`, `top`'s `%wa` | high `wa` = processes stuck waiting for **I/O**, not busy computing |
**Load average on Linux** counts tasks that are running, waiting for a CPU, *or* blocked on uninterruptible I/O — so load can be high with an idle CPU when the disk is the bottleneck.
**Signals:** `kill <pid>` sends **SIGTERM** (15, "please shut down" — the JVM runs its shutdown hooks); `kill -9` sends **SIGKILL** (cannot be caught — no cleanup). Kubernetes stops a pod with SIGTERM, waits the grace period (30 s by default), then SIGKILL. **Exit code 137 = 128 + 9 = killed by SIGKILL**, very often the **OOM killer** (the kernel killing the biggest process when memory runs out; `dmesg | grep -i killed` on a host, `OOMKilled: true` in `docker inspect`, `OOMKilled` in `kubectl describe pod`).

## 3. Network — "is it the network, DNS, or the app?"
```bash
curl -v http://localhost:8080/health                       # headers both ways; -i includes response headers
curl -s -o /dev/null -w "dns=%{time_namelookup} connect=%{time_connect} first-byte=%{time_starttransfer} total=%{time_total}\n" http://localhost:8080/health
dig example.com +short            # DNS answer (Day 71)       |  nslookup is the older tool
lsof -i :8080                     # which process has port 8080 open (works on Mac and Linux)
ss -tulpn                         # Linux: listening TCP/UDP sockets and their processes
```
The `curl -w` timings localise slowness: big `dns` → resolver; big `connect` → network / listen backlog; big gap between `connect` and `first-byte` → the **application** is slow; big gap to `total` → a large body or slow client.

## 4. Logs and services
`tail -f app.log | grep ERROR` (live filtered); `docker logs -f <container>`; `kubectl logs -f <pod>` (Days 95–99); on a systemd host `systemctl status myapp` and `journalctl -u myapp -f --since "10 min ago"`.

## 5. File descriptors — the "Too many open files" error
Every socket and file is a **file descriptor**; each process has a limit (`ulimit -n`, often 1024 by default). A service that leaks connections eventually fails with *Too many open files*. See the count with `ls /proc/<pid>/fd | wc -l` (Linux) or `lsof -p <pid> | wc -l`; the fix is closing resources (try-with-resources, Day 18) or pooling (connection pools), not just raising the limit. `/proc/<pid>/status` (Linux) shows `Threads:` and `VmRSS:` (resident memory).

## 6. A container is a process, not a tiny VM
- **Namespaces** give a process its own *view* of the system — its own process-ID space (it thinks it is PID 1), network interfaces, mounts, hostname, users.
- **cgroups** put *limits and accounting* on that process — CPU shares/quota (CPU **throttling**) and a **memory cap** (exceed it → the OOM kill above).
- An **image** is a stack of read-only filesystem layers (Day 43) shared between containers.
- **A VM** runs its own full operating-system kernel on virtual hardware; **a container shares the host's kernel** — so it starts in milliseconds and is lighter, but is less isolated. That is the interview answer to "containers vs VMs".
- Consequence for Java: the JVM reads the container's memory limit; set a heap *percentage* (e.g. `-XX:MaxRAMPercentage=75`) rather than a number that ignores the limit (Day 75's file).

## 7. SSH and cron in four lines
`ssh-keygen -t ed25519` makes a key pair; put the **public** key in the server's `~/.ssh/authorized_keys`; `ssh user@host`; `scp file user@host:/path`. **cron** schedule syntax is five fields — `minute hour day-of-month month day-of-week command` (`0 2 * * *` = 02:00 every day); Spring's `@Scheduled(cron=…)` on Days 130–131 uses the same idea.

## 8. The interview runbook: "the service is slow"
1. **Is it up, and since when?** health endpoint, recent deploys/config changes (most incidents follow a change — roll back first, investigate later).
2. **Resources, USE method per resource:** *Utilisation, Saturation, Errors* for **CPU** (`top`, load vs cores), **memory** (`free`, GC logs — Day 75, OOM kills), **disk** (`df`, `vmstat`), **network** (`curl -w`, `ss`).
3. **The service itself, RED method:** *Rate, Errors, Duration* (p99!) from metrics (Days 76–77), then **traces** (Day 137) to find the slow hop.
4. **Dependencies:** database slow-query log and `EXPLAIN` (Day 43's file), downstream latency, connection-pool exhaustion, thread-pool saturation (Day 64's file).
5. **Mitigate first** (rollback, scale out, shed load, restart a stuck instance), **root-cause afterwards**, then write the post-mortem (Day 141).

## 9. Exercise (≈ 20 min, inside the Ubuntu container from §0)
1. `yes > /dev/null &` starts a CPU hog. Find it with `ps aux --sort=-%cpu | head -3` and with `top`; stop it with `kill %1` (or `pkill yes`). Note its `%CPU`.
2. `dd if=/dev/zero of=/tmp/big bs=1M count=100` makes a 100 MB file; find it with `du -sh /tmp/*` and `df -h`; delete it.
3. Write the pipeline that prints the five most common words in a text file you create with `printf` (`tr -s ' ' '\n' < f | sort | uniq -c | sort -rn | head -5`).
4. From your Mac (not the container), start `todo-api`, then run the `curl -w` timing line from §3 against it, and `lsof -i :8080` to see the Java process. Say what each `curl` timing field tells you.
5. Say aloud in 60 seconds the runbook in §8 for "p99 latency doubled after the 3 pm deploy".

## Checklist
- [ ] Can set up a Linux container to practise in, and say why the Mac itself is not enough.
- [ ] Can use `grep`, pipes, `tail -f`, `sort | uniq -c`, `chmod`; can read `top`, load average vs cores, `free` (available), `df`/`du`.
- [ ] Can say SIGTERM vs SIGKILL, what exit code 137 and the OOM killer mean, and what "Too many open files" is.
- [ ] Can use `curl -w` timings, `dig`, `lsof -i` to localise a network vs DNS vs app problem.
- [ ] Can answer "container vs VM" with namespaces + cgroups + shared kernel; can recite the USE / RED runbook. Continue with Day 74's regular plan.
