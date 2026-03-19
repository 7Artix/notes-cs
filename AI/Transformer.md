# Preview

GPT, Generative Pre-trained Transformer.
- Generative: text generation.
- Pre-trained: 模型经历了从大量数据中学习的过程. 针对具体任务, 可以通过额外的训练进行微调.

Transformer可以构建不同功能的模型, 例如文本生成, 语音转文字, txt2img, 翻译等.

Google-Attention is all you need(2017). 演示的案例为文本翻译. 
后续例如ChatGPT之类的LLM是变种: 输入一段文字, 并可能伴随一些图片音频等数据, 预测文段接着出现的内容. 将可能出现的文本片段以概率分布的形式展示.

输入的内容首先被会切分成**Tokens**. 在文本中, Token通常是单词, 单词片段或其他字符组合; 对于图像和声音, Token通常是切分的小块. 每个Token都对应一个**向量**. 向量旨在表示Token片段的含义. 含义接近的向量往往对应坐标也接近(不光是夹角, 还有模长).
- 夹角(余弦相似度)是衡量语义相似度的核心指标, 是语义内容的本质.
- 模长在早期词向量中常备忽视, 但在Transformer中有明确的物理意义, 代表**量**与**确定性**. 通常**高频词**的Embedding模长更大. 模长也可以看做模型对该Token表达某种语义的"信心". **多义词**的向量模长通常较小.

**Attention Block & MLP Block**

Embedding会通过Attention Block进行处理, 使向量间能够**互相交流**, 通过互相传递信息**更新自身值**.

注意力模块的核心作用就是找出上下文中的哪些词会影响哪些词的含义, 以及如何更新含义. 使每个Embedding的含义更明确.

经过一次向量含义更新后, 所有向量会各自经历一次 **MLP (Feed-Forward Network, FNN)** 前向传播. 在此阶段, 向量间不会互相交流, 而是**并行**经过同一处理. 这一过程类似于向向量询问**一系列的问题**, 一系列问题的答案就是经过FNN的**新向量**. 所有向量经过的FNN是相同的.

注意力模块和MLP模块会层层堆叠, 不断交替进行.

Transformer的每一层都是**等宽的**, 若输入的Token数量是10(原始词向量), 则每一层都会输出10个向量. 
在**最后一层**, 第10个向量被认为是理解了所有的内容后得出的结果, 该向量会乘以一个矩阵, 输出一个代表**词表**中各个单词概率分布的得分, 经过**Softmax**之后得到真正的**概率值**, 然后进行选择.

ChatGPT**通过预设AI助手与用户的对话场景 (System Prompt)**, 以及将用户的提问(User Prompt)作为引导, 让模型预测AI助手会如何回答.

# Background

线性回归(Linear Regression)是最简单的机器学习. 其输入输出都是单个数字, 例如房价和面积的关系. 目的是找到一条最佳拟合线. 线由两个参数决定.

Transformer有更多的参数, 例如**GPT-3**有**175,181,291,520个参数**, 组成**27,938个矩阵**. 矩阵分为了8个类型:
- **Embedding**
- **Key**
- **Query**
- **Value**
- **Output**
- **Up-projection**
- **Down-projection**
- **Unembedding**

深度学习的输出输出通常都满足固定的模式.

输入必须是实数数组, 可以是1维, 2维, 与更常见的高维数组, 即张量. 输入数据通常被转换为多个层, 每个层都是实数数组.

模型工作的过程中, 一切基本都是向量与矩阵的乘法, 需要时刻注意区分, 防止混乱. 核心分为作为模型的预训练结果的**权重矩阵**, 与待处理的**数据向量**.

# Slicing & Embedding

将输入切分成Token, 并将其转换为向量. 模型有一个预设的**Token词汇库**, 包含所有可能的Token. 使用Token而不直接是单词有如下目的:
- 处理生僻词: 生僻词可以分解成多个Token而不是使词汇表无限膨胀.
- 节省空间: 常见词通常是单独Token, 罕见词会被拆分.
- 多语言统一: Token可以处理代码, 符号, 和不同语言.

## Tokenizer 分词器

分词器是**纯数学/统计学**算法, 不是任何形式的神经网络, 是一个贪婪字符串替换程序, 只有一张合并规则表.

分词器的获取过程是在海量的文本上进行统计, 计算哪些字符组合出现的概率最高, 并将这些组合记录下来.

分词器与后续的Transformer是解耦又耦合的:
- 解耦: 分词器在Transformer训练前就已经固化.
- 耦合: Transformer的输入基于分词器的规则, 如果更换分词器, Transformer必须从头训练.

例如从Llama2到Llama3, 分词器从32k Token词表升级到128k Token 词表, 因此所有权重无法继承, Llama3必须重新训练.

现在LLM使用的是Byte-level BPE (BBPE), 即字节级切分. 能够有效的处理各种语言, 公式, 代码的切分.

## Embedding 词嵌入

将Token转换为向量称为Embedding. 嵌入后的向量在高维度上体现含义. Embedding既代表将Token转换为向量的过程, 又代表转换后的向量本身.

Man-Woman的向量差 ≈ King-Queen的向量差. 实际会有偏差, 因为训练的数据中Queen并不全是King的女性化版本. 因此, 若向量空间中的某一个基向量编码性别信息, 则具有意义.

E(Hitler) + E(Italy) - E(Germany) ≈ E(Mussolini).

两个向量的点积值越大, 对齐程度越高.

例如令 $\vec{plur} = E(cats)-E(cat)$ . 将 $\vec{plur}$ 与各个数字做内积, 可能出现:
- $\vec{plur}\cdot E(one) = -2.40$ 
- $\vec{plur}\cdot E(two) = 0.79$ 
- $\vec{plur}\cdot E(three) = 1.27$ 
- $\vec{plur}\cdot E(four) = 1.80$ 

输入接触的首个矩阵是 **Embedding Matrix** $\color{red}W_E$ . 规模为 $V\times d$ .

- 行数 $V$ , **Vocabulary Size**, 对应词库的大小, 表示模型认识多少不同的Token. 如GPT-3有50257维.
- 列数 $d$ , **Embedding Dimension**, 信息维度/特征维度. 如GPT-3有12,288维.

总权重数量为: $50,257\times 12,288 = 617,558,016$

$W_E$ 的初始值也是**随机**的, 通过数据学习, 是整个模型中的第一组权重.

将Token转换成Embedding, **相同的Token在不同语境下的Embedding是相同的**, 对于Token在不同语境下的理解要在Attention的作用下才会完成. 即会在context作用下在向量空间内被拉扯, 逐渐趋向于一个更具体, 更准确的定义, 整个过程类似给词汇加定语. 最开始每个向量只能编码单个Token的含义, 没有上下文信息.

训练Embedding的本质是让模型自己决定在向量空间中, 词与词之间的距离, 从而让后续的Attention计算起来更顺畅.

**Embedding**本身也是一次对于输入内容的理解, 像是一个压缩后的百科全书. 训练Embedding的过程就是在不断修改百科全书, 让其描述更加准确. 后续Attention的过程是将百科全书的不同词条的内容串接起来理解. Embedding本质上就是**将语言文字转换成其真实携带的高维信息**. 是一个**翻译官**, 将字符翻译为计算机能理解的数字语言.

输入接触的首个矩阵是 **Embedding Matrix** $\color{red}W_E$ . 规模为 $V\times d$ .

- 行数 $V$ , **Vocabulary Size**, 对应词库的大小, 表示模型认识多少不同的Token. 如GPT-3有50257维.
- 列数 $d$ , **Embedding Dimension**, 信息维度/特征维度. 如GPT-3有12,288维.

总权重数量为: $50,257\times 12,288 = 617,558,016$

$W_E$ 的初始值也是**随机**的, 通过数据学习, 是整个模型中的第一组权重.

Embedding Matrix不是MLP, 而只是一个矩阵, 权重就是矩阵中的每一项. 因此Embedding部分的权重数量就是矩阵的项数, 而没有多余的偏置项等部分.

当根据输入文本创建向量组时, **每个Token对应的向量都是直接从嵌入矩阵中选取的**. 其具体操作是一次直接根据Token id的查找.

从数学层面上来看, Embedding被定义为一次**矩阵运算**. 设 $x$ 是一个 $1\times V$ 的**One-hot**列向量, 即只有第 $i$ 个元素为1, 其余为0, $\color{red}W_E$ 是 $V\times d$ 的Embedding Matrix. 则有:

$$
Embedding = x \cdot W_E
$$

当**反向传播**时, 损失函数产生的**梯度**流回Embedding层, 由于Embedding本质是通过查找得出的, 因此梯度会定向投递, 只有词表对应项的 $d$ 个值会被更新.

## Unembedding 解嵌入

在最后输出内容的时候, 需要将向量映射回词库列表. 对应的**Unembedding Matrix**是 $\color{red}W_U$ , $W_U$ 接近 $W_E^T$ , 在早期模型中, 会直接使用 $W_E^T$ 作为输出反映射的矩阵, 但后续模型认为应当为输出提供更灵活的表达方式, 因此训练单独的 $W_U$ .

最后预测的本质是一次**语义相似度搜索**. 最后一步的本质仍是向量的点积, 向量 $y$ 是 $d$ 维向量, 是理想中的下一个词. 
词库里的每一个词在 $W_E$ 矩阵里都有一列, 代表那个词的标准画像. 若想知道输出哪个词最合适, 将 $y$ 与 $W_E$ 中的每一列做**点积**, 计算相似度, 将得到的相似度做**Softmax**, 即得到输出每个词的可能性概率分布, 整个过程等价于:

$$
y \cdot W_U
$$

其中 $y$ 是 $1\times d$ 的向量, $W_U$ 是 $d\times V$ 的矩阵.

输入Token往往局限在向量空间的一个锥内, 若 $W_U$ 直接采用 $W_E$ 可能会出现对比度过低,导致输出内容不准确的问题. 在输入分析时, 模型往往把握语义的大致走向, 而在输出时, 模型需要敏锐的捕捉两个类似的词的区别, 从而生成更精准的描述, 使用单独的 $W_U$ 使输出可以不像输入那样温和地理解语义.

Unembedding Matrix的参数量与Embedding Matrix相同, 对于GPT-3, 为 $50,257\times 12,288 = 617,558,016$ .

在Softmax时, 引入参数T, 使Softmax对于 $x_i$ 的输出变为:

$$
x_i = \frac{e^{\frac{x_i}{T}}}{\displaystyle\sum_{n=0}^{N-1}{e^{\frac{x_n}{T}}}}
$$

- 当 $T=0$ 时, 所有概率全给输出值最大的一个.
- 当 $T=1$ 时, 是标准的Softmax.
- 当 $T>1$ 时, 会给输出值较低值更大的概率, 让概率分布更均匀.

通常, API会禁止选择大于2的T值.

输入到Softmax的原始值通常称为<span style="color: #ff0000"><b>Logits</b></span>, Softmax的输出值通常称为<span style="color: #ff0000"><b>Probabilities</b></span>.

## Context 上下文

context是将所有输入的Token, 经过Embedding后, 输入后续Blocks的内容. 其长度限制就是最多输入的Embedding向量数量. 例如GPT-3的context length仅有 $2,048$ , 而到GPT-5时, 已经支持400K的context length.

context长度限制了Transformer在预测下一个词时, 能结合的内容范围.

## Positional Encoding/Embedding

Transformer本质是一个并行处理器, 没有时间感, 它将所有Token变成一个巨大的矩阵, 一次性输入模型计算. 在Self-Attention的计算公式 $Attention(Q,K,V)$ 中, 只有矩阵乘法, 若将句子顺序打乱输入, 如果没有位置编码的情况下, 点积计算结果**完全相同**, 因此必须人为注入位置信息.

为了分辨不同语序对含义产生的影响, Transformer引入了**位置编码 (Positional Encoding/Embedding)**.

位置编码通常**只和位置顺序有关**, **和Token内容完全无关**.

位置编码通常分为**绝对位置编码 Absolute Positional Embedding (APE)** 和 **旋转位置编码Rotary Positional Embedding (RoPE)**.

**APE**

模型拥有一张位置表Position Embedding Matrix. 表形状为 $\text{MaxSequenceLength} \times d$ . 每一行对应第 $i$ 个位置的位置向量. 该表可以通过固定公式得出, 或通过学习得到. 
- 固定公式计算时, 通过三角函数计算得到一组数字作为位置向量.
- 通过学习更新时, 像Embedding Matrix一样, 位置向量初始也是随机化的, 通过训练更新权重.

$$
最终输入向量 = \text{Token Embedding} + \text{Positional Embedding}
$$

绝对位置编码是和Token Embedding同时进行的, 发生在进入Attention Blocks之前.

**RoPE**

RoPE不是在开始的Embedding层做的, 而是在每一个Attention Block内部.

# Attention

Embedding由Token得到, 经过Attention Block后, 使其通过上下文的全局理解, 更新含义. Attention Block允许Embedding之间交换信息. 
每次输入一个Embedding, Embedding原本只由Token直接得到, 但经过所有Attention Block更新后, 向量最终能够获得超越单个Token的信息量.

Transformer的输入输出分别为:
- 输入: 

## Single head of attention 单头注意力

## Query

## Key

## Value

