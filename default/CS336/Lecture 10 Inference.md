# Taking Shortcuts (lossy)

We want to further improve model inference throughput or latency based on limited given resources, since we have already pushed batch size or sequence length to the limit, we must pay extra price for the power.

## Reduce Key Value Cache Size

Recall that memory is the bottleneck for inference, so let's reduce the size of KV cache, but make sure we don't lose much accuracy.

### GQA

Group Query Attention, for a group of queries, now we only have 1 key value cache. Reduce memory cost significantly.

### MLA

Project each key and value vector from $N\times H$ (head numbers and head dimensions) dimensions to $C$ dimensions. A wrinkle is: MLA is not compatible with RoPE, so we have to add additional 64 dimensions for RoPE.