# Experimental independent completion for Pico

Apply `independent-done.patch` at Pico commit [`f2117916f2a4cac0f79fa41680ebd39bf5d42b51`](https://github.com/Picom-code/pico-dgx-ds4/commit/f2117916f2a4cac0f79fa41680ebd39bf5d42b51). Build the rank binary as usual and add `--independent-done` to the serving command. The default stays off. This test ran one four-node recipe with drafting enabled. See the [Lab Note](../../notes/03-pico-independent-turns.md) for results and scope.

Patch SHA-256: `0b82dd326e646bb5bc0c6c564200fb4c74644e00c65a8243ddbfc7a0a364dd1b`. Build: CUDA 13.0.88, GCC 13.3.0, Release. The included MIT license is from the pinned Pico source.
