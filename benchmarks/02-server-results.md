# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=6` ·
`ngl=0`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 59 | 1.04 | 7600 | 13000 | 15000 | 8.2 | 0.0% |
| 50 | 61 | 1.05 | 32000 | 47000 | 48000 | 31.7 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.01x** (20% of linear) |
| P95 latency | **3.62x** |
| Effective concurrency at 50 users | 31.7 vs `--parallel 4` slots (occupancy/slot ratio 7.92) |

**Saturated.** Throughput delivered only 1.01x for 5x the offered load, and effective concurrency (31.7) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.01x while P95 moved 3.62x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Reading

Server đã bão hoà rõ ở mức 50 users, và có thể bắt đầu trước đó. Offered load tăng
5× nhưng RPS chỉ từ 1.04 lên 1.05, trong khi P95 tăng từ 13 lên 47 giây. Compute
không tăng tương ứng, còn requests deferred có xuất hiện, nên phần latency tăng chủ
yếu là queue time. Nếu cần nâng goodput@SLO, em sẽ sweep `--parallel` trước thay vì
mặc định tăng slot, vì CPU-only có thể bị contention khi tăng concurrency.
