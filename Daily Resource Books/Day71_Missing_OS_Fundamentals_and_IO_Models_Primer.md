# Day 71 — Missing Primer: Operating-System Fundamentals and I/O Models for Backend Engineers

**Do this first on Day 71 (≈ 75 min, alongside Networking Fundamentals I), then continue with Day 71's regular plan.**
**Needed by:** Day 72 (HTTP, load balancers, connection handling), Day 76 (JVM/process metrics), Day 127 (LSM/page cache), Day 129 (fan-out, connection counts) — and SDE-2 interviews ("process vs thread", "what is a context switch", "how does epoll work", "why is Redis fast").
**Why it is missing:** "context switch", "virtual memory", "page fault", "file descriptor", "epoll" and "I/O multiplexing" appear in the books only in passing (Days 43, 106, 127, 134), never taught. Day 65's Reactor primer asserts that event loops scale better than thread-per-request; this explains *why*.
**Already in place:** threads, deadlock and locks (Days 29, 37–39), the Reactor primer (Day 65), TCP/UDP (Day 71's own theory).

---

## 1. Process vs thread
A **process** has its own virtual address space, file-descriptor table and resources. **Threads** live inside a process: they share its heap, code and open files, and each has its own **stack** and registers. A thread switch is cheaper than a process switch (no change of address space), but neither is free.

## 2. User mode, kernel mode, system calls
Your code runs in **user mode**. Anything touching hardware or kernel resources (`read`, `write`, `open`, `socket`, `mmap`) is a **system call** that traps into the kernel and back — much costlier than a function call. That is why code **buffers and batches** I/O.

## 3. Context switch
When the scheduler pre-empts a thread it saves registers, switches stacks (and the address space for a different process) and resumes another. The direct cost is microseconds; the hidden cost is **cold CPU caches and TLB entries**. Thousands of runnable threads → time lost to switching. (Linux tunes this with a fair scheduler; you only need the idea.)

## 4. Virtual memory
Each process sees a private, contiguous address space; **page tables** map it to physical frames, cached by the **TLB**. Touching a page that isn't mapped is a **page fault** — *minor* (the kernel just maps it) or *major* (read from disk/swap, milliseconds). If a container exceeds its **memory limit** the kernel's OOM killer ends it — why a JVM whose heap + metaspace + threads exceed the Kubernetes limit gets `OOMKilled` (Day 75's file).

## 5. File descriptors
On Unix, files, sockets and pipes are all **file descriptors** (integers per process). The per-process limit (`ulimit -n`) caps open files *and* connections: "**Too many open files**" usually means leaked connections or streams you never closed.

## 6. IPC and deadlock
Processes talk through pipes, sockets (TCP or Unix-domain), shared memory, signals. **Deadlock** needs four conditions together — mutual exclusion, hold-and-wait, no pre-emption, circular wait — so breaking any one (usually by a **global lock order**) prevents it.

## 7. I/O models — the part interviewers love
| Model | How it works | Cost |
|---|---|---|
| **Blocking** | the thread sleeps until data arrives | one thread per connection |
| **Non-blocking** | the call returns at once ("not ready") | you must poll |
| **I/O multiplexing** (`select`/`poll`/**`epoll`** on Linux, `kqueue` on macOS/BSD) | **one thread waits on thousands of descriptors** and is told which are ready | the model behind Netty, Nginx, Redis, Node |
| **Async I/O** (`io_uring`, IOCP) | the kernel completes the I/O and notifies you | newest, most complex |

The **C10K problem** (10,000 concurrent connections) is why thread-per-connection stopped scaling. Today's answers: an **event loop** over epoll (Netty — your Gateway, Day 65), or **virtual threads** that make blocking code cheap (Day 42's file).

## 8. Why real systems behave the way they do
| System | OS idea behind it |
|---|---|
| **Kafka** | sequential disk writes + the OS **page cache** + **zero-copy** `sendfile` from disk to socket |
| **Redis** | one thread executing commands over an epoll event loop — no locks, memory-bound |
| **PostgreSQL** | one **process per connection** → pool connections (HikariCP/PgBouncer), don't open thousands |
| **Tomcat** | thread-per-request pool (default 200) → exhaustion when handlers block on slow services (Day 64) |

## 9. Exercise (30 min, your Mac; Linux equivalents in brackets)
1. Start `todo-api`. Find its pid (`jps`). Count threads: `ps -M <pid> | wc -l` [`ps -L -p <pid>`]. Compare with `jcmd <pid> Thread.print | grep -c '^"'`.
2. Count open descriptors: `lsof -nP -p <pid> | wc -l`; list listeners: `lsof -nP -iTCP -sTCP:LISTEN`. Check `ulimit -n`.
3. Reproduce "Too many open files": `ulimit -n 256` in a shell, then run a small Java `main` that opens files in a loop without closing; read the `IOException`; fix it with try-with-resources.
4. Watch memory while the app runs (`top -pid <pid>` [`top -p <pid>`]; `vm_stat` for page-in/out). Write two sentences on what a page fault is.

## Checklist
- [ ] Can contrast process and thread, and explain what a context switch costs.
- [ ] Can explain virtual memory, a page fault, and why exceeding a container's limit kills the JVM.
- [ ] Can explain blocking vs non-blocking vs epoll multiplexing, and why Netty/Redis use an event loop.
- [ ] Can explain why Kafka is fast and why Postgres needs connection pools.
- [ ] Continue with Day 71's regular plan.
