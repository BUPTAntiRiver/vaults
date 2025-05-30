# Lecture 3 "Manual" Neural Networks
## Why deep networks?
We know that with only one hidden layer the neural network can approximate any continuous function, but we still build deeper networks, because they just work better on a fixed number of input parameters.
# Lecture 7 Neural Network Abstraction
## Elements of machine learning
1. The hypothesis class: tensor in, tensor out
2. The loss function: tensor in, scalar out
3. An optimization method
# Lecture 9 Normalization and Regularization
Initialization matters a lot, and can vary over the course of training to no longer be consistent in layers.
We can apply a normalization layer, which does a normalize computation to make it stable.
## Layer normalization
$$
\begin{align}
\hat{z}_{i+1} &= \sigma(W_{i+1}^\top z_i + b_{i+1})\\
z_{i+1} &= \dfrac{\hat{z}_{i+1} - \textbf{E}[\hat{z}_{i+1}]}{(\textbf{Var}[\hat{z}_{i+1}]+\epsilon)^{1/2}}
\end{align}
$$
## Batch normalization
Let's consider the matrix form of our updates, the layer normalization equals to normalizing the *rows* of the matrix, and when we try to normalize the *columns* of the matrix, we get batch normalization. As we are normalizing over the mini batch.
### Minibatch dependency
An interesting point is, somehow batch normalization shows that the result of batch normalization depends on how you choose the batch, which also means, the output of example 1 depends on example 2 which are in the same minibatch, because in computation, example 1 is normalized with example 2. But the output of a single sample, should only depend on itself.
A common solution is use the batch norm on *training* time, but when *testing*, compute a *running average* of mean/variance for all features at each layer and normalize by them.
## Regularization
A introduction, typically deep networks may have more params than the number of training examples, so the model would have almost 0 training error. Which may be argued that it is overfitting the training set and performs badly on test, while it doesn't but sometimes does.
So **regularization** comes up, limiting the complexity of model and make sure the model generalize better.
### L2 regularization a.k.a. weight decay
Restrict the complexity of model by keeping the params small. If the params are small, the model would be smoother.
Implemented by augmenting the loss function with a L2 norm of weights.
### Dropout
Randomly set some fraction of the activations at each layer to 0.
# Lecture 10 Convolutional Networks
One easy way to understand convolution calculation is, think each element in input as a vector with length of input channel number, and the same with output but vector with length of output channel number, for the kernel, think each of its element as a matrix of size input channels by output channels.
# Lecture 11 Hardware Acceleration
Strides representation of matrix: can apply transpose and slice with 0 copy by adjusting the stride and shape.
## How to accelerate matrix multiplication?
Usually take $O(n^3)$ time, but there are differences between the operations in matrix multiplication. Reading from L1 cache is much faster than reading from DRAM. We can store the data that would be used for multiple times from DRAM to L1 cache and register to decrease the times we need to read from DRAM and accelerate the multiplication.
# Lecture 12 GPU acceleration
In GPU, there are more computing units but fewer controllers, so it would be easier for GPU to do repetitive works rather than flexible works.
In GPU there are many blocks, and in each block there are many threads, each thread deals with the same task but with different ids.