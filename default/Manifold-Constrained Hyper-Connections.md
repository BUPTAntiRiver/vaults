Paper link: http://arxiv.org/abs/2512.24880

# Introduction

**Hyper-Connections** was introduced to replace **residual connection** in order to apply more control over the information passed down so that can improve model performance. From the result, we can see that it did it, but sacrificed training stability. This is because hyper-connections has no limit over the control. Let's examine the detail.

# Related Work

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

$$