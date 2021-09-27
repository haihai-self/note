# 背景

多任务学习通过在一个模型中同时学习多个不同的目标，如CTR和CVR，最近被越来越多的应用到线上的推荐系统中。MTL在一个模型中同时学习多个任务，并被证明可以通过任务之间的信息共享来提高学习效率, 当不同的学习任务之间较为相关时，多任务学习可以通过任务之间的信息共享，来提升学习的效率。但通常情况下，任务之间的相关性并不强，有时候甚至是有冲突的，此时应用多任务学习可能带来负迁移（negative transfer）现象，也就是说，相关性不强的任务之间的信息共享，会影响网络的表现。

此前已经有部分研究来减轻负迁移现象，如谷歌提出的MMoE模型。但通过实验发现，多任务学习中往往还存在seesaw phenomenon，也就是说，多任务学习相对于多个单任务学习的模型，往往能够提升一部分任务的效果，同时牺牲另外部分任务的效果。即使通过MMoE这种方式减轻负迁移现象，跷跷板现象仍然是广泛存在的。

# 多任务学习介绍

#### Single-Level MTL Models

Single-Level MTL Models主要包含以下几种形式：

![图片](../picture/21/640)

1）Hard Parameter Sharing：这也是最为常见的MTL模型，不同的任务底层的模块是共享的，然后共享层的输出分别输入到不同任务的独有模块中，得到各自的输出。当两个任务相关性较高时，用这种结构往往可以取得不错的效果，但任务相关性不高时，会存在负迁移现象，导致效果不理想。

2）Asymmetry Sharing（不对称共享）：可以看到，这种结构的MTL，不同任务的底层模块有各自对应的输出，但其中部分任务的输出会被其他任务所使用，而部分任务则使用自己独有的输出。哪部分任务使用其他任务的输出，则需要人为指定。

出现问题的原因: 主要是不同任务之间不是简单的相关或者不相关, 可能出现两个任务之间一部分样本相关但一部分样本不相关的情况, 这样的请款共享Asymmetry sharing就不能解决问题

3）Customized Sharing（自定义共享）：可以看到，这种结构的MTL，不同任务的底层模块不仅有各自独立的输出，还有共享的输出。2和3这两种结构同样是论文提出的，但不会过多的介绍。

4）MMoE：底层包含多个Expert，然后基于门控机制，不同任务会对不同Expert的输出进行过滤。

5）CGC：在Customized的基础上, 使用了门控结构

> MMoE与CGC的主要区别: 
>
> MMoE中, 不同的specific之间会存在相互影响的情况
>
> CGC中, 不同specific之间不存在相互影响, 不同知识的共享主要通过share expert

#### Multi-Level MTL Models

![图片](../picture/21/640-20210831142828676)

1）Cross-Stitch Network（“十字绣”网络）：出自论文《Cross-stitch Networks for Multi-task Learning》，上图中可能表示的不太清楚，可以参考下图：

<img src="../picture/21/640-20210831142848992" alt="图片" style="zoom:50%;" />

从上面的公式中可以看出，当aBA或者aAB值为0时，说明两者没有共享的特征，相反的，当两者的值越大，说明共享部分越大。

2）Sluice Network（水闸网络）：名字都比较有意思，哈哈。这个结构出自论文《Sluice networks: Learning what to share between loosely related tasks》，模型结构比较复杂，本文不做详述，感兴趣的同学可以阅读原文

3）ML-MMoE：这是MMoE的多级结构，expert之间通过门控结构相互连接

4）PLE：CGC的进阶版本，低层通过share expert共享信息, 解决了expert之间相互影响导致蹊跷板现象

# PROGRESSIVE LAYERED EXTRACTION介绍

#### seesaw phenomenon

我们先来看一下MTL中的seesaw phenomenon（跷跷板现象），论文主要基于腾讯视频推荐中的多任务学习为例进行介绍，其视频推荐架构如下图：

<img src="../picture/21/640-20210831143731265" alt="图片" style="zoom:50%;" />

这里主要关注VCR和VTR两个任务。VCR任务可理解为视频完成度，假设10min的视频，观看了5min，则VCR=0.5。这是回归问题，并以MSE作为评估指标。VTR表示此次观看是否是一次有效观看，即观看时长是否在给定的阈值之上，这是二分类问题（如果没有观看，样本Label为0），并以AUC为评估指标。

两个任务之间的关系比较复杂。首先，VTR的标签是播放动作和VCR的耦合结果，因为只有观看时间超过阈值的播放动作才被视为有效观看。其次，播放动作的分布更加复杂，在存在WIFI时，部分场景有自动播放机制，这些样本就有较高的平均播放概率，而没有自动播放且需要人为显式点击的场景下，视频的平均播放概率则较低。

上述所有结构的MTL在腾讯视频VCR和VTR两个任务上相对单任务模型的离线训练结果：

<img src="../picture/21/640-20210831143822430" alt="图片" style="zoom:50%;" />

可以看到，几乎所有的网络结构都是在一个任务上表现优于单任务模型，而在另一个任务上表现差于单任务模型，这就是所谓的跷跷板现象。MMoE尽管有了一定的改进，在VTR上取得了不错的收益，但在VCR上的收益接近于0。**MMoE模型存在以下两方面的缺点，首先，MMoE中所有的Expert是被所有任务所共享的，这可能无法捕捉到任务之间更复杂的关系，从而给部分任务带来一定的噪声；其次，不同的Expert之间也没有交互，联合优化的效果有所折扣。**

#### Customized Gate Control

Customized Gate Control（以下简称CGC）可以看作是PLE的简单版本，本文先对其进行介绍，其结构如下图所示：

<img src="../picture/21/640-20210831143932202" alt="图片" style="zoom:50%;" />

CGC可以看作是Customized Sharing和MMoE的结合版本。每个任务有共享的Expert和独有的Expert。对任务A来说，将Experts A里面的多个Expert的输出以及Experts Shared里面的多个Expert的输出，通过类似于MMoE的门控机制之后输入到任务A的上层网络中，计算公式如下：
$$
\begin{gathered}
g^{k}(x)=w^{k}(x) S^{k}(x) \\
w^{k}(x)=\operatorname{Softmax}\left(W_{g}^{k} x\right), \\
S^{k}(x)=\left[E_{(k, 1)}^{T}, E_{(k, 2)}^{T}, \ldots, E_{\left(k, m_{k}\right)}^{T}, E_{(s, 1)}^{T}, E_{(s, 2)}^{T}, \ldots, E_{\left(s, m_{s}\right)}^{T}\right]^{T}
\end{gathered}
$$
其中x表示下层输入, $w^k(x)$表示task k的权重矩阵函数, $W_{g}^{k} \in R^{\left(m_{k}+m_{s}\right) \times d}$表示参数矩阵, 其中$m_s, m_k$分别表示share task的数量以及task k中specific task的数量. $S^k(x)$表示选择矩阵, 最终task k的输出为
$$
y^{k}(x)=t^{k}\left(g^{k}(x)\right)
$$
其中$t^k$表示Tower k的网络

#### Progressive Layered Extraction

在CGC的基础上，Progressive Layered Extraction(以下简称PLE)考虑了不同的Expert之间的交互，可以看作是Customized Sharing和ML-MMOE的结合版本，其结构图如下：

<img src="../picture/21/640-20210831150352494" alt="图片" style="zoom:50%;" />

在下层模块，增加了多层Extraction Network，在每一层，共享Experts不断吸收各自独有的Experts之间的信息，而任务独有的Experts则从共享Experts中吸收有用的信息。

#### MTL训练优化

传统的MTL的损失是各任务损失的加权和：
$$
L\left(\theta_{1}, \ldots, \theta_{K}, \theta_{s}\right)=\sum_{k=1}^{K} \omega_{k} L_{k}\left(\theta_{k}, \theta_{s}\right)
$$
而在腾讯视频场景下，不同任务的样本空间是不一样的，比如计算视频的完成度，必须有视频点击行为才可以。不同任务的样本空间如下图所示：

<img src="../picture/21/640-20210831150530413" alt="图片" style="zoom:50%;" />

而本文则是在Loss上进行一定的优化，不同的任务仍使用其各自样本空间中的样本：
$$
L_{k}\left(\theta_{k}, \theta_{s}\right)=\frac{1}{\sum_{i} \delta_{k}^{i}} \sum_{i} \delta_{k}^{i} \operatorname{loss}_{k}\left(\hat{y}_{k}^{i}\left(\theta_{k}, \theta_{s}\right), y_{k}^{i}\right)
$$
其中$\delta_k^i$的取值为0或者1, 表示第i个样本是否属于第k个任务

其次是不同任务之间权重的优化。关于MTL的权重设置，最常见的是人工设置，这需要不断的尝试来探索最优的权重组合，另一种则是阿里提出的通过帕累托最优来计算优化不同任务的权重。本文也是人工设置权重的方式，不过在不同的训练轮次，权重会进行改变。在每一轮，权重的计算如下：
$$
\omega_{k}^{(t)}=\omega_{k, 0} \times \gamma_{k}^{t}
$$

## 实验结果

论文比较了在任务之间相关系数不同的情况下，Hard Parameter Sharing、MMoE和PLE的结果：

<img src="../picture/21/640-20210831150920556" alt="图片" style="zoom:50%;" />

可以看到，无论任务之间的相关程度如何，PLE均取得了最优的效果。

最后，论文对比了MMoE和PLE不同Expert的输出均值，来比较不同模型的Expert利用率（expert utilization）。为方便比较，将MMoE的Expert设置为3个，而PLE&CGC中，每个任务独有的Expert为1个，共享的为1个。这样不同模型都是有三个Expert。结果如下：

<img src="../picture/21/640-20210831150836427" alt="图片" style="zoom:50%;" />

可以看到，无论是MMoE还是ML-MMoE，不同任务在三个Expert上的权重都是接近的，这其实更接近于一种Hard Parameter Sharing的方式，但对于CGC&PLE来说，不同任务在共享Expert上的权重是有较大差异的，其针对不同的任务，能够有效利用共享Expert和独有Expert的信息，这也解释了为什么其能够达到比MMoE更好的训练结果。

