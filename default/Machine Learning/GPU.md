A **graphic processing unit (GPU)** is a specialized electronic circuit designed for digital image processing and to accelerate computer graphics. But nowadays it is widely used in boosting parallel computation tasks in machine learning. We all know that CPU is good at computing, so why does this GPU comes out? Let's dive into it.
# The difference between CPU and GPU
A common CPU is optimized to be as quick as possible to finish a task at a as low as possible latency, while keeping the ability to quickly switch between operations. It's nature is all about processing tasks in a serialized way.
A GPU is all about throughput optimization, allowing to push as many as possible tasks through is internals at once. It does so by being able to parallel process a task.
![[CPU and GPU 'cores'.png]]
This figure only shows the difference on number of cores, but that is not the only one.
The more important difference is that the number of memory caches, CPU has much more caches for each core, almost each core has multiple caches storing the instructions to be executed next. While GPU only has one cache for multiple cores. So this is why CPU is more capable of solving flexible tasks, and GPU handles repetitive tasks better.
**CPU architecture**
![[CPU architecture.png]]
**GPU architecture**
![[GPU architecture.png]]