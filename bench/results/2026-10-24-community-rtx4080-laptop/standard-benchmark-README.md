# Strata standard benchmark: NVIDIA GeForce RTX 4080 Laptop GPU

Protocol `strata-qwen38-thinking-v1-draft.1` (draft), run `20261024T182017Z-f4c30872`, started 2026-10-24T18:20:17Z. **Status: complete.**

Written by `strata bench run`. The machine-readable result is [results.json](results.json); every request, its raw usage and timings and the client event offsets are in [outputs.jsonl](outputs.jsonl).

## Hardware and software

- GPU: NVIDIA GeForce RTX 4080 Laptop GPU (12.0 GiB dedicated)
- CPU: 13th Gen Intel(R) Core(TM) i9-13950HX; RAM: 62.5 GiB
- Storage of the model files: not measured
- OS: Ubuntu 24.04.5 LTS 7.2.8-200.fc44.x86_64; drivers: gpu-driver 615.71.09; runtime: cuda 13.4
- Strata commit: `8183c6b67162340c09b67360860177c577a65f2e`; engine 0.1.49.1; release build
- Background work: not measured

## Model and configuration

- Model: qwen3.8-flash-next-iq3_s; quantization IQ3_S; revision not measured; source revision not measured
- Files: `Qwen3.8-Flash-Next-GSQ-RCO-IQ3_S-00001-of-00002.gguf` `4c1eb2ceb4915e11...`; `Qwen3.8-Flash-Next-GSQ-RCO-IQ3_S-00002-of-00002.gguf` `316b46f3a2dbd68c...`
- Track: **custom** (no notes given by the operator)
- Context 131072 tokens; server capacity 1 slot(s)
- Sampling: temperature 1.0, top_p 0.95, top_k 20, min_p 0.0, presence_penalty 0.0, repetition_penalty 1.0, reasoning_effort xhigh; thinking on, preserved
- Engine options: `config.json`; chat template: `provenance/chat_template.jinja`

## Method

The frozen requests of `bench/prompts/standard-v1` (corpus `e75eff8b9feea04f...`) were sent over loopback HTTP, streaming, one at a time, in this order: 1 warm-up (excluded), 3 repetitions of the short, 4K and 16K prompts, and the twelve sanity checks. Prompt and decode speed are the engine's own `timings`; time to first token is the client's monotonic clock from dispatch to the first nonempty reasoning, answer or tool-call delta. No request was retried. Time after the model loaded: 3 min 18 s.

## Results

| Case | Prompt tok/s | Decode tok/s | TTFT s | First answer s | Total s | Capped |
| --- | --- | --- | --- | --- | --- | --- |
| short | n/a | 66.5 (66.0-76.6) n=3 | 1.53 (1.40-1.58) n=3 | 2.64 (2.22-3.08) n=3 | 7.1 (6.4-9.3) n=3 | 1/3 |
| prompt-4k | 961.3 (958.9-968.2) n=3 | 58.5 (49.5-60.8) n=3 | 4.29 (4.26-4.30) n=3 | 9.60 (8.97-10.22) n=2 | 13.0 (12.7-14.6) n=3 | 3/3 |
| prompt-16k | 1292.0 (1291.2-1296.5) n=3 | 55.8 (52.2-56.8) n=3 | 12.72 (12.67-12.72) n=3 | not measured | 21.9 (21.7-22.5) n=3 | 3/3 |

Cells are median (min-max) n=samples. Short is a prompt of about 80 tokens: its prompt rate is dominated by the fixed cost of a request, so it is not shown; read prompt speed from prompt-4k and prompt-16k. Capped = requests that reached the 512-token cap, reasoning and answer together. A capped request's total time is the time to the cap, not to an answer, and it has no first-answer time when it was still reasoning.

| Phase | Planned | Completed | Truncated | Failed | Skipped |
| --- | ---: | ---: | ---: | ---: | ---: |
| warmup | 1 | 0 | 1 | 0 | 0 |
| performance | 9 | 2 | 7 | 0 | 0 |
| sanity | 12 | 12 | 0 | 0 | 0 |
| concurrency | 0 | 0 | 0 | 0 | 0 |

Sanity checks: **12 of 12**.

| Case | Result | Reason |
| --- | --- | --- |
| I1 | pass | the expected JSON value |
| I2 | pass | the expected JSON value |
| I3 | pass | the expected JSON value |
| I4 | pass | the exact expected text |
| R1 | pass | the exact expected text |
| R2 | pass | the exact expected text |
| R3 | pass | the exact expected text |
| R4 | pass | the expected JSON value |
| T1 | pass | the expected tool call |
| T2 | pass | the expected tool call |
| T3 | pass | the expected tool call |
| T4 | pass | the expected tool call |

## Failures, exclusions and limitations

- No request failed.
- Excluded from speed summaries: performance-prompt-16k-r1: first_answer_s is not available for this request
- Excluded from speed summaries: performance-prompt-16k-r2: first_answer_s is not available for this request
- Excluded from speed summaries: performance-prompt-16k-r3: first_answer_s is not available for this request
- Excluded from speed summaries: performance-prompt-4k-r3: first_answer_s is not available for this request
- Prompt-prefix reuse is the shared template prefix only; the cache was not emptied between requests, and the expert cache stays warm and evolves. These are not cold-model measurements.
- Output caps include reasoning; a request that reaches its cap is `truncated` and fails its sanity check.
- The runner cannot see other programs on the machine, the repeat-stop and loop-recovery switches of the server, or the quantization repository's revision; see `missing` in results.json for every value it could not read.
- The server had answered 1 request(s) before this run.
