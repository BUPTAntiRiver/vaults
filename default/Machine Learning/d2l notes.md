# 10. 注意力机制

## 10.4. Bahdanau 注意力

**why?**
机器翻译中，每个生成的词可能相关于源句子中的不同的词（在外语中尤其如此），seq2seq 不能直接对此建模（seq2seq 生成词语是根据源句子生成的隐状态以及生成的词语进行生成）。
**what?**
利用所有的 key 进行预测，而非只是利用最后的隐状态输出。
**how?**

- 编码器对每次词的输出作为 key 和 value
- 解码器 RNN 对上一个词的输出是 query
- 注意力的输出和下一个词的词嵌入合并进入

## 10.5. 多头注意力

**why?**
当给定相同的查询、键和值的集合时，我们希望模型可以基于相同的注意力机制学习到不同的行为，  然后将不同的行为作为知识组合起来， 捕获序列内各种范围的依赖关系 （例如，短距离依赖和长距离依赖关系）。
**what?**

**how?**

## 10.6. 自注意力和位置编码

#### 自注意力

传统的注意力机制是通过对生成语句的 query 和源语句的 key 进行矩阵乘法以及一系列归一化操作得出结果，而自注意力则是将生成语句同时作为 query 和 key 的对象进行运算得出结果，再与 value 相乘得到输出。由此体现出语句内不同 token 之间的关系，其中 query 和 key 矩阵都是可学习的参数。

#### 位置编码

**why?**
通过位置编码，可以体现出 token 之间的位置关系，从而进行位置操作。
**what?**
除了原本表示输入的 token embedding matrix $\mathbf{X}$还有表示位置编码的 position embedding matrix $\mathbf{P}$两者相加输出$\mathbf{X+P}$。
**how?**
位置编码嵌入矩阵第$i$行、第$2j$列和$2j+1$列上的元素为：

$$
\begin{array}{cc}
p_{i,2j}=\sin\left(\frac{i}{10000^{2j/d}}\right), \\
p_{i,2j+1}=\cos\left(\frac{i}{10000^{2j/d}}\right).
\end{array}
$$

## 11. 优化算法

### 11.2. 凸性

[11.2. 凸性 — 动手学深度学习 2.0.0 documentation](https://zh.d2l.ai/chapter_optimization/convexity.html)
