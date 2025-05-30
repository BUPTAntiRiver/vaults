# lecture 1

**5 main parts**

- Supervised learning
- Machine learning methods
- Unsupervised learning
- Deep learning
- Reinforcement learning

# lecture 2

## Supervised learning

The job of **learning algorithm**: take in testing data as input and produce a "hypothesis" as output.
The job of the "hypothesis": take in new data and produce an estimate output $\hat{y}$.
The inputs $x$ are often called **features**.
$$h_{\Theta}(x)=\sum_{j=0}^ix_j\Theta_j,x_0 = 1.$$
Choose $\Theta$ such that $h(x)\approx y$, by minimizing the cost function(loss function)
$$J(\Theta)=\frac{1}{2}\sum^m_{i=1}(h_\Theta(x^{(i)})-y^{(i)})^2$$

## Gradient descent

Start with some value of $\Theta$, we want to minimize the function $J(\Theta)$ as we said above. Keep changing $\Theta$ to reduce $J(\Theta)$.
$$\Theta_j:=\Theta_j-\alpha\frac{\partial}{\partial\Theta_j}J(\Theta)$$
$\alpha$ is the learning rate, the speed we update $\Theta$.
[[cs229-notes1.pdf#page=4&selection=120,0,140,1|update algorithm]]

For a single training example, this gives the update rule:
$$\theta_j := \theta_j+\alpha(y^{(i)}-h_\theta(x^{(i)}))x_j^{(i)}$$[[cs229-notes1.pdf#page=5&selection=4,0,41,1|LMS update rule]]
The rule is called **LMS** update rule("least mean squares"), also known as the **Widrow-Hoff** learning rule.
To apply this method on a whole training set we can modify it in 2 ways:

1. [[cs229-notes1.pdf#page=5&selection=104,0,155,1|Batch gradient descent]]
2. [[cs229-notes1.pdf#page=7&selection=57,0,66,0|Stochastic gradient descent]]
   Difference between stochastic gradient descent and batch gradient descent:

- The former is much quicker and has a good enough result so stochastic gradient descent is much more popular in practice.

# lecture 3 Locally Weighted & Logistic Regression

"**Parametric**" learning algorithm: fit fixed set of parameters to data;
"**Non-parametric**": the amount of data/parameters you need to keep grows linearly with the size of the data.
To evaluate $h$ at certain $x$:

- **LR**: Fit $\theta$ to minimize cost function
- **Locally weighted regression**: Fit $\theta$ to minimize a modified cost function $$\sum_{i=1}^mw^{(i)}(y^{(i)}-\theta^\top x^{(i)})$$where $w^{(i)}$ is a "weighted" feature$$w^{(i)}=\exp(-\frac{(x^{(i)}-x)^2}{\tau^2})$$

## Justification for lecture 2

Probabilistic interpretation of linear regression: Why least squares?
[[cs229-notes1.pdf#page=11&selection=295,0,295,28|Probabilistic interpretation]]

## Classification problem

### Logistic regression:

**new term**: _sigmoid_ or _logistic_ feature $$g(z)=\frac{1}{1+\exp(-z)}$$
$P(y|x;\theta)=h(x)^y(1-h(x))^{1-y},y\in\{0,1\}$.

## Newton's method:

Have $f$.
Want to find $\theta$ such that $f(\theta) = 0$.

# lecture 4

## Perceptron

Not used in practice because it can only solve a really small amount of problem.

## Exponential family

## GLM(generalized linear models)

**Assumptions / Design Choices**:

1. $y|x;\theta \sim \text{Exponential family}(\eta)$
2. $\eta = \theta^\top x,\theta \in \mathbb{R}^n,x\in \mathbb{R}^n$
3. Test time: output $\mathrm{E}[y|x;\theta]$ which is the hypothesis function

## Softmax regression

Can be used in multi-class classification.
**Cross entropy interpretation**

- $K = \#\text{classes}$
- $x^{(i)}\in\mathbb{R}^n$
- label $y$ uses one-hot code

# lecture 4 GDA & Naive Bayes

## New method: Generative Learning Algorithms

- Gaussian Discriminant Analysis(GDA)
- Generative & Discriminant comparison
- Naive Bayes

## Gaussian Discriminant Analysis

## Naive Bayes

Example: e-mail classification problem
First question, what is Feature vector $x$?
Answer: mapping the piece of e-mail to a vector by building a dictionary.
$x\in \{0,1\}^n, x_i=1\{\text{word}\ i\ \text{appears in e-mail}\}$
