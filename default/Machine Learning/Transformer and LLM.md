What do we have before Transformer?
# Pre-Transformer Era
## RNNs
- Limited efficiency, hard to model long-term relationships. It takes $O(\text{seq\_len})$ to interact with far tokens.
- Limited training parallelism, the states are strictly dependent on previous states.
## CNNs
Limited context.
# Transformer
## Tokenize words
A tokenizer maps a word to one/multiple tokens.
## Map tokens into embedders
**One-hot encoding**(not efficient): can be very long for large dictionary.
**Word embedding**: map the word index to a *continuous* word embedding through a *look-up table*, can be trained end-to-end.
## Word embedding through the model
### Multi-head attention
**Self-attention**:
$\text{Attention}(Q,K,V)=\text{softmax}(\dfrac{QK^T}{\sqrt{d_k}})V$
Project the embedding $E$ into query, key and value$(Q,K,V)$.
The query-key-value design is analogous to a retrieval system, let's take YouTube search as an example:
- Query: text prompt in the search bar
- Key: the titles/descriptions of videos
- Value: the corresponding videos
**Multi-heads** helps to learn more semantics of words.
### Feed-Forward Network (FFN)
There is no non-linear activation so far, then we use FFN to bring this functionality.
### Layer Normalization
## Positional encoding
- **Problem**: attention and FFN do no differentiate the **order** of the input tokens (unlike convolutions)
- Added to the raw word embedding to fuse in positional information
## Design Variants
### Encoder
- Encoder-Decoder
- Encoder-only
- Decoder-only
### Positional encoding
- Absolute
- Relative
### KV Cache Optimization
During Transformer decoding, we need to store the **Keys** and **Values** of all previous tokens so that we can perform the attention computation, namely the KV cache, but the KV cache can be very large for with long context. But we only need the **current** query token.
We can reduce the KV cache by reducing $\#\text{kv-heads}$.
- Multi-head attention (MHA): $N$ heads for query, $N$ heads for key/value
- Multi-query attention (MQA):$N$ heads for query, 1 heads for key/value
- Grouped-query attention (GQA): $N$ heads for query, $G$ heads for key/value (typically $G = N / 8$)
# Visual Transformer
## Basics of Visual Transformer (ViT)
**How to tokenize images?**
Convert the 2D image into a sequence of patches, then do a linear projection to flatten the patches.
## Efficient ViT
**Challenges**: *High resolution* is essential for achieving good performance, but ViT computation cost grows *quadratically* as the input resolution increases.
### Window attention
Instead of applying attention on the whole image, we separate the image into several windows and use attention on them to reduce computation cost. But here comes the **problem**, there are no cross window information. To solve this, we add a shift operation that shifts the window partition.
### Linear attention
Replace SoftMax attention with Linear attention.
$\text{Sim}(Q,K)=\exp(\dfrac{QK^T}{\sqrt{d}})\to \text{ReLU}(Q)\text{ReLU}(K)^T$
By change the order of matrix multiplication, reducing the complexity to $O(n)$.
But ReLU linear attention cannot produce sharp distributions. It is good at capturing global context information but not good at local ones.
How to **solve** this?
- Enhance linear attention by muti-scale aggregation
- Add depth-wise convolution in FFN to further enhance linear attention's ability to capture local context.
### Sparse attention
Sparse ViT learns to prune **irrelevant background** windows, while keeping **informative** windows.
## Self-supervised learning ViT
ViT needs **large amount** of data to train.
### Contrastive learning
### Masked Image Modeling
