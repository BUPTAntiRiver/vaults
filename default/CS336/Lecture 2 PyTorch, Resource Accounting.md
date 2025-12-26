## Tensor Memory
When learning programming languages, we know that we have at least two methods to store floating point values: `float`, `double`. `float` takes 32 bits, which is 4 bytes and `double` takes 64 bits, which is 8 bytes.
The bits consist of one *sign* bit, some *exponent* bits and some *fraction* bits. You may ask AI for more detail.
We usually use `float32` (32 bits) in deep learning, it is precise enough. But to save memory, we also produced `float16` and `bfloat16`, the former has better precision and the latter has larger range.
There are some bald choices like `float8`, which is more risky and unstable. So, in practice, mixed precision training is more common. e.g. we may use `float32` on accumulating operations like reduce or attention, while `bfloat16` on simple run through computations like feed forward.
## Compute Accounting
### Tensor Operations
Einops tutorial