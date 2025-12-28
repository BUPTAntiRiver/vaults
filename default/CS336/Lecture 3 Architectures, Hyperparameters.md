## Architectures

### RMSNorm

Let's look at the popular RMSNorm, it simply removed the bias term and mean term comparing to layer norm but it keeps the same performance. It reduced some FLOPs but the key part is it reduced memory usage of bias and mean.
This is because though FLOPs of normalization takes only a tiny part of total FLOPs, with less than 1%, but the run time we need for normalization can take 25%. A lot of memory transfer lead to such result.

### More generally

Most modern transformers don't have bias terms in order to use less memory and improve optimization stability.

### Activations

We have a whole zoo of activations now: ReLU, GeLU, SwiGLU, etc.
What are they? What is the difference? Does it matter?
Most modern models used *gated activations* (GLU).
GLUs modify the first part of a feed forward layer:
$$
\text{FF}(x)=\max(0, xW_{1})W_{2}
$$
Instead of a linear + ReLU, augment the above with an (entriywise) linear term:
$$
\max(0, xW_{1})\to\max(0,xW_{1})\otimes(xV)
$$
This gives the gated variant, notice that we have an extra parameter $V$.
So the difference between GeGLU and SwiGLU is that they use different function for non-linearity.

### Position Embedding

Sine embedding, absolute embedding, relative embedding, etc.
Nowadays, seems RoPE has won the game. Its idea is that we only care about the relative distance between two tokens, and it is very similar to inner product of two vectors, if the angle between two vectors stays the same, then their inner product won't change. They turn the offset of position into rotation, so that if the relative distance of two token does not change, their embedding relation won't change.
The rotation in 2 dimension is straight forward, but our token has huge embedding dimension, so the RoPE inventors find an simple way which is breaking down the token vector into pieces of 2 dimension and embed them separately.

## Hyperparameters
