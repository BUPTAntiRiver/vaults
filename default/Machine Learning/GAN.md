# What is GAN?
The full name of GAN is **generative adversarial network**. Because it uses the generative adversarial training method, period.
## Generator
GAN is a **generator** notating with $G$, so it is different from the classifier we have met before. We need to generate samples and try to make them as close to the target data distribution as possible.
In order to build a training system, we need to define a "distance" between generated sample and target data.
It's objective is: $\max_G\{-E_{z\sim\text{noise}}\log(1-D(G(z)))\}$
## Discriminator
**Discriminator** will do this job. Discriminator is able to tell whether a data is fake or real. But we don't have a discriminator now, so we treat it as part of the model and we will train a discriminator $D$.
It's objective is: $\min_D\{-E_{x\sim\text{Data}}\log D(x)-E_{z\sim\text{noise}}\log(1-D(G(z)))\}$
The workflow of GAN looks like below:
![[GAN.png]]
Put them together it became a "minmax" function.
# Conditional GAN
Both generator and discriminator receive a new input like a label or other conditional information so that it won't generate random data.