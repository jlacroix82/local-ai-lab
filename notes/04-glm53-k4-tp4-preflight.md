# Will full GLM-5.3 K4 fit on four Sparks?

*Lab Note 04 · November 16, 2026*

[Mia picked K4 for full GLM-5.3](https://x.com/MiaAI_lab/status/2122328290543227267) and said her four-Spark recipe was still in development. I ran a memory preflight for the [same public checkpoint](https://huggingface.co/wrldsuksgo2mars/GLM-5.3-EXL3-K4-MC/tree/e28931c918fa89403b1cd247f89c7dc25eeb67ac) using a [published TensorFold TP4 branch](https://github.com/bertholomus/TensorFold/tree/ddfad356dc16f4d6bed9cfcc9d60cd0ebc0a3595). On my four Sparks, this branch did not clear the memory gate.

## Fit check

With the four GPUs idle, I measured available host memory. The CPU and GPU clock caps were set to 2 GHz. Keeping an 8 GiB available-memory floor left **95.6 GiB** for the runtime on the tightest node. I used the checkpoint tensor headers and the branch’s memory estimator for an 8,192-token window, one session, FP8 KV cache, and 2,048-token prefill chunks. The compact views share the same K4 experts and use smaller dense cores.

| Dense core | Estimated peak per rank | Estimated peak excluding draft-layer weights |
| --- | ---: | ---: |
| Original BF16 | 105.83 GiB | 103.75 GiB |
| K8 | 100.91 GiB | 99.01 GiB |
| K6 | 99.74 GiB | 97.84 GiB |

These are estimates from the pinned runtime, not observed GPU peaks. Omitting the draft layer is a separate estimate; switching off generation drafts alone does not remove those weights from the loader. [Per-rank measurements and estimates](../data/glm53-k4-tp4-fit.csv).

## What I verified

I read the gate and down projections of one real K4 expert from the pinned checkpoint. Using TensorFold’s CPU decoder and TP4 split rules, I compared the four reconstructed parts with an unsplit forward pass on the same three input rows. The maximum absolute difference was zero for the gate and `1.89e-15` for the down projection. [Sample identities and results](../data/glm53-k4-cpu-parity.json).

The memory gate stopped the trial before a full weight download or GPU launch. This checks a specific TensorFold layout; Mia’s upcoming recipe may allocate memory differently. It leaves serving speed, RoCE behavior, and coding quality open.
