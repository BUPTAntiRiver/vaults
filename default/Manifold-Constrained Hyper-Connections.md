Paper link: http://arxiv.org/abs/2512.24880

# Introduction

**Hyper-Connections** was introduced to replace **residual connection** in order to apply more control over the information passed down so that can improve model performance. From the result, we can see that it did it, but sacrificed training stability. This is because hyper-connections has no limit over the control. Let's examine the detail.

# Related Work

## Instability

Hyper connections just replace residual connections, which originally looks like this:

$$
\mathbf{x}_{l+1}=\mathbf{x}_{l}+\mathcal{F}(\mathbf{x}_{l},\mathcal{W}_{l})
$$

The HC version is defined as:

$$
\mathbf{x}_{l+1}=\mathcal{H}^{\text{res}}_{l}\mathbf{x}_{l}+{\mathcal{H}^{\text{post}}_{l}}^{\top}\mathcal{F}(\mathcal{H}^{\text{pre}}_{l}\mathbf{x}_{l},\mathcal{W}_{l})
$$

Formally, HC computes the coefficients as follows:

$$
\begin{cases}
\tilde{\mathbf{x}}_{l}=\text{RMSNorm}(\mathbf{x}_{l}) \\
\mathcal{H}^{\text{pre}}_{l}=\alpha^{\text{pre}}_{l}\cdot \text{tanh}(\theta^{\text{pre}}_{l}\tilde{\mathbf{x}}^{\top}_{l})+\mathbf{b}^{\text{pre}}_{l} \\
\mathcal{H}^{\text{post}}_{l}=\alpha^{\text{post}}_{l}\cdot \text{tanh}(\theta^{\text{post}}_{l}\tilde{\mathbf{x}}^{\top}_{l})+\mathbf{b}^{\text{post}}_{l} \\
\mathcal{H}^{\text{res}}_{l}=\alpha^{\text{res}}_{l}\cdot \text{tanh}(\theta^{\text{res}}_{l}\tilde{\mathbf{x}}^{\top}_{l})+\mathbf{b}^{\text{res}}_{l},
\end{cases}
$$

So we can conclude that HC has no constraints on the transforms with input signal. It may lead to unbounded signal amplification or attenuation, result in instability during large-scale training.

## System Overhead

HC also adds a lot of memory access costs, which constitutes one of the primary bottlenecks in modern model architectures. We need to calculate additional parameters and increases memory cost by a factor approximately proportional to $n$. Further more, HC requires $n$-fold more communication cost in pipeline parallelism, leading to larger bubbles and decreasing the training throughput.

# Method

## Manifold-Constrained Hyper-Connections

Drawing inspiration from the identity mapping principle of residual connection, the core premise of $m$HC is to constrain the residual mapping $\mathcal{H}^{\text{res}}$ onto a specific manifold.