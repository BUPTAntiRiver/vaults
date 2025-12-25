A wrong interpretation to scaling law is: scale is all that matters, algorithms don't matter.
Right interpretation: algorithms that scale is what matters.
Accuracy = Efficiency $\times$ Resources
So the question is, what is the best model one can build given a certain compute and data budget? Which is, *maximum efficiency*.
## Basics
### Tokenization
In this lecture, we introduce Byte-Pair Encoding (BPE) tokenizer.
### Architecture
The starting point is Transformer, but since 2017, we have done a lot of things to make it work better:
- Activation functions: ReLU to SwiGLU
- Positional encoding: sinusoidal to [[RoPE]]
- Normalization: LayerNorm to RMSNorm
- Placement of normalization: pre-norm versus post-norm
- MLP: dense to [mixture of experts](MoE-Mixture-of-Experts)
- Attention: full, sliding window, liner, flash
- Lower-dimensional attention: group-query attention ([[Transformer and LLM#^GQA|GQA]]), multi-head latent attention ([[DeepSeek-V3#Multi-Head Latent Attention|MLA]])
- Challengers: state space models like [[Mamba]]

## Systems
