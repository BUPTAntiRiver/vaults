# 1-D Flow Models
## Summary
### Terms
$x$: **Original data space**, complex distribution to model
$z$: **Embedding space**, we enforce a simpler distribution $z\sim p_{Z}(z)$
$f_{\theta}$: Transformation to $x$, where $x = f_{\theta}^{-1}(z)$
### Training
$$
\max_{\theta}\sum_{i}\log p_{\theta}(x^{(i)})=
\max_{\theta}\sum_{i}\log p_{Z}(f_{\theta}(x^{(i)}))
+\log\left|\frac{\partial f_{\theta}}{\partial x}(x^{(i)})\right|
$$
### Inference
Evaluate training object.
### Sampling
First sample $z$ from $z\sim p_{Z}(z)$ then compute $x=f_{\theta}^{-1}(z)$
## Flows are general
We can transform any distribution into uniform by using the CDF of that distribution. So we can transform $x$ to $u$ with CDF of $x$ and then transform $u$ to $z$ with inverse of CDF of $z$.
#### Proof to the transformation
$$
\begin{align}
Z&=F_{\theta}(X) \\
F_{Z}(z)&=P(Z\leq z) \\
&=P(F_{\theta}(X)\leq z) \\
&=P(X\leq F_{\theta}^{-1}(z)) \\
&=F_{\theta}(F_{\theta}^{-1}(z)) \\
&=z
\end{align}
$$
Which means  $F_{Z}(z)$ is uniform distribution.
# 2-D Flows
## Auto-regressive Flow
The sampling process of a Bayes network is a flow.

# N-D Flows
## AF and IAF
# Dequantization
Add noise to discrete samples.