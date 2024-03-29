# GPT-4 架构，基础设施，训练数据集，成本，视觉，MoE

副标题：揭秘 GPT-4：引领 OpenAI 走向他们架构的工程权衡

## TL;DR
### Claude-2-100k@Poe

根据这篇文章,主要讲了以下内容:
1. OpenAI 之所以保密 GPT-4 的架构,不是因为担心存在风险,而是因为它的架构是可以被复制的。作者预计谷歌、脸书、Anthropic 等公司都能在近期拥有与 GPT-4 同样强大的模型。
1. 介绍了 GPT-4 的模型架构,包括参数量约 180 万亿,采用了混合专家(MoE)结构,以及训练数据集、标记数、层数等参数。
1. 分析了 GPT-4 在训练和推理方面的工程取舍,包括并行策略、多查询注意力(MQA)、推测解码等技术,以及如何克服巨大模型推理的瓶颈。
1. 估算了在 A100 和 H100 芯片上训练和推理 GPT-4 的计算成本。
1. 讨论了 GPT-4 的视觉多模态编码器设计,以及下一代 GPT-5 可能从零开始训练视觉并能生成图像。
1. 指出视觉模型的 IO 成本与文本模型不同,对未来硬件优化很重要。
1. 总体来说,文章比较全面地介绍和分析了 GPT-4 的架构、训练和推理等方面的细节。

重要数据
1. GPT-4 参数量约为 1.8 万亿,分布在 120 层,比 GPT-3 的 1750 亿 参数多 10 倍。
1. GPT-4 采用了混合专家(MoE)架构,包含 16 个专家,每个前向传播只使用 2 个专家,约 280 亿参数。
1. GPT-4 在约 13 万亿词元上进行预训练,包含多轮 epoch。
1. GPT-4 的批量大小达到了 6000 万。
1. GPT-4 在 25000 个 A100 GPU上训练,耗时 90-100 天,FLOPS 约为2.15e25。
1. 估计 GPT-4 的训练成本在 6300 万美元左右。
1. GPT-4 在 128 个 GPU 集群上进行推理,使用 8 路张量并行 + 16 路流水线并行。
1. GPT-4 的推理成本是 GPT-3 的 3 倍,一个原因是利用率较低。
1. GPT-4 使用了多查询注意力(MQA)等技术来减少内存需求。
1. GPT-4 整合了独立的视觉编码器,增加了参数量。


## 正文

概述
- 训练成本无关紧要
  - 在接下来的几年中，Google、Meta 和 OpenAI/Microsoft 等多家公司将在价值超过一千亿美元的超级计算机上训练模型
  - 这是新的太空竞赛
  - 人工智能可以在短期内从人类助手和自主代理身上带来有形的价值
- AI 瓶颈，是推理
  - 推理的成本超过了训练的成本数倍
  - 必须达到足够高的吞吐量
  - 训练和推理算力解耦
  - 进行稀疏模型架构；每个参数在推理过程中都不会被激活

GPT-4 模型架构
- 总共有约 1.8 万亿个参数,分布在 120 个层
- 利用混合专家(MoE)模型保持了合理的成本
- 550 亿个共享参数用于注意力机制
- 模型中使用了16 个专家，每个专家的 MLP 大约有 1110 亿个参数
- 每次前向传递推理（生成 1 个 token）只利用了约 280B 参数和约 560 TFLOPs。
  - 与纯密集模型每次前向传递所需约 1.8 万亿参数和约 3700 TFLOPs 形成了鲜明对比

数据集
- 在约 13 万亿 token 上训练了 GPT-4
- Deepminds 的 Chinchilla 训练使用 1.4 万亿 token
- Google 的 PaLM 模型训练使用 0.78 万亿 token
- PaLM 2 预计使用 5 万亿 token
- OpenAI 使用的批量大小达到了 6000 万，每个专家可见 750 万个 token

并行策略
- A100 GPU 8 路张量并行（这是 NVLink 的极限），15 路管道并行
- 在每个GPU上取 FP16 就需要大约 30GB 显存
- 可能使用了ZeRo Stage 1
- 可能使用了块级 FSDP，或者混合共享数据并行
- 为什么没有使用完整模型的 FSDP，可能是因为通信开销更大

训练成本
- GPT-4 的训练 FLOPS 约为 2.15e25
- 约使用 25000 个 A100  90~100 天，MFU 约 32% 到 36%
- 独立运行的训练成本将达到约 6300 万美元，每 A100 小时1美元
- 预训练可以在大约 55 天内用 ~8192 个 H100 完成，费用为 2150 万美元，每个 H100 小时 2 美元

混合专家
- 在推理过程中减少参数数量
- 使用 64 到 128 个专家能比 16 个专家取得更低的损失
- 更多的专家也可能更难达到收敛
- 较少的专家还有助于他们的推理基础设施

推理
- 权衡延迟、吞吐量、利用率
- LLM 推理主要是关于平衡两个主要点，内存带宽和计算
- 当批量大小为 256 或 512 时，每读入的内存字节有 512 FLOP/s 或 1024 FLOP/s
- 这个比例更接近 H100s 内存带宽与 FLOPS 之间的比率
- 这有助于实现更高的利用率，但也带来了更高延迟
- 较低的延迟通常可以通过减小批量大小来实现，但是较小的批量大小也会导致 MFU 恶化
- 从而导致每个 token 的总成本增加
- 离线推理，增大批量大小是最有效的
- 对于批处理大小为 512、上下文长度为 2048，KV 缓存总计达到 3TB，这是模型参数大小的3倍
- OpenAI 在一个由 128 个GPU组成的集群上运行推理
- 推理在 8 路张量并行和 16 路管道并行下完成
- 每个由 8 个 GPU 组成的节点只有大约 130B 参数，或者每个 GPU 在 FP16 下不到 30GB 显存需求，在FP8/int8 下不到 15GB。

推理成本
- GPT-4 的成本是 175B 参数 Davinchi 模型的 3 倍
- 使用 128 个 A100 进行 GPT-4 8k 序列长度的推理每 1k 个 token 的成本为 0.0049 美分
- 使用 128 个 H100 进行 GPT-4 8k 序列长度的推理每 1k 个 token 的成本为 0.0021 美分
- 这可能是一个错误的假设，因为很明显 OpenAI 有时候的利用率非常低
- 我们认为 OpenAI 在波谷小时候会关闭群集，并使用那些节点恢复从检查点开始训练较小的测试模型

多查询注意力
- 只需要 1 个“头”，KV 缓存的内存容量可以显著减少
- 32k 序列长度的 GPT-4 绝对不能 在40GB 的 A100 上运行
- https://arxiv.org/pdf/1911.02150.pdf

连续批处理
- OpenAI 实现了可变批大小和连续批处理
- 允许一定程度的最大延迟以及优化推理成本
- https://www.anyscale.com/blog/continuous-batching-llm-inference

推测性解码
- 听说 OpenAI 在 GPT-4 的推理上使用了推测性解码
- 推测性解码完全不会降低模型的质量
- 推测性解码以算力换取内存带宽
- 推测性解码的基本思想是使用一个更小、更快的草案模型提前解码几个 token
- 然后将它们作为一个批量输入到模型中
- 如果较大的模型拒绝了草稿模型预测的标记，那么剩下的批次将被丢弃
- 算法会恢复到标准的逐个标记解码
- 可以节省大量的内存带宽，从而节省每个 token 的时间
- 随着算术强度的增加，推测解码的回报迅速减小。
- 两个模型对于长连续的令牌序列达成一致的概率是指数级别的低

视觉多模态
- 这是一个与文本编码器分离的视觉编码器，但存在交叉注意力
- 在仅文本预训练之后，它使用另外约2万亿个 token 进行微调。
- OpenAI 希望从零开始训练它，但是它还不够成熟，所以他们希望通过从文本开始来降低风险
- 在视觉模型上，数据加载的IO大约是文本模型的150倍
- 每个标记600字节，而不是文本模型的4字节
- 目前正在进行大量的图像压缩工作

Nvidia
- Nvidia不断更新底层软件，通过更智能的数据在芯片、芯片之间、以及内存之间的移动，提高 FLOPS 的利用率
- Nvidia 的 FasterTransformer 推理库非常糟糕

Misc
- LLM 第一阶段是预填充，prompt 通过模型生成 KV 缓存和第一个输出对数。
- 这通常很快，因为整个 prompt 可以并行处理。
- LLM 第二阶段是解码，通常是自回归生成中最昂贵的部分
- 解码的算术强度（即，计算的 FLOP / 内存带宽的字节）在小批量运行时非常低
- 在 OpenAI 的 API 调用中，输入 token 比输出 token 便宜得多

## 术语
**多头注意力**（Multi-Head Attention）是一种注意力机制，常用于序列数据处理中。在多头注意力中，输入序列被分成多个子序列，并针对每个子序列学习不同的特征表示。这些不同的特征表示可以被拼接在一起，从而增强模型的表达能力。

**多查询注意力**（Multi-Query Attention）：为多头注意力的变体，其中键和值在所有不同的注意力“头”之间共享，大大减少了这些张量的大小，从而减少了增量解码的内存带宽要求。我们通过实验验证了生成的模型确实可以更快地解码，并且与基线相比仅产生轻微的质量下降。

**FLOPS** 是指每秒钟浮点运算次数（Floating-point Operations Per Second），是衡量计算机性能的一种指标。FLOPS通常用于评估计算机的处理速度和能力，特别是对于需要大量数值计算的应用程序，例如科学计算、深度学习等领域。

**管道并行**（Pipeline Parallelism）是一种模型并行的技术，用于在多个设备或处理器之间并行计算深度学习模型的不同部分，以加速模型的训练和推理。在管道并行中，模型被分成多个阶段，每个阶段分配到不同的处理器或设备上，每个处理器或设备负责计算相应阶段的输出，并将其传递给下一个阶段进行计算。

**混合专家（MOE）**是指同时擅长几个不同领域的专家。就像你可能擅长数学、语文和体育一样，一个混合专家可能会擅长设计、编程和营销等多个领域。这些专家能够在不同的领域之间进行交流和合作，从而提供更全面和综合的解决方案。

**连续批处理**是指在深度学习模型训练过程中，将多个数据批次连续地输入到模型中进行训练的过程。与传统的单个数据训练相比，连续批处理可以更好地利用计算资源，提高训练效率和速度。

**MFU**（Model FLOPs Utilization）是指模型在进行计算时，实际使用的浮点运算次数占总的浮点运算次数的比例。MFU 越高，意味着模型更加高效地利用了计算资源，计算效率也更高。MFU 通常会受到模型结构、数据集、批量大小等多种因素的影响

**MLP** 是多层感知器（Multilayer Perceptron）的缩写，是一种常见的神经网络模型。多层感知器由多个全连接层组成，每个全连接层接收上一层的所有神经元作为输入，并将它们作为输出传递给下一层。

密集模型架构是指在深度学习中使用的一种模型架构，用于处理低维密集数据。与稀疏模型不同，密集模型需要处理大量的数据，这些数据中的每个特征都是非零的。在密集模型中，每个输入特征都与输出之间存在一个全连接的权重矩阵，这需要大量的存储空间和计算资源。

**Nvlink** 是一种将两个或多个显卡连接在一起的技术，以便它们可以协同工作并共享处理能力。这有点像您和您的朋友如何在一个项目上合作以更快地完成它。

**批量大小**（Batch Size）是指在深度学习模型训练过程中，每个训练迭代中使用的样本数量。在每个训练迭代中，模型会根据批量大小从训练数据集中随机选择一定数量的样本。通常情况下，批量大小越大，训练速度越快，但需要更多的内存和计算资源。而批量大小越小，训练速度越慢，但可以更好地控制模型的过拟合，并且可以更容易地处理不同大小的数据集。

**视觉多模态**是指我们使用多个感官来获取信息。比如，当我们在观看电影时，我们不仅仅是看到画面，我们还能听到声音和音乐。这些不同的感官输入一起工作，让我们对电影的理解更加深入和全面。

**推测性解码**是指我们根据先前的经验和知识来预测和猜测未知信息的过程。这个过程在语言学中非常常见。比如，当我们听到一个句子的一部分时，我们可以通过以前的经验和语言知识来猜测下一个单词是什么。推测性解码在阅读和写作中也非常有用，因为它可以帮助我们更好地理解和表达信息。

**稀疏模型架构**（Sparse Model Architecture）是指在深度学习中使用的一种模型架构，用于处理高维稀疏数据。在稀疏数据中，大多数特征都是零，只有少数特征是非零。传统的深度学习模型需要处理所有特征，这会导致计算开销和存储开销非常大。而稀疏模型架构使用一些特殊的技术来处理这些高维稀疏数据，从而降低了计算和存储开销，并提高了模型的效率和性能。

**张量并行**（Tensor Parallelism）是一种数据并行的技术，用于在多个处理器或多个设备上并行计算张量操作，以加速深度学习模型的训练和推理。在张量并行中，大型的张量操作被划分成多个小的子操作，这些子操作可以在多个处理器或设备上并行计算。这种方式可以充分利用多个处理器或设备的计算能力，加速模型的训练和推理过程。


## 相关文档
[1] https://mp.weixin.qq.com/s/kOIoLc9nZDuM-bpNfHvW-A GPT-4 Architecture, Infrastructure,Training Dataset, Costs, Vision, MoE  
[2] https://www.semianalysis.com/p/gpt-4-architecture-infrastructure