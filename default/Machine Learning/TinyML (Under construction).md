# WHAT?
TinyML wants to reduce the memory and storage use of machine learning models from GB/TB level to MB/GB level so that it can be used on more tiny devices which are more widely used.
# WHY?
Todays models are too big, we want deep learning to be "tiny".
For example, nowadays we train big models on cloud which makes the training process easier to be attacked.
Besides, squeezing deep learning into IoT devices can lead to **lower** cost, **lower** power use and let the model have **wider** range of applications.
## Applications
# HOW?
## An example
Take CNNs as example, when we run CNNS, we store input and output of each step in Memory(SRAM) and store kernel weights in Storage(DRAM/Flash).
**Flash** usage **= model size**. Flash is static, need to hold the entire model.
**SRAM** usage **= input activation + output activation**. SRAM usage is dynamic and different to each layer, we care about *peak* SRAM usage.
Cloud/Mobile CNNs usually has 10 times the peak SRAM usage of the tiny controller's SRAM. So they can not fit in.
## Tiny neural network design
