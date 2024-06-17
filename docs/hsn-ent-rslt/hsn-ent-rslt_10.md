# 第 10 章\. 隐私保护记录链接

在前几章中，我们已经看到如何通过精确匹配和概率匹配技术解析实体，既使用本地计算又使用基于云的解决方案。这些匹配过程的第一步是将数据源汇集到一个平台上进行比较。当需要解析的数据源由一个共同的所有者拥有，或者可以完全共享以进行匹配，那么集中处理是最有效的方法。

然而，数据源往往可能是敏感的，隐私考虑可能阻止与另一方的无限制共享。本章考虑了如何使用隐私保护的记录链接技术，在两个独立持有数据源的各方之间执行基本的实体解析。特别是，我们将考虑私有集合交集作为识别双方已知实体的实际手段，而不会向任一方透露其完整数据集。

# 私有集合交集简介

私有集合交集（PSI）是一种加密技术，允许识别由两个不同方持有的重叠信息集合之间的交集，而不向任何一方透露非交集元素。

例如，如图 [10-1](#fig-10-1) 所示，Alice 拥有的集合 A 和 Bob 拥有的集合 B 的交集可以被确定为由元素 4 和 5 组成，而不会向 Alice 透露 Bob 对实体 6、7 或 8 的了解，或向 Bob 透露 Alice 对 1、2 或 3 的了解。

![](assets/hoer_1001.png)

###### 图 10-1\. 私有集合交集

一旦确定了这个交集，我们可以结合 Alice 和 Bob 关于解析实体 4 和 5 的信息，以便更好地决定如何处理这些实体。这种技术通常在单一方向上应用，比如 Alice（作为客户端）和 Bob（作为服务器），Alice 了解交集元素，但 Bob 不了解 Alice 的数据集。

# PSI 的示例用例

在隐私法域中的金融机构可能希望查看其客户是否与另一组织共享，而不透露其客户的身份。分享组织愿意透露他们共同拥有的个体，但不愿透露其完整的客户名单。

这是本章将要讨论的方法，其中信息集是由双方持有的实体列表，客户端试图确定服务器是否持有其集合中实体的信息，而在此过程中不会透露任何自己的实体。这听起来可能像是魔术，但请跟我一起来！

# PSI 的工作原理

在服务器愿意与客户端共享其数据集的客户端/服务器设置中，客户端发现交集的最简单解决方案是让服务器向客户端发送其数据集的完整副本，然后客户端可以在私下执行匹配过程。客户端了解哪些匹配元素也由服务器持有，并且可以构建更完整的共同实体图片，而服务器则不知情。

在实践中，这种完全披露的方法通常是不可能的，要么是因为服务器数据集的大小超过了客户端设备的容量，要么是因为虽然服务器愿意透露与客户端共有的交集元素的存在和信息描述，但不愿意或不允许透露整个集合。

如果服务器无法完全与客户端共享数据，那么一个常见的解决方案通常被提议，通常称为*朴素PSI*，即双方对其数据集中的每个元素应用相同的映射函数。然后服务器将其转换后的值与客户端共享，客户端可以将这些处理过的值与自己的相等值进行比较，以找到交集，然后使用匹配的客户端参考作为键查找相应的原始元素。*密码哈希函数*经常用于此目的。

# 密码哈希函数

密码哈希函数是一种哈希算法（将任意二进制字符串映射到固定大小的二进制字符串）。SHA-256是一种常用的密码哈希函数，生成一个256位的值，称为摘要。

虽然高效，但基于哈希的这种方法潜在地可以被客户端利用来尝试发现完整的服务器数据集。一个可能的攻击是客户端准备一个包含原始值和转换值的综合表，将此全面集合与所有接收到的服务器值进行匹配，然后在表中查找原始值，从而重建完整的服务器数据集。当使用哈希函数执行映射时，这种预先计算的查找表被称为彩虹表。

因此，出于这个原因，我们将继续寻找更强的密码解决方案。多年来，已经采用了几种不同的密码技术来实现PSI解决方案。第一类算法使用公钥加密来保护交换，以便只有客户端能够解密匹配元素并发现交集。这种方法在客户端和服务器之间所需的带宽效率非常高，但计算交集的运行时间较长。

通用安全计算电路也已应用于PSI问题，无意识传输技术也是如此。最近，完全同态加密方案被提出，以实现近似和精确匹配。

对于本书的目的，我们将考虑由 Catherine Meadows 在 1986 年提出的原始公钥技术，使用 *椭圆曲线 Diffie-Hellman*（ECDH）协议。^([1](ch10.html#id612)) 我们不会深入探讨加密和解密过程的细节或数学。如果您想更详细地了解这个主题，我推荐阅读 Phillip J. Windley 的 *Learning Digital Identity*（O’Reilly）作为一个很好的入门书。

# 基于 ECDH 的 PSI 协议

基本的 PSI 协议工作方式如下：

1.  客户端使用可交换的加密方案和自己的秘钥对数据元素进行加密。

1.  客户端将它们的加密元素发送给服务器。这向服务器透露了客户端数据集中不同元素的数量，但不透露其他任何信息。

1.  然后服务器进一步使用新的与此请求唯一的秘钥对客户端加密的值进行加密，并将这些值发送回客户端。

1.  然后客户端利用加密方案的可交换属性，允许其解密从服务器接收的所有服务器元素，有效地去除其应用的原始加密，但保留服务器秘钥加密的元素。

1.  服务器使用为此请求创建的相同方案和秘钥加密其数据集中的所有元素，并将加密值发送给客户端。

1.  然后客户端可以比较在第 5 步接收到的完整的服务器加密元素与自己的集合成员，现在仅由第 4 步的服务器秘钥加密，以确定交集。

此协议显示在 [Figure 10-2](#fig-10-2) 中。

在其基本形式中，此协议意味着对于每个客户端查询，整个服务器数据集以加密形式发送给客户端。这些数据量可能是禁止的，无论是计算还是空间需求。但是，我们可以使用编码技术大幅减少我们需要交换的数据量，以较小的误报率为代价。我们将考虑两种技术：Bloom filters 和 Golomb-coded sets (GCSs)。提供了用于说明编码过程的简单示例 *Chapter10GCSBloomExamples.ipynb*。

![](assets/hoer_1002.png)

###### Figure 10-2\. PSI 协议

## 布隆过滤器

*Bloom filters* 是一种能够非常高效地存储和确认数据元素是否存在于集合中的概率数据结构。一个空的 Bloom 过滤器是一个位数组，其位被初始化为 0。要向过滤器添加项目，需要通过多个哈希函数处理数据元素；每个哈希函数的输出映射到过滤器中的一个位位置，然后将该位置设置为 1。

要测试新数据元素是否在集合中，我们只需检查其哈希值所对应的位位置是否全部设为1。如果是，则新元素可能已经存在于集合中。我说“可能”，因为这些位可能独立设置来表示其他值，导致假阳性。但可以确定的是，如果任何位不是设为1，则我们的新元素不在集合中；即，没有假阴性。

假阳性的可能性取决于过滤器的长度、哈希函数的数量以及数据集中的元素数量。可以优化如下：

<math alttext="upper B l o o m f i l t e r l e n g t h left-parenthesis b i t s right-parenthesis equals left ceiling StartFraction minus m a x normal bar e l e m e n t s times log Subscript 2 Baseline left-parenthesis f p r right-parenthesis Over 8 times ln 2 EndFraction right ceiling times 8"><mrow><mi>B</mi> <mi>l</mi> <mi>o</mi> <mi>o</mi> <mi>m</mi> <mi>f</mi> <mi>i</mi> <mi>l</mi> <mi>t</mi> <mi>e</mi> <mi>r</mi> <mi>l</mi> <mi>e</mi> <mi>n</mi> <mi>g</mi> <mi>t</mi> <mi>h</mi> <mrow><mo>(</mo> <mi>b</mi> <mi>i</mi> <mi>t</mi> <mi>s</mi> <mo>)</mo></mrow> <mo>=</mo> <mrow><mo>⌈</mo> <mfrac><mrow><mo>-</mo><mi>m</mi><mi>a</mi><mi>x</mi><mo>_</mo><mi>e</mi><mi>l</mi><mi>e</mi><mi>m</mi><mi>e</mi><mi>n</mi><mi>t</mi><mi>s</mi><mo>×</mo><msub><mo form="prefix">log</mo> <mn>2</mn></msub> <mrow><mo>(</mo><mi>f</mi><mi>p</mi><mi>r</mi><mo>)</mo></mrow></mrow> <mrow><mn>8</mn><mo>×</mo><mo form="prefix">ln</mo><mn>2</mn></mrow></mfrac> <mo>⌉</mo></mrow> <mo>×</mo> <mn>8</mn></mrow></math>

其中

<math alttext="f p r equals f a l s e p o s i t i v e r a t e"><mrow><mi>f</mi> <mi>p</mi> <mi>r</mi> <mo>=</mo> <mi>f</mi> <mi>a</mi> <mi>l</mi> <mi>s</mi> <mi>e</mi> <mi>p</mi> <mi>o</mi> <mi>s</mi> <mi>i</mi> <mi>t</mi> <mi>i</mi> <mi>v</mi> <mi>e</mi> <mi>r</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi></mrow></math>

<math alttext="m a x normal bar e l e m e n t s equals max left-parenthesis n u m normal bar c l i e n t normal bar i n p u t s comma n u m normal bar s e r v e r normal bar i n p u t s right-parenthesis"><mrow><mi>m</mi> <mi>a</mi> <mi>x</mi> <mo>_</mo> <mi>e</mi> <mi>l</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <mi>t</mi> <mi>s</mi> <mo>=</mo> <mo form="prefix" movablelimits="true">max</mo> <mo>(</mo> <mi>n</mi> <mi>u</mi> <mi>m</mi> <mo>_</mo> <mi>c</mi> <mi>l</mi> <mi>i</mi> <mi>e</mi> <mi>n</mi> <mi>t</mi> <mo>_</mo> <mi>i</mi> <mi>n</mi> <mi>p</mi> <mi>u</mi> <mi>t</mi> <mi>s</mi> <mo>,</mo> <mi>n</mi> <mi>u</mi> <mi>m</mi> <mo>_</mo> <mi>s</mi> <mi>e</mi> <mi>r</mi> <mi>v</mi> <mi>e</mi> <mi>r</mi> <mo>_</mo> <mi>i</mi> <mi>n</mi> <mi>p</mi> <mi>u</mi> <mi>t</mi> <mi>s</mi> <mo>)</mo></mrow></math>

和

<math alttext="upper N u m b e r h a s h f u n c t i o n s equals left ceiling minus log Subscript 2 Baseline left-parenthesis f p r right-parenthesis right ceiling"><mrow><mi>N</mi> <mi>u</mi> <mi>m</mi> <mi>b</mi> <mi>e</mi> <mi>r</mi> <mi>h</mi> <mi>a</mi> <mi>s</mi> <mi>h</mi> <mi>f</mi> <mi>u</mi> <mi>n</mi> <mi>c</mi> <mi>t</mi> <mi>i</mi> <mi>o</mi> <mi>n</mi> <mi>s</mi> <mo>=</mo> <mo>⌈</mo> <mo>-</mo> <msub><mo form="prefix">log</mo> <mn>2</mn></msub> <mrow><mo>(</mo> <mi>f</mi> <mi>p</mi> <mi>r</mi> <mo>)</mo></mrow> <mo>⌉</mo></mrow></math>

使用布隆过滤器来编码并返回加密服务器值，而不是返回完整的原始加密值集合，使得客户端可以将这些集合中的元素应用相同的布隆编码过程来检查是否存在。

### 布隆过滤器示例

让我们逐步构建一个简单的布隆过滤器来说明这个过程。

假设我们逐步向长度为 32 位的布隆过滤器中使用 4 次哈希迭代添加十进制值 217、354 和 466。假设哈希迭代按照以下方式计算：

<math alttext="upper H a s h Baseline 1 equals upper S upper H upper A Baseline 256 left-parenthesis upper E n c r y p t e d v a l u e p r e f i x e d b y 1 right-parenthesis percent-sign 32"><mrow><mi>H</mi> <mi>a</mi> <mi>s</mi> <mi>h</mi> <mn>1</mn> <mo>=</mo> <mi>S</mi> <mi>H</mi> <mi>A</mi> <mn>256</mn> <mo>(</mo> <mi>E</mi> <mi>n</mi> <mi>c</mi> <mi>r</mi> <mi>y</mi> <mi>p</mi> <mi>t</mi> <mi>e</mi> <mi>d</mi> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <mi>e</mi> <mi>p</mi> <mi>r</mi> <mi>e</mi> <mi>f</mi> <mi>i</mi> <mi>x</mi> <mi>e</mi> <mi>d</mi> <mi>b</mi> <mi>y</mi> <mn>1</mn> <mo>)</mo> <mo>%</mo> <mn>32</mn></mrow></math>

<math alttext="upper H a s h Baseline 2 equals upper S upper H upper A Baseline 256 left-parenthesis upper E n c r y p t e d v a l u e p r e f i x e d b y 2 right-parenthesis percent-sign 32"><mrow><mi>H</mi> <mi>a</mi> <mi>s</mi> <mi>h</mi> <mn>2</mn> <mo>=</mo> <mi>S</mi> <mi>H</mi> <mi>A</mi> <mn>256</mn> <mo>(</mo> <mi>E</mi> <mi>n</mi> <mi>c</mi> <mi>r</mi> <mi>y</mi> <mi>p</mi> <mi>t</mi> <mi>e</mi> <mi>d</mi> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <mi>e</mi> <mi>p</mi> <mi>r</mi> <mi>e</mi> <mi>f</mi> <mi>i</mi> <mi>x</mi> <mi>e</mi> <mi>d</mi> <mi>b</mi> <mi>y</mi> <mn>2</mn> <mo>)</mo> <mo>%</mo> <mn>32</mn></mrow></math>

和

<math alttext="upper H a s h upper V a l u e equals left-parenthesis upper H a s h Baseline 1 plus upper I t e r a t i o n n u m b e r times upper H a s h Baseline 2 right-parenthesis percent-sign 32"><mrow><mi>H</mi> <mi>a</mi> <mi>s</mi> <mi>h</mi> <mi>V</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <mi>e</mi> <mo>=</mo> <mo>(</mo> <mi>H</mi> <mi>a</mi> <mi>s</mi> <mi>h</mi> <mn>1</mn> <mo>+</mo> <mi>I</mi> <mi>t</mi> <mi>e</mi> <mi>r</mi> <mi>a</mi> <mi>t</mi> <mi>i</mi> <mi>o</mi> <mi>n</mi> <mi>n</mi> <mi>u</mi> <mi>m</mi> <mi>b</mi> <mi>e</mi> <mi>r</mi> <mo>×</mo> <mi>H</mi> <mi>a</mi> <mi>s</mi> <mi>h</mi> <mn>2</mn> <mo>)</mo> <mo>%</mo> <mn>32</mn></mrow></math>

然后我们逐步在[表 10-1](#table-10-1)中构建布隆过滤器。

表 10-1\. 布隆过滤器示例

| 加密值 | 哈希迭代 | 哈希值（范围

0-31) | 布隆过滤器（位置 0–31，从右到左） |

| --- | --- | --- | --- |
| --- | --- | --- | --- |
| 空过滤器 | 00000000000000000000000000000000 |
| 217 | 0 | 24 | 0000000**1**000000000000000000000000 |
| 1 | 19 | 000000010000**1**0000000000000000000 |
| 2 | ***14*** | 00000001000010000**1**00000000000000 |
| 3 | 9 | 0000000100001000010000**1**000000000 |
| 354 | 0 | 5 | 00000001000010000100001000**1**00000 |
| 1 | 4 | 000000010000100001000010001**1**0000 |
| 2 | 3 | 0000000100001000010000100011**1**000 |
| 3 | 2 | 00000001000010000100001000111**1**00 |
| 466 | 0 | ***14*** | 00000001000010000**1**00001000111100 |
| 1 | 18 | 0000000100001**1**000100001000111100 |
| 2 | 22 | 000000010**1**0011000100001000111100 |
| 3 | 26 | 00000**1**01010011000100001000111100 |
| 完成的过滤器 | 00000101010011000100001000111100 |

这里我们可以看到一种碰撞，即第三个值的第一个哈希迭代将位置 14 的位设置为1，尽管它已经被第一个值的第三次迭代先前设置为1。

类似地，如果对于新值的哈希迭代所对应的所有位位置已经设为1，则会误以为元素已经存在于数据集中，而实际上并非如此。例如，如果我们想测试值十进制 14 是否在服务器数据集中，我们计算其哈希值如[表 10-2](#table-10-2)所示。

表 10-2\. 布隆过滤器测试

| 测试值 | 哈希迭代 | 哈希值（范围

0–31) | 布隆过滤器（位置0–31，从右到左） | 位检查 |

| --- | --- | --- | --- | --- |
| --- | --- | --- | --- | --- |
| 布隆过滤器 | 00000**1**010**1**0011000**1**00001000111**1**00 |   |
| 14 | 0 | 22 | 000000000**1**0000000000000000000000 | True |
| 1 | 2 | 00000000000000000000000000000**1**00 | True |
| 2 | *14* | 00000000000000000**1**00000000000000 | True |
| 3 | 26 | 00000**1**00000000000000000000000000 | True |

从这个简单的例子中，我们错误地得出结论，即服务器数据中存在值14，实际并非如此。显然，需要更长的布隆过滤器长度。

## Golomb编码集

*Golomb编码集*（GCS），像布隆过滤器一样，是一种能够提供数据集中元素存在更高效方式的概率性数据结构。为了构建数据集的GCS表示，我们首先将原始数据元素哈希成一组在设定范围内的哈希值。

哈希范围计算如下：

<math alttext="StartFraction upper H a s h r a n g e equals m a x normal bar e l e m e n t s Over f p r EndFraction"><mfrac><mrow><mi>H</mi><mi>a</mi><mi>s</mi><mi>h</mi><mi>r</mi><mi>a</mi><mi>n</mi><mi>g</mi><mi>e</mi><mo>=</mo><mi>m</mi><mi>a</mi><mi>x</mi><mo>_</mo><mi>e</mi><mi>l</mi><mi>e</mi><mi>m</mi><mi>e</mi><mi>n</mi><mi>t</mi><mi>s</mi></mrow> <mrow><mi>f</mi><mi>p</mi><mi>r</mi></mrow></mfrac></math>

与以前一样：

<math alttext="f p r equals f a l s e p o s i t i v e r a t e"><mrow><mi>f</mi> <mi>p</mi> <mi>r</mi> <mo>=</mo> <mi>f</mi> <mi>a</mi> <mi>l</mi> <mi>s</mi> <mi>e</mi> <mi>p</mi> <mi>o</mi> <mi>s</mi> <mi>i</mi> <mi>t</mi> <mi>i</mi> <mi>v</mi> <mi>e</mi> <mi>r</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi></mrow></math>

<math alttext="m a x normal bar e l e m e n t s equals max left-parenthesis n u m normal bar c l i e n t normal bar i n p u t s comma n u m normal bar s e r v e r normal bar i n p u t s right-parenthesis"><mrow><mi>m</mi> <mi>a</mi> <mi>x</mi> <mo>_</mo> <mi>e</mi> <mi>l</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <mi>t</mi> <mi>s</mi> <mo>=</mo> <mo form="prefix" movablelimits="true">max</mo> <mo>(</mo> <mi>n</mi> <mi>u</mi> <mi>m</mi> <mo>_</mo> <mi>c</mi> <mi>l</mi> <mi>i</mi> <mi>e</mi> <mi>n</mi> <mi>t</mi> <mo>_</mo> <mi>i</mi> <mi>n</mi> <mi>p</mi> <mi>u</mi> <mi>t</mi> <mi>s</mi> <mo>,</mo> <mi>n</mi> <mi>u</mi> <mi>m</mi> <mo>_</mo> <mi>s</mi> <mi>e</mi> <mi>r</mi> <mi>v</mi> <mi>e</mi> <mi>r</mi> <mo>_</mo> <mi>i</mi> <mi>n</mi> <mi>p</mi> <mi>u</mi> <mi>t</mi> <mi>s</mi> <mo>)</mo></mrow></math>

然后，我们按升序排序这些哈希数值并计算代表几何值范围的除数。如果选择的除数是2的幂，则称此变体为Rice编码，并可从升序列表计算如下：

<math alttext="upper G upper C upper S normal bar d i v i s o r normal bar p o w e r normal bar o f normal bar 2 equals max left-parenthesis 0 comma r o u n d left-parenthesis minus l o g 2 left-parenthesis minus l o g 2 left-parenthesis 1.0 minus p r o b right-parenthesis right-parenthesis right-parenthesis"><mrow><mi>G</mi> <mi>C</mi> <mi>S</mi> <mo>_</mo> <mi>d</mi> <mi>i</mi> <mi>v</mi> <mi>i</mi> <mi>s</mi> <mi>o</mi> <mi>r</mi> <mo>_</mo> <mi>p</mi> <mi>o</mi> <mi>w</mi> <mi>e</mi> <mi>r</mi> <mo>_</mo> <mi>o</mi> <mi>f</mi> <mo>_</mo> <mn>2</mn> <mo>=</mo> <mo form="prefix" movablelimits="true">max</mo> <mo>(</mo> <mn>0</mn> <mo>,</mo> <mi>r</mi> <mi>o</mi> <mi>u</mi> <mi>n</mi> <mi>d</mi> <mrow><mo>(</mo> <mo>-</mo> <mi>l</mi> <mi>o</mi> <msub><mi>g</mi> <mn>2</mn></msub> <mrow><mo>(</mo> <mo>-</mo> <mi>l</mi> <mi>o</mi> <msub><mi>g</mi> <mn>2</mn></msub> <mrow><mo>(</mo> <mn>1</mn> <mo>.</mo> <mn>0</mn> <mo>-</mo> <mi>p</mi> <mi>r</mi> <mi>o</mi> <mi>b</mi> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>)</mo></mrow></mrow></math>

其中

<math alttext="p r o b equals StartFraction 1 Over a v g EndFraction"><mrow><mi>p</mi> <mi>r</mi> <mi>o</mi> <mi>b</mi> <mo>=</mo> <mfrac><mn>1</mn> <mrow><mi>a</mi><mi>v</mi><mi>g</mi></mrow></mfrac></mrow></math>

<math alttext="a v g equals StartFraction left-parenthesis l a s t normal bar e l e m e n t normal bar i n normal bar a s c e n d i n g normal bar l i s t plus 1 right-parenthesis Over n u m b e r normal bar e l e m e n t s normal bar i n normal bar a s c e n d i n g normal bar l i s t EndFraction"><mrow><mi>a</mi> <mi>v</mi> <mi>g</mi> <mo>=</mo> <mfrac><mrow><mo>(</mo><mi>l</mi><mi>a</mi><mi>s</mi><mi>t</mi><mo>_</mo><mi>e</mi><mi>l</mi><mi>e</mi><mi>m</mi><mi>e</mi><mi>n</mi><mi>t</mi><mo>_</mo><mi>i</mi><mi>n</mi><mo>_</mo><mi>a</mi><mi>s</mi><mi>c</mi><mi>e</mi><mi>n</mi><mi>d</mi><mi>i</mi><mi>n</mi><mi>g</mi><mo>_</mo><mi>l</mi><mi>i</mi><mi>s</mi><mi>t</mi><mo>+</mo><mn>1</mn><mo>)</mo></mrow> <mrow><mi>n</mi><mi>u</mi><mi>m</mi><mi>b</mi><mi>e</mi><mi>r</mi><mo>_</mo><mi>e</mi><mi>l</mi><mi>e</mi><mi>m</mi><mi>e</mi><mi>n</mi><mi>t</mi><mi>s</mi><mo>_</mo><mi>i</mi><mi>n</mi><mo>_</mo><mi>a</mi><mi>s</mi><mi>c</mi><mi>e</mi><mi>n</mi><mi>d</mi><mi>i</mi><mi>n</mi><mi>g</mi><mo>_</mo><mi>l</mi><mi>i</mi><mi>s</mi><mi>t</mi></mrow></mfrac></mrow></math>

接下来，我们计算连续值之间的差异，移除任何值为0的差异，并将这些增量哈希值除以先前计算的GCS除数的2次方。这种除法产生商和余数。为了完成编码，我们使用一元编码表示商，使用二进制表示余数，并使用0填充到最大长度。

每个元素都以这种方式编码，并将位连接在一起形成GCS结构。要在结构中检查给定元素，我们通过位扫描来逐个重建每个元素，从而逐步累加我们获取的差值，以重建我们可以与测试值哈希进行比较的原始哈希值。

与布隆过滤器一样，由于哈希碰撞的可能性，存在误报的可能性，其概率取决于哈希范围的大小和要编码的元素数量。再次强调，不存在误报。

让我们考虑一个简短的例子。

### GCS示例

从相同的加密数值217、354和466以及十进制128的哈希范围开始。我们计算这些数值的SHA256哈希（作为字节），然后除以哈希范围以获得介于0和127之间的余数。这给出了在[表10-4](#table-10-4)中显示的数值。

表10-4\. GCS哈希值计算

| 加密数值 | 加密数值的SHA256哈希（十六进制） | 哈希值范围0-127 |
| --- | --- | --- |
| 217 | 16badfc6202cb3f8889e0f2779b19218af4cbb736e56acadce8148aba9a7a9f8 | 120 |
| 354 | 09a1b036b82baba3177d83c27c1f7d0beacaac6de1c5fdcc9680c49f638c5fb9 | 57 |
| 466 | 826e27285307a923759de350de081d6218a04f4cff82b20c5ddaa8c60138c066 | 102 |

将减少范围哈希值按升序排序，我们有57、102和120。因此，增量值为57（57–0）、45（102–57）和18（120–102）。

我们计算的除数幂为：

<math alttext="a v g equals StartFraction 120 plus 1 Over 3 EndFraction almost-equals 40.33"><mrow><mi>a</mi> <mi>v</mi> <mi>g</mi> <mo>=</mo> <mfrac><mrow><mn>120</mn><mo>+</mo><mn>1</mn></mrow> <mn>3</mn></mfrac> <mo>≈</mo> <mn>40</mn> <mo>.</mo> <mn>33</mn></mrow></math>

<math alttext="p r o b equals StartFraction 1 Over 40.33 EndFraction almost-equals 0.02479"><mrow><mi>p</mi> <mi>r</mi> <mi>o</mi> <mi>b</mi> <mo>=</mo> <mfrac><mn>1</mn> <mrow><mn>40</mn><mo>.</mo><mn>33</mn></mrow></mfrac> <mo>≈</mo> <mn>0</mn> <mo>.</mo> <mn>02479</mn></mrow></math>

<math alttext="upper G upper C upper S normal bar d i v i s o r normal bar p o w e r normal bar o f normal bar 2 equals max left-parenthesis 0 comma r o u n d left-parenthesis minus l o g 2 left-parenthesis minus l o g 2 left-parenthesis 1.0 minus 0.02479 right-parenthesis right-parenthesis right-parenthesis equals 5"><mrow><mi>G</mi> <mi>C</mi> <mi>S</mi> <mo>_</mo> <mi>d</mi> <mi>i</mi> <mi>v</mi> <mi>i</mi> <mi>s</mi> <mi>o</mi> <mi>r</mi> <mo>_</mo> <mi>p</mi> <mi>o</mi> <mi>w</mi> <mi>e</mi> <mi>r</mi> <mo>_</mo> <mi>o</mi> <mi>f</mi> <mo>_</mo> <mn>2</mn> <mo>=</mo> <mo form="prefix" movablelimits="true">max</mo> <mo>(</mo> <mn>0</mn> <mo>,</mo> <mi>r</mi> <mi>o</mi> <mi>u</mi> <mi>n</mi> <mi>d</mi> <mrow><mo>(</mo> <mo>-</mo> <mi>l</mi> <mi>o</mi> <msub><mi>g</mi> <mn>2</mn></msub> <mrow><mo>(</mo> <mo>-</mo> <mi>l</mi> <mi>o</mi> <msub><mi>g</mi> <mn>2</mn></msub> <mrow><mo>(</mo> <mn>1</mn> <mo>.</mo> <mn>0</mn> <mo>-</mo> <mn>0</mn> <mo>.</mo> <mn>02479</mn> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>=</mo> <mn>5</mn></mrow></math>

使用除数参数32（<math alttext="2 Superscript 5"><msup><mn>2</mn> <mn>5</mn></msup></math>），我们可以将这些值编码如[表格10-5](#table-10-5)所示。

表格10-5\. GCS二进制和一进制编码

| Delta hash value

范围0–127 | 商（/32） | 余数（%32） | 余数（二进制5位） | 一元商（从右到左带0的） | GCS编码 |

| --- | --- | --- | --- | --- | --- |
| --- | --- | --- | --- | --- | --- |
| 57 | 1  | 25 | 11001 | 10 | 1100110 |
| 45 | 1 | 13 | 01101 | 10 | 0110110 |
| 18 | 0 | 18 | 10010 | 1 | 100101 |

最后到第一个，从左到右，集合编码为：10010101101101100110。

# 示例：使用PSI过程

现在我们理解了基本的PSI过程，让我们将其应用到识别英国MCA列表和Companies House注册中存在的公司的挑战上。如果我们将MCA视为客户方，Companies House视为服务器方，那么我们可以研究如何找到那些MCA公司在Companies House注册中存在而不向Companies House透露MCA列表的内容。

*请注意，此示例仅供说明目的。*

## 环境设置

由于PSI过程需要大量计算资源，我们将使用Google Cloud暂时提供基础设施来运行这个例子。

在[第6章](ch06.html#chapter_6)中，我们对MCA和Companies House注册数据集进行了标准化，并将它们保存为经过标准化和清理的CSV文件。在[第7章](ch07.html#chapter_7)中，我们将这些文件上传到了Google Cloud Storage存储桶中。在本章中，我们将假设这些数据集由两个独立的方当中持有。

我们将把这些文件传输到一个单一的数据科学工作台实例上，在该实例上我们将运行服务器和客户端以演示交集过程。这个例子可以轻松扩展到在两台不同的机器上运行，以展示服务器和客户端角色的分离以及数据的分离。

### Google Cloud设置

要开始，我们在Google Cloud控制台的AI平台菜单中选择工作台。为了创建环境，我们选择用户管理的笔记本（而不是托管笔记本），因为这个选项将允许我们安装我们需要的包。

第一步是选择创建新的。从这里，我们可以将笔记本重命名为我们选择的名称。在环境部分，选择基本的Python3选项，然后点击创建。如同[第7章](ch07.html#chapter_7)中一样，如果您希望或接受默认设置，可以更改区域和区域设置。如果（可选）选择“IAM和安全性”，您将注意到将授予虚拟机的根访问权限。

# 成本

请注意，一旦创建了新的Workbench环境，您将开始产生费用，无论实例是否正在运行，磁盘空间成本都会产生费用，即使实例停止时也是如此。默认情况下，Workbench实例会创建2个100 GB的磁盘！

*您有责任确保停止或删除实例，以避免产生意外费用。*

创建了您的实例后，您将能够点击“打开JupyterLab”以打开一个本地窗口，访问托管在新的GCP Workbench上的JupyterLab环境。^([2](ch10.html#id626)) 从这里，我们可以在“其他”下选择“终端”以打开终端窗口来配置我们的环境。

我们将要使用的PSI包由OpenMined社区发布和分发。

# OpenMined

OpenMined是一个开源社区，旨在通过降低进入私人AI技术的门槛来使世界更加隐私保护。他们的PSI存储库提供了基于ECDH和Bloom过滤器的私人集交集基数协议。

在撰写本文时，OpenMined PSI包可以在[线上](https://oreil.ly/XaKJs)获取。从该网站我们可以下载与Google Cloud Workbench兼容的预构建发行版（当前是运行Debian 11操作系统的x86 64位虚拟机），我们可以方便地安装（选项1）。或者，如果您更喜欢使用不同的环境或自行构建该包，则可以选择（选项2）。

### 选项1：预构建PSI包

创建一个PSI目录并切换到该位置：

```py
>>>mkdir psi

>>>cd psi
```

复制兼容Python发行版的链接地址，并使用wget进行下载。目前的链接是：

```py
>>>wget https://files.pythonhosted.org/packages/2b/ac/
   a62c753f91139597b2baf6fb3207d29bd98a6cf01da918660c8d58a756e8/
   openmined.psi-2.0.1-cp310-cp310-manylinux_2_31_x86_64.whl
```

安装该软件包如下：

```py
>>>pip install openmined.psi-2.0.1-cp310-cp310-
    manylinux_2_31_x86_64.whl
```

### 选项2：构建PSI包

在终端提示符下，我们克隆OpenMined psi包的存储库：

```py
>>>git clone http://github.com/openmined/psi
```

然后切换到psi目录：

```py
>>>cd psi
```

要从存储库源代码构建psi包，我们需要安装适当版本的构建包Bazel。使用wget获取GitHub存储库中适当的预构建Debian发行版软件包：

```py
>>>wget https://github.com/bazelbuild/bazel/releases/download/
    6.0.0/bazel_6.0.0-linux-x86_64.deb
```

以root身份安装此软件包：

```py
>>>sudo dpkg -i *.deb
```

接下来，我们使用Bazel构建Python发行版，即一个wheel文件，具备必要的依赖项。这一步可能需要几分钟：

```py
>>>bazel build -c opt //private_set_intersection/python:wheel
```

一旦我们构建了wheel归档文件，我们可以使用OpenMined提供的Python工具将文件重命名以反映它支持的环境：

```py
>>>python ./private_set_intersection/python/rename.py
```

重命名实用程序将输出重命名文件的路径和名称。我们现在需要从提供的路径安装这个新命名的包，例如：

```py
>>>pip install ./bazel-bin/private_set_intersection/python/
   openmined.psi-2.0.1-cp310-cp310-manylinux_2_31_x86_64.whl
```

再次强调，此安装可能需要几分钟，但一旦完成，我们就拥有了执行样本问题数据上PSI所需的基本组件。

### 服务器安装

一旦我们安装了psi包，我们还需要一个基本的客户端/服务器框架来处理匹配请求。为此，我们使用Flask轻量级微框架，可以使用pip安装：

```py
>>>pip install flask
```

安装完成后，我们可以从psi目录上级导航，以便复制我们的示例文件：

```py
>>>cd ..

>>>gsutil cp gs://<your bucket>/<your path>/Chapter10* . 
>>>gsutil cp gs://<your bucket>/<your path>/mari_clean.csv .
>>>gsutil cp gs://<your bucket>/<your path>/basic_clean.csv .
```

要启动flask服务器并运行`Chapter10Server` Python脚本，我们可以在终端选项卡提示符中使用以下命令：

```py
>>>flask --app Chapter10Server run --host 0.0.0.0
```

服务器启动需要一些时间，因为它正在读取Companies House数据集并将实体组装成一系列连接的`CompanyName`和`Postcode`字符串。

一旦准备好处理请求，它将在命令提示符下显示以下内容：

```py
* Serving Flask app 'Chapter10Server'
...
* Running on http://127.0.0.1:5000
PRESS CTRL+C to quit
```

## 服务器代码

让我们通过打开Python文件*Chapter10Server.py*来查看服务器代码：

```py
import private_set_intersection.python as psi
from flask import Flask, request
from pandas import read_csv

fpr = 0.01
num_client_inputs = 100

df_m = read_csv('basic_clean.csv',keep_default_na=False)
server_items = ['ABLY RESOURCES G2 1PB','ADVANCE GLOBAL RECRUITMENT EH7 4HG']
#server_items = (df_m['CompanyName']+' '+ df_m['Postcode']).to_list()

app = Flask(__name__)
```

我们首先导入安装的PSI包，然后是我们需要的`flask`和`pandas`函数。

接下来，我们设置所需的误报率（`fpr`）和每个请求中将检查的客户端输入数。这些参数一起用于计算在GCS编码中使用的Bloom过滤器和哈希范围的长度。

然后，我们读取我们之前从云存储桶传输的经过清理的Companies House记录，指定忽略空值。然后，我们通过连接每个`CompanyName`和`Postcode`值（用空格分隔）来创建服务器项目列表。这使我们能够检查每个实体的精确名称和邮政编码匹配。

为了允许我们详细检查编码协议，我从MCA列表中选择了两个实体，并手动创建了它们的经过清理的名称和邮政编码字符串，作为服务器项目的另一组替代集。*要使用完整的Companies House数据集，只需从列表创建语句的注释标记（前导`#`）中删除以覆盖`server_items`：

```py
#server_items = (df_m['CompanyName']+' '+ df_m['Postcode']).to_list()
```

服务器文件的其余部分定义了一个保存服务器密钥的类，然后创建了`key`对象：

```py
class psikey(object):
   def __init__(self):
      self.key = None
   def set_key(self, newkey):
      self.key = newkey
      return self.key
   def get_key(self):
      return self.key

pkey = psikey()
```

Flask Web应用程序允许我们响应GET和POST请求。

服务器响应`/match`路径的POST请求时，会创建新的服务器密钥和一个`psirequest`对象。然后我们解析POST请求中的数据，使用新密钥处理（即加密）接收到的数据，然后在返回给客户端之前对这些处理后的值进行序列化。

```py
@app.route('/match', methods=['POST'])
def match():
   s = pkey.set_key(psi.server.CreateWithNewKey(True))
   psirequest = psi.Request()
   psirequest.ParseFromString(request.data)
   return s.ProcessRequest(psirequest).SerializeToString()
```

处理完匹配请求后，服务器可以响应客户端对不同编码方案的GET请求：原始加密值、Bloom过滤器和GCS。在每种情况下，我们重用在匹配请求期间创建的密钥，并提供所需的误报率和每个客户端请求中的项目数，以便我们可以配置Bloom和GCS选项。

```py
@app.route('/gcssetup', methods=['GET'])
def gcssetup():
   s = pkey.get_key()
   return s.CreateSetupMessage(fpr, num_client_inputs, server_items,
      psi.DataStructure.GCS).SerializeToString()

@app.route('/rawsetup', methods=['GET'])
def rawsetup():
   s = pkey.get_key()
   return s.CreateSetupMessage(fpr, num_client_inputs, server_items,
      psi.DataStructure.RAW).SerializeToString()

@app.route('/bloomsetup', methods=['GET'])
def bloomsetup():
   s = pkey.get_key()
   return s.CreateSetupMessage(fpr, num_client_inputs, server_items,
      psi.DataStructure.BLOOM_FILTER).SerializeToString()
```

## 客户端代码

包含客户端代码的笔记本是*Chapter10Client.ipynb*，开始如下：

```py
import requests
import private_set_intersection.python as psi
from pandas import read_csv

url="http://localhost:5000/"
```

与服务器设置一样，我们读取经过清理的MCA公司详细信息，创建客户端密钥，加密，然后序列化以传输到服务器。

```py
df_m = read_csv('mari_clean.csv')
client_items = (df_m['CompanyName']+' '+df_m['Postcode']).to_list()
c = psi.client.CreateWithNewKey(True)
psirequest = c.CreateRequest(client_items).SerializeToString()

c.CreateRequest(client_items)
```

在序列化之前，`psirequest`的前几行如下所示：

```py
reveal_intersection: true
encrypted_elements: 
    "\002r\022JjD\303\210*\354\027\267aRId\2522\213\304\250%\005J\224\222m\354\
    207`\2136\306"
encrypted_elements: 
    "\002\005\352\245r\343n\325\277\026\026\355V\007P\260\313b\377\016\000{\336\
    343\033&\217o\210\263\255[\350"
```

我们将序列化的加密值作为消息内容包含在POST请求中，请求的路径为`/match`，并在头部指示我们传递的内容是一个protobuf结构。然后，服务器响应包含客户端加密值的服务器加密版本，并被解析为`response`对象：

```py
response = requests.post(url+'match',
    headers={'Content-Type': 'application/protobuf'}, data=psirequest)
psiresponse = psi.Response()
psiresponse.ParseFromString(response.content)
psiresponse
```

### 使用原始加密服务器值

要检索原始加密的服务器值，客户端发送请求到`/rawsetup` URL路径：

```py
setupresponse = requests.get(url+'rawsetup')
rawsetup = psi.ServerSetup()
rawsetup.ParseFromString(setupresponse.content)
rawsetup
```

如果我们选择仅在服务器设置文件中使用两个测试条目，则可以预期设置响应中仅有两个加密元素。值将取决于服务器密钥，但结构将类似于此：

```py
raw {
  encrypted_elements: 
      "\003>W.x+\354\310\246\302z\341\364%\255\202\354\021n\t\211\037\221\255\
      263\006\305NU\345.\243@"
  encrypted_elements: 
      "\003\304Q\373\224.\0348\025\3452\323\024\317l~\220\020\311A\257\002\
      014J0?\274$\031`N\035\277"
}
```

然后，我们可以计算`rawsetup`结构中的服务器值和`psiresponse`结构中的客户端值的交集：

```py
intersection = c.GetIntersection(gcssetup, psiresponse)
#intersection = c.GetIntersection(bloomsetup, psiresponse)
#intersection = c.GetIntersection(rawsetup, psiresponse)

iset = set(intersection)
sorted(intersection)
```

这给我们匹配实体的列表索引，在这个简单的情况中：

```py
[1, 2]
```

然后我们可以查找相应的客户端实体：

```py
for index in sorted(intersection):
   print(client_items[index])

ABLY RESOURCES G2 1PB
ADVANCE GLOBAL RECRUITMENT EH7 4HG
```

成功！我们已在客户端和服务器记录之间解析了这些实体，确切匹配`CompanyName`和`Postcode`属性，而不向服务器透露客户端项目。

### 使用布隆过滤器编码的加密服务器值

现在让我们看看如何使用布隆过滤器对编码的服务器加密值进行编码：

```py
setupresponse = requests.get(url+'bloomsetup')
bloomsetup = psi.ServerSetup()
bloomsetup.ParseFromString(setupresponse.content)
bloomsetup
```

如果我们通过`/bloomsetup`路径提交请求，我们会得到一个类似的输出：

```py
bloom_filter {
  num_hash_functions: 14
  bits: "\000\000\000\000 ...\000"
}
```

服务器根据布隆过滤器部分中的公式计算过滤器中的位数。我们可以重新创建如下：

```py
from math import ceil, log, log2

fpr = 0.01
num_client_inputs = 10

correctedfpr = fpr / num_client_inputs
len_server_items = 2

max_elements = max(num_client_inputs, len_server_items)
num_bits = (ceil(-max_elements * log2(correctedfpr) / log(2) /8))* 8
```

错误阳性率设置为每次查询100个客户端项目的100中的1，因此总体（修正后）的fpr为0.0001。对于我们非常基本的示例，`max_elements`也等于100。这给我们一个布隆过滤器位长度为1920：

```py
num_hash_functions = ceil(-log2(correctedfpr))
```

这给我们14个哈希函数。

我们可以通过处理原始加密服务器元素来复现布隆过滤器：

```py
from hashlib import sha256

#num_bits = len(bloomsetup.bloom_filter.bits)*8
filterlist = ['0'] * num_bits
for element in rawsetup.raw.encrypted_elements:
   element1 = str.encode('1') + element
   k = sha256(element1).hexdigest()
   h1 = int(k,16) % num_bits

   element2 = str.encode('2') + element
   k = sha256(element2).hexdigest()
   h2 = int(k,16) % num_bits

  for i in range(bloomsetup.bloom_filter.num_hash_functions):
      pos = ((h1 + i * h2) % num_bits)
      filterlist[num_bits-1-pos]='1'

filterstring = ''.join(filterlist)
```

然后，当以相同顺序组装并转换为字符串时，我们可以将我们的过滤器与服务器返回的过滤器比较：

```py
bloombits = ''.join(format(byte, '08b') for byte in
   reversed(bloomsetup.bloom_filter.bits))
bloombits == filterstring
```

### 使用GCS编码的加密服务器值

最后，让我们看看如何使用GCS对编码的服务器加密值进行编码。

```py
setupresponse = requests.get(url+'gcssetup')
gcssetup =
   psi.ServerSetup()gcssetup.ParseFromString(setupresponse.content)
```

如果我们通过`/gcssetup`路径提交请求，我们会得到一个类似的输出：

```py
gcs {
  div: 17
  hash_range: 1000000
  bits: ")![Q\026"
}
```

要复现这些值，我们可以应用上述PSI部分中的公式：

```py
from math import ceil, log, log2

fpr = 0.01
num_client_inputs = 100
correctedfpr = fpr/num_client_inputs

hash_range = max_elements/correctedfpr
hash_range
```

这给我们1000000的哈希范围。

与布隆过滤器类似，我们可以通过处理原始加密服务器元素来复现GCS结构。首先，我们将原始加密服务器值哈希到`gcs_hash_range`中，按升序排序，并计算差值：

```py
from hashlib import sha256

ulist = []
for element in rawsetup.raw.encrypted_elements:
   k = sha256(element).hexdigest()
   ks = int(k,16) % gcssetup.gcs.hash_range
   ulist.append(ks)

ulist.sort()
udiff = [ulist[0]] + [ulist[n]-ulist[n-1]
   for n in range(1,len(ulist))]
```

现在我们可以计算GCS除数如下：

```py
avg = (ulist[-1]+1)/len(ulist)
prob = 1/avg
gcsdiv = max(0,round(-log2(-log2(1.0-prob))))
```

这给我们一个除数为17，然后我们可以使用它来计算商和余数，分别在一元和二元中编码这些位模式。我们将这些位模式连接在一起：

```py
encoded = ''
for diff in udiff:
   if diff != 0:
      quot = int(diff / pow(2,gcssetup.gcs.div))
      rem = diff % pow(2,gcssetup.gcs.div)

      next = '{0:b}'.format(rem) + '1' + ('0' * quot)
      pad = next.zfill(quot+gcssetup.gcs.div+1)
      encoded = pad + encoded
```

最后，我们填充编码字符串，使其成为8的倍数，以便与返回的GCS比特匹配：

```py
from math import ceil
padlength = ceil(len(encoded)/8)*8
padded = encoded.zfill(padlength)

gcsbits = ''.join(format(byte, '08b') for byte in
   reversed(gcssetup.gcs.bits))
gcsbits == padded
```

## 完整的MCA和Companies House样本示例

现在，我们已经看到了使用仅包含两个项目的微小服务器数据集进行端到端PSI实体匹配过程的结尾，我们准备使用完整的Companies House数据集。

打开*Chapter10Server.py*文件并取消注释：

```py
#server_items = (df_m['CompanyName']+' '+
   df_m['Postcode']).to_list()
```

然后停止（Ctrl+C或Cmd-C）并重新启动Flask服务器：

```py
>>>flask --app Chapter10Server run --host 0.0.0.0
```

现在我们可以重新启动客户端内核并重新运行笔记本，以获取MCA和Companies House数据的完整交集，解析`CompanyName`和`Postcode`上的实体。

我们可以请求原始、Bloom或GCS响应。请允许服务器大约10分钟来处理并返回。*我建议您跳过重现Bloom/GCS结构的步骤，因为这可能需要很长时间*。

跳转计算交集然后给出我们45个精确匹配项：

```py
ADVANCE GLOBAL RECRUITMENT EH7 4HG
ADVANCED RESOURCE MANAGERS PO6 4PR
...

WORLDWIDE RECRUITMENT SOLUTIONS WA15 8AB
```

# 整理

记得停止并删除您的用户管理笔记本和任何关联磁盘，以避免持续收费！

本PSI示例展示了如何在两个当事方之间解析实体，即使其中一方无法与另一方共享其数据。在这个基本示例中，我们能够仅查找两个属性的同时精确匹配。

在某些情况下，精确匹配可能足够。但是，当需要近似匹配时，并且至少有一方准备分享部分匹配时，我们需要一种更复杂的方法，这超出了本书的范围。目前正在研究使用全同态加密实现隐私保护模糊匹配的实用性，这将为该技术开辟更广泛的潜在用例领域。

# 摘要

在本章中，我们学习了如何使用私有集合交集来解析两个当事方之间的实体，而双方都不需透露其完整数据集。我们看到了如何使用压缩数据表示来减少需要在两个当事方之间传递的数据量，尽管会引入少量误报。

我们注意到，这些技术可以轻松应用于精确匹配场景，但更高级的近似或概率匹配仍然是一个挑战，也是积极研究的课题。

^([1](ch10.html#id612-marker)) Meadows, Catherine，“用于在没有连续可用第三方的情况下使用的更有效的加密匹配协议”，*1986 IEEE安全与隐私研讨会*，美国加利福尼亚州奥克兰，1986年，第134页，*https://doi.org/10.1109/SP.1986.10022*。

^([2](ch10.html#id626-marker)) 您可能需要在本地浏览器上允许弹出窗口。

^([3](ch10.html#id637-marker)) 请参阅专利“使用全同态加密方案进行紧凑模糊私有匹配”，[*https://patents.google.com/patent/US20160119119*](https://patents.google.com/patent/US20160119119)。
