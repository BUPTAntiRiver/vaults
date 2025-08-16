reference: [Mixture of Experts Explained](https://huggingface.co/blog/moe)
# What?
MoE is in the context of **Transformer** models, and a MoE has 2 main elements:
## Sparse MoE layers
Which replace FFN (Feed Forward Network) layers, MoE layers has a certain number of "**expert**s", each expert is a neural network. In practice they can be FFNs or more complex networks like MoE itself, which leads to hierarchical MoEs.
## Gate Network or Router
It determines which tokens are sent to which expert(s).
# Why?
MoE has **better compute efficiency**, which means you can **scale up** model size or dataset size with the same compute budget as a dense model.
Although MoEs provides efficient pretraining and faster inference compared to dense models, they also come with challenges:
- **Training**: struggled to generalize during fine-tuning, leading to overfitting.
- **Inference**: although a MoE model might have a lot of parameters, but its inference is pretty fast, because during inference only some of the experts are used. However all parameters need to be loaded in RAM, but actually it would need less space than number of experts times size of an expert, because there are some shared layers.
# How?
