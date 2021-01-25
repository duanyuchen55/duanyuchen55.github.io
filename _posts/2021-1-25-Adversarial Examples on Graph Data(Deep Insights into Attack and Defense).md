---
title: Adversarial Examples on Graph Data(Deep Insights into Attack and Defense)
tags: 
    - paper
    - GCN
    - Graph
    - Adversarial
mathjax: true
mathjax_autoNumber: true
comment: true
key: Adversarial Examples on Graph Data(Deep Insights into Attack and Defense)
---

# Adversarial Examples on Graph Data: Deep Insights into Attack and Defense

Huijun Wu1,2, Chen Wang2, Yuriy Tyshetskiy2, Andrew Docherty2, Kai Lu3, Liming Zhu1,2
1University of New South Wales, Australia
2Data61, CSIRO
3National University of Defense Technology, China
{first, second}@data61.csiro.au, kailu@nudt.edu.cn


## Summary

1. 攻击：引入积分梯度，积分梯度被用作度量来测量在图G中扰动特定特征或边的优先级。对于具有高扰动优先级的特征或边，只需将其翻转为不同的二进制值即可对其进行扰动。
2. 防御：在训练之前对给定的图进行预处理：检查图的邻接矩阵并检查边。选择连接具有低相似性分数的节点的所有边作为要移除的候选。



## Research Objective(s)

​		在本文中，我们提出了攻击和防御两种技术。对于攻击，我们证明了通过引入积分梯度可以很容易地解决离散性问题，它可以准确地反映扰动某些特征或边缘的效果，同时仍然受益于并行计算。在防御方面，我们观察到目标攻击的恶意操纵图在统计上与正常图不同。基于这一观察结果，我们提出了一种检测图形并恢复潜在敌方扰动的防御方法。

​		**现有的攻击方法中：添加 连接着具有不同特征的节点的边 在所有攻击方法中都起着关键作用。**

​		**本文证明了对图的邻接矩阵进行简单的预处理就能识别出被操纵的边。对于具有bag of word(BOW)特征的节点，Jaccard索引在度量连接节点之间的相似性时是有效的。通过去除连接非常不同节点的边，我们能够在不降低GCN模型的准确性的情况下防御有针对性的对抗性攻击。**



## Problem Statement

​        图的深度学习模型，如图卷积网络(GCN)，在处理图数据的任务上取得了显著的性能。与其他类型的深度模型类似，图深度学习模型经常遭受敌意攻击。然而，与非图数据相比，图数据的离散特征、图的连通性以及对潜伏扰动的不同定义给图数据的对抗性攻防带来了独特的挑战和机遇。

​		将在非图数据上的攻击方法，用在GCN上的挑战是离散输入问题。具体地说，图形节点的特征通常是离散的。边，特别是未加权图中的边，也是离散的。为了解决这个问题，基于贪婪的方法（下面的两篇文章[1]和[2]）来攻击基于图的深度学习系统。一种反复扰乱特征或图形结构的贪婪方法。图结构和特征统计在贪婪攻击期间被保留。在本文中，我们证明了尽管存在离散输入问题，但积分梯度仍然可以精确地逼近梯度。



## Method(s)


​		**攻击：现有的攻击方法中，添加 连接着具有不同特征的节点的边 在所有攻击方法中都起着关键作用。**

​		**本文证明了对图的邻接矩阵进行简单的预处理就能识别出被操纵的边。对于具有bag of word(BOW)特征的节点，Jaccard索引在度量连接节点之间的相似性时是有效的。**

​		**防御：通过去除连接非常不同节点的边，我们能够在不降低GCN模型的准确性的情况下防御有针对性的对抗性攻击。**

### Integrated Gradients Guided Attack：

图表中的节点特征通常是词袋类型的特征，可以是1，也可以是0。图中未加权的边也经常用来表示特定关系的存在，因此邻接矩阵中只有1或0。当攻击该模型时，敌意扰动被限制为将1改为0，或者反之亦然。在图模型中应用普通的FGSM和JSMA的主要问题是梯度不准确。

给定目标节点t，对于FGSM攻击，$\nabla J_{W^{(1)},W^{(2)}}(t)=\frac{\sigma J_{W^{(1)},W^{(2)}}(t)}{\sigma X}$ ∂X测量所有节点对损失函数值的特征重要性。这里，X是特征矩阵，它的每一行描述了图中一个节点的特征。对于节点n的特定特征i，$\nabla J_{W^{(1)},W^{(2)}}$的较大值表示扰动特征i为1，有助于使目标节点分类错误。但是，遵循此梯度可能没有用，原因有两个：

1. 首先，特征值可能已经是1，因此我们不能再对其进行扰动；
2. 其次，即使特征值是0，由于GCN模型可能无法学习该特征值在0和1之间的局部线性函数，所以这种扰动的结果是不可预测的。

换句话说，原始的梯度存在局部梯度问题。（以一个简单的RELU网络f(X)=relu(X)为例，当x从0增加到1时，函数值也增加了1，但在x=0时计算梯度为0，不能准确地捕捉到模型的行为。）

​		积分梯度定义如下：考虑一条从x0到输入x的直线路径，积分梯度是通过累加路径上所有点的所有梯度来获得的。

​		形式上，x的第i个特征，其积分梯度Integrate Gradient（IG）如下：
$$
IG_i(F(X))::=(x_i-\acute{x_i}) \times \int_{\alpha=0}^{1} \frac{\partial F(x'+\alpha x(x-x'))}{\partial x_i}d\alpha
$$
在给定邻接矩阵A、特征矩阵X和目标节点t的情况下，我们计算函数$F_{W^{(1)},W^{(2)}}(A,X,t)$，其中I是攻击的输入。I=A表示边缘攻击，而I=X表示特征攻击。当F为GCN模型的损失函数时，我们将这种攻击技术称为积分梯度类FGSM攻击，即IG-FGSM。

- 对于有目标的的IG-JSMA或IG-FGSM攻击，优化目标是最大化F的值。因此，对于值为1的特征或边，我们选择IG得分最低的特征/边，并将其扰动为0。
- 非目标IG-JSMA攻击旨在最小化获胜类的预测得分，以便我们尝试将IG得分高的输入维度增加到0。

为了删除边：我们要对全0矩阵逐渐增加边，来达到目前的状态，所以要把A或者X设置为全0矩阵；
为了添加边：我们要对全1矩阵逐渐删除边，来达到目前的状态，所以要把A或者X设置为全1矩阵。

![image-20210125190927497](C:\Users\duanyanwen\AppData\Roaming\Typora\typora-user-images\image-20210125190927497.png)

算法1显示了非目标IG-JSMA攻击的伪代码。我们计算获胜类别c的预测分数的积分梯度，即A和X的条目。
然后，积分梯度被用作度量来测量在图G中扰动特定特征或边的优先级。注意，边和特征值被考虑，并且仅计算可能扰动的分数(见等式(7))。（例如，我们只计算在以前不存在边的情况下添加边的重要性。）因此，对于具有高扰动优先级的特征或边，我们只需将其翻转为不同的二进制值即可对其进行扰动。

![image-20210125191012073](C:\Users\duanyanwen\AppData\Roaming\Typora\typora-user-images\image-20210125191012073.png)



### Defense for Adversarial Attack：

由于现有的针对GCN的attack都是有效的，因为attacked graph被直接用于训练新模型。基于此，一种可行的defense方法是使得邻接矩阵变得可训练。

按照nettack的方法，初始化adversarial graph，然后直接训练GCN模型。通过如此简单的防御方法，攻击后目标节点被正确分类的可信度高达0.912。

上述防御有效的原因：

1. 对边进行扰动比修改特征更有效。攻击方法倾向于添加边而不是删除边；
2. 邻居较多的节点比邻居较少的节点更难攻击。这也与[Zügner等人，2018年]中的观察结果一致，即度越高的节点在干净图和被攻击图中的分类精度都更高。
3. 攻击倾向于将目标节点连接到具有不同特征和标签的节点。我们发现这是最有效的攻击方式。

使用 CORA-ML数据集来验证，由于CORA-ML数据集的特征是bag of word，所以我们使用Jaccard相似度来衡量特征之间的相似性。
$$
J_{u,v}=\frac{M_{11}}{M_{01}+M_{10}+M_{11}}
$$
$M_{01}$是feature number，其中特征值在节点u中为0，而在节点v中为1。其他类似。

下图中可以看到，Adversarial Attack显著增加了与目标节点相似度得分较低的邻居节点的数量。

![image-20210125114527890](C:\Users\duanyanwen\AppData\Roaming\Typora\typora-user-images\image-20210125114527890.png)

图神经网络本质上是根据图形结构聚合特征。对于目标节点，恶意创建的图试图将具有不同特征和标签的节点连接起来，以污染目标节点的表示，从而使目标节点与其正确类中的节点不那么相似。

相应地，在删除边的同时，攻击倾向于删除连接与目标节点有许多相似之处的节点的边。边缘攻击更有效，因为添加或移除一条边会影响聚合过程中的所有特征维度。相反，修改一个特征只影响特征向量中的一个维度，并且这种扰动很容易被高度节点的其他邻居掩盖。

基于这些观察结果，我们提出了另一个**假设，即上述防御方法之所以有效，是因为模型为连接目标节点的边赋予了较低的权重，这些边连接到与目标节点特征相似度较低的节点。**为了验证这一点，我们绘制了从目标节点开始的边的末端节点的学习权重和Jaccard相似性得分(参见下图)。请注意，对于我们选择的目标节点，目标节点的每个邻居与其自身之间的Jaccard相似性得分在干净的图中大于0。相似度得分为零的边都是由攻击添加的。正如预期的那样，该模型对大部分相似度分数较低的边学习到了低权重。

![image-20210125114912808](C:\Users\duanyanwen\AppData\Roaming\Typora\typora-user-images\image-20210125114912808.png)

**为了使defense更有效，我们甚至不需要使用可学习的边权重作为defense。边权值的学习不可避免地会给模型引入额外的参数，这可能会影响模型的可扩展性和准确性。**基于以下几点，一种简单的方法可能同样有效：

1. 普通节点通常不会连接到许多与其没有相似之处的节点；
2. 学习过程基本上是将较低的权重分配给连接两个不同节点的边。

==**我们在训练之前对给定的图进行预处理：检查图的邻接矩阵并检查边。选择连接具有低相似性分数(例如，=0)的节点的所有边作为要移除的候选。**==（虽然clean graph也可能有少量这样的边，但我们发现删除这些边对目标节点的预测几乎没有什么坏处。相反，在某些情况下，删除这些边缘可能会改善预测。这是很直观的，因为聚合来自与目标截然不同的节点的特征通常会过度平滑节点表示。）

简化反而可能会导致性能提升，例如这些工作：

![image-20210125115222154](C:\Users\duanyanwen\AppData\Roaming\Typora\typora-user-images\image-20210125115222154.png)

## Evaluation

作者如何评估自己的方法，实验的setup是什么样的，有没有问题或者可以借鉴的地方。

我们使用了广泛使用的CORA-ML、CITESEER和Polblog数据集。

我们将每个图分为已标记节点(20%)和未标记节点(80%)。在标记的节点中，一半用于训练，另一半用于验证。对于Polblog数据集，由于没有特征属性，我们将属性矩阵设置为单位矩阵。



## Conclusion

图形神经网络(GNN)显著提高了对多种类型图形数据的分析性能。然而，与其他类型数据中的深度神经网络一样，GNN也存在健壮性问题。本文对图卷积网络(GCN)中的鲁棒性问题进行了深入研究。我们提出了一种综合的基于梯度的攻击方法，在攻击性能上优于现有的迭代和基于梯度的攻击方法。我们还分析了针对GCN的攻击，发现健壮性问题根源于GCN中的本地聚集。为了提高GCN模型的稳健性，我们给出了一种有效的防御方法。我们在基准数据上验证了我们方法的有效性和高效性。



## Notes




## References

[1]: [Wang et al., 2018] Xiaoyun Wang, Joe Eaton, Cho-Jui Hsieh, and Felix Wu. 
Attack graph convolutional networks by adding fake nodes. 
arXiv preprint arXiv:1810.10751,2018



[2]: [Zügner et al., 2018] Daniel Zügner, Amir Akbarnejad, and Stephan Günnemann.
Adversarial attacks on neural networks for graph data.
In Proceedings of the 24th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining, pages 2847–2856. ACM, 2018.

