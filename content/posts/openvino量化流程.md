---
tags: []
categories: ["work"]
description:
summary:
draft: false
isCJKLanguage: true
date: "2022-05-28"
lastmod: "2022-06-10"
title: openvino 量化
---

# openvino 量化

关于模型量化的更多信息，请参考[白皮书](https://www.intel.com/content/www/us/en/developer/articles/technical/lower-numerical-precision-deep-learning-inference-and-training.html)中深度学习中低精度的简要部分。

> [!cite] 参考
> [Post Training Quantization with OpenVINO Toolkit (learnopencv.com)](https://learnopencv.com/post-training-quantization-with-openvino-toolkit/)

初步理解 openvino 的量化过程应该属于 [[神经网络量化#静态训练后量化|静态训练后量化]]
在使用 POT 进行量化的过程中，通过指定的硬件在模型图中插入一个名为 `FakeQuantize` 的操作。

![](https://s2.loli.net/2022/06/06/f7MmEuhlHyz8aWD.png)

在运行期间，这些 FakeQuantize 层将卷积层的输入转换成 Int8。例如，如果下一个卷积层有 Int8 权重，那么该层的输入也将被转换为 Int8。再往下一步的精度取决于下一个操作，如果下一个操作需要全精度格式，那么输入将在运行时被重新转换为全精度。

## FakeQuantize

> [!cite] 参考
> [FakeQuantize — OpenVINO™ documentation — Version(latest)](https://docs.openvino.ai/latest/openvino_docs_ops_quantization_FakeQuantize_1.html)
> [Quantized networks compute and restrictions — OpenVINO™ documentation — Version(latest)](https://docs.openvino.ai/latest/openvino_docs_ie_plugin_dg_quantized_networks.html?highlight=fake%20quantize)
> [OpenVINO™ Low Precision Transformations — OpenVINO™ documentation — Version(latest)](https://docs.openvino.ai/latest/openvino_docs_OV_UG_lpt.html)

FakeQuantize 将浮点输入值按元素线性量化为一组离散的浮点值。
输入和输出范围以及量化级的数量由专门的输入和属性指定。对输入张量的每个元素或元素组（通道）可以有不同的限制，否则所有元素使用相同的限制。这取决于指定限制的输入的形状和适用于输入张量的常规广播规则。运算器的输出是一个与输入张量相同类型的浮点数。一般来说，有四个值指定每个元素的量化：`input_low`, `input_high`, `output_low`, `output_high`。`input_low` 和 `input_high` 属性指定量化的输入范围。所有超出这个范围的输入值在实际量化前都会被剪切到这个范围内。 `output_low` 和 `output_high` 指定输出端的最小和最大量化值。
**FakeQuantize 中的 Fake 意味着输出张量与输入张量同为浮点类型而不是整数类型**（未理解）(训练过程中为浮点参数，保存为定点)
```python
if x <= min(input_low, input_high):
    output = output_low
elif x > max(input_low, input_high):
    output = output_high
else:
    # input_low < x <= input_high
    output = round((x - input_low) / (input_high - input_low) \
    * (levels-1)) / (levels-1) \
    * (output_high - output_low) + output_low
```
**levels**：量化级别的数量（例如，2 用于二进制化，255/256 用于 int8 量化）

### 由运行解释 FakeQuantize

在模型加载期间，每个插件都可以解释 FakeQuantize 操作中表达的量化规则：

- 独立基于 _FakeQuantize_ 操作的定义。
- 使用特殊的低精度变换（LPT）库，该库对泛型运算（如卷积，全连接，Eltwise 等）应用通用规则，并将“假量化”模型转换为具有低精度运算的模型。

在这里，我们仅提供 FakeQuantize 解释规则的高级概述。在运行时，每个 FakeQuantize 可以分为两个独立的操作：**Quantize**和**Dequantize**。前者旨在将输入数据转换为目标精度，而后者将结果值转换回原始范围和精度。在实践中 _，去量化 _ 运算可以通过线性运算（如 _ 卷积 _ 或 _ 全连通 _）向前传播，并且在某些情况下与下一层的以下 _ 量化 _ 运算融合到所谓的 _ 重量化 _ 运算中（见图 1）。
![图 1.量化操作在运行时传播。Q、DQ、RQ 代表 Quantize、Dequantize 和 Requantize](https://s2.loli.net/2022/06/07/VCMoEedDriLIpJh.png)

从计算的角度来看，FakeQuantize 公式也相应地分为两部分
$$output = round((x - input\_low) / (input\_high - input\_low) * (levels-1)) / (levels-1) * (output\_high - output\_low) + output\_low$$

量化操作：$$q = round((x - input\_low) / (input\_high - input\_low) * (levels-1))$$
去量化操作：$$r = q / (levels-1) * (output\_high - output\_low) + output\_low$$
刻度/零点表示法下的去量化操作：$$r = (output\_high - output\_low) / (levels-1) * (q + output\_low / (output\_high - output\_low) * (levels-1))$$
因此可定义：
- **缩放**为$(output\_high - output\_low) / (levels-1)$
- **零点**为$-output\_low / (output\_high - output\_low) * (levels-1)$

![](https://s2.loli.net/2022/06/07/DlPGXa5Tp1AzwiM.png)

## DefaultQuantization

![](https://s2.loli.net/2022/06/05/YZVjWCxdXH9eBJR.png)
1. 首先使用 POT 向算法提供全精度模型。
2. 在算法内部首先对训练好的模型应用通道对齐。这也被称为激活通道对齐，它对齐卷积层的激活范围以减少量化误差。通常情况下，在这个过程中：
	1. 首先计算出激活值的中位数。
	2. 然后将它们对齐，在一定范围内剪掉激活值。
3. 接下来，MinMax 量化方法将 FakeQuantize 层插入模型图中。
4. 然后，量化后的模型会通过偏差校正，根据卷积层或全连接层的量化误差，调整各层偏差。对深度学习模型进行量化，往往会使网络的统计数据偏离所学的分布。通过在神经网络的每一层的每个通道的偏置项中加入一个常数，偏置校正有助于克服这个问题。

该算法使用两阶段统计收集过程，其中模型在校准子集上推理出来，因此量化的实际时间基本上取决于子集的大小。

### Pipeline

`deepcopy`：完全复制一个新对象，对原对象的复杂子对象（如列表）的改变也不会影响新对象

### ActivationChannelAlignment

ActivationChannelAlignment 被用作量化前的初步步骤，允许调整卷积层的输出激活范围以减少量化误差。

对于无 `Convolution producer` 或有与许多节点相连的 `Convolution producer` 的卷积算子，不调整激活范围

### MinMaxQuantization

MinMaxQuantization 是一种原始的量化方法，它根据指定的目标硬件自动将 FakeQuantize 操作插入模型图中，并利用校准数据集上收集的统计信息对其进行初始化。

#### get_quantized_model

1. Quantize the model
2. Calculate quantization config for FQ nodes
3. Collect the weight stats based on config
4. Calibrate \[min, max\] for inserted fq nodes

### FastBiasCorrection

BiasCorrection 根据卷积层和全连接层的量化误差调整层的偏置，使整体误差无偏差。

## AccuracyAwareQuantization

AccuracyAwareQuantization 算法的特别之处在于，它不会让产生的量化模型下降到预定的精度范围以下。

![](https://s2.loli.net/2022/06/05/Tb8tmgOl3EZwPjL.png)

1. 输入全精度模型，通过 DefaultQuantization 算法输出一个快速的 8 位量化模型。
2. 然后，这个量化的模型在样本验证集上进行推导，提供一个准确度分数。
	- 如果准确度令人满意，并且与所需的阈值相匹配，所产生的模型将作为输出。
	- 如果准确度得分不符合标准，在到达输出阶段之前可能需要一些额外的步骤：
3. 如果是第一次迭代，那么就会进行层级排序，以检查哪一层对准确性的影响最大。
4. 对准确率下降贡献最大的量化层被完全恢复到原来的全精度格式。
5. 该模型针对验证集运行，以获得一个新的准确度分数，并与阈值进行比较。
6. 如果精度仍然不符合标准，就继续迭代。而在随后的每一次迭代中，将下一个最有问题的层转换回全精度。在第 3 步中完成的逐层排名有助于你确定下一步要针对哪一层。
7. 一旦准确度得分达到要求的最低值，输出所得到的模型。
