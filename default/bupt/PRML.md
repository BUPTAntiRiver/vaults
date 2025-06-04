# Lecture 1 介绍

什么是机器学习？
从拥有某种表现度量方法$P$的任务$T$的经验$E$中学习，并且能够提升$T$在$P$下的表现。

## 机器学习的种类

1. Supervised learning **有监督学习**
   - $E$ are feature label pairs. 输入$x_i$和输出$y_i$**对**。当$y_i$是**离散**值时，称为**分类**问题 classification，当$y_i$是**连续**值时，称为**回归**问题 regression。
2. Unsupervised learning **无监督学习**
   - $E$ are only inputs. 数据**只有**输入$x_i$，目标是发掘出输入中的某些模式。
3. Reinforcement learning 强化学习
   - 课上没讲。

## 机器学习方法

- Generalization 泛化（训练）
  1.  收集数据
  2.  选择模型
  3.  估计参数
- Inference 推理，找到问题的最优解。

---

# Lecture 2 决策树

以数据的 Attribute 为节点，根据 Attribute 不同的值进行分割，生成相应数量的新节点，可以继续用其他的 Attribute 分割，或者以分割得到的子集中**个数最多**的元素作为节点的 Value 生成一个叶子节点。当分割结束决策树生成完毕。
以上生成过程中，还有一个步骤没有确定，就是如何选择 Attribute。

## 分类

### Entropy

$$\text{Entropy}(S)=\sum_{i=1}^n-p(i)\log p(i)$$
$n$ 是种类的数量。

### Information Gain

$$\text{Gain}(S, A)=\text{Entropy}(S)-\sum_{v\in\text{values}(A)}\dfrac{|S_v|}{|S|}\text{Entropy}(S_v)$$
等式减号右侧为根据 Attribute 分割后的条件熵，原来的熵减去分割后的条件熵就是 Information Gain。
选择 Information Gain **最大**的。

### Gain Ratio

$H_A(S)=-\sum_{i=1}^v\dfrac{|S_i|}{|S|}\log\dfrac{|S_i|}{|S|}$，其中 $v$ 是对应 Attribute 含有的 Value 的种类数。
$$\text{GainRatio}(S,A)=\dfrac{G(S,A)}{H_A(S)}$$
可以排除一些过于细致的 Attribute 的影响，例如名字。
选择 Gain Ratio **最大**的。

### Gini Index

$$\text{Gini}(D)=1-\sum_{k=1}^K(\dfrac{|C_k|}{|D|})^2$$
$$\text{Gini}(D,A)=\dfrac{|D_1|}{|D|}\text{Gini}(D_1)+\dfrac{|D_2|}{|D|}\text{Gini}(D_2)$$
选择 Gini Index **最小**的。

### Misclassification Error

$$E(D)=1-\underset{k}{\max}\dfrac{|C_k|}{|D|}$$
分割后的 ME 添加分割属性的权重。
其中，$D$ 为数据集，$C_k$ 为数据集中 label 属于第 $k$ 类的样本集合。
选择 Misclassification Error **最小**的。

## 回归

将特征空间分为 $M$ 个区域，分别为 $R_1,R_2,\ldots,R_M$，每一个区域对应回归树的一个 leaf node，并且在每一个区域固定一个输出 $c_m$，回归树的模型可以表示为：
$$f(x)=\sum_{m=1}^Mc_mI(x\in R_m)$$
$I$ 为指示函数。
规定将 leaf node 的**输出值**设定为此区域中所有样本输出值的平均值。
现在要处理的问题就是如何确定**分割**空间。
遍历切分变量 $j$，以及切分点 $s$ 选择预测误差最小的情况。
$$\min_{j,s}[\min_{c_1}\sum_{x_i\in R_1(j,s)}(y_i-c_1)^2+\min_{c_2}\sum_{x_i\in R_2(j,s)}(y_i-c_2)^2]$$
此处的 $R_1,R_2$ 即切分后的左右两侧。

---

# Lecture 3 Perceptron & SVM

## Perceptron 感知机

一种二类分类的线性模型，输入为特征向量，输出为类别，一般用 $+1,-1$ 表示。旨在求解出将训练数据进行划分的超平面。

### 定义

$$f(x)=\text{sign}(w\cdot x+b)$$

### 学习策略

采用梯度下降算法进行学习，该如何设计损失函数呢？
如果为误分类点的总数，不好优化。于是设置为误分类点到超平面 $S$ 的总距离。
对于误分类的数据来说，满足下式：
$$-y_i(w\cdot x_i+b)>0$$
$M$ 为误分类点集合，误分类点到超平面 $S$ 的总距离为：
$$-\dfrac{1}{||w||}\sum_{x_i\in M}y_i(w\cdot x_i+b)$$
损失函数去掉系数：
$$L(w,b)=-\sum_{x_i\in M}y_i(w\cdot x_i+b)$$
损失函数的梯度为：
$\nabla_wL(w,b)=-\sum_{x_i\in M}y_ix_i$
$\nabla_bL(w,b)=-\sum_{x_i\in M}y_i$
随机选取一个误分类点的数据进行更新，其中 $\eta$ 为学习率。
$w\leftarrow w+\eta y_ix_i$
$b\leftarrow b+\eta y_i$
不断重复该过程。

#### 对偶形式

从原形式中我们发现，每次对 $w,b$ 进行更新变化的值都是固定的，因此我们可以通过记录某个误分类点被选取的次数来进行计算。

$$
\begin{aligned}
w&=\sum_{i=1}^N\alpha_iy_ix_i\\
b&=\sum_{i=1}^N\alpha_iy_i\\
\alpha_i&=n_i\eta
\end{aligned}
$$

$n_i$ 表示第 $i$ 个实例点由于误分类而进行更新的次数.
于是当我们进行更新时, 可以简化为如果 $y_i(\sum_{j=1}^N\alpha_jy_jx_j\cdot x_i+b)\leq 0$ 那么更新次数加一
$\alpha_i\leftarrow \alpha_i + \eta$
$b\leftarrow b+\eta y_i$

## SVM 支持向量机

只是成功将数据分割还不够好，SVM 能够做到更优质的分割，让样本点到分割线的距离尽可能大，实现更加良好的分类性能。

### Maximum Margin

- Functional Margin $\hat\gamma_i=y_i(w\cdot x_i +b)$ 很像感知机。
- Geometry Margin $\gamma_i=y_i(\dfrac{w}{||w||}\cdot x_i +\dfrac{b}{||w||})$ 几何间隔相比函数间隔增加了**系数**。
- 以上只是样本点的间隔，真正的间隔取其中的**最小值**。
  最终我们的问题变为一个**凸优化**问题：
  $$
  \begin{align}\min_{w,b}&\quad \dfrac{1}{2}||w||^2\\
  \text{s.t.}&\quad \ y_i(w\cdot x_i+b)-1\geq0,\ i=1,2,\ldots,N\end{align}
  $$

### 拉格朗日对偶

相关证明此处略，见：[[拉格朗日对偶 Lagrange duality]]
构造拉格朗日函数，其中 $\alpha_i\geq0$ 为拉格朗日乘子：
$$L(w,b,\alpha)=\dfrac{1}{2}||w||^2-\sum_{i=1}^N\alpha_i[y_i(w\cdot x_i+b)-1]$$

根据拉格朗日对偶性，可以将原始问题转化成对偶问题：
$$\max_\alpha\min_{w,b}L(w,b,\alpha)$$
先求极小，令偏导为 0：

$$
\begin{align}
\nabla_wL(w,b,\alpha)&=w-\sum_{i=1}^N\alpha_i y_ix_i=0\\
\nabla_bL(w,b,\alpha)&=\sum_{i=1}^N\alpha_iy_i=0
\end{align}
$$

代入并化简后我们得到：
$$\mathcal{L}(w,b,\alpha)=\sum_{i=1}^n\alpha_i-\dfrac{1}{2}\sum_{i,j=1}^ny_iy_j\alpha_i\alpha_jx_ix_j.$$
于是我们得到了一个全新的优化问题：

$$
\begin{align}
\max_\alpha\quad &W(\alpha)=\sum_{i=1}^n\alpha_i-\dfrac{1}{2}\sum_{i,j=1}^ny_iy_j\alpha_i\alpha_jx_ix_j.\\
\text{s.t}\quad &\alpha_i\geq0,\quad i=1,\ldots,n\\
&\sum_{i=1}^n\alpha_iy_i=0
\end{align}
$$

求解即可。

### 正则化与线性不可分情况

为了能够**处理线性不可分**的情况并且**减小异常值的影响**，我们引入正则化（$\mathscr{l}_1$ **regularization**），改变我们的优化目标，最终的对偶问题变成如下形式：

$$
\begin{align}
\max_\alpha\quad &W(\alpha)=\sum_{i=1}^n\alpha_i -\dfrac{1}{2}\sum_{i,j=1}^ny_iy_j\alpha_i\alpha_jx_ix_j\\
\text{s.t} \quad &0\leq \alpha_i\leq C,\quad i=1,\ldots,n\\
& \sum_{i=1}^n\alpha_iy=0
\end{align}
$$

我们发现其实就是多了一个对于 $\alpha_i$ 的上限的约束。

### Kernel trick 核

为了解决线性不可分的问题，我们可以使用 kernel trick 将问题映射到更高的维度，将非线性问题转换成线性的，例如：
$$\phi(x)=\begin{pmatrix}1 \\ x_1 \\ x_2 \\ x_1^2 \\ x_1x_2 \\ x_2^2\end{pmatrix}$$
核函数的定义：
$$K(x,z)=\left\langle\phi(x),\phi(z)\right\rangle$$
其中 $\left\langle\right\rangle$ 表示点积，核函数相当于高维空间中的点积。
并非所有的函数可以当作核函数，需要使用时可以搜索。

---

# Lecture 4 Maximum Entropy

## 最大熵原理

最大熵模型认为在学习概率模型时，在所有可能的概率模型，即概率分布中，**熵最大**的模型就是最好的模型。
熵满足下列不等式：
$$0\leq H(P)\leq \log|X|$$
式中，$|X|$ 表示 $X$ 的取值个数，当且仅当 $X$ 的分布是**均匀分布**时，右边的等号成立，即**熵最大**。

## 最大熵模型的定义

假设分类模型是一个条件概率分布 $P(Y|X)$，表示对于给定的输入 $X$ 以 $P(Y|X)$ 输出 $Y$。
对于给定的训练数据集，我们可以确定 $P(X,Y)$ 和 $P(X)$ 的经验分布。用 $\tilde P(X,Y)$ 和 $\tilde P(X)$ 表示。
用**特征函数** $f(x,y)$ 描述输入输出 $x,y$ 之间的某一个事实，其定义是

$$
f(x,y)=\begin{cases}
1, & x,y\ 满足某种条件\\
0, & 否则
\end{cases}
$$

它是一个认为定义的二值函数，此处所说的“某种条件”比较难以理解，其实特征函数就是一个**筛选函数**，满足这个条件的样本才会被纳入考虑，这也是为什么我们会人为设定特征函数。
最大熵模型是一个**优化问题**，优化问题就涉及到**目标函数**和**约束条件**，最大熵模型的目标函数就是**条件熵**，约束条件是实际的概率分布的期望与数据的概率分布的**期望相同**。
于是建模如下：

$$
\begin{align}
\min_{P\in\text{C}}\quad &-H(P)=\sum_{x,y}\tilde P(x)P(y|x)\log P(y|x)\\
\text{s.t} \quad &E_P(f_i)-\tilde E_P(f_i)=0,\quad i=1,\ldots,n\\
&\sum_y
P(y|x)=1\end{align}
$$

同样的为了求解优化问题我们进行拉格朗日**对偶化**

$$
\begin{align}
L(P,w)\equiv &-H(P)+w_0\left(1-\sum_yP(y|x)\right)+\sum_{i=1}^nw_i\left(E_P(f_i)-\tilde E_P(f_i)\right)\\
=&\sum_{x,y}\tilde P(x)P(y|x)\log P(y|x)+w_0\left(1-\sum_yP(y|x)\right)+\\
&\sum_{i=1}^nw_i\left(\sum_{x,y}\tilde P(x,y)f_i(x,y)-\sum_{x,y}\tilde P(x)P(y|x)f_i(x,y)\right)
\end{align}
$$

两个期望的定义就在此处，前面就不重复了。
求解就是先极小再极大，令偏导为 0 等等。

---

# Lecture 5 Ensemble Learning 集成学习

多个弱模型合起来变成一个强模型。
主要介绍 Boosting 方法。

## AdaBoost

AdaBoost 主要做两件事情：

1. 每一轮改变训练数据的权重或者分布
2. 不断创建新的弱分类器并赋予其权重

**算法**

1. 初始化训练数据的起始权值分布：
   $D_1=(w_{11},\ldots, w_{1i},\ldots, w_{1N})\quad w_{1i}=\dfrac{1}{N},\quad i=1,2,\ldots,N$
2. 更新获得第 $m$ 个分类器：
   a) 在权值 $D_m$ 下训练数据集，得到弱分类器 $G_m(x)$
   b) 计算 $G_m$ 的训练误差：$e_m=\sum_i w_{mi}I(G_m(x_i)\neq y_i)$
   c) 计算 $G_m$ 在最终分类器中的系数：$\alpha_m=\dfrac{1}{2}\log\dfrac{1-e_m}{e_m}$
   d) 更新训练数据集的权值分布：$w_{m+1,i}=\dfrac{w_{mi}}{Z_m}\exp(-\alpha_m y_iG_m(x_i)),\quad i=1,2,\ldots,N$
   其中 $Z$ 是规范化因子。
3. 构建最终分类器：
   $G(x)=\text{sign}(\sum_m\alpha_mG_m(x))$
   不断重复知道达到终止条件如准确率。

## 提升树 Boosting Tree

提升方法采用**加法模型**（即基函数的线性组合）与**前向分布算法**，以决策树为基函数的提升方法即为**提升树**。对于分类（回归）问题，我们就使用二叉分类（回归）树。
提升树模型可以表示为决策树的加法模型：
$$f_M(x)=\sum_{m=1}^MT(x;\Theta_m)$$
其中，$T(x;\Theta_m)$ 表示决策树，$\Theta_m$ 表示决策树的参数，$M$ 为树的个数。

### 提升树算法

注：在提升树算法中所使用的决策树都是决策树桩，即只有一个分裂节点的决策树。
提升树采用前向分布算法。首先确定初始提升树 $f_0(x)=0$，第 $m$ 步的模型是
$$f_m(x)=f_{m-1}(x)+T(x;\Theta_m)$$
通过经验风险极小化确定下一颗树的参数 $\Theta_m$
$$\hat\Theta_m=\arg\min_{\Theta_m}\sum_{i=1}^NL(y_i,f_{m-1}(x_i)+T(x_i;\Theta_m))$$
此处的损失函数为前向分布算法中的指数损失函数
$$L(y,f(x))=\exp(-yf(x))$$
其实对于**分类问题**来说提升树可以理解为 Adaboost 的一个特殊情况，接下来讨论**回归问题**。
其实差别也不大，算法没什么改变。只是损失函数会发生变化，可以使用**平方误差损失函数**，如果使用更加一般的损失函数，那么就引出了**梯度提升算法**，其实也是一个道理，求偏导置零的思想，前面的所有损失函数都可以理解为特例。

---

# Lecture 6 Expectation Maximization

用于处理含有**隐变量**的概率模型。
假设我们有一个估计问题，我们有一个训练集 $\{x^{(1)},\ldots,x^{(n)}\}$ 由 $n$ 个相互独立的实例组成。我们有含有隐变量的模型 $p(x,z;\theta)$ 其中 $z$ 是隐变量。$x$ 的概率密度可以通过对 $z$ 求和得到：
$$p(x;\theta)=\sum_zp(x,z;\theta)$$
我们希望通过最大似然估计来拟合 $\theta$ ：

$$
\begin{align}
\mathscr{l}(\theta)& =\sum_{i=1}^n\log p(x^{(i)};\theta)\\
&=\sum_{i=1}^n\log\sum_{z^{(i)}}p(x^{(i)},z^{(i)};\theta)
\end{align}
$$

但是如果要直接对 $\theta$ 通过最大似然估计求解，会导向一个困难的非凸优化问题。在这里，$z$ 是隐含的随机变量，如果 $z$ 能够被观测到，那么最大似然估计求解会简单很多，EM 算法也就由此引出。
EM 算法不直接进行最大似然估计，而是不断地构建 $\mathscr{l}$ 的下界，然后优化这个下界。这两个过程分别就是 EM 算法的 E 步和 M 步。
于是求和 $\sum_{i=1}^n$ 就不是必要的了，我们可以先进行简化的 EM 算法，只考虑一个样本 $x$ 然后优化他的概率 $\log p(x)$。后续将 $n$ 个样本的结果组合起来即可。现在我们的优化目标是：
$$\log p(x) = \log\sum_z p(x,z;\theta)$$
令 $Q$ 为 $z$ 的一个分布，则：

$$
\begin{align}
\log p(x;\theta) & =\log\sum_z p(x,z;\theta)\\
&=\log\sum_zQ(z)\dfrac{p(x,z;\theta)}{Q(z)}\\
&\geq \sum_zQ(z)\log \dfrac{p(x,z;\theta)}{Q(z)}
\end{align}
$$

最后一步应用了 Jensen's inequality。详见 CS229 main notes p154.
对于任意的 $Q$，上述公式给出了 $\log p(x;\theta)$ 的一个下界。该如何从这么多分布中选取呢？自然想到要让下界更紧，比如达成上述不等式中的相等条件。满足下列条件时，可以实现：

$$
\begin{align}\dfrac{p(x,z;\theta)}{Q(z)}&=c \\
Q(z)&=p(z|x;\theta)
\end{align}
$$

我们称该下界为 evidence lower bound 简记为 ELBO：
$$\text{ELBO}(x,Q;\theta)=\sum_zQ(z)\log\dfrac{p(x,z;\theta)}{Q(z)}$$
可以将原先的不等式重新写为：
$$\forall Q,\theta,x,\quad \log p(x;\theta)\geq\text{ELBO}(x;Q,\theta)$$
EM 算法所做的就是：

1. 更新 $Q(z)=p(z|x;\theta)$ 使得 上述不等式等号成立。
2. 根据 $\theta$ 最大化 ELBO，保持 $Q$ 不变。
   目前为止我们只是针对一个样本进行优化，对于训练集 $\{x^{(1)},\ldots,x^{(n)}\}$ 中任意样本，有下式：
   $$\log p(x^{(i)};\theta)\geq\text{ELBO}(x^{(i);}Q_i,\theta)=\sum_{z^{(i)}}Q_i(z^{(i)})\log\dfrac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}$$
   对所有样本求和后，我们就得到了 log-likelihood 的下界
   $$
   \begin{align}
   \mathscr{l}(\theta) & \geq \sum_i\text{ELBO}(x^{(i)};Q_i,\theta)\\
   &=\sum_i\sum_{z^{(i)}}Q_i(z^{(i)})\log\dfrac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
   \end{align}
   $$
   使等式成立的 $Q_i$ 满足
   $$Q_i(z^{(i)})=p(z^{(i)}|x^{(i)};\theta)$$
   完整的 EM 算法如下：
   Repeat until convergence {
   (E-step) For each $i$, set
   $$Q_i(z^{(i)}):=p(z^{(i)}|x^{(i)};\theta)$$
   (M-step) Set
   $$
   \begin{align}
   		\theta &:= \arg\max_\theta\sum_{i=1}^n\text{ELBO}(x^{(i)};Q_i,\theta)\\
   		&=\arg\max_\theta\sum_i\sum_{z^{(i)}}Q_i(z^{(i)})\log\dfrac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
   		\end{align}
   $$
   }

---

# Lecture 7 隐马尔可夫模型 HMM

### What?

隐马尔可夫模型是一种用于建模**有隐状态的随机过程**的统计模型。广泛应用于**序列建模问题**。

### Why?

我们通常只能观测到表象而无法直接观测到表象之下的本质，这便是隐状态。例如通过一个人外出的着装来推测天气。HMM 的目标就是根据给出的观测序列推理出最有可能的隐状态序列。
主要解决三类问题：

- **评估问题** Evaluation 给定模型参数和序列，计算概率 $P(O|\theta)$
- **解码问题** Decoding 给定观测序列，找出最可能的隐状态序列 $Q$
- **学习问题** Learning 已知观测序列，估计最优的模型参数 $(A,B,\pi)$

### How?

#### HMM 的组成部分

1. 隐状态 Hidden States 我们不能直接观测到的状态
2. 观测值 Observations 直接观测到的“表面”现象
3. 状态转移概率 $A$ 表示从一个隐状态转移到另一个隐状态的概率
   $A[i][j]=P(i\rightarrow j)$
4. 观测概率 $B$ 表示在某个状态观测到某个值的概率
   $B[j][k]=P(k|j)$
5. 初始状态概率 $\pi$

#### 评估问题

##### Forward recursion for HMM

定义前向变量 $\alpha_k(i)$ 表示观测到序列 $o_1o_2\ldots o_k$ 且在第 $k$ 步的隐状态是 $s_i$的联合概率。
$$\alpha_k(i)=P(o_1o_2\ldots o_k,q_k=s_i)$$

###### 初始化

假设有 $N$ 个隐状态，序列长度为 $K$
$$\alpha_1(i)=P(o_1,q_1=s_i)=\pi_i b_i(o_1),1\leq i\leq N$$

###### 前向递归

$$\alpha_{k+1}(i)=[\sum_j\alpha_k(j)a_{ji}]b_i$$

###### 结束

$M$ 表示参数。
$$P(o_1o_2\ldots o_K|M)=\sum_i \alpha_K(i)$$

##### Backward Recursion for HMM

类似前向递归，定义后向变量 $\beta_k(i)=P(o_{k+1}\ldots o_K|q_k=s_i)$

###### 初始化

$$\beta_K(i)=1,i=1,2,\ldots,N$$

###### 后向递归

$$\beta_k(i)=\sum_j\beta_{k+1}(j)a_{ij}b_j(o_{k+1}),k=K-1,K-2,\ldots ,1\quad i=1,2,\ldots,N$$

###### Termination

$$P(o_1o_2\ldots o_K|M)=\sum_i\pi_ib_i(o_1)\beta_1(i)\quad i=1,2,\ldots,N$$

#### 学习问题

**Baum-Welch algorithm**
大致思路：

$$
\begin{align}a_{ij}&=P(s_j|s_i)\\
b_i(o_m)&=P(o_M|s_i)\\
\pi_i&=P(s_i)
\end{align}
$$

##### Expectation step

定义变量 $\xi_k(i,j)$ 表示观测到序列 $o_1o_2\ldots o_k$ 情况下，在 $k$ 步处于 $s_i$，在 $k+1$ 步处于 $s_j$ 的概率。
$$\xi_k(i,j)=\dfrac{\alpha_k(i)a_{ij}b_j(o_{k+1})\beta_{k+1}(j)}{\sum_i\sum_j\alpha_k(i)a_{ij}b_j(o_{k+1})\beta_{k+1}(j)}$$
定义变量 $\gamma_k(i)$ 表示观测到序列 $o_1o_2\ldots o_k$ 时，在 $k$ 步处于 $s_i$ 的概率。
$$\gamma_k(i)=\dfrac{\alpha_k(i)\beta_k(i)}{\sum_i\alpha_k(i)\beta_k(i)}$$

##### Maximization step

$$
\begin{align}
a_{ij} &= \dfrac{\sum_k\xi_k(i,j)}{\sum_k\gamma_k(i)} \\
b_i(o_m) &= \dfrac{\sum_{k,o_k=v_m}\gamma_k(i)}{\sum_k\gamma_k(i)}\\
\pi_i&=\gamma_1(i)
\end{align}
$$

#### 解码问题

**Viterbi algorithm**
目标找到隐状态序列 $Q=q_1\ldots q_K$ 最大化 $P(Q|o_1o_2\ldots o_K)$
暴力搜索有着指数级复杂度，Viterbi 算法更加高效。

##### 初始化

$$
\begin{align}
\delta_1(i)&=P(q_1 = s_i,o_1)=\pi_ib_i(o_1),1\leq i\leq N\\
\psi_1(i)&=0,1\leq i\leq N
\end{align}
$$

##### 前向递归 $2\leq k\leq K$

$$
\begin{align}
\delta_k(i)&=\max_j[\delta_{k-1}(j)a_{ji}]b_i(o_k)\quad 1\leq i\leq N\\
\psi_k(i)&=\arg\max_j[\delta_{k-1}(j)a_{ji}]\quad 1\leq i\leq N
\end{align}
$$

##### 结束

$$P^*=\arg\max_i[\delta_K(i)],\ i^*_K =\arg\max_i[\delta_K(i)]\quad 1\leq i\leq N$$

##### 反向计算最优路径

$$i^*_k=\psi_{k+1}(i^*_{k+1})\quad k=K-1,K-2,\ldots,1$$
最优路径 $I^*=(i^*_1,\ldots,i^*_K)$

---

# Lecture 8 条件随机场

CRF 对一个观测序列下某个标签序列的条件概率建模。
**C Conditional** 他学习条件概率 $P(Y|X)$，其中 $X$ 是观测到的输入序列，$Y$ 是标签序列。
**RF Random Field** 属于图类模型，随机变量之间的关系用图来表示。

## 线性链条件随机场

两个预设：

- 每个 label 取决于前一个 label 就像在 HMM 中一样
- 每个 label 取决于整个观测序列
  $$
  \begin{align}
  P(y|x)&=\dfrac{1}{Z(x)}\exp\left(\sum_{i,k}\lambda_kt_k(y_{i-1},y_i,x,i)+\sum_{i,l}u_ls_l(y_i,x,i)\right)\\
  Z(x)&=\sum_y\exp\left(\sum_{i,k}\lambda_kt_k(y_{i-1},y_i,x,i)+\sum_{i,l}u_ls_l(y_i,x,i)\right)
  \end{align}
  $$
  $t_k$ 和 $s_l$ 是特征函数，当满足特征条件时取值为 1 否则 0，$\lambda_k$ 和 $u_l$ 是对应的权值。
  可以对式子进行化简，把 $k$ 和 $l$ 接起来。
  $$
  \begin{align}
  P(y|x)&=\dfrac{1}{Z(x)}\prod_{t=1}^T\phi_t(y_t,y_{t-1},x)\\
  \phi_t(y_t,y_{t-1},x)&=\exp\left(\sum_{k}\lambda_kf_k(y_t,y_{t-1},x,t)\right)
  \end{align}
  $$

### 矩阵表示

理解起来比较简单, 要注意规范化. 规范化因子可通过观察矩阵乘法结果获得.
