---
title: Adversarial Attack on Community Detection by Hiding Individuals
tags: 
    - Paper
    - Adversary
    - Community Detection
mathjax: true
mathjax_autoNumber: true
comment: true
key: Adversarial Attack on Community Detection by Hiding Individuals
---

[toc]

# Adversarial Attack on Community Detection by Hiding Individuals

Jia Li
The Chinese University of Hong Kong
lijia@se.cuhk.edu.hk
Honglei Zhang
Georgia Institute of Technology
zhanghonglei@gatech.edu
Zhichao Han
The Chinese University of Hong Kong
zchan@se.cuhk.edu.hk
Yu Rong
Tencent AI Lab
yu.rong@hotmail.com
Hong Cheng
The Chinese University of Hong Kong
hcheng@se.cuhk.edu.hk
Junzhou Huang
Tencent AI Lab
joehhuang@tencent.com



## Summary



可参考的博客：https://jjzhou012.github.io/blog/2020/07/24/Adversarial-Attack-on-Community-Detection-by-Hiding-Individuals.html



# Introduction

随着社区检测算法的迅速发展，人们意识到他们的隐私被过度挖掘了。在这种情况下，一些工作开始研究允许隐藏个人，社区或降低社区检测算法的整体性能的技术，这些技术主要基于启发式算法或遗传算法。

最近，深度图学习模型在许多图学习任务中都取得了卓越的性能。同时，一些研究注意到，深图模型很容易在诸如节点/图分类之类的任务上受到攻击。受这些发现的启发，在这项工作中，我们旨在解决以下问题：基于图学习的社区检测方法有多脆弱？我们可以通过不可察觉的图扰动来隐藏个人吗？一个很好的解决方案可以使许多实际应用受益，例如，个人隐私保护，欺诈逃避理解fraud escape understanding

社区检测算法正在用作其他目的（例如广告）的后端，这阻止了目标模型与个人之间的直接交互。为了解决这一挑战，我们设计了一种替代社区检测模型，该模型基于广泛使用的图神经网络（GNN）和流行的社区检测度量归一化切割normalized cut。我们攻击此替代社区检测模型，并验证攻击是否也可以转移到其他基于图学习的社区检测模型。

在对图数据进行对抗攻击的文献中，一个普遍观察到的困难是很难量化各种攻击的对抗成本adversarial costs。尽管人们可以在图像领域中检查不到察觉的扰动，但在图领域中却无法采用相同的策略。当前，大多数现有方法或者通过离散budget来限制允许更改的数量，或者预定义的分布（例如幂律分布[37]）间接解决此问题。然而，前者基于离散变化是有帮助的，但远远不够，而后者强调幂律分布实际上在实践中是罕见的。
$\Downarrow$
在这项工作中，我们提出了一个清晰易懂的图形目标，它从两个角度来测量扰动的程度：局部接近度 local proximity 和全局接近度global proximity。

我们面临的另一个挑战是选择适当的候选边/节点进行修改的巨大计算空间。开发可随图形大小缩放的解决方案并非易事。现有的解决方案依赖于启发式方法来绕过该问题，但是，该方法无法得出最优选择，尤其是对于属性图而言。
$\Downarrow$
在这项工作中，我们设计了一个新颖的图生成模型，该模型学习选择合适的候选者。通过这种方法，我们不仅可以生成适当的对抗图来攻击社区检测模型，而且可以将不易察觉的扰动要求带入学习过程。

贡献：

攻击cdattack、代理模型（社区检测模型）、如何判断两个图相似、图生成网络（可以提供对抗图给社区检测算法，且满足离散约束。

- 我们通过隐藏一组节点来研究基于图学习的社区检测模型的对抗性攻击。我们提出的解决方案CD-ATTACK可为所有competitor竞争对手提供出色的攻击性能。
- 我们提出了一种新的基于图学习的社区检测模型，该模型依赖于广泛使用的GNN和normalized cut归一化切割。它充当攻击的代理模型；此外，它还可用于解决一般的无监督，不重叠的社区检测问题
- 我们定义了一个可理解的图相关目标，从两个角度来衡量各种攻击的adversarial cost对抗成本：局部接近度和全局接近度。
- 我们设计了一种新颖的图生成神经网络，它不仅可以为社区检测算法生成对抗图，而且可以满足离散约束。
- 我们在四个实际数据集上评估CD-ATTACK。我们的方法在两个方面大大优于竞争方法。此外，我们验证了通过我们的方法生成的对抗图可以转移到其他两个基于图学习的社区检测模型。

# Problem Definition

一组节点$V=\{v_1,v_2,...,\}$表示现实世界的实体。（例如，共同作者网络中的作者，user-user交易网络中的用户）

使用NxN的邻接矩阵来表示节点之间的连接，为1代表有无向边。

本文专注于**==无向图==**，但是也可以被用在有向图上。使用$X=\{x_1,x_2,...,x_N\}$代表V中节点的属性值，$x_i$是一个d维向量。

社区检测问题旨在将图$G=(V,A,X)$划分为K个不相交的子图$G_i=(V_i,A_i,X_i)$。在我们的研究中，我们采用这种产生不重叠社区的方式（$V_i$的并集就是$V$，$V_i \cap V_j$为空且$i \neq j$）

---

在社区检测的背景下，我们对一组个体$C^+ \subseteq V $感兴趣，他们希望避免作为社区或社区一部分的检测。

给定一个社区检测算法f，**community detection adversarial attack problem 社区检测对抗攻击问题**被定义为学习一个攻击函数g，对$G=(V,A,X)$进行小扰动，从而得到$\hat{G}=(\hat{V},\hat{A},\hat{X})$，使得

![](https://cdn.jsdelivr.net/gh/duanyuchen55/ImageHosting/jekyll_pic/20210207151321.png)

L函数用来衡量有关目标$C^+$的社区检测结果的质量。
$Q(G,\hat{G}) < \varepsilon$用于确保察觉不到的扰动。

在本文中，==**专注于边的扰动**==，边的插入和删除。即，$\hat{G}=(\hat{V},\hat{A},\hat{X})=(V,\hat{A},X)$。

直观地讲，我们希望通过注入较小的扰动来最大程度地降低与子集C +相关的社区检测性能的下降。



# Methodology

## Framework

框架有两个主要模块：

1. 对抗性攻击者$g(\cdot)$，对原图执行不明显的扰动，隐藏一组个体；
2. 社区检测算法$f(\cdot)$，以无监督的方式将一个图分成几个子图。

这两个模块有很强的交互性，攻击模块需要检测模块的反馈，来判断是否达到攻击要求；检测模块依赖攻击模块来增强它的鲁棒性。

对于图结构数据的离散导致梯度难以计算，作者通过利用Actor-Critic算法中的策略梯度作为两个模块间的交互信号。

对于黑盒攻击设置，意味着没有具体的社区检测算法可以利用，作者通过设计具有高泛化性和鲁棒性的替代模型来实例化攻击对象。

---

后续参考https://jjzhou012.github.io/blog/2020/07/24/Adversarial-Attack-on-Community-Detection-by-Hiding-Individuals.html

## Method(s)

作者解决问题的方法/算法是什么？是否基于前人的方法？基于了哪些？



## Evaluation

作者如何评估自己的方法？实验的setup是什么样的？感兴趣实验数据和结果有哪些？有没有问题或者可以借鉴的地方？



## Conclusion

作者给出了哪些结论？哪些是strong conclusions, 哪些又是weak的conclusions（即作者并没有通过实验提供evidence，只在discussion中提到；或实验的数据并没有给出充分的evidence）?



## Notes

(optional) 不在以上列表中，但需要特别记录的笔记。



## References

(optional) 列出相关性高的文献，以便之后可以继续track下去。