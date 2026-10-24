# Community benchmark on a 12 GB RTX 4080 Laptop (sm_89)

Measured on 2026-10-24 on this laptop, in the Strata container from source (engine 0.1.49.1). The card is the
laptop RTX 4080 (compute capability **8.9**, 58 SMs, 12 GB) - a card the engine's own per-architecture tables say
"nobody measured" (docs/MMVQ_IL_TABLE.md). Its speed tables previously used the shared values measured on an
RTX 3060 / RTX 5070; this report adds an actual sm_89 measurement.

Main limitation: the configuration is **not the release defaults** - it is the tuned container configuration of this
machine (`track custom` under the standard benchmark), and decode is CPU/PCIe-bound here, so absolute numbers are
not comparable with the maintainers' default-config README rows. Run-to-run decode spread is about +-8% (draft
acceptance and RAM pressure); every mean is several runs.

## Hardware and software

- GPU: NVIDIA GeForce RTX 4080 Laptop GPU, 12 GB (12,282 MiB), sm_89, 58 SMs, PCIe 4.0 x8, on AC (~105-120 W)
- CPU: Intel Core i9-13950HX, 8 P + 16 E cores, 32 threads, no AVX-512 (AVX2 + AVX-VNNI)
- RAM: 62 GB (about 58-59 used while serving; the model is fully resident)
- OS: Fedora 44 (kernel of the day), NVIDIA driver 615.71.09, CUDA UMD 13.4
- Strata: engine 0.1.49.1, Terminal 0.2.4.1, built from source as a container image (CUDA 13.0, `CUDA_ARCHITECTURES=89`),
  served by the podman Quadlet `strata.service`; rootless, GPU via `--device nvidia.com/gpu=all`, no API key (this
  machine's long-standing choice)
- Model: Qwen3.8-Flash-Next IQ3_S (`Qwen3.8-Flash-Next-GSQ-RCO-IQ3_S`), 128K (131,072-token) context, MTP draft layer,
  resident expert arena (all ~47 GB of experts in RAM)

## Configuration (custom track)

Engine args (the saved `/data/install/strata-iq3_s.json`):
`--expert-cache auto --prefill auto --spec 4 --spec-min-p 0.5 --max-context 131072 --kv q4_0 --aux-cpus auto
--pool-workers 23 --adapt-async 1`
env: `STRATA_PREFILL_CPU_SHARE=0 STRATA_IQ3S_MT1=1 STRATA_EMB_REUSE_ACCOUNT=1`
an `api_monitor`-free server; shared defaults `reasoning_effort=medium`; `fit_max_tokens=true`,
`reasoning_budget_tokens=4000`.

Kernel contribution on this card (already submitted separately): the sm_89 row of the interleaved small-batch
matvec table (`native_mmvq.cu` `kIlRows_89`), measured here with `mmvq_il_parity --bench --emit-table`
(bit-exact, `mmvq_il_parity: OK`); it replaces the shared sm_86-based table for the dense projections of the
decode/verify windows.

## Method

- **Standard benchmark**: `strata bench run --url http://127.0.0.1:8080` (the draft `strata-qwen38-thinking-v1`
  protocol; thinking on, model-card sampling, 512-token cap). Fresh engine load, no other GPU jobs. Artifacts are
  the `standard-benchmark-*` files in this folder.
- **Quick check** (`docs/COMMUNITY_BENCHMARKS.md`, `tools/strata_bench.py` style): decode = three 256-token greedy
  prompts (thinking off); prefill = disjoint slices of the repo's own files at ~31K / ~52K tokens.

## Measurements

### Standard benchmark (thinking on, 512-token cap; engine's own timings, medians of 3)

| Case | Prompt tok/s | Decode tok/s | TTFT s | |
| --- | ---: | ---: | ---: | --- |
| short (~80 tokens) | n/a | 66.5 (66.0-76.6) | 1.53 | 1/3 capped |
| prompt-4K | 961.3 (958.9-968.2) | 58.5 (49.5-60.8) | 4.29 | 3/3 capped |
| prompt-16K | 1,292.0 (1,291.2-1,296.5) | 55.8 (52.2-56.8) | 12.72 | 3/3 capped |

Sanity checks 12/12 (instruction 4/4, retrieval 4/4, tool calls 4/4). Status `complete`; `track custom`.

### Quick check (thinking off; this machine's earlier, broader measurements)

| | median / mean | range |
| --- | ---: | ---: |
| Decode, 3 x 256-token greedy | ~62 tok/s | 55-77 (fresh starts at the high end) |
| Prefill ~31K tokens | ~1,615 tok/s | - |
| Prefill ~52K tokens | ~1,713 tok/s (1,528 without `STRATA_EMB_REUSE_ACCOUNT`) | - |
| Prefill ~100K tokens | ~1,687 tok/s | - |

## What was and was not tested

- Tested: decode and prefill at the sizes above; the standard suite's 12 sanity checks; quality spot-checks
  (arithmetic and reasoning answers correct); `mmvq_il_parity` bitwise parity of the sm_89 table.
- Not tested: the full 15-task quality suite; long-document needles beyond the standard suite's retrieval checks;
  Windows; a second sm_89 card; the release-default configuration (this machine runs its own tuned config).

## Reproducing

- Image: build the repo's `Dockerfile` for `CUDA_ARCHITECTURES=89` (see LOCAL_SETUP_NOTES.md in the owner's repo
  for the `STRATA_COMMIT` build-arg workaround and the keyless entrypoint override).
- Engine args, env and tuning in `LOCAL_SETUP_NOTES.md`; full `mmvq_il_parity --bench` output and the ncu kernel
  profile notes are recorded with the owner.
- Date of measurement: 2026-10-24, America/Chicago; laptop on charger at full performance governor.
