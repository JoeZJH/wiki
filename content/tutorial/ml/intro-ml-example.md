---
title: "第1.0讲：一个例子入门机器学习"
layout: page
date: 2018-02-05
---
[TOC]

## 关于
本讲内容将通过一个例子，入门机器学习。在这一讲中，你将学习到：

1. 什么是机器学习？
2. 机器学习能做什么？
3. 机器学习在代码上具体如何实现？

学习本讲，希望你

1. 有基本的 python 知识，你可以通过[python快速入门](/wiki/tutorial/ml/intro-python.html)快速了解python如何使用。

## 机器学习的本质
### 机器学习历史简述
现在人们通常将机器学习和人工智能联系在一起，实际上，人工智能涉及的领域更加宽泛，机器学习只是其中一种手段。人工智能的起源可以追溯到上世纪50年代，1956年举办的达茂思会议([Dartmouth Conference](https://en.wikipedia.org/wiki/Dartmouth_Workshop))，在这次会议上，信息论之父Shannon和IBM科学家Nathan Rochester等人，一起探讨了一个议题：精确地描述学习过程和智能的特征并用机器进行模拟。说人话！就是用机器模拟出人类的智能！

人工智能发展的初期，研究者致力于将人类的知识表达为一些逻辑规则，然后利用搜索进行逻辑推理，进而实现智能，到后来演变到利用知识库构造专家系统，实现所谓的智能。这期间，比较有名的成就有IBM的国际象棋程序深蓝打败国际象棋冠军。这一阶段的人工智能实现，更像人类的演绎推理，利用少量的规则，加上知识库，进行推演，从而得出结论。但是，规则的归纳需要人类专家干预，限制了这种模式的发展。2000年以后，随着互联网和摩尔定律的发展，产生了大量的数据和计算资源，使得人们可以利用机器从数据中自动归纳出规则，也就是数据驱动的智能。这其中的工具就是机器学习！

所以，**机器学习就是利用一种程序从数据中自动归纳出有价值的知识的一种方法**。


所谓演绎推理(Deductive Reasoning)，就是从一般性的前提出发，通过推导即“演绎”，得出具体陈述或个别结论的过程。演绎推理的逻辑形式对于理性的重要意义在于，它对人的思维保持严密性、一贯性有着不可替代的校正作用。我们熟知的很多数学证明方法，例如通过简单的几条公理，推导出整个欧式几何大厦的推理过程，就是典型的演绎推理。 下面是演绎推理里面一个典型的三段论推理的例子：

> 1. 知识分子都是应该受到尊重的，
> 2. 人民教师都是知识分子
> 3. 所以，人民教师都是应该受到尊重的。

演绎推理的核心思想就是从一般到特殊，将一些已经为真的通用性结论应用到具体的问题当中，得到具体的情况下的结论。这种推理方式保证了推理的严密性！在上述例子中，前两条就是一般性结论，知识分子是比人民教师更大的概念，第三条的结论就是将第一条结论应用到人民教师这个具体的个体上得到的更具体的结论！有趣的是，柯南道尔的著名小说中《福尔摩斯》中的大侦探福尔摩斯也十分推崇“演绎法”！为此，老美还专门拍了一部剧《福尔摩斯：基本演绎法》！ 只不过，福尔摩斯所声称的一般性结论和推理方式非常人能理解！

而归纳法是根据一类事物的部分对象具有某种性质的有限观察，推出这类事物的所有对象都具有这种性质的推理，叫做归纳推理（简称归纳）。归纳是从特殊到一般的过程，它属于合情推理。通常归纳法难以保证结论是可靠的，例如，下面就是经典的归纳法的例子：

> **黑天鹅事件**：17世纪之前，欧洲看到过的天鹅都是白色的，所以当时欧洲人归纳出一个结论：天鹅都是白色的！ 直到后来，欧洲人发现了澳洲，看到了当地的黑天鹅，人们才认识到这个结论是错误的！

从有限的经验归纳出来的结论当然不见得是可靠的，但是数学上也有完全归纳法，可以保证结论是可靠的的例子，我们以前学过的数学归纳法。从逻辑推理的角度来看，我们现在所用的机器学习就是先观察到一些数据，然后从这有限的数据中归纳出一些有用的规律的过程！ 因此，**机器学习本质上就是在做归纳推理，并且是不完全的归纳法**！我们前面说到，这种不完全归纳法无法保证结论的正确性，所以如果机器学习模型预测错了，请不要怪他，因为它是在做不完全归纳，肯定会犯错的！但是这并不意味着就没有用，事实上我们人类很多经验都是通过不完全归纳法归纳出来的，甚至可以说几乎所有实际的经验都来源于不完全归纳，完全归纳法只有在数学上才存在。只要归纳的结论大多数情况下是对的，那么他就是有用的！

### 机器学习可以干啥

具体来讲，机器学习可以用来做很多事情，目前已经有成功案例的就有很多。例如

- 计算广告：利用用户历史的行为数据，做广告的点击率(CTR)预估 <https://tech.meituan.com/deep-understanding-of-ffm-principles-and-practices.html>
- 推荐系统：利用用户历史行为做商品推荐 <https://book.douban.com/subject/10769749/>
- 游戏：Alpha Go <https://deepmind.com/research/alphago/>
- 金融：大数据风控
- 自然语言处理：机器翻译
- 语音识别
- 图像识别
- etc

接下来，我们就用一个实际的例子来解释机器学习是如何从数据中学到有用知识的。


## 从0开始机器学习

接下来，我们将利用一个简单的分类任务，给读者展示机器学习如何从数据中学到有用知识的。

### 任务与数据

本任务采用鸢尾花(iris)数据集，你可以从UCI网站上下载<https://archive.ics.uci.edu/ml/datasets/Iris>。如果已经安装了 scikit-learn，那么可以利用提供的dataset接口直接调用。鸢尾花数据集是著名的统计学家 Fisher 提供的。下面我们用一段简单的Python代码加载该数据集看一看。


```python
from sklearn.datasets import load_iris
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sn

plt.style.use('seaborn-talk')

iris = load_iris()
df = pd.DataFrame(iris.data, columns=iris.feature_names)
df['target'] = iris.target

# 随机选取几条数据
idx = range(df.shape[0])
np.random.shuffle(idx)

print iris.target_names
df.iloc[idx].head(10)
```

    ['setosa' 'versicolor' 'virginica']





|   | sepal length (cm) | sepal width (cm) | petal length (cm) | petal width (cm) | target  
|---|---|---|---|---|---  
91 | 6.1 | 3.0 | 4.6 | 1.4 | 1  
77 | 6.7 | 3.0 | 5.0 | 1.7 | 1  
99 | 5.7 | 2.8 | 4.1 | 1.3 | 1  
65 | 6.7 | 3.1 | 4.4 | 1.4 | 1  
14 | 5.8 | 4.0 | 1.2 | 0.2 | 0  
108 | 6.7 | 2.5 | 5.8 | 1.8 | 2  
142 | 5.8 | 2.7 | 5.1 | 1.9 | 2  
127 | 6.1 | 3.0 | 4.9 | 1.8 | 2  
24 | 4.8 | 3.4 | 1.9 | 0.2 | 0  
2 | 4.7 | 3.2 | 1.3 | 0.2 | 0




该数据集的每一条记录代表一个样本，每一个样本有4个属性变量：

- sepal length (cm) 萼片长度
- sepal width (cm) 萼片宽度
- petal length (cm) 花瓣长度
- petal width (cm) 花瓣宽度

每一个样本有1个目标变量target，target有3个取值，每一种取值的意义如下：

- 0： setosa 山鸢尾
- 1： versicolor 变色鸢尾
- 2： virginica 维吉尼亚鸢尾

每一种鸢尾花的图片如下，从左到右分别是 setosa,versicolor,virginica

![Iris](/wiki/static/images/iris.png)

### 建模

我们的目标是，建立一个模型，输入鸢尾花的4个属性变量，能够对鸢尾花的种类进行判别。这样一旦模型建立好了之后，对新看到的鸢尾花，只要测量了这4个属性，就可以利用模型对它的类别进行预测了。

数学地角度来说，我们要确定一个函数 $f: R^4 \rightarrow \{0,1,2\}$，输入是一个4维向量 $\vec{x} = (x_1, x_2, x_3, x_4)$，每一维代表一个属性变量的值，输出一个分类变量 $y \in \{0,1,2\}$，代表该样本属于哪个类别。所谓的建模过程，就是利用我们已经观测到的数据集，去确定这个函数 $f$ 的具体形式和参数。这里每一个属性我们都称作一个特征，输出分类变量我们陈做目标（或建模目标），这里的函数 $f$ 就是我们通常所说的模型。

#### 简单规则模型

在建立复杂模型之前，我们先来建立一种简单规则模型。所谓的简单规则，就是对一个属性，通过规则判定，确定该样本属于哪一个类。比如，我们可以进行数据分析，观察每一种花的萼片长度、萼片宽度、花瓣长度、花瓣宽度的平均值。


```python
ax = plt.gca()
df.groupby(by='target').plot(kind='line', x='petal length (cm)', y='petal width (cm)', style='.', ax=ax)
ax.set_xlim([0,8])
ax.set_ylim([0,3])
plt.legend(iris.target_names, loc='best')
plt.xlabel('petal length (cm)')
plt.ylabel('petal width (cm)')

df.groupby(by='target').mean().plot(kind='bar')
plt.xticks([0,1,2], iris.target_names);
plt.xlabel('')
plt.ylabel(u'平均值')
```


![svg](/wiki/static/images/iris-01.svg)

![svg](/wiki/static/images/iris-02.svg)


通过上述分析，可以看到三种花的花瓣长度(petal length)差异比较大，setosa的平均花瓣长度在1.5cm左右，versicolor的平均花瓣长度在4.2cm左右，而 virginica的平均花瓣长度在5.6cm左右。因此，一种简单规则模型可以归纳为

$$
target =
\begin{cases}
0, \text{petal length} \lt 2.8 \\\\
1, \text{petal length} \in [2.8, 4.9) \\\\
2, \text{petal length} \ge 4.9
\end{cases}
$$

这里分割点的值取的是两种花平均中的平均数。

好了，到目前为止，你已经学会了数据挖掘过程中的最简单情景了。通过数据分析，归纳出规则，然后将规则编码成一个函数，从而得到一个预测模型，可以用来做预测。

很快，我们会发现，这种方法需要人工进行数据分析，总结出规则，那么能不能够让程序自动地找到这些规则，甚至发现更复杂的规则呢？答案是肯定的，决策树就是这样一种模型，自动地发现这些规则，甚至高阶组合规则。

#### 决策树模型

前面我们已经说过，决策树是这样一种模型，它可以自动地发现一些区分目标的规则，甚至高阶组合规则。决策树如何发现这些规则我们暂时不要去深究，作为一个入门课程，我们重点是了解模型能干啥。这里，我们利用 scikit-learn 软件包里面的决策树模型工具，建立模型。

决策树模型分为回归和分类，如果目标变量是类别变量，这样的问题成为分类，我们这个任务就是分类。而如果目标变量是连续变量，例如预测房价，那么这样的问题就是回归。这里我们只用分类就好了，对应的类是 `DecisionTreeClassifier`，为了便于观察，我们限定树的深度为2。为了让决策树模型能够从数据中学会规则，我们需要调用模型的 `fit` 方法，并将数据（包括特征`iris.data`和目标`iris.target`）传给它。

模型从数据中自动学会这些规则的过程，我们称之为**训练**或者**拟合**。因此，`fit`方法实际上就是在做**模型训练**！

模型训练好了之后，我们可以将决策树画出来，进行观察。


```python
from sklearn.tree import DecisionTreeClassifier, export_graphviz
import graphviz

clf = DecisionTreeClassifier(max_depth=2)
clf.fit(iris.data, iris.target)

dot_data = export_graphviz(clf, feature_names=iris.feature_names,
                           class_names = iris.target_names,
                           out_file=None, filled=True)

graphviz.Source(dot_data)
```




![svg](/wiki/static/images/iris-3.svg)



决策树模型是由规则组成的一棵决策树，它的节点分为内部节点和叶子节点。内部节点对应一条分裂规则，叶子节点对应一个判决或者输出，对于分类任务输出的类别是满足这个规则样本最多的那个类别。

上面的决策树是一颗二叉树，根节点对应规则是petal length (cm) <= 2.45，如果满足这条规则，就到了左子树，左子树是一个叶子节点，满足这条规则到达左子树的样本有50个，且全部为setosa这个类别，因此输出类别是 setosa。

这和前面我们通过数据分析得出的规则 petal length < 2.8 则为setosa，非常接近。



#### 线性模型

线性模型和决策树模型行为完全不同，线性模型的出发点是对每一类，计算一个分数

$$
score_i = b_i + \vec{w} _ i^T \vec{x}
$$

然后选择出得分最大的类作为预测的类。模型的权重$\vec{w} _ i$称作模型参数，通过优化算法优化得到。优化模型参数的过程就叫做模型训练！

线性模型（确切地说是线性多分类模型）里面最典型的是多分类逻辑回归，它将每个类的分数做归一化，得到样本归属于该类的概率

$$
P(i|\vec{x}) = \frac{e^{score_i}}{\sum_i e^{score_i}}
$$

下面利用`scikit-learn`多分类逻辑回归工具 `LogisticRegression` 进行建模。


```python
from sklearn.linear_model import LogisticRegression

lr = LogisticRegression(C=0.1)
lr.fit(iris.data, iris.target)

plt.bar(np.arange(4)-0.3, lr.coef_[0], width=0.2)
plt.bar(np.arange(4)-0.1, lr.coef_[1], width=0.2, color='r')
plt.bar(np.arange(4)+0.1, lr.coef_[2], width=0.2, color='g')
plt.xticks(range(4), iris.feature_names);
plt.ylabel(u'权重')
plt.legend(iris.target_names, loc='best')
```



![svg](/wiki/static/images/iris-4.svg)


从模型参数来看，sepal length 和 sepal width 数值大的类别setosa的得分更高，petal length 数值较大的 versicolor的得分更高，petal length 和 petal width 数值大的 virginica 得分更高。

### 预测

一旦模型建立好了之后，我们就可以利用模型进行预测了，所谓的预测，是指对于一个新的样本，比如我在某个路边看到了一朵鸢尾花，不知道到底是哪一类，就可以利用这个模型进行预测。首先，我们需要测量模型预测所需要的4个数据（特征），花萼的长度和宽度，花瓣的长度和宽度，然后输入的模型中去。

对预测前面简单规则模型，只需要花瓣的长度数据即可预测。对于决策树模型，实际上只需要花瓣的长度和宽度数据也可预测，如果我们将决策树深度变得更深，那么就可能要用到所有数据。首先，决策树从根节点开始搜索，根节点对应一条规则 petal length (cm) <= 2.45，如果满足这条规则，就到左子树，预测输出为setosa。如果不满足，那么就到右子树，右子树根节点还是一个规则 petal width (cm) <= 1.75。我们重复这个过程，直到找到该样本满足规则的叶子节点，叶子节点对应的输出值就是模型预测结果。

## 总结

这里我们以鸢尾花分类任务为例，构建了一个决策树模型进行预测。总结起来，所谓的建模过程，就是利用已有的标注数据（已知目标变量的值的数据），自动学习到一个函数 $f:R^n \rightarrow Y$，根据观察到的特征向量，计算得到目标变量的值。这个任务就是一个3分类的函数。虽然这个任务简单，但是和更复杂的任务都具有以下3个基本步骤：

- 收集（标注）数据
- 建立模型
- 预测

不同的业务可能收集到的数据不同，收集到的原始数据需要加工成模型能用的数据（即特征）。

不同的任务可以设立不同的目标进行建模，比如预测性别，那么目标变量是男和女；预测年龄，那么目标变量是个0-100之间的连续值；预测股价涨跌，那么目标变量就是涨和跌。

相同数据和目标的情况下，也可以选择不同的模型，决策树是一个久经沙场的模型，它的两个变体**随机森林**和**梯度提升树**应用非常广泛。近年来的深度神经网络，可以利用非常原始的数据进行建模，减少了人工特征工程的工作量。但是本质上，他们都在干同一件事情，学习这样一个函数！

## 作业
一个编程小练习，探索决策树的深度与预测的准确率的关系，并解释一下观察到的现象的原因。

在编程之前，请先配置好环境：

- 安装 Python <https://www.python.org/downloads/release/python-2713/>
- 安装Python包
    - scikit-learn <http://scikit-learn.org/stable/install.html>
    - pandas 数据处理包
    - matplotlib 绘图包
    - seaborn  可视化包(本作业暂时不用)
    - graphviz 可视化包(本作业暂时不用)

可以用python包管理器`pip`安装相关包，`pip`默认会使用国外的软件源，在国内下载较慢，建议使用国内的镜像：

- USTC：<https://lug.ustc.edu.cn/wiki/mirrors/help/pypi>
- THU：<https://mirrors.tuna.tsinghua.edu.cn/help/pypi/>

```python
from sklearn.datasets import load_iris
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.tree import DecisionTreeClassifier

plt.style.use('seaborn-talk')

iris = load_iris()
df = pd.DataFrame(iris.data, columns=iris.feature_names)
df['target'] = iris.target

depths = range(2,10)
errors = []

"""下面是你的代码，请完成功能：
   1. 用深度为2-10的不同决策树分别对数据进行建模，计算每一颗决策树的预测准确率。准确率是预测正确的样本数目 / 总样本数目。
   2. 画出深度-误差的折线图

   Hint:
      1. sklearn每一个模型都会有一个`predict`方法，可以用来预测结果。
      2. matplotlib 的画图函数 `plt.plot` 是有用的画图工具。
"""

```


<img src="/wiki/static/images/support-qrcode.png" alt="支持我" style="max-width:300px;" />