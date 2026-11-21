# Inkling 2 Mini on one GB10 at 2 GHz

*Lab Note 05 · November 20, 2026*

On one GB10 with 2 GHz CPU and GPU caps, enabling DSpark raised median end-to-end output at 16 concurrent code requests from **131.0 to 191.6 tok/s**, a 46% gain. The same code workload's first-token p95 rose from **5.89 to 12.27 seconds**. Across the two modes I measured 176 timed waves, 686 request streams and 24 checked coding and tool attempts.

![Inkling 2 Mini on one GB10. Code throughput and first-token latency by concurrency, long-prompt cache replay, C4 checked work, and thermal results.](../media/inkling2-mini-gb10.png)

## Throughput and first-token latency

Each scaling wave sends distinct 1,024-token prompts and generates 256 tokens per response. The code prompts draw from Python sources and the documentation prompts from technical Markdown in the llama.cpp tree. Each table cell shows **DSpark off → on**. Rates are medians across three waves per mode at C1–C8 and five at C16. Latencies are pooled request medians.

| Concurrent requests | Code output (tok/s) | Code first token (s) | Docs output (tok/s) | Docs first token (s) |
| ---: | ---: | ---: | ---: | ---: |
| 1 | 33.1 → 59.8 | 0.39 → 0.41 | 32.9 → 71.1 | 0.42 → 0.43 |
| 2 | 48.1 → 65.7 | 0.63 → 0.79 | 48.5 → 70.9 | 0.65 → 0.79 |
| 4 | 68.3 → 93.6 | 1.49 → 1.48 | 71.3 → 125.2 | 1.49 → 1.52 |
| 8 | 94.7 → 144.6 | 2.52 → 2.55 | 96.6 → 129.0 | 2.54 → 2.57 |
| 16 | 131.0 → 191.6 | 4.31 → 4.68 | 131.7 → 208.1 | 4.34 → 4.41 |

C16 code waves ranged from **128.8–132.8 tok/s** with DSpark off and **165.4–222.9 tok/s** with it on. The median shared-decode rate on code was 34.7 → 67.2 tok/s at C1 and 157.8 → 317.4 tok/s at C16. Across 24 shorter C1 probes per mode with 512 input and 96 output tokens, first token was 0.25 → 0.28 s at the median and 0.26 → 0.30 s at p95. Those probes delivered 32.6 → 45.8 tok/s end to end.

On larger code prompts, C8 at 2,048 input tokens delivered 83.4 → 114.1 tok/s and a 4.76 → 4.74 s median first token. C4 at 8,192 input tokens delivered 38.5 → 51.3 tok/s and a 9.39 → 9.41 s median first token. Both lanes generated 256 tokens per response. [All speed cells](../data/inkling2-mini-gb10-speed.csv) · [Timed waves](../data/inkling2-mini-gb10-waves.csv) · [Request timings](../data/inkling2-mini-gb10-requests.csv).

## Context, cache reuse, and foreground delay

The context ladder sends one request at a time and generates 128 tokens. Each measured size has three waves per mode. The cache column records the median reported hit for an exact replay.

| Input tokens | Cold first token (s), off → on | Replay first token (s), off → on | Cached tokens on replay |
| ---: | ---: | ---: | ---: |
| 8,192 | 3.03 → 3.04 | 0.17 → 0.16 | 8,128 |
| 16,384 | 6.22 → 6.16 | 0.16 → 0.17 | 16,320 |
| 30,720 | 12.14 → 12.03 | — | — |

Appending 32 tokens to the 8K prompt kept the 8,128-token hit and reached first token in 0.19 → 0.17 s. The foreground probe used a 512-token request. I ran it alone, then sent it about 150 ms behind three cold 4,096-token requests. DSpark off went from 0.26 s idle to 4.58 s busy. DSpark on went from 0.28 s idle to 4.60 s busy. All four figures are medians of three waves. [Cache and foreground requests](../data/inkling2-mini-gb10-requests.csv).

## Checked coding and tool work

The pack contains three Python fixes with external assertions and three stateful tasks that read a local JSON fixture and save a checked result. The harness verifies the final answer and exactly one successful write per passed job. I ran the six tasks once per worker limit and mode using the chat endpoint, low reasoning effort, up to four turns, 768 generated tokens per turn, and a 100-second task budget.

| Worker limit | DSpark | Passed | First complete tool call p50 | Checked completion p50 | Checked tasks/min |
| ---: | --- | ---: | ---: | ---: | ---: |
| 1 | Off | 6/6 | 3.07 s | 12.97 s | 4.45 |
| 1 | On | 6/6 | 0.57 s | 4.31 s | 9.72 |
| 4 | Off | 6/6 | 5.76 s | 21.59 s | 8.36 |
| 4 | On | 5/6 | 9.25 s | 18.77 s | 8.21 |

The latency medians use successful attempts. Checked tasks/min includes the time spent on every attempt. One DSpark C4 Python fix reached the turn token limit midway through its submission call and made zero writes. Each C4 result comes from one six-task run. [Workflow summaries](../data/inkling2-mini-gb10-tasks.csv) · [Every attempt](../data/inkling2-mini-gb10-task-attempts.csv).

## Runtime and measurement details

Both runs used the official Thinking Machines checkpoint at `ed40e7da625f594fc9c4990f5f66be24873c22d7` and the same pinned ARM64 vLLM image (`0.1.dev25855+g7cb5dae22`). The text-only profile used native NVFP4/MXFP8, a 32,768-token context limit, up to 16 active sequences, a 4,096-token batch budget, a 0.50 GPU memory utilization target and prefix caching. DSpark used seven draft tokens with probabilistic draft sampling. I ran DSpark on first and the control second, with the client on the serving host. [Model card](https://huggingface.co/thinkingmachines/Inkling-2-Mini) · [vLLM recipe](https://recipes.vllm.ai/thinkingmachines/Inkling-2-Mini) · [Pinned runtime and checkpoint hashes](../data/inkling2-mini-gb10-results.json).

| Observed resource or runtime metric | DSpark off | DSpark on |
| --- | ---: | ---: |
| Server startup to API readiness | 143.4 s | 227.5 s |
| Engine-reported KV capacity | 148,089 tokens | 121,754 tokens |
| Maximum active requests in engine logs | 16 | 16 |
| Minimum available RAM | 43.7 GiB | 44.8 GiB |
| Peak GPU temperature | 60°C | 60°C |
| Peak ACPI temperature | 63.2°C | 64.5°C |
| Peak driver-reported GPU draw | 32.8 W | 33.1 W |

The guards used an 8 GiB available-memory floor, an 80°C GPU cutoff and a 90°C ACPI cutoff. Available RAM and temperature include the effect of resident host services. Power comes from sampled GPU driver readings.

- **End-to-end output** divides native generated tokens by the interval from first client send to last terminal response. The speed lanes use fixed output budgets, greedy sampling and native token IDs. Cold lanes verify zero cached input tokens. Timed rows exclude warmups and startup.
- **Shared decode** counts native tokens in the middle 80% of the interval when every stream is generating. First-token latency starts at client send and stops at the first streaming event with a generated token ID. The speed CSV includes p95 for cells with at least 20 request observations.
- **Workflow completion** includes parsing the model's tool calls, running the fixture tools and checking the result. First-tool latency stops when the streamed tool-call turn ends and the harness can run the parsed call.
- **Output agreement** was exact on 34 of 343 pairs with matched input token IDs and greedy sampling. Within each mode, all six exact prefix replays also returned a different sequence from their cold counterpart. All 686 timed streams returned their expected token count. Across the speed campaign, aggregate engine counters recorded 61,423 accepted draft tokens out of 149,632 proposed, including warmups between timed blocks. The source of the output differences remains open.

[Complete results and configuration](../data/inkling2-mini-gb10-results.json) · [Lab index](../README.md).
