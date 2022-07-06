---
tags: []
categories: ["chaos"]
description:
summary:
draft: true
isCJKLanguage: true
date: "2020-02-12"
lastmod: "2022-06-10"
title: Effective Python
---

# Effective Python

主要记录阅读《Effective Python》一书中发现的对于现阶段我编写代码有帮助的内容。

## 主要目标

以 *Pythontic* 方式来编写程序。

## 第一章

### 遵循 PEP8 风格指南

#### 空白

- 使用空格而不是制表符来表示缩进

- 每行字符数不超过 79

- 多行的长表达式在换行时应多加一层缩进

- 文件中的函数与类用两个空行隔开

- 同一个类中各方法用一个空行隔开

#### 命名

- 函数、变量、属性：小写字母，单词间下划线连接 ==> `lowercase_underscore`
- 类与异常：每个单词首字母均大写  ==> `CapitalizedWord`
- 模块级别的常量应全部采用大写字母拼写 ==> `ALL_CAPS`
- 类的实例方法的首个参数命名为 `self`，表示该对象自身类方法的首个参数命名为 `chs`，表示该类自身

###　表达式和语句

- 采用内联形式的否定词，`if a is not b`  √  `if not a is b`  ×
- 不通过检测长度的办法（`if len(somelist) == 0`）判断 `somelist` 是否为空，采用 `if not somelist`，`if somelist` 判断
- 不编写单行的 if 语句、for 循环、while 循环及 except 复合语句
- 始终将 import 语句放在文件开头
- 总是使用绝对名称引用模块，`from bar import boo`  √  `import boo`  ×
- 按顺序将 import 语句分为三个部分：标准库模块，第三方模块，自用模块，每个部分各语句按模块字母排序

### 用辅助函数替代复杂表达式

代码清晰程度：if/else 表达式 > Boolean 操作符（or，and)

使用辅助函数代替复杂的表达式

### 切割序列

`somlist[-0:]` 为原列表的一份拷贝

单次切片操作内不要同时指定 start、end 和 stride `[start:end:stride​]`

### 列表推导

#### 用列表推导取代 map 和 filter

```python
a = [1,2,3,4,5,6,7,8,9,10]

# 计算平方值
# map
squares = map(lambda x: x**2, a)
# list comprehension(列表推导)
squares = [x**2 for x in a]

# 计算偶数平方值
# map, filter
alt = map(lambda x: x**2, filter(lambda x: x % 2 == 0, a))
# list comprehension
even_squares = [x**2 for x in a if x % 2 == 0]
even_squares == list(alt)
```

#### 不使用含两个以上表达式的列表推导

```python
my_list = [
    [[1,2,3], [4,5,6]],
    # ...
]
# 多表达式
flat = [x for sublist1 in my_lists
        for sublist2 in sublist1
        for x in sublist2]
# for循环
flat = []
for sublist1 in my_lists:
    for sublist2 in my_lists:
        flat.extend(sublist2)
```

### 用生成器表达式改写数据量较大的列表推导

生成器可减少内存消耗，防止因处理大数据量消耗内存造成的程序崩溃

将列表推导的 `[]` 改为 `()` 即可构成生成器表达式

生成器表达式可相互组合

```python
# 列表推导
value = [len(x) for x in open('my_file.txt')]
# 生成器-->使用()包含表达式
it = (len(x) for x in open('my_file.txt'))
# 生成器组合
roots = ((x, x**0.5) for x in it)
```

### 用 enumerate 代替 range

enumerate 可指定开始下标：

```python
for i,value in enumerate(somelist,1)
# i 从1开始
```

### 用 zip 同时遍历两个迭代器

zip 可将两个或两个以上的迭代器封装为生成器

**注意事项**：如果输入的迭代器长度不同，zip 会在其中一个耗尽时停止，此时可采用 `zip_longest` 函数替代

## 第二章

### 用异常来表示特殊情况，而不是 None

在作为判断条件时，None、0 或者空字符串等都会视为 False，所以在特殊情况下抛出异常而非返回 None 值

### 使用 nonlocal 以在闭包里使用外围作用域中变量

在表达式中引用变量时，python 解释器按如下顺序遍历所有域：

1. 当前函数的作用域
2. 任何外围作用域（例如包含当前函数的其他函数）
3. 包含当前代码的模块的 作用域（也叫全局作用域）
4. 内置作用域

给变量赋值时，如果当前作用域中无此变量，python 将此次赋值视为对该变量的定义，新变量作用域即为当前作用域。

在函数中可使用 `nonlocal` 表明应在上层作用域中查找该变量。

**限制**：`nonlocal` 不能延伸到模块级别。

```python
def func_fir(a,b):
    x = True
    def func_sec(y):
        nonlocal x
        return x
```

尽量在简单函数内使用 `nonlocal`，当代码复杂时，应将相关状态封装为辅助类。

### 用生成器改写直接返回列表的函数

`yield`

### 迭代注意事项

- 迭代器只能产生一轮结果，重复迭代无输出但常常不会报错

- 将迭代器传给 `iter` 函数，会返回该迭代器，将容器类型对象传给 `iter` 函数，每次都会返回一个新的迭代器对象，可用此特性来判别迭代器
- 把 `__iter__` 方法实现为生成器即可定义自己的容器类型。

### 函数参数优化

#### 数量可变的位置参数

在最后的位置参数前加 `*` 以代表任意数量的位置参数

```python
def func_one(a, *b):
    return a + sum(b)

#enable
func_one(1)
func_one(1, 2)
func_one(1, 2, 3)
```

传入已有列表时在列表前加上 `*` :

```python
list_one = [2, 3, 4]
func_one(1, *list)
```

**注意事项**：

- 变长参数在传给函数时会先转化为元组，传入生成器可能消耗大量内存导致程序崩溃。
- 在以后为函数添加新的位置参数时，应该使用只能以关键字形式指定的参数来扩展。

#### 用 None 和文档字符串来描述具有动态默认值的参数

函数参数的默认值，会在每个模块加载进来时求出，在 Python 中实现动态默认值的正确做法是把参数默认值设为 `None`，并在文档字符串中将 `None` 所对应的实际行为描述出来，编写函数代码时，若该参数值为 `None`，就将其设为实际的默认值

### 使用只能以关键字形式指定的参数

在参数列表中加入 `*` 号，标志位置参数在此终结，之后的参数只能以关键字形式指定。

```python
def func(a, b, *, x=True, y=False):
    pass

# TypeError
func(1, 2, True, False)

# Enable
func(1, 2)
func(1, 2, x=True)
```

## 第三章

### 尽量用辅助类维护程序的状态

避免使用多层嵌套，不要使用包含其他字典的字典

在保存内部数据的字典变得复杂时，应将这些代码拆解为多个辅助类

### 简单的接口应接受函数而非类的实例

* 未理解此条意思，感觉此条实际上一步步展示如何更好地调用类的实例作为 hook 函数 *

在类中使用 `__call__` 的特殊方法，可使类的实例能够像普通的 python 函数一样得以调用

