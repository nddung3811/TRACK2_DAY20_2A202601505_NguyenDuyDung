# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=3` `ngl=0` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3176 | 432 / 571 | 32.0 / 58.2 | 2399 / 3536 / 3536 | 31.3 |
| UD-Q2_K_XL | 0.39 | 2812 | 556 / 694 | 32.6 / 43.2 | 2551 / 3279 / 3279 | 30.7 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` and `Q4_K_M` decode within 2% of each other here, for 0.11 GB difference on disk.

## Observation

UD-Q2_K_XL nhỏ hơn 0.11 GB (0.39 so với 0.50 GB), nhưng decode gần như không cải
thiện: 30.7 so với 31.3 tok/s, chậm khoảng 2%. Với máy này em sẽ dùng Q4_K_M vì
chênh lệch dung lượng nhỏ, còn latency ổn định hơn. Em chưa so trực tiếp chất lượng
bằng hai server riêng, nên kết luận này chỉ dựa trên số liệu hiệu năng và dung lượng.
