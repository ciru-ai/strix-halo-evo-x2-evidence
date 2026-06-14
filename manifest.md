# Public Manifest

Public community evidence artifact for GMKtec EVO-X2 / NixOS / IOMMU-on / NPU-aware Strix Halo benchmarking.

## System Snapshot

- System: GMKtec NucBox_EVO-X2, SKU EVO-X2-001, hardware version 1.0
- CPU/APU: AMD Ryzen AI MAX+ 395 w/ Radeon 8060S
- RAM: 123 GiB visible system memory, 128 GB class
- Firmware: EVO-X2 1.12, firmware date 2025-12-09
- OS: NixOS 26.05 (Yarara), build 26.05pre987561.1c3fe55ad329
- Kernel: Linux 7.0.1
- GPU: Radeon 8060S Graphics, RADV STRIX_HALO, Mesa 26.0.5, Vulkan 1.4.341 instance
- NPU: AMD Strix/Krackan/Strix Halo Neural Processing Unit at PCI c7:00.1, exposed as `/dev/accel/accel0`
- IOMMU: enabled in the boot path; current kernel cmdline includes `iommu.passthrough=0`
- BIOS VRAM setting: 2 GB; the workload relies on UMA access to the 128 GB class system memory pool rather than a large fixed VRAM carveout
- IOMMU rationale: keep an always-on NPU sidecar available for cron jobs, Hermes tasks, and benchmark supervision without materially slowing the main CPU/iGPU workload

## Public Files

- `README.md`: public evidence story and headline metrics
- `MODEL_LINKS.md`: Hugging Face links for named model routes
- `docs/strix-halo-guide-evidence-report-20260612.html`: visual evidence report
- `data/public_evidence.sqlite3`: compact public highlights database
- `data/public_evidence_metrics.csv`: CSV view of the compact public highlights database
- `data/db/llama-results.sanitized.sqlite3`: full llama.cpp throughput database, 1,687 benchmark rows
- `data/db/quality-results.sanitized.sqlite3`: full quality-results database, 1,029 quality rows
- `data/csv/`: CSV exports for the main public views

## Benchmark Families

- llama.cpp throughput measurements for prompt processing, token generation, context scaling, KV type, split mode, batch/ubatch, backend, and flash attention
- llama.cpp server/API measurements for served behavior, especially MTP/speculative routes
- NPU context matrix and FastFlowLM-NPU results
- NPU 64k contention/impact study
- [Qwopus](https://huggingface.co/Jackrong/Qwopus3.6-27B-v2-MTP-GGUF) / [Chadrock](https://huggingface.co/jcbtc/qwopus3.6-27b-v2-chadrock-rocmfp4-mtp) / [Qwen3.6](https://huggingface.co/plunderstruck/Qwen3.6-27B-MTP-ROCmFP4-GGUF) MTP settings, context, and served-speed tuning
- ROCm vs Vulkan served/backend comparisons
- [Ace Saber](https://huggingface.co/jcbtc/chadrock-35b-ace-saber-rocmfp4-mtp) / [Chadrock](https://huggingface.co/jcbtc/CHADROCK3.6-35B-UNCENSORED-MTP-STRIX-LEAN) ROCmFP4 and MTP tuning runs
- [Gemma 4 QAT/MTP](https://huggingface.co/collections/unsloth/gemma-4-qat) tuning and quality runs
- [CrownV7 Qwen3.6 35B](https://huggingface.co/jcbtc/qwen3.6-35b-a3b-crown-halo-mtp-dynamic) dynamic-route throughput and quality evidence
- EvalPlus, BigCodeBench, BFCL, HermesAgent-20, LiveCodeBench, and mini-swe-agent quality measurements

## Data Tables

- `llama-results.sanitized.sqlite3`
  - `benchmark_rows`: 1,687 rows
  - `llama_bench_comparable`: 1,315 rows
  - `llama_bench_strict`: 1,241 rows
- `quality-results.sanitized.sqlite3`
  - `quality_rows`: 1,029 rows
  - `quality_result_rows`: 1,029 rows
  - `quality_evalplus_aggregate`: 27 rows
- `public_evidence.sqlite3`
  - `results`: 10 curated public result summaries
  - `metrics`: 41 headline metrics

Paths are retained where they provide benchmark provenance. The public screen is focused on credentials and private payloads.
