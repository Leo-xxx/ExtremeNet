## Anchor-free 目标检测之 ExtremeNet

路一直都在 [我爱计算机视觉](javascript:void(0);) *前天*

点击我爱计算机视觉标星，更快获取CVML新技术

------



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNia4hwiazNKjYicNzubwnJTo5fOxOTm4ZhHKdVtvg4cChcZ5R7AfYmRGJA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



本文转载自知乎用户“路一直都在”的OUCer&CVer专栏，原文地址：

https://zhuanlan.zhihu.com/p/71249684



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNvQhLkURyeTskzAWBaXuDmhztr5YF6VjpXNdXawS7ZmiarNEagdwC24A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





该文被CVPR 2019接收，作者来自德克萨斯州大学奥斯汀分校。



论文地址:

https://arxiv.org/abs/1901.08043

开源代码:

https://github.com/xingyizhou/ExtremeNet

## 1 Abstract

随着深度学习的发展，目标检测问题已经从一个由下到上的问题转变为一个由上到下的问题。（下文有详细介绍）当前最先进的目标检测算法首先枚举密集的目标可能存在的位置，然后对每一个位置分类判断是否存在目标，是前景还是背景。

在本文中，作者认为，从下到上的方法仍然能够取得很好的性能。利用关键点预测网络预测四个极值点（最顶部、最左侧、最底部、最右侧）和一个中心点共五个关键点。

如果五个关键点是几何对齐的，我们将它们分组到一个bounding box中，那么，目标检测问题就转化为一个纯的关键点估计问题，而不用进行分类或特征学习。实验表明，在COCO数据集上，AP有43.7%。

## 2 Introduction

利用从上到下的方法进行目标检测是当今的主流方法。常用的目标检测器将目标检测转变为矩形区域分类，通过裁剪，特征提取或者anchor-based方法，获得目标可能存在的区域位置。

这就是我们说的从上到下的方法，但是这种top-down方法并不是没有缺陷的，显而易见的是，绝大多数的目标都不是常规的矩形，如果我们强制用矩形表示目标，那么会包括除了目标本身外很多的背景像素，这些像素并不是我们需要的。

此外，这种方法需要列举大量候选框可能存在的位置，而不是真正对图像的语义信息进行理解，这种方法的计算成本是非常庞大的。而且，bounding box是远远不能和目标本身划等号的，在形状，姿态等方面有很大的差距。

在本文中，作者提出了ExtremeNet，一种从下到上的目标检测方法：通过预测每个对象类别的四个多峰热图（heatmaps），找到四个极值点（最顶部、最左侧、最底部、最右侧），此外，再利用一个heatmaps进行中心点预测，并将该点作为x维和y维两个边界框的平均值。

使用几何方法对极值点进行分类，如果四个极值点的几何中心在中心热图中的得分高于阈值，那么这四个极值点就会被归为一组。枚举所有可能的极值点组合，然后选择符合条件的，分为一组。

如下图所示：首先通过四个热点图选出极值点（下图四种颜色的点，每种颜色的点代表四类中一类的极值点）的候选，一个热图选出中心点的候选。然后对于极值点候选，每个类选择一个，进行组合，组合出来的四个点，求其几何中心，如果该几何中心在中心点预测的热图上有超过阈值的分数，那么这四个极值点就是一组的，得到一个bounding box。否则，拒绝，进行下一种组合。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNia6XHJlnzGPCFS32B7RGoYRl1RSeciaCyMdYTFljYfiaW6u2dsHiaSTDlA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



近两年来，有很多基于关键点预测的目标检测方法，比如CornerNet（在我的专栏中也有详细解读），通过预测左上角和右下角的角点，构成bounding box。

在角点分类的方法上，借鉴了人体姿态估计的方法，进行向量嵌入进行分组。与CornerNet相比，本文提出的方法有两大不同：关键点的定义和分组方式。

角点是bounding box的一种特殊形式，比如CornerNet，知道左上角和右下角的点后就形成了一个bounding box。再比如CenterNet，在CornerNet的基础上加了一个中间点，利用三个点形成一个bounding box。

但是这种bounding box的生成方式是有缺陷的，因为这种点通常在真正物体的外部，并没有很强的外观特征，甚至说，它本不应该属于物体，只是因为我们要用bounding box表示而已。但是本文提出的极值点是位于物体上的，因此在视觉上是可区分的，具有一致的局部外观特征。

举例来说，对于一个人的目标检测，最顶端的点通常是头部位置；对于一个车辆或者飞机的目标检测来说，最底部的点通常是轮子的位置。这种特性让ExtremeNet检测更简单和具有优势。

ExtremeNet是完全基于外观的，没有进行任何的隐式学习。在ICCV 2017中有这样一个工作：《Extreme clicking for efficient object annotation》，说的是关于边界框标注的问题。通常目标检测的边界框标注是用一个矩形圈定范围。

在注释ILSVRC时，获得一个高质量的标注框大约需要35s，在通过改变标注方式，将画框改为极值点标注后，时间能缩短到7s。此外，与边界框相比，极值点的标注能获得更细粒度的表示。

实验表明，

，这个成绩，可以傲视整个一阶段目标检测器了。

## 3 Related work:

在这个模块，我附上其他几篇基于关键点进行目标检测的论文解读，如果你想了解更多，可以点击阅读

RepPoints：用代表性点集进行目标检测

https://zhuanlan.zhihu.com/p/69957858

CornerNet:另辟蹊径的目标检测方法

https://zhuanlan.zhihu.com/p/63854126

CenterNet: Keypoint Triplets for Object Detection

https://zhuanlan.zhihu.com/p/66326413



## 4 Preliminaries（预备知识）

- Extreme and center points（极值和中心点）

计算机视觉，尤其是目标检测任务中，边界框标注是必不可少的一项准备工作，想一想常规的边界框标注我们是怎么做的？以我用LabelImg标注的体验，通常先点击一个边界框左上角的位置，然后拉到右下角的位置，生成一个矩形将目标覆盖。

如下图所示，我们通过这种方法选出来的左上角和右下角本身不属于目标本身，而且要想得到一个高质量的边界框，需要的时间平均为34.5s



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNZUpWKL4Bia3mQFLm6e3qQptw8WW6uljhgYJXt8IfXPJbvaibFwtN7ERg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



于是，新的标注方式被提出，标注四个极值点即最顶，最底，最左，最右。若四个点坐标如下所示：



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNpKol9O2XgUrVh0bcy93bYmbhNgb65lmftqAYTYYibxdibGk2aneG9FCA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



那么我们真正需要的，其实是下列这四个值：最左，最右坐标的x方向上的值；最顶最低坐标的y方向上的值。



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNSIXeSVibnuXOzyLibObbxZm2fdyu2hdK7DNtSJBETQa0cvBhwKpfYgsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



那么相对应的中心点坐标就是：



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNuYicb0GHk4tF9icoJjibnVPRDM1SGNSqyfDnQFmf1jdMBU3EFoDpWIh3g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



这种标注方式平均花费时间是7.2秒，而标注效果却毫不逊色。于是本文就利用了这种标注方式。

- Hourglass Network（沙漏网络）

沙漏网路最初是在ECCV 2016中的工作，通过重复自底向上和自顶向下并联合中间结果的监督用在人体姿态估计中可以很好地利用身体不同部位的空间关系，一个沙漏网络的结构可以通过下列图示表示：基本思想是先进行降采样，在降采样的过程中，C1a-C4a是对应层的副本，从C7开始上采样，与C1a-C4a对应元素相加，得到C1b-C4b，最后再经过两个1x1的卷积进行处理，得到最终输出。

沙漏模块首先通过一系列卷积层和最大池化层对输入特性进行下采样。然后通过一系列的上采样和卷积层将特征上采样回原来的分辨率。由于细节在最大池化层中丢失，因此添加了跳过层用来将细节带回到上采样的特征



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNxOaNnZxJ1ZxJcfdO0m7YxSaRvppARZicmicl3vWd8ObMLyZ6NCqnlnUA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 5 ExtremeNet for Object detection

ExtremeNet利用沙漏网络为每个类别的五个关键点（四个极值点和一个中心点）进行检测，在损失函数设计和偏移预测上，遵循和CornerNet相同的策略。偏移预测是类别无关但是极值点相关的。

由于中心点是通过四个极值点的几何计算得到的，所以对于中心点并不进行偏移预测。

因此，整个网络的输出是这样的：heatmaps(热点图) 5xC维，其中C为要分类的数目，在COCO数据集上C的值是80；4x2 偏移图（4个极值点，2个方向）。下图是ExtremeNet的图示：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsN5HOGLKZA4lYjdEa5XXItf8Gf4YG6icR3NCpmvu8iasJPz0KicCQAojMBA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



一旦提取出极值点后，利用纯几何的形式将极值点分组。

- Center Grouping（极值点分组）

显而易见，极值点位于目标的不同边上，这就使将极值点分组变得复杂。在CornerNet上，借鉴了人体姿态估计中的向量嵌入方法将角点进行分组，但是作者认为这种方法可能缺少足够的全局视图信息。于是本文采取了截然不同的策略进行分组。也就是Center Grouping。

整个模块的输入是每一个类别的5个热图即一个中心点热图和4个极值点热图，可分别用下列公式表示：



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsN3cV0paYUwdibLLGdJKKJH443hVsa4SZk7ic5iaJgWDCic8LYLez3SEqYzQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)1个中心热点图

![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNXvkMURlnrFGDDSpyWSP1VlYWodqicBCNQgAfu1CyeEMW0Arqc1y53Bw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)4个极值点热点图



给定一个热点图，首先通过检测峰值（peaks）将所有关键点提取出来。峰值的定义是热点图中所有得分超过阈值*τp* 的点。提取方法是设置一个3x3的窗口，找到窗口中的局部最大值。这个过程也就是ExtractPeak，提取极值点阶段。

第一步的提取关键点阶段，从上文定义的四张热点图中提取出四个极值点，*t*, *b*, *r*, *l ，*通过下列公式计算出中心点：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNicvdC3cqEO9LsvUJOXYSBjJjUEMicFnohqN6ibSncC8iayjzicQicmMkibylg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



如果这个点c在中心热点图中有高响应，就认为这四个极值点是一个有效的检测，高响应的评判标准是：



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNuMJyfQWiacRFVwicaxUeSTF9gmEibsgCrj2CuU91GKrdSjs80tfH8Hj9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



这只是一组极值点的检测方法，下面要做的就是推广开来，枚举四类极值点所有可能的组合，重复上述检测方法，算法流程如下所示：

输入：某类的4个极值点热点图和1个中心点热点图

输出：bounding box的得分

主要流程概述：提取出的关键点，根据顶部点，底部点，左端点，右端点标准分为四个点集，从四个点集中各抽取一个点，共四个点，得到四个极值，计算几何中心坐标，然后找到该中心坐标在中心点热点图中的得分，如果高于阈值，那么这四个极值点组成的bounding box是有效地，返回一个最终得分，得分的计算方式是五个点得分的平均值。

*本文中， *τp* = 0*.*1，*τc* = 0*.*1



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNtqmpkVOjmdk9eWico9AZia97DNEl2Tql43E8mjJKaNg4MARxWwCWeLLw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Center Grouping 算法



- Ghost box suppression（Ghost box抑制）

先解释一下什么是Ghost box。仔细想想Center grouping算法，可能会出现这样一种情况,中心分组可以对三个相同大小的等距共线目标进行高置信false-positive检测。这时位于中间位置的对象有两个选择，一种是选择正确的小框，另一种预测一个更大的框，里面有它相邻物体的极值点。

把这种false-positive的形式称为ghost boxes。为了消除这种错误，利用nms进行抑制，nms原先是用在消除重叠框上的，在这里同样有效。ghost box包含许多其他小的检测对象，我们采用soft nms解决这个问题。若某个box所包含的所有框的分数和大于其自身的3倍，则最终分数除以2，这可以看做是一种惩罚，惩罚的对象是ghost box。

- Edge aggregation（边缘聚合）

有时候，极值点并不是唯一的，举例来说，一辆汽车，可能顶部这一条类似于水平的线上，每一个位置都有可能成为最顶部的极值点。

也就是说，若极点来自物体的垂直或水平边，则沿边缘任何一点都可被认为是极点。因此，我们的网络沿着对象的任何对齐边缘产生弱响应，而不是单一的强峰值响应。这种弱响应有两个不足：

1. 弱响应点的得分可能会小于阈值，那么极值点就会被错过
2. 即使检测到了这些极值点，它的得分将低于具有强峰值响应的稍微旋转的对象。

为了解决这个问题，提出了edge aggregation。对每一个取局部最大值得到的极值点，若是左边和右边的极值点，那么选择竖直方向进行聚合；同理，若是顶部，底部的极值点，则选择从水平方向进行分数聚合。聚合的方法为：沿着聚合方向，聚合那些单调递减的score，并在达到局部最小值的时候停止聚合。假定m为极值点，定义一个包含m点的水平线在heatmaps上的分数集：



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNWRGfTbdQ3ItIic2ShcN8ZpIbUIuc9wnApgPibuNYnicIBHtg2picGypiaUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



由于m是极值点，极值点的选择机制是局部最大原则，所以，需要在极值点m的左边和右边找到两个局部最小值，记为i0和i1，应该满足：



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNQzgfdzjI8GhRg5jf9OjUqwSX0gG01dD83u3UYGjdV3A02EsfXVQMhA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNlmLSmRlo0waqrMZwsPwTImZeEQh3BQmRpMtBMXcvRuJKHRbiae861bQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



那么，此时原始极值点m的更新为：



![img](https://mmbiz.qpic.cn/mmbiz_png/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNLNvicO1ghsiaS93Jl85Hibkib7WaSqqo0asdZsXs5y6KqGb3KpDrIUZCjA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



其中，*λaggr* 是聚合权重，在本文中，设置为0.1。

下图是经过边缘聚合后的效果图：在原始heatmaps中，位于边缘的极值点，反馈回一个很弱的响应值，但是经过边缘聚合处理后，大大增强了中间像素的响应值。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNg0Gw1knvaXxVKfKzYEXgtzzuAJjRiaDicQ6O3uXbaHq3vO1NMFMTEKZw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## Experiments



![img](https://mmbiz.qpic.cn/mmbiz_jpg/BJbRvwibeSTsQxwvxOG8oVYllUjZ3iatsNaRXRY1rLD5wkSiaGxrL98tv0YqFKiaOt6W3zJSkWHqynaHCGwmaEuo6g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## Conclusion

本文提出了一种基于自底向上极值点估计的目标检测框架---ExtremeNet。框架提取了四个极值点，并以纯粹的几何方式将它们分组。实现了anchor-free且性能提升。

*原文中分割部分有详细介绍，由于目前专注于目标检测，故将大部分分割内容略去，请谅解

*如果您想获取最新计算机视觉/多任务的论文解读，欢迎关注作者的知乎专栏~

------



**目标检测交流群**



关注最新最前沿的目标检测技术，欢迎加入52CV检测交流群，扫码添加CV君拉你入群（如已为CV君好友，请直接私信，**不必重复添加**），

**（请务必注明:检测）：**

**![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)**

喜欢在QQ交流的童鞋可以加52CV官方QQ群：702781905。

（不会时时在线，如果没能及时通过还请见谅）

------

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

长按关注我爱计算机视觉







![img](https://mp.weixin.qq.com/mp/qrcode?scene=10000004&size=102&__biz=MzIwMTE1NjQxMQ==&mid=2247487549&idx=1&sn=babb359387cd7aa5d5acb96c3fe8bd03&send_time=)

微信扫一扫
关注该公众号