# Taking Shortcuts (lossy)

We want to further improve model inference throughput or latency based on limited given resources, since we have already pushed batch size or sequence length to the limit, we must pay extra price for the power.

## Reduce Key Value Cache Size

Recall that memory is the bottleneck for inference, so let's reduce the size of KV cache, but make sure we don't lose too much accuracy.