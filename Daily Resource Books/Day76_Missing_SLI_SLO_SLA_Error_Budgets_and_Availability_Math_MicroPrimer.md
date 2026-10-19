# Day 76 — Missing Primer: SLIs, SLOs, SLAs, Error Budgets, Golden Signals, and Availability Math

**Do this first on Day 76 (≈ 45 min), then continue with Day 76's regular plan.**
**Needed by:** Day 77 (Grafana dashboards and alerts), Day 137 (tracing and bottleneck analysis), HLD answers (every "99.99 % available" claim), SDE-2 reliability questions.
**Why it is missing:** SLI, SLO, error budget and the golden signals appear nowhere in the repo; "nines" appear only in passing.
**Already in place:** Actuator, Micrometer and Prometheus (Day 76's own theory), percentiles/latency intuition (Day 72), replication and availability ideas (Days 59–61).

---

## 1. The vocabulary
- **SLI** (indicator): a measurement — "the share of requests that succeeded", "the 99th-percentile latency".
- **SLO** (objective): your internal target for an SLI — "99.9 % of requests succeed over 30 days".
- **SLA** (agreement): the *contract* with consequences (credits) — looser than the SLO.
- **Error budget** = 100 % − SLO. At 99.9 % you may "spend" 0.1 % failures per window. Budget left → ship features; budget gone → freeze and fix reliability.

## 2. What to measure
**Golden signals** (Google SRE): **latency**, **traffic**, **errors**, **saturation**. **RED** for services: **R**ate, **E**rrors, **D**uration. **USE** for resources: **U**tilization, **S**aturation, **E**rrors.
Use **percentiles**, not averages: an average hides the slow tail that your unluckiest users feel (p99 is often 10× p50). Prometheus records latency as **histograms**, so `histogram_quantile()` can compute p95/p99.

## 3. Availability math
| Availability | Downtime / year | / 30 days |
|---|---|---|
| 99 % | 3.65 days | 7.2 h |
| 99.9 % ("three nines") | 8.76 h | 43.2 min |
| 99.99 % | 52.6 min | 4.3 min |
| 99.999 % | 5.26 min | 26 s |
- **In series** (a request needs A *and* B): multiply — two 99.9 % services give ≈ 99.8 %. Every hard dependency lowers your ceiling.
- **In parallel / redundant** (either works): 1 − (1 − a)ⁿ — two independent 99 % copies give 99.99 %.
- Redundancy only helps if failures are independent (two replicas in one AZ are not).

## 4. Alert on symptoms, not causes
Page when users are hurt (error-rate or latency SLO **burning fast**), not when a CPU hits 80 %. Use *burn rate*: how quickly you are consuming the budget.

## 5. Spring + Prometheus recipes (Day 76/77)
```
# error ratio (availability SLI), last 5 minutes
sum(rate(http_server_requests_seconds_count{status=~"5.."}[5m]))
  / sum(rate(http_server_requests_seconds_count[5m]))
# p99 latency (needs histogram buckets enabled)
histogram_quantile(0.99, sum by (le) (rate(http_server_requests_seconds_bucket[5m])))
```
Enable buckets with `management.metrics.distribution.percentiles-histogram.http.server.requests=true`.

## 6. Exercise (≈ 20 min)
1. Define three SLIs and SLOs for `order-service` (e.g. 99.9 % of `POST /orders` return non-5xx; p99 < 400 ms; 99.5 % of payment events processed within 5 s). Compute each one's monthly error budget.
2. Your Gateway (99.95 %) calls Order (99.9 %), which calls Product (99.9 %) in series: what is the best end-to-end availability? What changes if Product has a fallback that serves cached data?
3. Write one alert for the error-ratio SLI above in plain words (threshold, window, who is paged).

## Checklist
- [ ] Can define SLI, SLO, SLA, error budget and give an example of each.
- [ ] Can name the golden signals and say why p99 beats the average.
- [ ] Can compute downtime for N nines and combine serial/parallel availability.
- [ ] Continue with Day 76's regular plan.
