---
title: Adversarial Sets for Regularising Neural Link Predictors
tags: 
    - Paper
    - Adversary
    - Discriminator
mathjax: true
mathjax_autoNumber: true
comment: true
key: Adversarial Sets for Regularising Neural Link Predictors
---


[toc]

# Adversarial Sets for Regularising Neural Link Predictors

Pasquale Minervini1	Thomas Demeester2	Tim Rocktäschel3	Sebastian Riedel1
University College London, London, United Kingdom1
Ghent University - iMinds, Ghent, Belgium2
University of Oxford, Oxford, United Kingdom3





## Background / Problem Statement

研究的背景以及问题陈述：作者需要解决的问题是什么？

在对抗学习中，一系列模型通过追求竞争目标在一起学习，通常被定义为single data instances。
在关系学习和其他非独立同分布领域，目标通常也被定义在instances set上。

这里我们使用这样的一种假设为了得到一个不一致损失inconsistency loss（用于测量模型在对抗性生成的一组示例中，违反假设的程度） 

训练目标被定义为最小化最大值问题。adversary通过最大化inconsistency loss来找到最有攻击性的Adversarial examples，并且模型通过在Adversarial examples上共同最小号一个监督损失和inconsistency loss来进行训练。

这产生了第一种方法，该方法可以使用无函数Horn子句来规范化任何Neural link predictor，其复杂性与域大小无关。

我们证明了：对于几种链接预测模型，对手所面临的优化问题都有有效的闭合解。关于link predictor基准的实验表明，只要有适当的先验知识，我们的方法就可以在所有相关指标上显着改善神经链接预测器。



---

Adversarial training对抗训练是一种两个或多个模型通过追求相互竞争的目标共同学习的情境。这种competing goals被定义在single data instances。例如：GAN：生成器被训练来生成虚假数据（让辨别器将其识别为真实数据real），辨别器被训练来辨别real和fake数据。但是，对于诸如链接预测或知识库填充之类的关系任务，其中对象可以彼此交互，也可以根据多个实例定义此类目标。

另外一种，是在一个集合上的巴拉巴拉。。。Adversary的目标是找到导致预测不一致的输入，而预测器的目标将是恢复此类输入的一致性。

本文中，我们引入了对抗集正则化ASR，一种通过使用背景知识对神经链接预测模型进行正则化的通用的可拓展的方法。架构如下图：

![](https://cdn.jsdelivr.net/gh/duanyuchen55/ImageHosting/jekyll_pic/20210131125238.png)

==**在ASR中，我们先以无函数的First-orderLogic一阶逻辑（FOL）子句的形式为多个问题实例定义了一组约束。从这些字句中，我们可以得出inconsistency loss，用来衡量违反约束的程度。**==

架构由两个模型组成，一个是攻击者adversary，一个是鉴别器discriminatory，有两个competing goals：

- 攻击者：根据鉴别器(link prediction模型)，攻击者试图找到一个对抗性的输入表示集合，对于这些表示，约束是不成立的。这样的集合是通过最大化不一致性损失来找到的。
- 鉴别器：使用对抗性输入表示的inconsistency loss来regularising（规范）其训练过程。

提出的训练算法可以被看做是一个零和博弈问题：

1. 第一个玩家，link predictor，需要通过一系列real输入预测一个target graph，同时还要确保在生成的对抗输入集合上的全局一致性。
2. 另外一个玩家，adversary，需要生成对抗输入表示，来让link predictor无法构建consistent graphs。

link predictor模型通过共同最小化data loss和对抗输入集合inconsistency loss来训练，而攻击者通过改变输入集合来最大化inconsistency loss。

## Method(s)

作者解决问题的方法/算法是什么？是否基于前人的方法？基于了哪些？

Contribution：

1. 我们引入了一种新颖的方法来基于先前的关系假设（例如传递性）来规范化神经链接预测模型-这是第一项使用对抗性输入集进行此操作的工作；
2. 我们提出了一种用于解决潜在的极小极大问题的优化算法；
3. 我们推导了内部最大化问题的封闭式解决方案，该解决方案可以加快训练速度，提供对敌方目标的直观见解，并证明Demeester等人的方法是有效的。



ADVERSARIALSETS

即使训练数据可能与我们可以对图形做出的各种假设相一致，但在看不见的三元组的集合上，分类器的局部性质仍可能导致不一致。以IS-A为例，我们可能会看到（IS-A，CAT，FELINE。）和（IS-A，FELINE，ANIMAL）的高分，但是（IS-A，CAT，ANIMAL）的低分，违反了IS-A上位关系的传递性。

为了解决此问题，我们生成了对抗性输入集，并鼓励该模型修复与这些输入有关的不一致之处。更具体地说，我们找到了一个对抗实体embedding集合作为模型scoring layer	的输入，而不是一组实际实体对。这有两个核心好处。

1. 它使我们能够解决连续优化问题（在嵌入中），而不是组合优化问题（在实体上）。前者甚至可以具有封闭形式的解决方案。
2. 它迫使模型学习关系之间的一般关联，而不是通过编码器来了解有关特定事实和实体的知识。

为了清楚起见，我们现在考虑单个假设A，例如关系r的传递性。概括多个假设只需要为每个假设实例化一个对手和一个不一致损失。在本文中，我们使用Horn子句（FOL公式的子集）来表达我们的假设。例如，上位关系的可传递性可以表示为

![](https://cdn.jsdelivr.net/gh/duanyuchen55/ImageHosting/jekyll_pic/20210131214453.png)

其中蕴涵右侧的原子称为子句的开头，左侧原子的合点称为子句的主体，并且所有变量都被普遍量化。





论文后部分看不懂的方法太多了，暂时搁置。。。

