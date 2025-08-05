# Probabilistic Latent Variable Models
## Probabilistic Models
Def: models that represents distributions.
Like $p(x)$ or $p(y|x)$, for example: if $y$ is linear with $x$ then we can model $p(y|x)$ by a Gaussian distribution with some noise. And we also have something that is conditional distributed in RL: the policy $\pi_{\theta}(\mathbf{a}|\mathbf{s})$.
## Latent Variable Models
Usually used in mixture distributions.
$p(x)=\sum_{z}p(x|z)p(z)$, $z$ here is the mixture element (latent variable), which decides categories.
### Latent Variable Models in General
$p(x)$ is a not simple distribution, $p(z)$ is simple, then $p(x|z)$ is also simple, e.g. $p(x|z)=\mathcal{N}(\mu_{\text{nn}}(z),\sigma_{\text{nn}}(z))$. Since $p(x)=\int p(x|z)p(z)\text{d}z$, we can represent a complex distribution with product and integral of two simple distributions.
### Latent Variable Models in RL
# Variational Inference
## The Variational Approximation
**How to calculate $p(z|x_{i})$?**
Approximate with $q_{i}(z)=\mathcal{N}(\mu_{i},\sigma_{i})$. This can bound $\log p(x_{i})$.
$$
\begin{align}
\log p(x_{i})&=\log \int_{z} p(x_{i}|z)p(z) \\
&=\log \int_{z}p(x_{i}|z)p(z)\frac{q_{i}(z)}{q_{i}(z)} \\
&=\log E_{z\sim q_{i}(z)}\left[\frac{p(x_{i}|z)p(z)}{q_{i}(z)}\right] \\
&\geq E_{z\sim q_{i}(z)}\left[ \log \frac{p(x_{i}|z)p(z)}{q_{i}(z)}\right]
\end{align}
$$
**Jensen's inequality:** $\log E[y]\geq E[\log y]$
# Amortized Variational Inference
# Generative Models: Variational Auto-encoders
