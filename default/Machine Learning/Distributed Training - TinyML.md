Models are too big, so we need multiple devices to train them.
# Parallelization Methods
## Data Parallelism
Same model on $N$ GPU, split the Training Dataset across them.
Two different *roles* in framework:
- **Parameter Server**: receive gradients from workers and send back the aggregated results
- **Workers**: compute gradients using split dataset and send to parameter server
Compare with single node training, DP has two more synchronization steps:
```
For iteration i in T:
	1. Replicate / Pull gradients form parameter server
	2. Sample data from datasets x(i, j), y(i, j)
	3. Compute gradients w(i, j)
	4. Push gradients to parameter server and sum
	5. Perform gradient step: w(i, j) = w(i, j) - lr * avg(w(i))
```
### Communication Primitives
**Send**: send from $n_{0}$ to $n_{1}$
**Receive**: receive from $n_{1}$ to $n_{0}$
**Scatter**: split a tensor to all other workers, e.g. $[1,2,3,4]$ to $[1],[2],[3],[4]$
**Gather**: the reverse of scatter
**Reduce**: a bit different from gather, e.g $[1],[2],[3],[4]$ to $[1+2+3+4]=[10]$
**Broadcast**: send identical copies to all other workers
**All-Reduce**: perform reduce on all workers
**All-Gather**: perform gather on all workers
### Use the primitives
So how can we describe the communication schemes used in DP? Broadcast then Reduce! But if we check the bandwidth requirement we will find out that each worker demands $O(1)$ while Parameter Server requires $O(N)$, this can be a **bottleneck**.
Can we perform the aggregation without a central server? **ALL-REDUCE!** However naive All-Reduce will require $O(N)$ time and bandwidth on all workers, but we have better implementations like **Ring** and **Parallel Reduce**. Ring has $O(N)$ time and $O(1)$ bandwidth, Parallel Reduce has $O(1)$ time and $O(N^2)$ bandwidth. There is also a method called **Recursive Halving** which achieves $O(\log N)$ time.
### DeepSpeed - Reducing Memory
Even the best GPU cannot fit the model weights of current large language models, so we have DeepSpeed ZeRO-1/2/3 which proposes to partition **optimizer states, gradients and weights**, so that each GPU only needs to store part of the data. In PyTorch, ZeRO-3 is implemented via `FullyShardedDataParallel`, known as FSDP.
## Pipeline Parallelism
Single copy of data, but split the model across the GPU.
We can imagine the naive implementation be like, we have 4 GPU, and they run forward 1 to 4 sequentially, then backward 4 to 1 sequentially, this works but will have huge amount of idle time, the utilization will be very low.
How to improve? We can separate the batch into micro batches: $[16,10,512]\to 4*[4,10,512]$ this improves more working overlap, which elevates utilization from $25\%$ to around $57\%$.
## Tensor Parallelism
Single copy of data, split the activations then synchronize.
Even with optimization, PP still has a lot of idle time, we can keep improving this by making the model partition **more fine-grained**. Which leads to Tensor Parallelism, split a weight tensor into $N$ chunks and paralleled.
## Sequence Parallelism
Suppose we have long text for training and it looks like: $[\text{Batch Size, Tokens, Hidden Dimension}]$.
Data parallelism splits batch size, while sequence parallelism splits tokens.
Sequence Parallelism in fully connected layers is just like Data Parallelism, but in attention layer it becomes different, because the computation is quadratic about sequence length. We need all to all communication to solve this.
## Solution 1: Head partition
We will have head $1,\dots ,d$ of $Q,K,V$ about the partitioned token in each GPU. So we can only compute one meaningful result for each head, in total we can get $N$ attention. But we need to recompose them, and that cost is huge. We propose all to all communication to get all $Q,K,V$ about one head in one GPU, so in the GPU itself it can compute all attention of this head, and greatly reduced communication cost. Think it like arranging $Nd$ compare to arranging $N^{2}$ objects.
## Solution 2: Ring Attention
After each computation, we switch $K,Q$ with each other, so after a round, we can compute all the attention, and this also only needs to arrange $Nd$ objects.