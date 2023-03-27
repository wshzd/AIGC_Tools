# 清华Multi-Modal Diffusion:UniDiffuser_zh

> 该论文提出了一个为多模态设计的概率建模框架 UniDiffuser，除了单向的文生图，还能实现图生文、图文联合生成、无条件图文生成、图文改写等多种功能。

据悉 GPT-4 将于本周发布，多模态将成为其一大亮点。当前的大语言模型正在成为理解各种模态的通用接口，能够根据不同模态信息来给出回复文本，但大语言模型生成的内容也仅仅局限于文本。另一方面，当前的扩散模型 DALL・E 2、Imagen、Stable Diffusion 等在视觉创作上掀起一场革命，但这些模型仅仅支持文到图的单一跨模态功能，离通用式生成模型还有一定距离。而多模态大模型将能够打通各种模态能力，实现任意模态之间转化，被认为是通用式生成模型的未来发展方向。

清华大学计算机系朱军教授带领的 TSAIL 团队近期公开的一篇论文《One Transformer Fits All Distributions in Multi-Modal Diffusion at Scale》，率先发布了对多模态生成式模型的一些探索工作，实现了任意模态之间的相互转化。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagedl3cQ2Wib0Yy9LuArBzBLdtEic9eH3sWqQv68mOYEOYBFVvBepjlCsA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

论文链接：https://ml.cs.tsinghua.edu.cn/diffusion/unidiffuser.pdf

开源代码：https://github.com/thu-ml/unidiffuser

该论文提出了一个为多模态设计的概率建模框架 UniDiffuser，并采用该团队提出的基于 transformer 的网络架构 U-ViT，在开源的大规模图文数据集 LAION-5B 上训练了一个十亿参数量的模型，使得一个底层模型能够高质量地完成多种生成任务（图 1）。简单来讲，除了单向的文生图，还能实现图生文、图文联合生成、无条件图文生成、图文改写等多种功能，大幅提升文图内容的生产效率，也进一步提升了生成式模型的应用想象力。

该论文一作鲍凡目前博士在读，是此前 Analytic-DPM 的提出者，凭借在扩散模型方面的优秀工作荣获 ICLR 2022 的 outstanding paper award（目前唯一一篇大陆单位独立完成的获奖论文）。

此外，机器之心之前还报道过 TSAIL 团队提出的 [DPM-Solver 快速算法](http://mp.weixin.qq.com/s?__biz=MzA3MzI4MjgzMw==&mid=2650861050&idx=1&sn=09df47b3b1e26c290bf90ce567661e92&chksm=84e52984b392a0920ef1adc10d7efc4292325c7664ac0b6262cd7ea3f3ee051ed50313ffd4fb&scene=21#wechat_redirect)，目前仍是扩散模型最快的生成算法。多模态大模型正是该团队在深度概率模型的算法和原理方面上长期深入积累的一个集中展示。该工作的合作者包括人民大学高瓴人工智能学院的李崇轩、北京智源研究院的曹越等。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagzXj0jznwM7YpLIqkGnvd1MScRsHRN80KJS8kN6yicKCDftVtN6Bj25A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

值得注意的是，该项目的论文和代码均已开源。

**效果展示**

如下的图 8 展示了 UniDiffuser 在图文联合生成的效果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagFttCbw09luNmLnTqT9ZHBj7VVibzlV7BjJWI1BS9rwYWuP1ISicD9GMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如下的图 9 展示了 UniDiffuser 在文到图上的效果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagw2uml5o3Gg5GibrWNLiaMMGplhee3eW3xsQxzmhAnS5Iiaia0wYkNz0BRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如下的图 10 展示了 UniDiffuser 在图到文上的效果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiag1MPmD0xuCupF31wxRQ5E7EA82npicPpxESmrC0UkHiaK1pT613DJ8QDw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如下的图 11 展示了 UniDiffuser 在无条件图像生成上的效果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagE6u1YZ6U3fFbkrP4rLibiagwuxKVeMdk4QK6HtyPmlxzK5IgqrNic0iaQg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如下的图 12 展示了 UniDiffuser 在图像改写上的效果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiag7Ko3SVbed7ozvxwtbu7dlGXtYPx7mzib9983fhopzwD8WFlv5g6SDrQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如下的图 15 展示了 UniDiffuser 能够实现在图文两个模态之间的来回跳跃 ：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagLN7mj5yqntNT7mrV6iaALKde1SticTfpAOXrqOEsf7QUueicjqMiaQLEwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如下图 16 展示了 UniDiffuser 能对真实的两张图像进行插值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiaggmyGmPCVjDPIzfNicrKBqbibJ04K16e5Of9mSY2RIDmaArYPUQERTRdw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**方法概览**

研究团队将针对通用生成式模型的设计划分成了两个子问题：

- 概率建模框架：是否能寻找到一个概率建模框架，能同时建模出模态之间所有的分布，例如图文之间的边缘分布、条件分布、联合分布等？
- 网络架构：是否能设计出一个统一的网络架构，来支持各种不同模态的输入？

**概率建模框架**

针对概率建模框架，研究团队提出 UniDiffuser，一个基于扩散模型的概率建模框架。UniDiffuser 能够显示地建模多模态数据中包括边缘分布、条件分布、联合分布在内的所有分布。研究团队发现，关于不同分布的扩散模型学习都可以统一成一个视角：首先向两个模态的数据分别加入某种大小的噪声，然后再预测两个模态数据上的噪声。其中两个模态数据上的噪声大小决定了具体的分布。例如，将文本的噪声大小设置为 0，则对应了文生图的条件分布；将文本噪声大小设置为最大值，则对应了无条件图像生成的分布；将图文噪声大小设置为相同，则对应了图文的联合分布。根据该统一的视角，UniDiffuser 只需要将原始扩散模型的训练算法做少许的修改，便能同时学习上述的所有分布 — 如下图所示，UniDiffuser 同时向所有模态加噪而非单个模态，输入所有模态对应的噪声大小，以及预测所有模态上的噪声。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagiaeIHNRvnlmgefTO6GY2Llr8B48Og1Up4r0aufxciaaWnNmv2dbOOldg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

以双模态为例子，最终的训练目标函数如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiag1emvQk0FA9S2R5KInjLr3vGgagwGiagZzqftcWiaOUuYjiaErrZKFA1vQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagVjNNA4iaqOTGz3FsuGzHatcZsk49eIn0bOdnAl1gFwmDDa2YvBzagVw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)代表数据，![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagLc9Dyiaew3lGvOG6UNdL2lsJcOhnuOASG1FkhRCGcmmAAiakXj2ZVGYw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)代表加入到两个模态中的标准高斯噪声，![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiage9f7aMe2gkKL7erL5NNUjibOygXF66DSqicETNpyicmqPqU9OvSrwIW2Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)代表两个模态加入噪声的大小（即时间），两者独立的从 {1,2,…,T} 中采样，![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagfqRibsOvyXMeDJPNOibaaSQW1NhhpyByJGDca2RyqibNxwttEBMUyAhYg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)为噪声预测网络，同时预测两个模态上的噪声。

在训练后，通过向噪声预测网络设置两个模态合适的时间，UniDiffuser 能够实现无条件、条件以及联合生成。例如将文本的时间设置为 0，可以实现文到图生成；将文本的时间设置为最大值，可以实现无条件图像生成；将图文时间设置为相同值，可以实现图文联合生成。

下面罗列了 UniDiffuser 的训练和采样算法，可见这些算法相对原始的扩散模型均只做了微小的改动，易于实现。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagK0qJ96c4X3ADB9NQzLQzasT7JcWWTVdPhDTpBMmhB5htndYVJPFodA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

此外，由于 UniDiffuser 同时建模了条件分布和无条件分布，因此 UniDiffuser 天然地支持 classifier-free guidance。下面的图 3 展示了 UniDiffuser 的条件生成和联合生成在不同的 guidance scale 下的效果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagXFrAAiapDGBRicnn316ic3yKpFhjyY8QO29Iib1lhQotcaT7FvCMot5lyA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**网络架构**

针对网络架构，研究团队提出使用基于 transformer 的架构来参数化噪声预测网络。具体地，研究团队采用了最近提出的 U-ViT 架构。U-ViT 将所有的输入都视作 token，并在 transformer 块之间加入了 U 型连接。研究团队也采用了 Stable Diffusion 的策略，将不同模态的数据都转换到了隐空间再进行扩散模型的建模。值得注意的是，U-ViT 架构同样来自该研究团队，并且已被开源在 https://github.com/baofff/U-ViT。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagpj4yu8PktCnaV6ukzDq0KtG5W7HrXdzgNKbiayPR9UvuPTNTOkeGibSA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**实验结果**

UniDiffuser 首先和 Versatile Diffusion 进行了比较。Versatile Diffusion 是过去的一个基于多任务框架的多模态扩散模型。首先 UniDiffuser 和 Versatile Diffusion 进行了文到图上的效果比较。如下面的图 5 所示，在不同的 classifier-free guidance scale 下，UniDiffuser 在 CLIP Score 和 FID 指标上均要好于 Versatile Diffusion。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiaguEDVwvSSl7mibpv6l9zVlUlrpy1GwlTCyaFaye1mDjc33EDibvZZJLibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后 UniDiffuser 和 Versatile Diffusion 进行了图到文上的效果比较。如下面的图 6 所示，UniDiffuser 在图到文上有更好的 CLIP Score。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiagMnwLe4Cn8gN4RCORPmQy92SqzgTPEKevHmJ4DUZqxhPJ1AG1Us0BOg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

UniDiffuser 也和专用的文到图模型在 MS-COCO 上进行了 zero-shot FID 的比较。如下面的表 1 所示，UniDiffuser 可以和专用的文到图模型取得可比的效果。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicQprCK8NjAfWYMCVWsKqiageDiaL4f35hzgKLJhPfuHTiar4FMqEgMF1B6Rjic7ibt3BLJ8kI3vRTiaacg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)









