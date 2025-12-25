# **一、实验背景**

语音合成（Text-to-Speech, TTS）是深度学习的重要应用方向，其中 Tacotron2 负责将文本转为梅尔频谱，WaveGlow 负责将梅尔频谱转换为真实语音。本实验要求同学通过加载 NVIDIA 提供的预训练模型，在指定硬件（如 GPU/MLU）上完成端到端语音合成推理。

本实验旨在帮助学生理解：

1. 序列到序列模型在语音合成中的角色；
    
2. Tacotron2 模型的结构与推理流程；
    
3. WaveGlow 流模型的语音生成原理；
    
4. 模型推理过程中张量维度管理及后处理网络（Postnet）的使用；
    
5. 深度学习框架（PyTorch）中模型加载、device 管理、错误调试等操作。

# **二、实验原理**

## **1. Tacotron2 模型原理**

Tacotron2 包含三个主要部分：

1. **Encoder**：将字符序列编码为高维隐藏序列；
    
2. **Decoder + Attention**：利用注意力机制依次生成梅尔频谱帧；
    
3. **Postnet**：对初始梅尔频谱进行卷积增强，提高细节质量。
    

Tacotron2 输出形状为：

- initial mel: ([B, T, 80]) 或 ([B, 80, T])（视实现而定）
    
- 必须转换为 Channel-first 才能送入 Postnet。
    

## **2. WaveGlow 原理**

WaveGlow 是一种基于流模型（Flow-based Model）的语音生成模型，通过多层 invertible 1×1 卷积和 affine coupling 层，实现从高斯噪声到语音波形的映射。

推理流程：

```
text → Tacotron2 → mel spectrogram → WaveGlow → waveform
```

## **3. 模型推理流程关键点**

- **模型路径正确性检测**；
    
- **Torch 的 map_location 加载 CPU/MLU 时的差异**；
    
- **输入维度：Postnet 需要 [B, 80, T]**；
    
- **float64 → float32 自动转换警告**；
    
- **注意力矩阵对齐方式**。
    



# **三、实验流程**

实验主要步骤如下：

## **1. 环境准备**

- Python 3.7
    
- PyTorch / torch_mlu
    
- NVIDIA 预训练模型目录：
    
    ```
    model/pretrained/
        nvidia_tacotron2pyt_fp32_20190427
        nvidia_waveglowpyt_fp32_20190427
    ```
    

## **2. 加载模型**

```python
state_dict = torch.load(checkpoint, map_location='cpu')['state_dict']
```

注意：若模型路径错误会导致 FileNotFoundError。

## **3. Tacotron2 推理实现**

关键代码片段：

```python
embedded_inputs = self.embedding(inputs)
encoder_outputs = self.encoder.infer(embedded_inputs)
mel_outputs, _, alignments, mel_lengths = self.decoder.infer(
    encoder_outputs, input_lengths
)

# 修正维度，Postnet 必须是 [B, 80, T]
mel_outputs = mel_outputs.transpose(1, 2)

mel_outputs_postnet = self.postnet(mel_outputs)
mel_outputs_postnet += mel_outputs
```

## **4. WaveGlow 推理**

通过 Flow computation 将 mel 转换为 waveform：

```python
audio = waveglow.infer(mel_outputs_postnet, sigma=sigma)
```


# **四、实验数据与分析**

## **1. 实验输入**

- 文本长度：约 167 字符
    
- 批大小：1
    
- 梅尔帧数：约 750 帧（取决于文本长度）
    

## **2. 关键观察结果**

| 模块            | 输入形状         | 输出形状       | 备注         |
| ------------- | ------------ | ---------- | ---------- |
| Decoder       | [1, 748, 80] | [1, T, 80] | 输出为 T-Last |
| 调整后输入 Postnet | [1, 80, 748] | -          | 维度纠正成功     |
| 最终 Mel        | [1, 80, T]   | -          | 后处理增强调制效果  |

## **3. 常见错误分析**

报错：

```
expected input[B, C=80, T], but got [1, 748, 80]
```

原因：decoder.infer 输出为 `[B, T, 80]`，需要转置。

解决方案：

```python
mel_outputs = mel_outputs.transpose(1, 2)
```


# **五、实验结论**

本实验成功完成了 Tacotron2 + WaveGlow 的端到端语音合成推理流程。学生掌握了：

1. Tacotron2 编码器、解码器、Postnet 的数据流向；
    
2. mel 频谱维度管理在推理中的关键作用；
    
3. 预训练模型加载过程中的路径和设备配置；
    
4. WaveGlow 流模型的推理机制；
    
5. 深度学习推理代码调试方法。
    

实验结果表明，通过正确的维度调整和模型加载，可以顺利生成高质量语音。本实验提高了对序列模型、信号处理以及推理工程实践的理解，为后续的语音相关研究奠定了基础。
# 六、课程心得
在写模型相关代码的时候一定要注意把握tensor的形状，明确输入输出的形状，不要嫌写注释麻烦。