# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Nguyễn Duy Dũng
**Cohort:** _A20-K3_
**Ngày submit:** 2026-08-20

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 10
- **CPU:** 11th Gen Intel Core i5-11400H @ 2.70GHz
- **Cores:** 6 physical / 12 logical
- **CPU extensions:** AVX2
- **RAM:** 15.8 GB
- **Accelerator:** NVIDIA GeForce GTX 1650 4 GB (run CPU-only, `ngl=0`)
- **llama.cpp asset đã tải:** `llama-b10488-bin-win-cpu-x64.zip`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** Q4_K_M + UD-Q2_K_XL (từ `models/active.json`)

**Chạy ở đâu:** laptop của em
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Máy có GTX 1650 nhưng CUDA runtime không phản hồi ổn định ở request đầu tiên, nên em
dùng prebuilt CPU runtime với `ngl=0`. Gemma tải không ổn định qua mạng, vì vậy em
dùng Qwen3.5 0.8B để hoàn thành phép đo local. Các số liệu trong báo cáo là CPU-only.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3176 | 432 / 571 | 32.0 / 58.2 | 2399 / 3536 / 3536 | 31.3 |
| UD-Q2_K_XL | 0.39 | 2812 | 556 / 694 | 32.6 / 43.2 | 2551 / 3279 / 3279 | 30.7 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

UD-Q2_K_XL nhỏ hơn 0.11 GB nhưng decode P50 chỉ đạt 30.7 tok/s, thấp hơn Q4_K_M
là 31.3 tok/s. Q4_K_M cũng có TTFT P50 thấp hơn (432 ms so với 556 ms). Em chưa
so trực tiếp chất lượng bằng hai server riêng; với phần chênh dung lượng nhỏ này,
em chọn Q4_K_M cho các test sau.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 1.04 | 7600 | 13000 | 15000 | 8.2 | 0.0% |
| 50 | 1.05 | 32000 | 47000 | 48000 | 31.7 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 1.01×
- **P95 tăng:** 3.62×
- **Effective concurrency ở 50 users:** 31.7 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.94 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hoà rõ ở 50 users: RPS chỉ tăng 1.01× khi offered load tăng 5×, trong khi
P95 tăng từ 13 lên 47 giây. Peak busy slots 3.94/4 và deferred requests cho thấy slot
đầy, nên latency tăng chủ yếu là queue time. Em sẽ sweep `--parallel` trước và đo
goodput@SLO, vì tăng slot trên CPU-only có thể làm contention nặng hơn.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost only | stub |
| N17 Data pipeline | không có batch job/DAG | stub |
| N18 Lakehouse | `TOY_DOCS` trong memory | stub |
| N19 Vector + features | keyword overlap | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.0 ms
- llm: 5909.4 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck, đúng với pipeline nhỏ dùng keyword overlap nên embed/retrieve gần
như không tốn thời gian. Nếu cần giảm latency 2×, em sẽ giới hạn output tokens trước
vì retrieval chỉ mất 0.0–0.1 ms. Em chỉ thử CUDA lại sau khi runtime ổn định và đo lại
được cùng cấu hình.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** giảm thread count từ 24 xuống 6

```
before:  17.1 tok/s (-t 24)
after:   37.5 tok/s (-t 6)
speedup: 2.19×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

Điểm tốt nhất nằm ở 6 threads, đúng bằng số core vật lý. Khi tăng lên 12 và 24
threads, tốc độ giảm từ 37.5 xuống 27.7 và 17.1 tok/s. Đây là thay đổi rõ nhất trong
thread sweep, nên em dùng -t 6 cho server và các phép đo sau.

Decode ở cấu hình này có vẻ bị giới hạn bởi memory bandwidth. Thread bổ sung không
tạo thêm băng thông mà tăng tranh chấp scheduler và cache, nên hyperthreading và
oversubscription không giúp workload này.

---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

Em bất ngờ vì tăng từ 6 lên 24 threads làm tg128 giảm từ 37.5 xuống 17.1 tok/s.

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [x] Repo GitHub ở chế độ **public**
- [x] Đã paste public URL vào VinUni LMS
- [x] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
