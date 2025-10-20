# Latent Variable Models
## Training
### Objective
Latent Variable: $z$
Output: $x$
$$
\max_{\theta}\sum_{i}\log p_{\theta}(x^{(i)})=\sum_{i}\log \sum_{z}p_{Z}(z)p_{\theta}(x^{(i)}|z)
$$
Scenario 1: $z$ can only take on a small number of values $\to$ exact objective tractable
Scenario 2: $z$ can take on an impractical number of values to enumerate $\to$ approximate
### Prior Sampling
If $z$ can take many values $\to$ sample $z$
$$
\sum_{i}\log \sum_{z}p_{Z}(z)p_{\theta}(x^{(i)}|z)\approx \sum_{i}\log \frac{1}{K}\sum_{k = 1}^Kp_{\theta}(x^{(i)}|z_{k}^{(i)})
$$
$$
z_{k}^{(i)}\sim p_{Z}(z)
$$
Challenges:
Suppose we have $N$ clusters then when we are using a $N$ Gaussian model to model it. Then if we sample $z$ uniformly, it would only have $\frac{1}{N}$ terms being useful for each data point only lands in one of the clusters, but $z$ might be in any one of them.
When $z$ corresponds to many higher level properties, it becomes impossible for $z$ to match be a good match.
### Importance Sampling
#### Motivation and Background
**Problem setting**: want to compute $E_{z\sim p_{Z}(z)}[f(z)]$
But: 1. hard to sample from $p_{Z}(z)$; 2. samples from $p_{Z}(z)$ are not very informative, like where the distribution has a high density, the function has very low value, and in reverse.
Out latent variable model is example of problem 2.
#### Algorithm
**Formulation**:
$$
\begin{align*}
\mathbb{E}_{z\sim p_{Z}(z)}&=\sum_{z}p_{Z}(z)f(z)\\
&=\sum_{z} \frac{q(z)}{q(z)}p_{Z}(z)f(z)\\
&=\mathbb{E}_{z\sim q(z)}\left[  \frac{p_{Z}(z)}{q(z)}f(z)  \right] \\
&\approx \frac{1}{K}\sum_{k=1}^{K} \frac{p_{Z}(z^{(k)})}{q(z^{(k)})}f(z^{(k)}) \quad \text{with} \quad z^{(k)}\sim q(z)
\end{align*}
$$
Can sample from $q$ to compute expectation w.r.t. $p$.
Now the problem is how to choose $q(z)$?
$q(z)=p_{\theta}(z|x^{(i)})$ is a easy starting point, because it shows corresponding relationship between $z$ and $x$, but this is also very hard to get, so we should try to approximate this distribution.
**Optimize to find $q$**:
$$
\begin{align}
&\min_{q(z)}\text{KL}(q(z)||p_{\theta}(z|x^{(i)})) \\
=&\min_{q(z)}\mathbb{E}_{z\sim q(z)}\log\left(  \frac{q(z)}{p_{\theta}(z|x(i))}  \right)
\end{align}
$$
So this solution needs to solve such equation for *all* $x^{(i)}$, so we can develop a *amortized version*:
$$
\min_{\phi}\sum_{i}\text{KL}(q_{\phi}(z|x^{(i)})||p_{\theta}(z|x^{(i)}))
$$
Instead of setting up a separate mean and variance for each data point, we are going to use a neural network.
But the trade off is **faster and regularization** but **not as precise**.