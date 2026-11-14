# How many Hermes workers when someone is waiting?

*Lab Note 02 · November 14, 2026*

With eight background workers, Hermes returned its first usable foreground tool call in **3.52 seconds**. With sixteen, that took **7.65 seconds**, but the background jobs finished at a **26% higher rate**. Both settings worked through the same 16 jobs. The larger setting let more of them start at once.

![At foreground arrival, 8 or 16 initial background requests had already been sent. Eight workers reached the first usable foreground tool call in 3.52 seconds versus 7.65 at sixteen. Sixteen completed background jobs at a pooled rate of 4,855 per active hour versus 3,859 at eight.](../media/hermes-worker-tradeoff.png)

## What the foreground request ran into

The trial ran DeepSeek V4.1 Flash 1017 on eight DGX Sparks with the GPUs capped at 2 GHz. Each wave started a queue of 16 independent Hermes jobs. Each job read a record, calculated a result, and saved it once. A worker slot opened as soon as a job finished, and the next job started if one was waiting. The separate foreground request went out about 150 ms after the background release.

By then, the client had sent eight background first requests at the eight-worker limit and all sixteen at the sixteen-worker limit, in every repeat. Both settings used the same short batching window for background requests while the scheduler was idle. The foreground request skipped that window, but it couldn’t preempt work already scheduled. More background requests were competing for the same engine at sixteen. That’s consistent with the longer wait; the test didn’t isolate how much came from queueing, prompt processing, or generation. Sixteen active jobs also didn’t mean one 16-request GPU batch.

## The tradeoff

| Background worker limit | Median to first usable foreground tool call | Median to checked foreground result | Checked background jobs per active hour |
| --- | ---: | ---: | ---: |
| 8 | **3.52 s** | **10.98 s** | 3,859 |
| 16 | 7.65 s | 11.93 s | **4,855** |

Sixteen cleared the same 16-job queue sooner in all three blocks: **12.13 seconds** at the median, versus **14.67 seconds** with eight workers. Both settings passed all 48 background jobs and all three foreground probes. The difference in foreground latency was clearest at the start: about **4.1 seconds** to the first usable tool call versus about **one second** to the checked result. Both clocks start when the foreground request is sent. “First usable” means the client had a tool name, call ID, and complete arguments it could parse.

The hourly rates are pooled from three short waves per setting and include model calls, queueing, and tool execution during those waves. They aren’t counts from an hour of production work.

For this setup, I’d start with eight background workers while someone is waiting and use sixteen to clear a queue of independent jobs. I’d check time to the first usable action and completed jobs separately before making either setting a default for longer tasks.

[Original methods and all five policies](https://github.com/jlacroix82/DeepSeek-V4.1-Flash-1017-SGLang-TP8-8x-DGX-Spark/blob/07d2569b71fc11899752a87721a3c79cf3afe40d/docs/HERMES_BATCHING.md#five-policy-comparison) · [Measured waves](https://github.com/jlacroix82/DeepSeek-V4.1-Flash-1017-SGLang-TP8-8x-DGX-Spark/blob/07d2569b71fc11899752a87721a3c79cf3afe40d/results/hermes-fleet-waves.csv) · [Chart data](../data/hermes-worker-tradeoff.csv) · [Back to the lab index](../README.md)
