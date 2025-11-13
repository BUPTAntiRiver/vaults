[Computational Graphs in Deep Learning - GeeksforGeeks](https://www.geeksforgeeks.org/deep-learning/computational-graphs-in-deep-learning/)
Have you ever wondered how does the magical `.backward()` work in PyTorch? How is the gradients calculated and then descent automatically? These are all because of one thing called **Computational Graph**.
# What is computational graph?
First it is a **Directed Acyclic Graph**. And it has some key terminologies:
- A *variable* is represented by a *node* in a graph, it can be a scalar, vector, matrix or tensor.
- A *function argument* and *data dependency* are both represented by an *edge*.
- A simple *function* of one or more variables is called an *operation*. There is a set of permitted operations (developer implemented), some other complex operations (user defined) can be represented by combing these primitive operations.
For example: $Y=(a+b)*(b-c)$ might looks like this in a computational graph.
![[Pasted image 20251020210209.png]]
Then after forward computation, we can calculate the gradients according to chain rule:
![[Pasted image 20251020210407.png]]
![[Pasted image 20251020210415.png]]
# 2 kinds of computational graph
## Static
Explicitly defined by user. Easier to implement some kind of optimization.
## Dynamic
Implicitly defined by the library, which is more adaptable but has limits on optimization.
# How to update variable?
So far we only talked about how to compute the gradients, but how to update them?
That is what the optimizer do, it will update all the parameters in your model.