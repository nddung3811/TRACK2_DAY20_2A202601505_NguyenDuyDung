# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 6578.5 | 6578.6 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 5071.5 | 5071.7 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 6078.1 | 6078.2 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **5909.4** · total **5909.5**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput is more useful than raw throughput** because it filters out requests that do not meet their specific Service Level Objectives (SLOs).

While raw throughput measures total requests per second, it ignores the constraints of the system. The context explicitly states that "Throughput at saturation ignores SLOs." Goodput, by contrast, counts only requests that m

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation** in GPU memory.

By storing the Key-Value (KV) cache in non-contiguous pages, the model avoids the wasted space that would exist if the KV cache were stored contiguously (like a contiguous array). This non-contiguous layout allows the engine to skip prefill entirely when a shared prefix is used, thereby optimizing memory usage and perf

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps when **prefill is compute-bound and decode is memory-bound**, as stated in the context.

This is because the context explains that prefilling the attention matrix requires significant computation (likely GPU compute), while decoding the output requires significant memory bandwidth. By separating these operations into distinct pools:
1.  **Prefill** can be optimiz


## Which N16-N19 pieces are real

N16 Cloud/IaC: stub, chạy localhost. N17 Data pipeline: stub, không có batch job
hay DAG trong pipeline này. N18 Lakehouse: stub, dùng `TOY_DOCS` trong memory. N19
Vector + features: stub, retrieval bằng keyword overlap, không dùng vector index hay
embedding server. N20 Serving: real, gọi `llama-server` qua HTTP.

LLM chiếm gần như toàn bộ 5909.5 ms total, phù hợp với pipeline nhỏ và retrieval
fallback. Nếu cần giảm latency 2×, em sẽ giảm decode work trước bằng cách giới hạn
output tokens hoặc dùng runtime/GPU ổn định hơn; retrieval hiện gần 0 ms.
