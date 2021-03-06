---
title: "第03.0讲：机器学习建模实战"
layout: page
date: 2018-02-06
---
[TOC]

## 关于
本讲内容将通过一个例子，深入理解机器学习建模的整个过程。在这一讲中，你将学习到：

1. 什么是过拟合与泛化？为什么要划分训练集、测试集、验证集。
2. 什么是交叉验证？
3. 如何评估一个分类模型的好坏？
4. 方差-偏差分解
5. 决策树的减枝与泛化

学习本讲，希望你

1. 至少有高中数学水平。
2. 了解决策树，如果还不了解，可以参看[决策树模型](/wiki/tutorial/ml/intro-dt.html)
2. 如果你需要完成实践部分，需要有基本的 python 知识，你可以通过[python快速入门](/wiki/tutorial/ml/intro-python.html)快速了解python如何使用。

本讲所用的数据集还是采用鸢尾花(iris)数据集，你可以从UCI网站上下载<https://archive.ics.uci.edu/ml/datasets/Iris>。如果已经安装了 scikit-learn，那么可以利用提供的dataset接口直接调用。鸢尾花数据集是著名的统计学家 Fisher 提供的。下面我们采样少量的数据看一看。

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

这个数据集一共有150个样本，这三种花都有50个样本。上面显示出来的只是随机选取的一部分数据。每一种鸢尾花的图片如下，从左到右分别是 setosa,versicolor,virginica

![Iris](/wiki/static/images/iris.png)

## 分类准确率
在上一讲中，我们说到决策树生成算法，每一步都是穷举所有可能的分裂规则，利用信息增益准则找到最佳规则，将最佳规则加入决策树中，持续这个步骤知道满足停止准则。在前面，我们介绍了三个停止准则，分别是：如果集合中只有一类样本，就停止分裂；如果集合中样本个数少于某个阈值，就停止分裂；如果树的深度达到某个阈值，就停止分裂。显然，如果停止准则不同，那么我们得到的决策树也不同，这么多决策树如何评估哪一个好，哪一个不好呢？

以上述鸢尾花分类任务为例，分类得好和坏可以通过分类转确率来衡量，分类准确率=分类正确的样本数目/总样本数目。如下图所示，分别是3棵决策树，深度分别为0、1、2，叶子节点中的3个数字分别代表训练样本中3类花的样本数目。第一棵决策树只有一个叶子节点，也就是对所有样本都预测同一个值（训练样本中最多的那一类，在这个例子中3类样本一样更多，任意一个预测类别都一样），假设预测为第0类，那么准确率只有1/3。第二棵决策树深度为1，根据花瓣长度是否小于2.45将特征空间划分为两部分，每一部分对应一个叶子节点，左边的叶子节点预测为第0类，而右边的叶子节点预测为第1类（由于第1类样本和第2类样本一样多，所以预测为第1类和第2类效果是一样的）。因此，分到左子树的50个样本都会预测为第0类，都预测准确，而分到右子树的100个样本只有一半预测准确，总的预测准确样本数是100，准确率为2/3。随着决策树深度的加深，训练样本预测的准确率会越来越高，知道每个叶子节点上都只有一类样本，那么训练样本的预测准确率将达到100%！如果以训练集上的准确率为评估指标，那么显然越深的决策树预测效果越好。而且可以想象，对任何一个训练集，总可以不断地将决策树加深，直到每个叶子节点上都只有一类样本。那么，训练集上的准确率越高是否就代表模型的预测能力越好呢？

![决策树](/wiki/static/images/dt-03.png)


## 交叉验证
我们前面讲过，机器学习就是从数据中自动发现一些有意义的规律。这些规律是有意义的，意味着可以用来指导实践，用来对没有见到过的数据进行预测。以鸢尾花为例，我们希望通过机器学习从已经观测到的那150个样本，找到一些关于4个属性与类别的规律，这样我们就可以通过这4个属性对没有见过的鸢尾花预测它的类别了。我们用X表示用来预测的属性，也就是特征，用Y表示预测的目标类别，也就是标签，那么在所有可能的数据中，可以用一个联合概率分布P(X, Y)来表示X和Y的概率关系，它的意义是在所有的数据中特征等于X且类别为Y的概率。这些所有可能的数据构成的集合我们称作总体，P(X, Y)就是总体的概率分布，已经观测到的数据 D={(x,y)| (x,y)~P(X, Y)} 只是总体的一个采样结果，它们只是总体很少的一部分并且通常假设这些样本是独立选取的。你可以理解为1个样本是否选取与其他样本没有关系，因此样本间统计独立，又因为他们都来自同一个总体，所以他们是独立同分布(i.i.d)的！机器学习的目标就是从这有限的观测数据D中学习到关于总体的规律。

![经验和总体](/wiki/static/images/cross_validate01.png)

因此，我们当然不希望模型学到的只是训练集上看到的有限的规律，甚至只有少数样本表现出来的假规律。这也是因为机器学习采用的是不完全归纳法，所以存在被经验误导的可能性，我们希望模型学到这种错误规律的风险尽可能小，这样模型才是有价值的，所以我们应该在全量的总体上来评估模型的效果才靠谱。以鸢尾花任务为例，确切来说就是要求模型在未知的来自同一个总体的样本上，分类准确率尽可能高。如果我们用函数h(x)表示训练好的模型，对于样本的特征为x，模型的预测结果$\hat{y} = h(x)$。用示性函数$I(h(x) = y)$表示模型是否预测正确，预测正确结果为1，预测错误结果为0.那么模型在训练集上的准确率为

$$
\hat{P} _ c = E_{(x, y) \sim D} I(h(x) = y)
$$

上述准确率并不是一个评估模型效果好坏的指标，而应该用模型在总体上的准确率

$$
P _ c = E_{(x, y) \sim P(x, y)} I(h(x) = y)
$$

这两个准确率的唯一区别就是求期望是在观测数据上还是在总体上计算的结果。在总体上的期望值才有意义，才能表达模型在未知数据上的预测效果。但是，这个期望值我们无法求，因为总体对我们来说是未知的，那怎么办呢？一个简单的方法是，将观测数据划分成两部分，一部分用来训练模型，而用另外一部分计算准确率作为模型在总体上准确率的估计值。这样划分的两个集合我们叫做训练集和测试集，在训练集上训练模型，而在测试集上评估模型的效果。

有了测试集后，就可以用测试集上的预测效果作为模型在总体上的预测效果的一个较好的估计。前面说过，在训练集上，深度越深的决策树准确率约高，但是在测试集上的效果就不见得。那么我们能不能通过测试集上的效果来选择最佳的模型呢？答案是不能的！模型的建立过程中不能涉及到任何测试集上的信息，否则测试集上的评估结果就不能很好地反应出模型真实的性能。为了从很多模型中选择一个最好的，我们还需要将训练集继续划分成训练集和验证集！用训练集训练多个模型，用验证集选出最好的一个模型，测试集只能用于评估！但是在实际建模任务中，我们会用真实的未观测数据来评估模型，比如训练好一个推荐的模型，上线之后运行一段时间来验证实际效果，因此也有只将训练集划分成训练集和验证集两个集合的做法，而用线上效果来评估模型。而在学术论文当中，为了与同行比较效果，那么就需要划分成3个集合，用验证集选择模型，而用测试集报告本论文方法的效果与同行进行比较。曾经有学者在选择模型用到了测试集的数据，最后被发现了，这是严重的学术不端行为！

将训练集划分成训练集和验证集选择模型的方法也叫做交叉验证(Cross Validate)，交叉验证还有一些其他方法，这里再介绍两种：k折叠和留一法。k折叠是为了解决训练集数目少的问题，如果训练集数目很大，上述简单的划分就可以了，但是如果训练集较小，某一次划分带来的统计波动很大，使得这种验证方法不稳定。为了解决这个问题，可以将训练集随机划分成k个相等的集合，用其中k-1个集合训练模型，而用剩下一个计算评估指标，但是这个评估指标并不是最终选择模型的指标。选择评估集合可以有k种不同的选择方式（k个集合任何一个都可以作为评估集合），因此可以用相同的超参数（我们将决策树深度、每个叶子节点最少样本数目等等这种在训练模型之前就需要确定的参数叫做模型的超参数）训练k次，可得到k个准确率，然后用这k个准确率的平均值作为最终的评估指标来选择**不同的超参数**。

![k折叠交叉验证](/wiki/static/images/k-fold.png)

留一法可以看做k等于训练样本数目的特殊情况，也就是每次只留下一个样本来评估。在k折叠交叉验证中，k越大，那么评估集上估计的统计误差就越小，因为评估指标是k个评估结果的平均值！但是计算消耗的资源就越多，留一法是评估指标最接近总体上的评估指标的，但是计算消耗的资源也最多，一般应根据训练集合的大小来选择合适的k值，一般k取3到10是比较合理的。

以鸢尾花任务为例，为了选择最佳的决策树深度这个超参数，我们可以利用5折叠交叉验证。对每一个深度，利用5折叠交叉验证计算出k折叠平均准确率，平均准确率最高的那个深度值就是最佳的！然后我们可以将深度设置为这个最佳值，在全量训练样本中重新训练决策树模型，作为最终的预测模型！

## 过拟合与决策树减枝
将数据集划分成训练集合测试集，可以评估训练集上训练的模型的好坏。训练集上效果越好并不代表在测试集上的效果越好，当然也不代表在总体上的效果越好。下图是在鸢尾花任务中，将数据按照8:2的比例随机划分为训练集和测试集，限制决策树的最大深度为不同值时，训练集(train)和测试集(test)上的误差变化。随着深度的增加，训练集上的误差逐渐降低至0，而测试集上的误差先变低后变高并发送波动（实际效果跟划分结果有关）。而一般的数据集中，测试集上的效果会随着训练集误差降低反而增加！这种训练集和测试集效果不一致的现象，我们说模型发生了过拟合现象。模型过度拟合了训练集，但是在未见过的测试集上的效果随训练集拟合精度提高反而下降了！

![过拟合](/wiki/static/images/over-fitting.svg)

过拟合是机器学习建模中经常遇到的问题，过拟合现象可以通过交叉验证发现，同时通过限制模型复杂度等措施在一定程度上降低过拟合的程度。例如可以限制决策树深度、每个叶子节点上的样本数目等措施，减少模型过拟合风险。此外还可以通过对决策树减枝的方法解决过拟合问题。事实上，通过限制叶子节点上最少样本数目就是在做减枝，防止决策树过度生长，这种在生成决策树过程中减少决策树的分支的方法叫做预减枝。也可以先让决策树充分生长，然后测试决策树每个分支上的最后一次分裂是否有足够大的信息增益，如果没有就合并这两个叶子节点，这种方法叫后减枝。这两种减枝的方法都是通过限制模型复杂度减少决策树过拟合的风险。

关于模型选择有个奥卡姆剃刀原理，这个原理说如果两个模型都能解释数据，那么应该选择最简单的那一个。用过拟合来解释就是说更复杂的那个模型有更高的过拟合风险，因此如果模型效果没有明显提升的话，不应该选择更复杂的模型。


### 过拟合的方差偏差解释
根据偏差-方差分解，模型的预测误差可以分解为偏差和方差。对给定的待预测样本$x$，估计出来的预测函数$\hat{f}$会随着训练集的不同而改变，是一个随机变量。假设实际的关系是$y = f(x) + \epsilon$，$\epsilon$是噪声随机变量，所以$y$也是一个随机变量，但是$f(x)$是常数。那么模型预测值$\hat{f}$和实际值$y$之间的期望误差可以分解为

$$
\begin{align}
E[(y - \hat{f}(x))^2] &= E[y^2 + \hat{f}^2 - 2y\hat{f}] \\\\
&=Var[y]+ Var[\hat{f}] +[Ey]^2 +[E\hat{f}]^2 - 2 f E\hat{f} \\\\
&= Var[y] + Var[\hat{f}] + (f - E\hat{f})^2 \\\\
&= \sigma^2 + Var[\hat{f}] + Bias[\hat{f}]^2
\end{align}
$$

误差可以分为三项，第一项是该问题由于信息缺失等问题带来的固有误差，无法消除；第二项是$\hat{f}$因为训练集选取的不同所带来的统计涨落误差，称为方差；第三项是把所有可能的训练集都训练一遍，得到的函数预测值平均消除统计涨落后还无法消除的偏差！

模型的过拟合可以看做方差很大，对特定的训练集合误差很小而对其他训练集合误差很大，所以对某个固定的观测样本，模型预测的结果的波动受训练集的选择影响很大，也就是模型的方差很大。

![方差-偏差解释](/wiki/static/images/variance-bias.png)

## 特征工程
### 类别特征
在实际问题中常常会碰到类别特征，例如性别分为男和女，职业有学生、白领、蓝领、无业等，文章的分类有社会、科技、经济等等。这一类的特征和鸢尾花任务的特征不同的是，这类特征的取值是有限个并且没有明确的数值关系。对于类别特征，我们通过one-hot编码的方式将它们对应为数值向量。one-hot编码的方法是，如果这个特征有n个不同的取值，那么就用一个n维向量来表示这一个特征，每一维对应一个取值，如果这个特征取第k个值，那么就将这个特征第k维置1，而其他位置都置0。以上述的职业特征为例，假设职业取值有5个，分别是学生、白领、蓝领、无业、未知。那么我们需要用一个5维的向量来编码职业这个特征，对于某个样本职业特征为白领，那么这个5维的向量为[0, 1, 0, 0, 0]，除了第2位为1其他全为0。通过这种编码之后，类别特征也可以和连续值特征一样进行相同的处理了，这个方法在以后介绍的模型中会大量用到。

不过，决策树还有一个更方便的处理类别特征的方法，直接分裂为多个子树而不是两个！这样一来，决策树的中间节点可能存在多个分支，每个分支对应该特征的一个取值，那么也就是一个等于规则，如下图所示，最左边的分支对应“职业=学生”这条规则。

![决策树-类别特征](/wiki/static/images/category-feature.png)

决策树处理类别特征的另外一个方法是子集划分，这个方法在微软的LightGBM中被用到。一个类别特征的n个不同的取值构成的集合，划分成两个不同取值不同的子集的方法有$2^n$个，因此遍历每一种划分要求的计算复杂度很高。这种问题的难点来自于n个取值的无序性，试想如果n个取值是有序的，那么这就与之前见到的连续值特征一样的处理就好了。基于这个思考，可以考虑根据特征与目标变量的相关性为这些值赋予一定的序关系。例如，在建模用户的收入水平问题中，可以根据高收入占比排序，将职业的5个取值排序，假设从低到高分别是：无业、未知、学生、蓝领、白领。我们只将这5个值划分成两个子集A和B，其中A中的职业中高收入占比都比B要低，因此这种划分方式只有4种，一般情况下只有n-1种划分方式。相比于遍历无需集合所有子集大大减少了计算量。

![类别特征的划分](/wiki/static/images/category-split.png)

### 缺失值
在实际问题中第二类常遇到问题是特征缺失值，一般用NULL表示一个样本的某个特征值缺失，这种缺失可能是多种原因造成的，例如确实未知、录入的时候忘记填了、脏数据等等。对于缺失值的处理，一般可以根据先验知识填充对应的值。例如在年龄特征缺失时，可以用这个任务下的用户的平均值、中位数等全局统计指标填充；在性别缺失的时候，可以用数据中的众数填充，例如在唯品会上的建模，性别大多数女性，所以对性别缺失值用女性填充。对于类别特征，还可以将缺失值当做一个新的特殊类别处理，例如性别取值为男和女，缺失值可以作为第3个取值对待。

对于决策树模型，缺失值还有一些其他处理方式。在C4.5算法中，缺失值的样本会同时进入到分裂后的各分支中，为了确保缺失值样本与非缺失值样本贡献相同，保证公平，同时进入各分支的缺失值样本会被加权，权重归一到1，保证所有的缺失值贡献之和为1，与非缺失值的贡献相同。为此，可以在初始化时为每一个样本赋予一个权重$w_i=1$，在以后的每一次分裂中，如果只分到一个分支的样本，权重不变；而分到多个分支的样本在每个分支的权重会被再次加权，变为$r_i * w_i$，$r_i$是非缺失样本在该分支的占比。以职业为例，学生、白领、蓝领、无业，未知就是缺失值，那么用多叉树进行分裂就会得到4个分支，假设职业不缺失的样本有100个，在每个分支分别为30、30、30、10，而职业缺失的样本有10个，这10个样本会同时进入这4个分支，且同一个样本在每个分支的权重会从原来的$w_i$更新为$r_i * w_i$，4个分支的比例系数$r_i$分别为0.3、0.3、0.3、0.1！而在计算信息增益的时候，只在非缺失值样本上计算，并且计算的时候概率不再是简单的数目之比，而且要考虑权重，最后将结果再乘上非缺失样本的比例。

决策树另外一种处理确实值的方式是只将缺失值放到一个分支，这个方法在著名的XGBoost中被采用。多个分支应该选择哪个分支呢？很简单，每个分支都试一下，选择信息增益(或者其他准则)最大的那一种方法即可！

## 决策树的其他分裂准则

### 信息增益率
信息增益准则在二叉树(即每个中间节点都只有两个子节点的树)中是一个很好的判断分裂好坏的准则，但是在多叉树中就不见得了。在多叉树中，取值很多的类别特征会天然地有很高的信息增益，这使得这类特征被选择进行分裂的概率比连续特征和只有两个取值的类别要高很多。为了解决这个问题，信息增益率准则被提出来了。信息增益率是在信息增益的基础上除以属性取值的"固有值"，属性a的固有值定义为

$$
IV(a) = - \sum_v \frac{|D_v|}{D} \log _ 2 \frac{|D_v|}{D}
$$

上式中D和$D_v$分别是待分裂样本总数和其中属性a=v的样本数目，也就是根据样本中属性a的取值形成一个概率分布$P(v) = \frac{|D_v|}{D}$，这个分布的熵就是属性a的固有值。因此，取值越多的属性IV通常越大，以此在一定程度上纠正信息增益对取值多的属性的选择偏好。利用IV，信息增益率可以写为

$$
Gain\\_ ratio(D, a) = \frac{Gain(D, a)}{IV(a)}
$$

其中$Gain(D, a)$就是前面所说的对集合D按照属性a划分的信息增益$\Delta I$。与信息增益相反，信息增益率会天然的偏好取值数目少的属性，在C4.5算法中，使用了一种启发式方法，先找出信息增益高于平均水平的属性，然后从中选出信息增益率最高的，相当于在信息增益和信息增益率两者中取了一个折中的方案。


### 基尼系数
决策树的基尼系数与经济学上度量贫富差距的基尼系数并不是一回事，这里的基尼系数是说对一个集合D，D中的样本属于不同类别，随机取两个样本他们的类别不同的概率。这个概率越低，说明集合的纯度越高，这个概率等于1说明集合中只有一个类别。假设集合D中每一类的比例为$p_k$，那么根据这种概率定义可得基尼系数为

$$
\begin{align}
Gini(D) &= \sum_k p_k \sum_{k' \ne k} p_{k'} \\\\
&=\sum_k p_k(1-p_k) \\\\
&=1 - \sum_k p_k^2
\end{align}
$$

按照某个分裂规则分裂后，每个集合都可以计算一个基尼系数$Gini(D_v)$，我们用平均基尼系数$\sum_v r_v Gini(D_v)$来衡量按照属性a的某个分裂规则分裂后的总体不纯度，其中$r_v$是每个集合的样本占总体的比例。这个平均基尼系数越低，说明分裂规则越好！因此，在决策树分裂算法中，用分裂后最小平均基尼系数选择最佳属性和分裂点即可，这就是CART（分类回归树）中用到的分裂准则，在著名的GBDT和XGBoost中也经常用到。

## 复现代码
### 过拟合现象代码
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import load_iris

iris = load_iris()

mask = np.random.randn(iris.data.shape[0]) < 0.8 # 20% for test
train_x, train_y = iris.data[mask], iris.target[mask]
test_x, test_y = iris.data[~mask], iris.target[~mask]

train_err = []
test_err = []
depths = range(1, 10)
for n in depths:
    clf = DecisionTreeClassifier(max_depth=n)
    clf.fit(train_x, train_y)
    train_err.append(1 - clf.score(train_x, train_y))
    test_err.append(1 - clf.score(test_x, test_y))

plt.plot(depths, train_err, '.-')
plt.plot(depths, test_err, '.-')

plt.legend(['train error', 'test error'])
plt.xlabel('depth')
plt.ylabel('error')

plt.show()
```

## 思考与实践
1. 举例说明在最终的决策树中，缺失值样本在所有叶子节点中的权重之和等于1！
2. 实现k折叠交叉验证，选择最佳决策树模型，最佳模型参数包括决策树深度、叶子节点上最少样本数目、分裂准则这三个。


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import load_iris
from sklearn.cross_validation import KFold


iris = load_iris()

mask = np.random.randn(iris.data.shape[0]) < 0.8 # 20% for test
train_x, train_y = iris.data[mask], iris.target[mask]
test_x, test_y = iris.data[~mask], iris.target[~mask]

"""实现K-Fold算法
可以利用sklearn的 sklearn.model_selection.KFold 实现集合划分，然后依次执行fit训练模型，
和 score 评估在评估集上的效果，将k次的效果平均得到在该组超参数下的平均效果。最后从多组超参数中
选择出最佳的超参数重新训练模型，在测试集上评估结果。
"""
best_err = 1
best_params = {'depth':0, 'min_samples_leaf': 1, 'criterion': 'entropy'}
k = 5
clf = DecisionTreeClassifier()

### YOUR CODE, 定义 kfold 对象

### END YOUR CODE

for depth in range(1,10):
    for min_samples_leaf in [1,3,10,30]:
        for criterion in ['entropy', 'gini']:            
            ### YOUR CODE，实现K-Fold算法

            ### END YOUR CODE

print('Best error: %.4f, depth=%d, min_samples_leaf=%d, criterion=%s'.format(best_err, best_params['depth'], best_params['min_samples_leaf'], best_params['criterion']))

### YOUR CODE, 在上述最佳超参数下重新在训练集上训练模型，并在测试集上评估效果

### END YOUR CODE
```
