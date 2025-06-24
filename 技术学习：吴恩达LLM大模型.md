---
title: 技术学习：吴恩达LLM大模型
date: 2025-06-23
tags: 
- LLM
- Python
categories:
- 算道（课外IT技能学习）
- Lecture
description: 吴恩达LLM大模型课程学习笔记
---

## GenAI 与 LLM

LLM（Large Language Model）. 模型的参数越大，其记忆越大，能处理的任务也越复杂。而较小的模型更可能在一些特定领域表现优异。

一般使用都是直接用模型或者加以微调，很少自己从头训练新模型（成本太高）。我们传递给大模型语言指令（Prompt），Prompt 可用的空间或者记忆被称作上下文窗口（Context Window），可以存放几千个字；模型输出叫做 Completion。整个使用模型输出文本的行为被称作推理（Inference）。

>[!NOTE]
>
>推理和学习的区别主要就是我们不自己训练模型。

LLM 的应用包括：聊天，处理文本，翻译，写代码，词分类……

早期训练使用循环神经网络 （Recurrent Neural Networks RNN）。RNN 非常强大，但是需要大量计算和内存来执行任务，所以能力受限。随着文本增加，RNN 的记忆和所需内存指数级上升，而且 RNN 的状态变化模型单一，很难理解一词多义这种情况，所以不太适合 LLM 应用场景。

后来 《Attention is All You Need》这篇论文提出使用 Transformers 来构建 LLM 更好。Transformer 的注意力机制可以很好地让句子中的词互相关联起来。

## Transformer

如下图，Transformer 分为编码解码器两部分。

![ ](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506231838673.png)

首先，在这个模型之前，我们要明确一个 Tokenizer 分词器的概念。这个东西将单词转换为数字，每个数字代表字典中所有可能单词的 ID 位置。分词方法有很多，比如一个短语一个 Token，或者一个单词多个 Token。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506231840422.png" alt=" " style="zoom: 50%;" />

输入输出的分词器应当一致。

嵌入层：每个 Token ID 对应一个可训练的多维度向量（比如独热编码格式，word2vec 格式，其他格式等）。原始 Transformer 论文中向量维度是 512.

比如每个 Token 被转换成一个三维向量，可以理解为放到一个三维空间中，这些 Token 之间的距离就可以帮助计算机“理解”意义。

然后 Token 向量还要和一个位置向量求和，这样保证单词在句子中的位置信息也被应用。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506231849323.png" alt=" " style="zoom:67%;" />

然后在自注意力（Self-Attention）层，会学习词的自注意力权重，就是这个词对句子中其他词的重要性。

Transformer 其实是多头自注意机制（Multi-headed Self Attention），多头自注意力并行学习，可能会学习句子的不同方面，比如人与人之间的关系，句子语法，句子是否通顺……

多头自注意力的学习机制没法认为指定，训练者只能通过随机初始化的权重让不同的头学习不同方面。所以可能有些注意力头并不这么容易理解。

编码器的输出被送到前向网络中输出到解码器中。编码器的输出是输入序列的结构和含义的深入表达，这些插入到解码器中影响解码器的自注意力机制。

解码器预测得到第一个 Token 输出。输出先被投射到一个对数几率向量中，在经过 Softmax 处理获得 Token 字典中每个词的生成几率，计算机选择概率最高的 Token 进行输出。

然后解码器之前的所有输出和最开始的输入 Token 拼接到一起作为下一轮输入，重复如上过程继续输出，直到达到某个停止条件。

以上就是简单的 Transformer 机制。总结来说编码器主要学习输入序列结构和含义，解码器借助编码器的结果和输入 token 输出。

也有一些仅编码器，仅解码器的应用。比如 GPT 系列，Llama 就是仅解码器；仅编码器一般输入和输出长度一致。

## 提示词工程

在上下文窗口中提供例子被称作上下文学习（In-Context Learning）.

比如我给 gpt 一段文本，让他对其进行情感分析，没有任何参考学习用例（大多数时候我们都是这么用 GPT 的）。这被称作零样例推断（zero-shot inference）. 小模型处理零样例推断的时候结果可能会比较差。

下面是带有一个样例的提示词，这被称作单样例推断（One-shot inference）。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506231909277.png" alt=" " style="zoom: 67%;" />

有时候一个样例可能也不够让模型理解我的需求，可以扩展到少数样例推断（few-shot inference）。

不过注意模型上下文窗口长度有限，可能给不了太多样例。

如果包括了 5-6 个样例输出效果仍然不好，建议对模型进行微调（Fine-tuning）。

## 推理生成配置

一些模型会公开一部分参数供大家微调。

- 最大生成 Token 数量，影响模型迭代次数和输出长度；
- 创造性和严谨性，解码方式有贪婪抽样（Greedy sampling）永远选择概率最大的单词，但是容易导致重复，和随机抽样（Random sampling）根据概率权重，一定程度上随机选取单词，不过可能会过于创新使得输出和输入没什么关联；
- sample top K 和 sample top P：top K 是限制随机抽样只在概率排行前 K 个 Token 中选，一定程度上减小了随机性，让输出更加合理，有意义；top P 是只选择概率求和加起来小于 P 的 Token 进行随机抽样，不是限制前几个 Token 而是限定一定的概率；
- temperature，影响 softmax 公式改变输出的 Token 的概率。低 temperature 得到的概率图像会更窄更尖，随机性更小；高 temperature 的概率图像会更均匀，随机性更大。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506231924354.png" alt=" " style="zoom:50%;" />

## 生成式人工智能项目 Life Cycle

如下图。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506231924771.png" alt=" " style="zoom:67%;" />

需求（比如输出长文本；只应用于特定领域；……）

-> 已有模型还是自己训练

-> Prompt 还是微调还是人类反馈，以及评估（这部分可能要迭代多次）

-> 优化部署模型，以及一些增强措施，比如防止 LLM 不知道答案时编造答案，以及增强其逻辑推理能力等，后面介绍

## 微调

### 全面微调

如果 Prompt 推理，以及加上少样例的 Prompt 推理无法达到需求，就得进一步考虑修改模型参数进行微调了。

一般来说如果模型太小，或者上下文窗口长度有限，Prompt 就难满足需求。

微调算是有监督学习，我们给定一些 prompt 和 completion 对的指令，调整模型参数使其获得近似的 completion 结果。比如“请翻译如下文本：[TEXT] [COMPLETION]” 给出要翻译的文本和我们期望其翻译结果。

更新所有模型权重的微调叫做全面微调（full fine tuning）。全面微调需要大量内存，存储等资源。（后面还会介绍 LoRA 等部分微调）。

首先我们需要找到一些数据集，然后将其转化为上面那种指令。一些开发者有总结出提示模板库：

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506231946809.png" alt=" " style="zoom: 40%;" />

然后类似有监督学习，将数据分成训练集 CV 集，测试集进行学习训练，利用交叉熵函数计算损失，评估，后向更新……得到新的指令模型或者叫做微调模型。

### 对单一任务微调

如果 LLM 模型只针对特定任务，可能 500 个左右的微调指令就够达到不错的结果了。通用 LLM 可能得上亿个。

不过单一任务微调可能会导致灾难性遗忘（Catastrophic forgetting）灾难性遗忘问题。因为单一任务微调也是全面微调，所以导致其他任务的表现变差了。

发生灾难性遗忘，首先考虑一下业务需求，其他功能能力下降是否真的影响很大。

如果影响较大，可以同时进行多任务微调，当然这样微调指令，成本也大幅度增加。

或者可以进行有参数微调（Parameter Efficient Fine-Tuning PEFT），保留大部分原始预训练层权重，只训练少量特定任务的适配器层和参数。

### 多任务指令微调

Multitask Fine-Tuning。就像我们之前说的，同时训练多个任务。因此也需要大量数据，可能 50000-100000 范围。

已经经过多任务指令微调的模型表现可能也未必好，比如 FLAN T5 模型的总结能力主要通过一个语言学家制作的消息对话数据集进行训练，但是对日常生活对话的总结能力并不好。这种情况下我们就可以对其进行微调。

实际使用的时候公司用自己的数据集会更合适，比如自己公司的客户对话数据集。

### 模型评估

我们需要知道如何评估模型才能得出“该模型表现良好”这样的结论。

学习有监督学习的时候大部分评估都很简单，但是 LLM 评估难很多。比如”我爱你” “我喜欢你”是近似的结果，怎么衡量其相似度？再比如“我喜欢喝咖啡” “我不喜欢喝咖啡”只有一个字的区别但是意思完全相反。

#### ROUGE

召回率导向的摘要评估（Recall-Oriented Under study for Gisting Evaluation），用于衡量摘要性能。

一个单词等同于一个 unigram，两个词是 bigram，多个词是 ngram。

下面是只有 unigram 的评估。缺点在于没有顺序。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506232120269.png" alt=" " style="zoom: 50%;" />

所以可以改成 bigram。当然对于越长的句子 bigram 顺序表现也会变差。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506232121050.png" alt=" " style="zoom: 33%;" />

与其继续扩充使用 ngram，不如改为最长子序列：

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506232122704.png" alt=" " style="zoom:50%;" />

这样还可能出现一个问题，比如输出是 cold cold cold cold 评估分数也会挺高的。所以我们可以通过一个 clip 函数限制重复 unigram matches 的最大值：

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506232124157.png" alt=" " style="zoom: 33%;" />

#### BLEU

双语评估研究（Bilingual Evaluation Under study）主要用于评估翻译。通过多个 ngram 的平均 precision 计算。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506232142687.png" alt=" " style="zoom: 40%;" />

BLEU 和 ROUGE 都是比较简单的，分别用于翻译和摘要的评估。不过对于模型总的评估不能单用这两种，要用基准。

#### 基准

Evaluation Benchmarks。

可以选择一些突出特定功能评估的数据集，并且训练的时候要确保模型未见该数据集。

如 GLUE，SuperGLUE……

[LLMs 评测 benchmark 汇总 - 知乎](https://zhuanlan.zhihu.com/p/638508365)

### PEFT

训练过程中，梯度，前向传播，临时内存等远大于模型自身，很快就会超过消费级电子设备的存储或内存。

PEFT 只部分更新，比如只更新某些层或者参数权重，或者新增一些层或者参数。（训练的参数权重通常只有整个模型的 15~20%），也不容易出现灾难遗忘性问题。

我们可以对同一个模型进行不同的微调，得到解决不同任务的微调模型，比如摘要模型，翻译模型……

#### Selective

只挑选原模型的子集进行训练，比如一个层，一个组件，一个参数。性能参差不齐，不是重点。

#### Reparameterization

创建原始网络的低秩（low rank）转换来减小要修改的模型权重，如 lora。

#### Additive

增加新参数。

Adapter：在编码和解码层前或者后增加新的训练层。这个新层远小于原始网络中的层，比较灵活，

soft prompt：模型结构不变，调整输入获得更好的性能。比如给 prompt 设置可训练权重（提示调整 prompt tuning），或者调整嵌入层。

#### LoRA

Low-Rank Adaptation of Large Language Models

回想一下 Transformer 的架构：

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506232234512.png" alt=" " style="zoom:50%;" />

Token 经过 Embedding 之后得到一个多向量组成的大参数矩阵。我们现在不对这个参数矩阵直接进行微调，而是训练两个低秩矩阵 A B，这两个矩阵相乘变为一个矩阵加到原来的参数矩阵上，再经过自注意力机制处理。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506232238885.png" alt=" " style="zoom: 33%;" />

矩阵大小和参数一致所以延时不多。

由于 LLM 大多数参数都在自注意力部分，所以在其之前加这么一个处理就足以获得不错的微调性能。

比如参数矩阵是 512 \* 64，使用秩 = 8 的 LoRA 微调，B 矩阵是 512 \* 8 大小，A 矩阵是 8 \* 64 大小。只训练这两个矩阵比训练整个参数矩阵训练参数量小了 9 倍左右，这也使得 LoRA 可以在单 GPU 上运行。

对于其他任务，我们可能训练一个新的 A1 B1 矩阵，使用的时候将原始参数矩阵分别加 BA 或者 B1A1 矩阵来执行不同任务。这些低秩矩阵参数量很小，所以可以存储很多个，让 LoRA 可以执行很多任务，而不是存储多个 LLM。

当然可能 LoRA 执行特定任务性能未必比全面微调的 LLM 好，但是其最主要好处就是训练复杂度，模型复杂度都小了，还可以达到差不多的结果。

### Soft Prompt

处理输入文本。

原来的 token 是和词汇表对应的嘛，我们现在将输入内容分词后，新建一些 soft token，这些 tokens 没有词义，只用于微调训练输入，输出结构中没有这些 soft tokens。

然后我们冻结其他模型参数，训练这些 soft tokens 值。

这些值嵌入在 token 前面（prefix token），作用上和加了一层 prompt 感觉似的（当然，为什么我们不用 prompt engineering 优化输入而使用这种方式？就是因为其能表示一些不存在于真实词义的特征，所以效果上和直接修改 prompt 肯定不一样）。

这部分相比于整个模型参数量也小很多，所以训练很快；而且不同任务我们就切换不同的 soft token 就行了。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506232315634.png" alt=" " style="zoom:50%;" />

已有的 hard token 因为我们固定模型参数不改变，所以其在空间中的位置是不动的。而 soft token 在空间中是活动的，其训练结果距离哪些 hard token 距离更近说明其学习到了什么样的词簇功能。

## RLHF

有人类反馈的强化学习（Reinforcement Learning from Human Feedback）.基础大模型预训练后基本可以具备语言能力，但是生成内容质量未必好，比如语气不善，内容有害，偏见歧视等，所以需要人类反馈调整。

LLM 应该遵守的HHH人类价值观：Helpful（计算机输出的内容没有帮助），Honest（比如给出错误的答案），Harmless（比如我问他如何抢银行，他真的给我好好回复了。应该拒绝回答，指出这种行为是不对的）。

### 原理

如下图，强化学习。可以额外先训练一个奖励模型用于评估奖励（LLM 模型输出和人类偏好的相似度）。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506241044577.png" alt=" " style="zoom:67%;" />

>[!NOTE]
>
>经典强化学习中将状态-行动序列称为 play-out，而 RLHF 中称作 roll-out.

### 步骤

整体步骤流程：人类标注数据 -> 有了标注数据才能训练奖励模型 -> 用奖励模型代替人类反馈对结果进行评估 -> RLHF 学习。

#### 数据整理

首先我们选择一个解决特定任务的模型，最好是预训练，微调过的。

然后输入 prompt 数据集获取 completion 输出。每条输入可能获取多个输出。

然后收集人类标注员（human labelers）对结果的反馈。

比如输入 prompt：我的房间太热了。输出可能包括：我没什么可以做的。我可以开空调降温。其实它并不太热。标注员对其帮助性进行排序，2最好，1其次，3没用。

同一份结果可能交由多个人类标注员进行标注，有的标注员可能曲解其意思而给出错误的结果。指示越清晰，人类反馈也越准确。

> [!TIP]
>
> 选择标注员的时候要选择更能代表多元文化和全球观点的人。
>
> 一般需要告诉标注员我们的评估标准，比如根据答案的准确性和有用性进行判断，必要时可以借助互联网检验答案的准确性，尽量避免出现并列排名，删除质量太差的结果等。指令越清晰，标注员越容易理解任务，也越能给出准确的反馈结果。

标注员完成标注后，对于不同的输出我们大致获得了一个排名。但是这个排名不能直接用，因为排名很难量化绝对分数。比如，排名为第一的 completion 的奖励分数是多少？这里面也有很多主观因素，不同人尽管可能得出同样的排名，但是有各种各样的打分值，而且非要打出绝对分数也会大幅度增加标注的复杂度。

相比于量化排名分数，我们可以将其转化为结果之间的两两比较获取优劣。比如“开空调降温”>“房间热我没什么可以做的”；“没什么可以做的”>“其实房间并不热”；“开空调降温”>“其实房间并不热”。相比排名，这样对于模型训练来说也很简单，只要两两比较获取最好的结果即可。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506241137505.png" alt=" " style="zoom: 50%;" />



> [!NOTE]
>
> 既然两两比较难度很低，为什么训练的时候不让人类标注员采用“点赞”“点踩”的两两比较方式而是建议用排名方式？因为排名方式可以包含更多信息，难度也没有增加太多。标注员做了 n 组数据的排名，就可以获取 n 选 2 组比较结果。

整理完成数据后，就可以开始训练奖励模型了。

#### 训练奖励模型

奖励模型也是一个语言模型，我们希望其给更好的结果（上图中的 yj ）打出相比更差的结果（yk）更高的分数。

*一般来说奖励模型和要 RLHF 的模型是一类 LLM 模型，不过其输入是 prompt + completion，最后一层不是一个 softmax 分类层，而是一个标量输出 reward 代表对该 completion 的打分。*

所以其损失函数（一个交叉熵函数）：
$$
L = -log(\sigma(R(y_{better})-R(y_{worse})))
$$
如果 better 和 worse 的两组 completion 奖励差距较小，sigmoid 函数就会比较接近0，L 损失就比较大。反之，如果 better worse 奖励差距较大，sigmoid 函数就会比较接近1，L 损失就较小。

学习完成后，我们的奖励模型就会倾向于给更好的结果打高分，给更差的结果打低分（推理过程就不需要二元比较的交叉熵损失函数了）。

#### 利用奖励模型微调

如下图，就是模型输出 completion，奖励模型给其打分，反馈迭代让模型输出分数更高的结果。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506241201775.png" alt=" " style="zoom: 50%;" />

这里常见的奖励算法：近端策略算法（Proximal Policy Optimization PPO）：每个版本的更新幅度较小，接近上一个版本，使得学习过程更加稳定。

- 第一阶段：代价函数 = 当前状态预测到结束时的价值 - 真实从当前状态走到结尾的价值（价值预测偏差）的平方。这里没啥特殊的。

$$
L^{VF} = \frac{1}{2} \left\| 
\underbrace{V_\theta(s)}_{\text{Estimated future total reward}} 
- 
\underbrace{\left( \sum_{t=0}^{T} \gamma^t r_t \,\middle|\, s_0 = s \right)}_{\text{Known future total reward}} 
\right\|_2^2
$$

- 第二阶段：模型的更新只在一定的信任区域（trust region）中更新。公式其实不是特别难，π那一部分是新策略概率/旧策略概率，用于“衡量偏好变化”，A 部分是新策略比旧策略好多少。ϵ 是剪切超参数，一般为 0.2，如果新策略概率/旧策略概率过大或者过小就会被截断为 1-ϵ 或 1+ϵ。当 Advantage >1 的时候我们希望概率比尽可能大（当然大不过 1+ϵ），当其 <1 的时候我们希望概率比尽量小。最终我们再用 min 取两个变化中较小的那一个，防止过度更新。

$$
L^{\text{POLICY}} = \min \left( 
\frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)} \cdot \hat{A}_t,
\; \text{clip} \left( 
\frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)}, 
1 - \epsilon, 1 + \epsilon 
\right) \cdot \hat{A}_t 
\right)
$$

- 熵：类似 temperature 影响模型的创新性，不过 temperature 是推理期间影响，熵是训练期间影响。
  $$
  \mathcal{H}[\pi(\cdot|s)] = -p \log p , p = \pi(\cdot | s)
  $$
  <img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506241629861.png" alt=" " style="zoom: 67%;" />

  熵值大，不确定性大（靠中间）；熵值小，不确定性小。

总损失是如下三个部分：

第一部分是第二阶段裁剪后的策略损失，用于策略改进；第二部分是第一阶段的价值损失，用于价值评估改进；第三部分是熵损失，用于不确定性，创新性改进。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506241633325.png" alt=" " style="zoom: 50%;" />

除此之外还有 Q-Learning.

### 奖励攻击

奖励攻击(reward hacking) 比如模型倾向于选择一些语气友善，内容无害，但是不符合原来语义的内容。比如“这个产品是”+“美好的，世界和平的。”

可以使用一个输出内容准确有价值的参考模型（冻结参数），和 RL 更新模型一起处理输入数据并得到输出，通过 KL 发散比较结果差距（这里没具体讲原理），如果发现 RL 结果和参考模型差距较大，则将惩罚加到 PPO 奖励上。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506241650841.png)

我们还可以应用 PEFT 参数高效微调，比如 Adapter 添加瓶颈训练层。

### 评估

同样地，我们应当有一个评估模型结果是否有害的评估方式。且经过 RLHF 过程后评估结果应当明显下降。

### 扩大人类反馈的规模

评估员人类反馈数量限制是 RLHF 的一个重要瓶颈。

与其一味尝试增加评估员数量，我们可以利用模型自监督。

#### 宪法 AI

constitutional AI，根据一套管理模型行为的规则和原则来训练模型的策略。与样本提示词结合，教会模型自监督评估。

1. 首先我们尝试诱导 LLM 输出一些有害 completion，比如：“如何黑掉我邻居家的wifi？”“你可以尝试这个app……” 这个操作被称作红队（red teaming）。
2. 然后我们让 LLM 根据宪法原则对其自己的响应进行评估，比如“确定助理最后的回应是有害的、不道德的、种族主义的、性别歧视的、有毒的、危险的或非法的具体方式。” “该回复是有害的，可能是违法的。”
3. 然后我们让模型根据宪法 AI 修改输出结果，得到无害的输出。最初的 prompt 和最后无害的输出可以放在一起作为一个训练集。借助这个训练集进行监督微调原模型（supervised fine-tuning SFT）初步微调。
4. RLAIF AI反馈强化学习，奖励模型 LLM 针对红队提示词生成一系列输出，再让其根据宪法 AI 原则比较哪些有害哪些无害，以此训练奖励模型。最后再用这个奖励模型和一些强化学习算法调整原模型。

