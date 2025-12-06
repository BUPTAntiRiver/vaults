Traditional attention takes $O(N^{2})$ complexity, quadratic to sequence length, to compute, which is high time cost. So they proposed a method to change the attention from traditional *softmax* attention to a feature map based dot product attention, which achieved linear complexity while has pretty good performance.
# Dive into Transformer
Let $x \in \mathbb{R}^{N\times F}$ denote a sequence of $N$ feature vectors of dimension $F$. A transformer is a function $T:\mathbb{R}^{N\times F}\to \mathbb{R}^{N\times F}$ defined by the composition of $L$ transformer layers like:
$$
T_{l}(x)=f_{l}(A_{l}(x)+x)
$$
$f(\cdot)$ represents feed forward network. $A(\cdot)$ represents self attention function, the $x$ stands for residual connection. And the self attention is the key part to change.
The self attention function $A(\cdot)$ computes, for every position, a weighted average of the feature representations of all the other positions with a weight proportional to a **similarity score** between the representations. In traditional attention, such score is computed with softmax, since we only need a similarity score, we can use some other ways to do so.
# Linearized Attention
