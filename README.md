# James’s Local AI Lab

I run local AI on hardware I can measure. I’m interested in how long it takes to get a patch that passes tests, whether an agent’s tool calls do what they should, and what happens when several people share a model. The experiments include the settings and the evidence behind the numbers.

## Lab Notes

- **[04: Will full GLM-5.3 K4 fit on four Sparks?](notes/04-glm53-k4-tp4-preflight.md)** The weights passed a small CPU split check. The TensorFold launch estimate exceeded the available memory, even with a compact core.
- **[03: When a finished turn waits for its batch](notes/03-pico-independent-turns.md)** Independent completion reduced the warm foreground tool wait on Pico TP4 from 28.7 seconds to 0.82 seconds.
- **[02: How many Hermes workers when someone is waiting?](notes/02-hermes-worker-tradeoff.md)** Eight background workers reached the first usable foreground tool call sooner. Sixteen cleared the same background queue faster.
- **[01: When batching helps local AI](notes/01-when-batching-helps.md)** Batching improved a synchronized short benchmark by 21.6%. In a separate replay of staggered agent traffic, it barely changed throughput.

## Find a project by hardware

| Hardware | What to read | Where it stands |
| --- | --- | --- |
| 1 DGX Spark / GB10 | [Qwen3-TTS independent test](https://github.com/jlacroix82/qwen3-tts-dgx-spark-independent-test) | Independent audio reproduction on one Spark. |
| 1 DGX Spark / GB10 | [FreeToken field guide](https://github.com/jlacroix82/freetoken-gb10-guide) | Historical setup; check the documented environment. |
| 2 DGX Sparks / GB10 | [Muse Spark Open on two Sparks](https://github.com/jlacroix82/muse-spark-open-2x-dgx-spark) | Measured serving recipe using llama.cpp RPC. |
| 8 DGX Sparks / GB10 | [DeepSeek 1017: concurrency, latency, and agent tests](https://github.com/jlacroix82/DeepSeek-V4.1-Flash-1017-SGLang-TP8-8x-DGX-Spark) | Original SGLang measurements and later vLLM and Hermes studies. The portable SGLang rebuild is still unqualified. |
| 8 DGX Sparks / GB10 | [DeepSeek 1017 in VeloGB10](https://github.com/jlacroix82/veloGB10-TP8-DeepSeek-1017) | Independent experimental port based on [sf-stav/veloGB10](https://github.com/sf-stav/veloGB10), with measured results and limitations. |

For agent workflows, see [Hermes dispatch and warm context](https://github.com/jlacroix82/DeepSeek-V4.1-Flash-1017-SGLang-TP8-8x-DGX-Spark/blob/07d2569b71fc11899752a87721a3c79cf3afe40d/docs/HERMES_BATCHING.md). For desktop and serving tools, see [vLLM and SGLang adapters for NVIDIA Personal AI Router](https://github.com/jlacroix82/pair-multi-engine) and [Unsloth Desktop on ARM64 Linux](https://github.com/jlacroix82/unsloth-desktop-aarch64).

## Upstream recipes I follow

- [DeepSeek 1017 on two Sparks with TensorFold](https://github.com/jlacroix82/DeepSeek-V4.1-Flash-1017-EXL3-TensorFold-2x-DGX-Spark), from [Elevatormusic’s recipe](https://github.com/Elevatormusic/DeepSeek-V4.1-Flash-1017-EXL3-TensorFold-2x-DGX-Spark).
- [DeepSeek V4.1 Flash on three or four Sparks with SGLang](https://github.com/jlacroix82/DeepSeek-v4.1-Flash-DGX-Sparks), from [Mia’s AI Lab](https://github.com/MiaAI-Lab/DeepSeek-v4.1-Flash-DGX-Sparks).
- [spark-ds41f for DeepSeek V4.1 Flash](https://github.com/jlacroix82/spark-ds41f), from [Christopher Owen’s recipe](https://github.com/christopherowen/spark-ds41f).

These links are to forks I follow. The original authors’ recipe numbers belong to their setups.

## How to read the results

A capped benchmark measures how quickly a server generates a fixed amount. A trace replay shows how it handles a recorded pattern of requests. A checked coding or tool task measures whether the work succeeded and how long that took. Each note says which question it answers and links to the original method and results.

If you reproduce a result or get a different one, include your hardware, exact model and engine versions, clock settings, and workload. Share it in [GitHub Issues](https://github.com/jlacroix82/local-ai-lab/issues) or join the [NVIDIA forum discussion](https://forums.developer.nvidia.com/t/deepseek-v4-1-flash-on-8x-dgx-spark-tp8-ep8-concurrency-results/387537).
