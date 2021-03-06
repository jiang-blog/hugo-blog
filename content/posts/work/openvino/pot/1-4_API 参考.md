---
tags: []
categories: ["work", "openvino_pot"]
description:
summary:
draft: false
isCJKLanguage: true
date: "2022-05-28"
lastmod: "2022-06-10"
title: 1-4-API参考
---

# 1-4-API参考

培训后优化工具API提供了一整套接口和助手，允许用户为各种类型的DL模型（包括级联或复合模型）实现自定义优化pipeline。下面是这个API的完整规范:

## DataLoader

```python
class openvino.tools.pot.DataLoader(config)
```

所有DataLoaders的基类。

`DataLoader`从一个数据集加载数据，并对它们进行预处理，提供对预处理数据的索引访问。

所有的子类都应该覆盖`__len__()`函数，它应该返回数据集的大小，以及`__getitem__()`，它支持在0到`len(self)`范围内的整数索引。`__getitem__()`方法可以以一种可能的格式返回数据：

```python
(data, annotation)
```

或

```python
(data, annotation, metadata)
```

`data`是在推理时传递给模型的输入，所以它应该被适当地预处理。`data`可以是`numpy.array`对象或字典，其中key是模型输入的名称，value是与此输入对应的`numpy.array`。`annotation`的格式应该与`Metric`类的期望一致。`metadata`是一个可选的字段，可以用来存储后处理所需的额外信息。

## Metric

```python
class openvino.tools.pot.Metric()
```

一个抽象的类，代表一个精度指标。

所有实例应覆盖以下属性。

- `value` - 返回最后一个模型输出的精度指标值，格式为`Dict[str, numpy.array]`。

- `avg_value` - 以`Dict[str, numpy.array]`的格式返回收集的模型结果的平均精度指标。

- `higher_better`应该返回`True`，如果指标的值越高，说明性能越好，否则，返回`False`。默认实现返回`True`

和方法：

- `update(output, annotation)` - 使用最后的模型输出和注释来计算和更新精度指标值。模型的输出和注释应该在这个方法中传递。它还应该包含模型特定的后处理，以防模型返回原始输出。

- `reset()` - 重置收集的精度指标。

- `get_attributes()` - 返回一个度量衡属性的字典。

  ```python
  {metric_name: {attribute_name: value}}。
  ```

  必要的属性:

  - `direction` - (`higher-better`或`higher-worse`)一个字符串参数，定义在精度感知算法中是否应该增加度量值。

  - `type` - 公制类型的字符串表示。例如，'准确度'或'平均值'。

## Engine

```python
class openvino.tools.pot.Engine(config, data_loader=None, metric=None)
```

所有Engines的基类.

该引擎提供模型推理、激活统计数据收集和数据集准确度指标的计算。

_参数_

- `config` - 引擎特定的配置。

- `data_loader` - `DataLoader` 实例来迭代数据集。

- `metric` - `Metric` 实例，用于计算模型的准确度指标。

所有子类都应覆盖以下方法：

- `set_model(model)` - 设置/重置模型。

  _参数_

  - `model` - `CompressedModel` 实例用于推理。
- `predict(stats_layout=None, sampler=None, metric_per_sample=False, print_progress=False)` - 对指定的数据子集执行模型推断。

  _参数_

  - `stats_layout` - 统计收集函数字典。 一个可选参数。

  ```python
  {
      'node_name': {
          'stat_name': fn
      }
  }
  ```

  - `sampler` - `Sampler` 实例，提供了一种迭代数据集的方法。 （详见下文）。

  - `metric_per_sample` - 如果指定了 `Metric` 并且此参数设置为 True，则应为每个数据样本计算度量值，否则为整个数据集。

  - `print_progress` - 打印推理进度。

  _返回_

  - 如果 `metric_per_sample` 为 True，则为每个样本和整体度量值的字典元组

  ```python
  (
      {
          'sample_id': sample_index,
          'metric_name': metric_name,
          'result': metric_value
      },
      {
          'metric_name': metric_value
      }
  )
  ```

  否则，一个整体指标的字典。

  ```python
  { 'metric_name': metric_value }
  ```
- 收集的统计数据字典

  ```python
  {
      'node_name': {
          'stat_name': [statistics]
      }
  }
  ```

## Pipeline[¶](#pipeline "Permalink to this headline")

```
class openvino.tools.pot.Pipeline(engine)
```

pipeline类表示优化pipeline。

_参数_

- `engine` - 用于模型推理的`Engine` 类的实例。

可以通过调用 `run(model)`方法将pipeline应用于 DL 模型，其中 `model` 是 `NXModel` 实例。

### 创建pipeline

POT Python* API 提供了用于创建和配置pipeline的实用程序函数：

```python
openvino.tools.pot.create_pipeline(algo_config, engine)
```

_参数_

- `algo_config` - 定义优化pipeline中包含的优化算法及其参数的列表。它们在优化pipeline中应用于模型的顺序由列表中的顺序决定。

  pipline的算法配置的例子：

  ```python
  algo_config = [
      {
          'name': 'DefaultQuantization',
          'params': {
              'preset': 'performance',
              'stat_subset_size': 500
          }
       },
      ...
  ]
  ```

- `engine` - 用于模型推理的`Engine` 类示例

_返回_

- `Pipeline` 类实例。

#### 帮助器和内部模型表示法

为了简化优化pipeline的实施，我们提供了一套随时可用的帮助器。这里我们也描述了DL模型的内部表示，以及如何使用它。

## IEEngine[¶](#ieengine "Permalink to this headline")

```python
class openvino.tools.pot.IEEngine(config, data_loader=None, metric=None)
```

IEEngine是一个辅助工具，它基于[OpenVINO推理引擎Python* API](https://docs.openvino.ai/latest/ie_python_api/api.html)实现引擎类。这个类支持同步和异步模式的推理，可以在自定义pipeline中原样重用，也可以做一些修改，例如，在推理结果的自定义后处理中。

以下方法可以在子类中被重写。

- `postprocess_output(output, metadata)` - 使用数据加载时获得的图像元数据处理模型输出数据。

  _参数_

  - `outputs` - 每个输出名称的输出数据字典。

  - `metadata` - 用于推理的数据信息。

  _返回_

  - 输出数据的列表，如果使用了准确度指标，则以准确度指标所期望的顺序排列。

`IEEngine`支持由`DataLoader`返回的数据格式:

```python
(data, annotation)
```

或

```
(data, annotation, metadata)
```

由`Metric`实例返回的度量值应该是这样的格式。:

- 对于 `value()` :

  ```python
  {metric_name: [metric_values_per_image]}
  ```

- 对于 `avg_value()` :

  ```python
  {metric_name: metric_value}
  ```

为了实现一个自定义的 `Engine`类，你可能需要熟悉以下接口：

## CompressedModel

Python* POT API提供了`CompressedModel`类作为一个接口，用于处理单一和级联的DL模型。它被用来加载、保存和访问模型，在级联模型的情况下，访问级联模型的每个模型。

```python
class openvino.tools.pot.graph.nx_model.CompressedModel(**kwargs)
```

`CompressedModel`类提供了一个DL模型的表示。一个单一的模型和级联的模型可以被表示为这个类的实例。级联模型被存储为一个模型的列表。

_属性_

- `models` - 级联模型的模型列表。

- `is_cascade` - 如果加载的模型是级联模型，则返回 True。

## 从OpenVINO IR中读取模型

Python* POT API提供了从OpenVINO中间表征(IR)中加载模型的实用函数。

```python
openvino.tools.pot.load_model(model_config)
```

_参数_

- `model_config` - 描述一个模型的字典，包括以下属性。

  - `model_name` - 模型名称。

  - `model` - 网络拓扑结构的路径（.xml）。

  - `weights` - 模型权重的路径（.bin）。

  单个模型的`model_config`的例子:

  ```python
  model_config = {
      'model_name': 'mobilenet_v2',
      'model': '<PATH_TO_MODEL>/mobilenet_v2.xml',
      'weights': '<PATH_TO_WEIGHTS>/mobilenet_v2.bin'
  }
  ```

  级联模型的`model_config`的例子:

  ```python
  model_config = {
      'model_name': 'mtcnn',
      'cascade': [
          {
              'name': 'pnet',
              "model": '<PATH_TO_MODEL>/pnet.xml',
              'weights': '<PATH_TO_WEIGHTS>/pnet.bin'
          },
          {
              'name': 'rnet',
              'model': '<PATH_TO_MODEL>/rnet.xml',
              'weights': '<PATH_TO_WEIGHTS>/rnet.bin'
          },
          {
              'name': 'onet',
              'model': '<PATH_TO_MODEL>/onet.xml',
              'weights': '<PATH_TO_WEIGHTS>/onet.bin'
          }
      ]
  }
  ```

_返回_

- `CompressedModel` 实例

### 将模型保存为IR

Python* POT API提供了保存模型为OpenVINO中间表征（IR）的实用功能。

```python
openvino.tools.pot.save_model(model, save_path, model_name=None, for_stat_collection=False)
```

_参数_

- `model` - `CompressedModel`实例。

- `save_path` - 保存模型的路径。

- `model_name` - 模型将被保存的名称。

- `for_stat_collection` - 模型是否被保存用于统计收集或正常推理（仅影响级联模型）。如果设置为False，将从节点名称中删除模型前缀。

_返回_

- 带有路径的字典列表：

  ```python
  [
      {
          'name': model name,
          'model': path to .xml,
          'weights': path to .bin
      },
      ...
  ]
  ```

## Sampler

```python
class openvino.tools.pot.samplers.Sampler(data_loader=None, batch_size=1, subset_indices=None)
```

所有Sampler的基类。

Sampler提供了一种在数据集上迭代的方法。

所有的子类都覆盖了`__iter__()`方法，提供了一种对数据集进行迭代的方法，以及一个`__len__()`方法，返回返回迭代器的长度。

_参数_

- `data_loader` - `DataLoader`类的实例，用于加载数据。

- `batch_size` - batch中的项目数，默认为1。

- `subset_indices` - 要加载的样本的索引。如果`subset_indices`被设置为None，那么sampler将从整个数据集中获取元素。

## BatchSampler

```python
class openvino.tools.pot.samplers.batch_sampler.BatchSampler(data_loader, batch_size=1, subset_indices=None):
```

如果指定了 "subset_indices"，sampler提供一个数据集子集的迭代，或者在给定的 `batch_size`下提供整个数据集的迭代。返回一个数据项的列表。
