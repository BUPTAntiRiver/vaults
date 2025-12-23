We are going to talk about four parts about diffusion model in this little article:
- Forward process: add noise to data
- Reverse process: learn to denoise
- Training objective: from Hierarchical VAE to L2 loss
- Noise Conditional Network: represent distributions

# What is Noise?
Adding Gaussian noise to a data point equals to sampling $x\sim\mathcal{N}(x\mid x_{0},\sigma)$, for data distribution, we are doing a convolution (of PDF) between $p_{\text{data}}(x)$ and normal distribution.
The process of adding noise is like doing convolution repeatedly until it becomes a noise distribution.

# Forward Process
From initial distribution $x_{0}$ to final noisy distribution $x_{T}$, the distribution passed a lot of steps. In each step we have:
$$
x_{t} = \sqrt{ 1-\beta_{t} } x_{t-1}+\sqrt{ \beta_{t} } \epsilon,\quad \epsilon\sim\mathcal{N}(0,\mathbf{I})
$$
The coefficients are designed to preserve variance. $t$ i called "schedule", key to Diffusion Model's success. Former part is mean of $x_{t}$, later part is std of $x_{t}$. So we can rewrite the equation as a conditional distribution:
$$
q(x_{t}\mid x_{t-1})=\mathcal{N}(x_{t}\mid \sqrt{ 1-\beta_{t} }x_{t-1},\beta_{t}\mathbf{I})
$$
If we concatenate transitions from $x_{0}$ to $x_{t}$:
$$
\begin{align*}
q(x_{t}\mid x_{0})&=\mathcal{N}(x_{t}\mid \sqrt{ \bar{\alpha}_{t} }x_{0}, (1-\bar{\alpha}_{t})\mathbf{I}) \\
\text{where } \alpha_{t} &:=1-\beta_{t}\\
\bar{\alpha}_{t}&:=\prod ^{t}_{s=1}\alpha_{s}
\end{align*}
$$
$\beta_{t}$ is some function about $t$, it might be linear or cosine, etc.
**In short**, forward process has predefined conditional distributions (schedule $\beta$), Gaussian with controllable mean and std, *divide* and conquer strategy (time step)
# Reverse Process
We use a parameterized network to model the reverse process:
$$
p_{\theta}(x_{t-1}\mid x_{t}) \text{ models unknown target } q(x_{t-1}\mid x_{t}) 
$$
Wait, why it is *unknown*?
$$
q(x_{t-1}\mid x_{t}) = \frac{q(x_{t-1},x_{t})}{q(x_{_{t}})}=\frac{q(x_{t}\mid x_{t-1})q(x_{t-1})}{q(x_{t})}
$$