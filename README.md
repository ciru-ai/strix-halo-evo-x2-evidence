# GMKtec EVO-X2 Strix Halo Evidence

This is a public community evidence artifact for GMKtec EVO-X2 / NixOS / IOMMU-on / NPU-aware Strix Halo benchmarking. It is shaped for the `hogeheer499-commits/strix-halo-guide` community evidence lane.

The useful story is not just that Strix Halo can run large local models. The evidence here is that an EVO-X2 can keep IOMMU on, use the NPU as a low-overhead sidecar, and still deliver strong speed-plus-quality results from tuned 27B, 35B, and Gemma QAT/MTP routes.

## System

- GMKtec NucBox_EVO-X2, Ryzen AI MAX+ 395, Radeon 8060S
- 128 GB class memory, 123 GiB visible to Linux
- BIOS VRAM setting: 2 GB, using UMA for the large shared memory pool
- NixOS 26.05 pre-release, Linux 7.0.1
- RADV STRIX_HALO, Mesa 26.0.5, Vulkan 1.4.341
- AMD Strix/Krackan/Strix Halo NPU exposed through `/dev/accel/accel0`
- IOMMU enabled with `iommu.passthrough=0`

IOMMU stays on because the NPU is useful as a separate always-on execution lane. The goal is to run sidecar work, such as cron jobs, Hermes tasks, and benchmark supervision, without materially slowing the main CPU/iGPU model path.

## Headline Evidence

### NPU Sidecar

In the 64k contention study, the main iGPU workload saw only **+3.29%** total latency with concurrent NPU load. A comparable concurrent iGPU auxiliary model caused **+68.96%** main total latency.

That is the practical NPU result: the NPU can carry useful background model work while preserving the iGPU for the main llama.cpp workload.

The FastFlowLM-NPU context matrix also shows why NPU footprint should be discussed with process RSS/HWM, not just sysfs VRAM/GTT counters. For example, [LFM2.5 1.2B](https://huggingface.co/LiquidAI/LFM2.5-1.2B-Instruct-GGUF) at 32k shows about **1646 prompt tok/s**, **38.18 decode tok/s**, and about **2.09 GiB RSS**.

### 27B Chadrock / Qwopus

[Qwopus3.6 27B Chadrock](https://huggingface.co/jcbtc/qwopus3.6-27b-v2-chadrock-rocmfp4-mtp) is one of the strongest results in this package.

On the 164-task HumanEval workload, the run reached **0.9451 HumanEval+** while recording:

- 45,033 completion tokens
- 1346.7948 s cumulative request latency
- 59.081965 tok/s mean total-token request speed
- 60.037325 tok/s median total-token request speed
- 33.437165 tok/s completion-only predicted speed
- 37.137693 tok/s peak active completion speed

The stored original [Qwopus3.6 27B v2 Q5_K_M](https://huggingface.co/Jackrong/Qwopus3.6-27B-v2-GGUF) HumanEval row recorded 3834 s generation time and 0.8963 HumanEval+. On the same 164-task workload, the Chadrock route is about **2.85x lower recorded request-generation time** while scoring higher.

[Qwen3.6 27B MTP ROCmFP4](https://huggingface.co/plunderstruck/Qwen3.6-27B-MTP-ROCmFP4-GGUF) also reached **0.9451 HumanEval+**, giving a second 27B quality anchor.

### 35B Chadrock / Ace Saber

The 35B tuned routes are central to the evidence, not side notes.

[CHADROCK3.6 35B Uncensored MTP Strix Lean](https://huggingface.co/jcbtc/CHADROCK3.6-35B-UNCENSORED-MTP-STRIX-LEAN) reaches about **0.91 HumanEval+** while quality-run speed rows show about **95.6** and **108.2 peak predicted tok/s**.

[Ace Saber 35B ROCmFP4 MTP](https://huggingface.co/jcbtc/chadrock-35b-ace-saber-rocmfp4-mtp) reaches **0.9024 HumanEval+** with about **104.35 peak predicted tok/s** and about **101.33 last-active predicted tok/s**.

Those rows are important because they pair high coding quality with over-90 and over-100 tok/s speed evidence.

### CrownV7

[CrownV7 Qwen3.6 35B dynamic route](https://huggingface.co/jcbtc/qwen3.6-35b-a3b-crown-halo-mtp-dynamic) anchors the tuned 35B long-context lane.

The public evidence includes:

- 128k prompt-processing evidence up to **515.33 tok/s**
- standalone TG around **60.71 tok/s** in the diagnostic row
- EvalPlus HumanEval+ **0.8902**
- BFCL v4 non-live accuracy **0.83**
- HermesAgent-20 later-run average **0.70**

This is the clearest stable tuned 35B route package: long-context prompt processing, usable decode, coding quality, tool/function-calling signal, and agent-workflow signal.

### Gemma 4 QAT / MTP

[Gemma 4 26B A4B QAT/MTP](https://huggingface.co/unsloth/gemma-4-26B-A4B-it-qat-GGUF) has a strong speed headline: a 512-token API row generated 512 tokens in 4319.5 ms with 150.2 ms TTFP, implying about **122.8 decode tok/s after TTFP** and about **118.5 end-to-end tok/s**.

The June 13 no-cache context sweep keeps the 26B QAT/MTP route over 100 tok/s at 4k and 8k, with **120.14 tok/s at 8k** and **96.36 tok/s at 16k**. Longer-context TTFP grows, so the clean 512-token row is the speed headline and the context rows are supporting evidence.

[Gemma 4 12B QAT/MTP](https://huggingface.co/unsloth/gemma-4-12B-it-qat-GGUF) answers the quality question for the fast n-max 5 route: HumanEval base **158/164**, HumanEval+ **153/164**, or **0.9329 HumanEval+**, with no harness failures, token caps, or length-finish rows. The speed evidence includes a best 512-token sweep row around **104.07 decode tok/s**.

[Gemma 4 31B QAT/MTP](https://huggingface.co/unsloth/gemma-4-31B-it-qat-GGUF) is included in the model-link map as part of the broader Gemma QAT/MTP family.

## Practical Takeaways

- Use the NPU as a sidecar for auxiliary work when preserving iGPU main-model latency matters.
- Vulkan/RADV is the practical default path for most of this evidence.
- ROCm and hybrid profiles are useful where measured, especially around specific ROCmFP4/tuned routes, but the public guidance should be evidence-specific rather than a blanket backend claim.
- Served/API measurements matter for MTP/speculative routes; standalone TG is not the whole user-facing speed story.
- 32k and 128k are the useful context planning points in this evidence.
- The strongest public story is speed and quality together: Chadrock/Qwopus 27B, Chadrock/Ace 35B, CrownV7, Gemma 26B, Gemma 12B, and the NPU sidecar lane.

## Files

- [manifest.md](manifest.md): system and benchmark-family appendix
- [MODEL_LINKS.md](MODEL_LINKS.md): Hugging Face links for named model routes
- [HTML evidence report](docs/strix-halo-guide-evidence-report-20260612.html): visual summary of the same public evidence
