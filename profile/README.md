# ZeroStack RF Engineering (`@zerostack-rf`)

> **Official ZeroStack Overview & Firmware (`Release Zero (0.9.9-beta.1)`)**: [https://smlight.tech/software/zerostack](https://smlight.tech/software/zerostack)
> **Interactive Web Log Analyzer**: [https://smlight-tech.github.io/zerostack/](https://smlight-tech.github.io/zerostack/)
> **Technical Whitepaper — Part 1 (PDF)**: [Scaling Zigbee 3.0 / R23 Beyond the 100-Node Wall](https://smlight-tech.github.io/zerostack/whitepapers/zerostack-whitepaper-part-1.pdf)
> **Technical Whitepapers & Silicon Notes**: [https://github.com/smlight-tech/zerostack](https://github.com/smlight-tech/zerostack)

**ZeroStack** is a deterministic, O(1) Zigbee 3.0 / PRO R23 Coordinator & Router runtime engine engineered for **SMLIGHT `SLZB-06` and `SLZB-07` series coordinators** powered by the **TI SimpleLink F2 (`CC2652P`, `CC2652P7`, `CC1352P7`, `CC2674P10`)** platform.

ZeroStack **Release Zero (`0.9.9-beta.1`)** eliminates the architectural bottlenecks of legacy 2006-era OSAL/Z-Stack (`ZNP 3.x`) firmware:
- **O(1) In-RAM Security & Idempotent NVM State Deduplication**: Eliminates O(N⋅P) linear flash log scans during per-frame AES-CCM* security (`82.7 µs` encrypt / `79.7 µs` decrypt, `0` flash reads) and TCLK key exchange (`6.8 µs` vs `30.96 ms`, **4,559× faster**), while eliminating redundant flash writes on unchanged state (**26.7× faster**, `0 B` flash wear, **14.7× lower compaction frequency**).
- **Constant-Time O(1) Memory Manager (`218` cycles)**: Eliminates first-fit O(N) `OSAL_Memory` heap fragmentation (`0x10: MEM_ERROR`, `0x11: BUFFER_FULL`, `0xC7: NWK_TABLE_FULL`) with guaranteed 100% heap recovery (Δheap = 0 B).
- **Indirect Queue Quota Isolation & Zero Flash-Read Stalls in Sleepy Poll Windows**: Prevents sleeping or offline Zigbee End Devices (ZEDs) from hoarding the shared `nwkDB` buffer pool (`50%` pool cap, max `2` queued frames per sleeping child) while O(1) in-RAM security (`0` flash reads) ensures queued indirect frames are dequeued immediately while a polling ZED's RX window is open.
- **Zigbee 3.0 / PRO R23 Coordinator & Router Firmware**: Supports DMA-driven UART with **Live Auto-Baud (`115200`, `460800`, `1000000` baud)** and wire-speed recovery from hostile serial framing (`0` lost pings vs `8.84 s` blackout).

---

## Supported Hardware — Release Zero (`0.9.9-beta.1`)

| Coordinator Model | Radio SoC | SRAM / Flash | Hardware RTS/CTS |
| :--- | :--- | :--- | :--- |
| **`SLZB-06`** | `TI CC2652P` | `88 KB` / `352 KB` | No |
| **`SLZB-06U`** | `TI CC2652P` | `88 KB` / `352 KB` | Yes (`DIO10` / `DIO11`) |
| **`SLZB-06P7`** / **`SLZB-06P7v1`** | `TI CC2652P7` / `CC1352P7` | `144 KB` / `704 KB` | No |
| **`SLZB-06P7U`** | `TI CC2652P7` / `CC1352P7` | `144 KB` / `704 KB` | Yes (`DIO10` / `DIO11`) |
| **`SLZB-07P7`** | `TI CC2652P7` | `144 KB` / `704 KB` | No (USB Dongle) |
| **`SLZB-06P10`** | `TI CC2674P10` | `256 KB` / `1024 KB` | Yes (`DIO10` / `DIO11`) |
| **`SLZB-07P10`** | `TI CC2674P10` | `256 KB` / `1024 KB` | No (USB Dongle) |

---

## Verified Hardware Validation Benchmarks (`TI CC1352P7` @ `48.0 MHz`, `DWT->CYCCNT`)

| Benchmark (`1 µs = 48 cycles`) | Legacy TI Z-Stack (`ZNP 3.x`) | ZeroStack `Release Zero (0.9.9-beta.1)` | Improvement |
| :--- | :--- | :--- | :--- |
| **TCLK Lookup (300-entry miss / first contact)** | `1,486,228` cyc (`30.96 ms`) | **`326` cyc (`6.8 µs`)** | **4,559× faster** (`0` flash reads) |
| **TCLK Lookup (oldest-record hit, age 383)** | `620,396` cyc (`12.92 ms`) | **`324` cyc (`6.75 µs`)** | **1,915× faster** |
| **Per-Frame NWK Encrypt (`D=150`)** | `533,156` cyc (`11.11 ms`) | **`3,968` cyc (`82.7 µs`)** | **134.4× faster** (O(1)) |
| **Per-Frame NWK Decrypt (`D=150`)** | `532,182` cyc (`11.09 ms`) | **`3,824` cyc (`79.7 µs`)** | **139.2× faster** (O(1)) |
| **Routed Hop Security Ceiling (`D=150`)** | `22.19 ms` (`~45 pkt/s`) | **`162 µs` (`>6,100 pkt/s`)** | **135× higher ceiling** |
| **Per-Frame APS TX Security (`D=0..100`)** | `52,909–409,301` cyc (`8.53 ms`) | **`5,410–5,424` cyc (`113 µs`)** | **Up to 75.5× faster** |
| **NVM Compaction Frequency** | Write **#69** (`163.9 ms` stall) | Write **#1,013** (`13.7 ms` stall) | **14.7× lower frequency** |
| **Malformed Secured OTA Frames (12 classes)** | **Remote CPU `HardFault`** (`BusFault`) | **`0` crashes (`320–6,631` cyc)** | **Zero crash + up to 402× faster** |
| **1,000-Frame Replay Flood (`D=150`)** | `2.99 s` CPU lockup (`3.0 ms`/frame) | **`32.4 ms` total (`32 µs`/frame)** | **92.8× faster** |
| **Torn-Record Brownout Reboot Recovery** | `20,232 B` written (`62` page erases) | **`27 B` written (`0` page erases)** | **749× lower flash write volume** |
| **Hostile UART Framing (`300 × SOF` storm)** | **`8.84 s` serial blackout** (`44` lost pings) | **`0` lost pings (`3–55 ms` recovery)** | **Zero serial blackout** |

---

## Repositories & Tools

- **[`zerostack-rf/zerostack-mcp`](https://github.com/zerostack-rf/zerostack-mcp)**: Model Context Protocol (MCP) server for AI-assisted Zigbee/Thread log diagnosis, silicon register decoding, and ZeroStack hardware benchmark queries.
- **[`smlight-tech/zerostack`](https://github.com/smlight-tech/zerostack)**: Silicon root-cause diagnostics, hardware validation whitepapers (Part 1 & Part 2), and the [Interactive Web Log Analyzer](https://smlight-tech.github.io/zerostack/).
- **[Official ZeroStack Page](https://smlight.tech/software/zerostack)**: Firmware overview, release notes, and supported SMLIGHT coordinator matrix.
