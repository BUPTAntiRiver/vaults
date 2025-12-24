使用 CIFAR-10 数据集，采用了一层隐层的全连接层，使用 `ReLU` 激活，最后用 `Softmax` 得到输出结果。
尝试了以下几种方法：
1. MSE + MiniBatch GD
2. MSE + Adam
3. MSE + Adam + L2 正则
4. CrossEntropy + Adam + L2 正则
均训练 20 epochs，根据测试结果，MiniBatch GD 训练速度比 Adam 更快，但收敛效率更低，而且需要的学习率比 Adam 大很多。
正则化的引入对于当前问题似乎影响并不大。
CrossEntropy 在使用 Softmax 的情况下相较于 MSE 具有更好的效果。