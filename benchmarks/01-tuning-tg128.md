# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **6 physical · 12 logical** cores · `ngl=0` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 19.0 | 51% |
| 3 | 28.9 | 77% |
| 6 | 37.5 | 100% |
| 12 | 27.7 | 74% |
| 24 | 17.1 | 46% |

**Best**: `-t 6` at 37.5 tok/s
**Slowest tested**: `-t 24` at 17.1 tok/s (2.19x spread)
**Against the physical-core default** (`-t 6`, 37.5 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=6 make bench
```

## Explanation

Điểm tốt nhất nằm ở 6 threads, đúng bằng số core vật lý. Khi tăng lên 12 và 24
threads, tốc độ giảm từ 37.5 xuống 27.7 và 17.1 tok/s.

Decode ở cấu hình này có vẻ bị giới hạn bởi memory bandwidth. Thread bổ sung không
tạo thêm băng thông mà tăng tranh chấp scheduler và cache, nên hyperthreading và
oversubscription không giúp workload này.
