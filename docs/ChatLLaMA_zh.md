# ChatLLaMA_zh

上篇文章《[Meta开放小模型LLaMA，性能超过GPT-3](ChatLLaMA_zh.md)》分享了LLaMA论文中在bechmark上的评估效果，例如，LLaMA-13B 在大多数基准上都优于 GPT-3(175B)，而模型大小却小了 10 倍以上，LLaMA-65B 与最好的模型 Chinchilla70B 和 PaLM-540B 性能相当。参数量的减少对于普通研究者和商业机构来说都是好事，LLaMA 在其他任务上表现如何？来自一位名叫 @Enryu 的 Medium 作者在“解释笑话”、“零样本分类”和“代码生成”三个任务与ChatGPT进行了对比评估，相关博客文章为《Mini-post: first look at LLaMA》。

**解释笑话**

这是谷歌原始 PaLM 论文中展示的一个用例：给出一个笑话，让模型来解释它为什么好笑。该任务需要将世界知识和一些基本逻辑相结合。PaLM 之前的所有模型都无法做到这一点。作者从 PaLM 论文中提取了一些示例，比较了 LLaMA-7B、LLaMA-13B、LLaMA-33B 与 ChatGPT 的表现。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicHsGdj258fyg3aujOh9lMUnIGDVflgib8uDljkrpGenB6NYx1CuCZYV001gPBbSSicjSzZXDBprPicw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到，结果很糟糕。这些模型 get 到了一些笑点，但无法真正理解，它们只是随机生成一些相关的文本流。ChatGPT 虽与 LLaMA-33B 一样表现很差（其他几个模型更差），但它遵循了不一样的策略：生成了一大堆文本，希望自己的回答至少有一部分是正确的（但大部分显然不是），是不是很像大家考试时应对问答题的策略？

不过，ChatGPT 起码 get 到了关于 Schmidthuber 的笑话。但总的来说，这些模型在零样本笑话解释任务上的效果与 PaLM 相差甚远（除非 PaLM 的示例是精心挑选）。

**零样本分类**

作者考虑的第二项任务更具挑战性 —— 标题党（clickbait）分类。由于连人类也无法就什么是标题党达成一致，作者在 prompt 中为这些模型提供了一些示例（因此实际上是小样本而非零样本）。如下为 LLaMa 的 prompt：

```
I will tell whether the following news titles are clickbait:1) The WORST care homes in England: Interactive map reveals the lowest-rated 2,530 residences - so is there one near you?Clickbait: yes2) Netflix's top 10 most-watched movies of all timeClickbait: yes3) Peering Through the Fog of InflationClickbait: no4) You’ll never believe which TV cook this cheeky chap grew up to beClickbait: yes
```

下图为 LLaMA-7B、LLaMA-13B、LLaMA-33B 与 ChatGPT 的更多示例结果。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicHsGdj258fyg3aujOh9lMUqXyfu7YW71wPNibR1RyfDUSrSibJHriat7jBJZ5exd7iaV5B8LrpOvSvibQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

很明显，赢家为 LLaMA-33B，它是唯一一个能够遵循所有请求格式（yes/no）的模型，并且预测合理。ChatGPT 也还可以，但有些预测不太合理，格式也有错误。较小的模型（7B/13B）不适用于该任务。

**代码生成**

虽然 LLM 擅长人文学科，但在 STEM 学科上表现糟糕。LLaMA 虽然有基准测试结果，但作者在代码生成领域尝试了一些特别的东西，即将人类语言零样本地转换为 SQL 查询。这并不是很实用，在现实生活中直接编写查询会更有效率。这里只作为代码生成任务的一个示例。

在 prompt 中，作者提供表模式（table schema）以及想要实现的目标，要求模型给出 SQL 查询。如下为一些随机示例，老实说，ChatGPT 看起来效果更好。

```
-- We have a table `events` with columns `event_id`, `timestamp`, `user_id`, `rpc_method`.-- We want to count how many users issued request "Login" over the last week.-- SQL query for this is:

-- 7bSELECT count (*)FROM eventsWHERE user_id IN (SELECT user_id FROM events WHERE timestamp >= NOW () - INTERVAL 7 DAY) AND rpc_method = 'Login';

-- 13bSELECT COUNT (*) FROM events WHERE rpc_method = "Login" AND UNIX_TIMESTAMP (timestamp) >= UNIX_TIMESTAMP (CURRENT_DATE - INTERVAL 7 DAY)

-- 33bSELECT user_id, COUNT (DISTINCT user_id) AS totalFROM eventsWHERE timestamp BETWEEN TIMESTAMP '2013-08-14 00:00:00' AND TIMESTAMP '2013-08-21 00:00:00' AND rpc_method = 'Login'GROUP BY user_id

-- ChatGPTSELECT COUNT (DISTINCT user_id)FROM eventsWHERE rpc_method = 'Login'AND timestamp >= DATE_SUB (NOW (), INTERVAL 1 WEEK);
```

从测试结果来看，LLaMA 在一些任务上表现还不错，但在另一些任务上和 ChatGPT 还有一些差距。如果能像 ChatGPT 一样加入一些「训练秘籍」，效果会不会大幅提升？

**结论：**

从上述三个任务来看，作者认为LLaMA模型离SOTA不太远，如果像ChatGPT一样使用RLHF，效果会怎样呢？

**Nebuly AI 开源 ChatLLaMA**

可喜的是，在 LLaMA 发布三天后，初创公司 Nebuly AI 开源了 RLHF 版 LLaMA（ChatLLaMA）的训练方法。它的训练过程类似 ChatGPT，该项目允许基于预训练的 LLaMA 模型构建 ChatGPT 形式的服务。项目上线刚刚 2 天，狂揽 5.6K 星。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicHsGdj258fyg3aujOh9lMUv9XDtprNjG4Q7zvYcsgZAEb4qeKc3QEVVzLpLhCtp8N5eG1rnUaAWA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

项目地址：https://github.com/nebuly-ai/nebullvm/tree/main/apps/accelerate/chatllama

ChatLLaMA 训练过程算法实现主打比 ChatGPT 训练更快、更便宜，我们可以从以下四点得到验证：

- ChatLLaMA 是一个完整的开源实现，允许用户基于预训练的 LLaMA 模型构建 ChatGPT 风格的服务；
- 与 ChatGPT 相比，LLaMA 架构更小，但训练过程和单 GPU 推理速度更快，成本更低；
- ChatLLaMA 内置了对 DeepSpeed ZERO 的支持，以加速微调过程；
- 该库还支持所有的 LLaMA 模型架构（7B、13B、33B、65B），因此用户可以根据训练时间和推理性能偏好对模型进行微调。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWicHsGdj258fyg3aujOh9lMU8S8fvBic1YJbZM2icC0rC5NjuwnSmYp3mYxGIVYmSibmUibQxFDePLEcNQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

*图源：https://openai.com/blog/chatgpt*

![图片](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDRJagFUcAxY6dK8libycfwcmdiaUtPzqGnBHo8C7iaibDGc1FwVZIOrOsa482JLnT0JhibJHz6ef4TdEQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**ChatLLaMA的关键技术**

点进ChatLLaMA项目主页来看，会发现它实际上集成了四个部分——

DeepSpeed、RLHF方法、LLaMA和基于LangChain agent生成的数据集。

其中，**DeepSpeed**是一个开源深度学习训练优化库，包含名叫Zero的现存优化技术，用于提升大模型训练能力，具体指帮模型提升训练速度、降低成本、提升模型可用性等。

**RLHF**则会采用奖励模型来对预训练模型进行微调。奖励模型即先用多个模型生成问题问答，再依靠人工对问答进行排序，让它学会打分；随后，基于奖励学习给模型生成的回答进行打分，通过强化学习的方式增强模型能力。

**LangChain**是一个大语言模型应用开发库，希望将各种大语言模型整合起来，结合其他知识来源或计算能力创建一个实用的应用程序。LangChain agent则会像思维链一样放出GPT-3思考的全过程，将操作记录下来。

**参考文献**：

[1] https://medium.com/@enryu9000/mini-post-first-look-at-llama-4403517d41a1

[2] https://arxiv.org/pdf/2204.02311.pdf

[3] https://mp.weixin.qq.com/s/kImwfWWtXMmEDVOhJZ4dJg

[4] https://github.com/nebuly-ai/nebullvm/tree/main/apps/accelerate/chatllama

[5] https://twitter.com/omarsar0/status/1630211059876339713

[6] https://mp.weixin.qq.com/s/Qyf7Ng2mhuJggpyDp_84zQ
