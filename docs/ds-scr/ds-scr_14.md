# 第十三章：贝叶斯朴素分类

> 为心灵保持天真，为思想保持成熟。
> 
> 阿纳托尔·法朗士

如果人们不能进行社交，那么社交网络就没什么用了。因此，DataSciencester 拥有一个受欢迎的功能，允许会员发送消息给其他会员。虽然大多数会员是负责任的公民，只发送受欢迎的“最近好吗？”消息，但一些不法分子坚持不懈地向其他成员发送关于致富计划、无需处方的药物和盈利数据科学证书项目的垃圾邮件。您的用户已经开始抱怨，因此消息副总裁要求您使用数据科学找出一种过滤这些垃圾邮件的方法。

# 一个非常愚蠢的垃圾邮件过滤器

假设一个“宇宙”，由从所有可能消息中随机选择的消息组成。设*S*为事件“消息是垃圾邮件”，*B*为事件“消息包含词*bitcoin*”。贝叶斯定理告诉我们，包含词*bitcoin*的消息是垃圾邮件的条件概率是：

<math alttext="upper P left-parenthesis upper S vertical-bar upper B right-parenthesis equals left-bracket upper P left-parenthesis upper B vertical-bar upper S right-parenthesis upper P left-parenthesis upper S right-parenthesis right-bracket slash left-bracket upper P left-parenthesis upper B vertical-bar upper S right-parenthesis upper P left-parenthesis upper S right-parenthesis plus upper P left-parenthesis upper B vertical-bar normal not-sign upper S right-parenthesis upper P left-parenthesis normal not-sign upper S right-parenthesis right-bracket" display="block"><mrow><mi>P</mi> <mo>(</mo> <mi>S</mi> <mo>|</mo> <mi>B</mi> <mo>)</mo> <mo>=</mo> <mo>[</mo> <mi>P</mi> <mo>(</mo> <mi>B</mi> <mo>|</mo> <mi>S</mi> <mo>)</mo> <mi>P</mi> <mo>(</mo> <mi>S</mi> <mo>)</mo> <mo>]</mo> <mo>/</mo> <mo>[</mo> <mi>P</mi> <mo>(</mo> <mi>B</mi> <mo>|</mo> <mi>S</mi> <mo>)</mo> <mi>P</mi> <mo>(</mo> <mi>S</mi> <mo>)</mo> <mo>+</mo> <mi>P</mi> <mo>(</mo> <mi>B</mi> <mo>|</mo> <mo>¬</mo> <mi>S</mi> <mo>)</mo> <mi>P</mi> <mo>(</mo> <mo>¬</mo> <mi>S</mi> <mo>)</mo> <mo>]</mo></mrow></math>

分子是消息是垃圾邮件且包含*bitcoin*的概率，而分母只是消息包含*bitcoin*的概率。因此，您可以将这个计算视为简单地表示为垃圾邮件的*bitcoin*消息的比例。

如果我们有大量的已知是垃圾邮件的消息和大量的已知不是垃圾邮件的消息，那么我们可以很容易地估计*P*(*B*|*S*)和*P*(*B*|*¬S*)。如果我们进一步假设任何消息都有相等的可能性是垃圾邮件或不是垃圾邮件（所以*P*(*S*) = *P*(*¬S*) = 0.5），那么：

<math alttext="left-bracket upper P left-parenthesis upper S vertical-bar upper B right-parenthesis equals upper P left-parenthesis upper B vertical-bar upper S right-parenthesis slash left-bracket upper P left-parenthesis upper B vertical-bar upper S right-parenthesis plus upper P left-parenthesis upper B vertical-bar normal not-sign upper S right-parenthesis right-bracket right-bracket" display="block"><mrow><mi>P</mi> <mo>(</mo> <mi>S</mi> <mo>|</mo> <mi>B</mi> <mo>)</mo> <mo>=</mo> <mi>P</mi> <mo>(</mo> <mi>B</mi> <mo>|</mo> <mi>S</mi> <mo>)</mo> <mo>/</mo> <mo>[</mo> <mi>P</mi> <mo>(</mo> <mi>B</mi> <mo>|</mo> <mi>S</mi> <mo>)</mo> <mo>+</mo> <mi>P</mi> <mo>(</mo> <mi>B</mi> <mo>|</mo> <mo>¬</mo> <mi>S</mi> <mo>)</mo> <mo>]</mo></mrow></math>

例如，如果垃圾邮件中有 50%的消息包含*bitcoin*这个词，而非垃圾邮件中只有 1%的消息包含，那么包含*bitcoin*的任意邮件是垃圾邮件的概率是：

<math alttext="0.5 slash left-parenthesis 0.5 plus 0.01 right-parenthesis equals 98 percent-sign" display="block"><mrow><mn>0</mn> <mo>.</mo> <mn>5</mn> <mo>/</mo> <mo>(</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>01</mn> <mo>)</mo> <mo>=</mo> <mn>98</mn> <mo>%</mo></mrow></math>

# 一个更复杂的垃圾邮件过滤器

现在想象我们有一个包含许多词*w*[1] ... *w*[n]的词汇表。为了将其转化为概率理论的领域，我们将*X*[i]写成事件“消息包含词*w*[i]”。还想象一下（通过某种未指定的过程），我们已经得出了一个估计值*P*(*X*[i]|*S*)，表示垃圾邮件消息包含第*i*个词的概率，以及类似的估计*P*(*X*[i]|¬*S*)，表示非垃圾邮件消息包含第*i*个词的概率。

贝叶斯方法的关键在于做出（大胆的）假设，即每个单词的存在（或不存在）在消息是垃圾邮件与否的条件下是独立的。直观地说，这个假设意味着知道某个垃圾邮件消息是否包含词*bitcoin*并不会告诉你同一消息是否包含词*rolex*。用数学术语来说，这意味着：

<math alttext="upper P left-parenthesis upper X 1 equals x 1 comma period period period comma upper X Subscript n Baseline equals x Subscript n Baseline vertical-bar upper S right-parenthesis equals upper P left-parenthesis upper X 1 equals x 1 vertical-bar upper S right-parenthesis times ellipsis times upper P left-parenthesis upper X Subscript n Baseline equals x Subscript n Baseline vertical-bar upper S right-parenthesis" display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> <msub><mi>X</mi> <mn>1</mn></msub> <mo>=</mo> <msub><mi>x</mi> <mn>1</mn></msub> <mo>,</mo> <mo>.</mo> <mo>.</mo> <mo>.</mo> <mo>,</mo> <msub><mi>X</mi> <mi>n</mi></msub> <mo>=</mo> <msub><mi>x</mi> <mi>n</mi></msub> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow> <mo>=</mo> <mi>P</mi> <mrow><mo>(</mo> <msub><mi>X</mi> <mn>1</mn></msub> <mo>=</mo> <msub><mi>x</mi> <mn>1</mn></msub> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow> <mo>×</mo> <mo>⋯</mo> <mo>×</mo> <mi>P</mi> <mrow><mo>(</mo> <msub><mi>X</mi> <mi>n</mi></msub> <mo>=</mo> <msub><mi>x</mi> <mi>n</mi></msub> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow></mrow></math>

这是一个极端的假设。（这个技术名称中带有*天真*也是有原因的。）假设我们的词汇表*仅仅*包括*比特币*和*劳力士*这两个词，而一半的垃圾邮件是关于“赚取比特币”，另一半是关于“正品劳力士”。在这种情况下，朴素贝叶斯估计一封垃圾邮件包含*比特币*和*劳力士*的概率是：

<math alttext="upper P left-parenthesis upper X 1 equals 1 comma upper X 2 equals 1 vertical-bar upper S right-parenthesis equals upper P left-parenthesis upper X 1 equals 1 vertical-bar upper S right-parenthesis upper P left-parenthesis upper X 2 equals 1 vertical-bar upper S right-parenthesis equals .5 times .5 equals .25" display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> <msub><mi>X</mi> <mn>1</mn></msub> <mo>=</mo> <mn>1</mn> <mo>,</mo> <msub><mi>X</mi> <mn>2</mn></msub> <mo>=</mo> <mn>1</mn> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow> <mo>=</mo> <mi>P</mi> <mrow><mo>(</mo> <msub><mi>X</mi> <mn>1</mn></msub> <mo>=</mo> <mn>1</mn> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow> <mi>P</mi> <mrow><mo>(</mo> <msub><mi>X</mi> <mn>2</mn></msub> <mo>=</mo> <mn>1</mn> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow> <mo>=</mo> <mo>.</mo> <mn>5</mn> <mo>×</mo> <mo>.</mo> <mn>5</mn> <mo>=</mo> <mo>.</mo> <mn>25</mn></mrow></math>

因为我们假设了*比特币*和*劳力士*实际上从不会同时出现的知识。尽管这种假设的不现实性，这种模型通常表现良好，并且在实际的垃圾邮件过滤器中历史悠久地使用。

我们用于“仅比特币”垃圾邮件过滤器的相同贝叶斯定理推理告诉我们可以使用以下方程来计算一封邮件是垃圾邮件的概率：

<math alttext="upper P left-parenthesis upper S vertical-bar upper X equals x right-parenthesis equals upper P left-parenthesis upper X equals x vertical-bar upper S right-parenthesis slash left-bracket upper P left-parenthesis upper X equals x vertical-bar upper S right-parenthesis plus upper P left-parenthesis upper X equals x vertical-bar normal not-sign upper S right-parenthesis right-bracket" display="block"><mrow><mi>P</mi> <mo>(</mo> <mi>S</mi> <mo>|</mo> <mi>X</mi> <mo>=</mo> <mi>x</mi> <mo>)</mo> <mo>=</mo> <mi>P</mi> <mo>(</mo> <mi>X</mi> <mo>=</mo> <mi>x</mi> <mo>|</mo> <mi>S</mi> <mo>)</mo> <mo>/</mo> <mo>[</mo> <mi>P</mi> <mo>(</mo> <mi>X</mi> <mo>=</mo> <mi>x</mi> <mo>|</mo> <mi>S</mi> <mo>)</mo> <mo>+</mo> <mi>P</mi> <mo>(</mo> <mi>X</mi> <mo>=</mo> <mi>x</mi> <mo>|</mo> <mo>¬</mo> <mi>S</mi> <mo>)</mo> <mo>]</mo></mrow></math>

朴素贝叶斯假设允许我们通过简单地将每个词汇单词的概率估计相乘来计算右侧的每个概率。

实际操作中，通常要避免将大量概率相乘，以防止*下溢*问题，即计算机无法处理太接近 0 的浮点数。从代数中记得<math alttext="log left-parenthesis a b right-parenthesis equals log a plus log b"><mrow><mo form="prefix">log</mo> <mo>(</mo> <mi>a</mi> <mi>b</mi> <mo>)</mo> <mo>=</mo> <mo form="prefix">log</mo> <mi>a</mi> <mo>+</mo> <mo form="prefix">log</mo> <mi>b</mi></mrow></math> 和<math alttext="exp left-parenthesis log x right-parenthesis equals x"><mrow><mo form="prefix">exp</mo><mo>(</mo> <mo form="prefix">log</mo> <mi>x</mi> <mo>)</mo> <mo>=</mo> <mi>x</mi></mrow></math> ，我们通常将<math alttext="p 1 asterisk ellipsis asterisk p Subscript n"><mrow><msub><mi>p</mi> <mn>1</mn></msub> <mo>*</mo> <mo>⋯</mo> <mo>*</mo> <msub><mi>p</mi> <mi>n</mi></msub></mrow></math> 计算为等效的（但更友好于浮点数的）形式：

<math alttext="exp left-parenthesis log left-parenthesis p 1 right-parenthesis plus ellipsis plus log left-parenthesis p Subscript n Baseline right-parenthesis right-parenthesis" display="block"><mrow><mo form="prefix">exp</mo><mo>(</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <msub><mi>p</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <msub><mi>p</mi> <mi>n</mi></msub> <mo>)</mo></mrow> <mo>)</mo></mrow></math>

唯一剩下的挑战是估计<math><mrow><mi>P</mi> <mo>(</mo> <msub><mi>X</mi><mi>i</mi></msub> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow></math> 和<math><mrow><mi>P</mi> <mo>(</mo> <msub><mi>X</mi><mi>i</mi></msub> <mo>|</mo> <mo>¬</mo> <mi>S</mi> <mo>)</mo></mrow></math> ，即垃圾邮件（或非垃圾邮件）包含单词<math><msub><mi>w</mi> <mi>i</mi></msub></math> 的概率。如果我们有大量标记为垃圾邮件和非垃圾邮件的“训练”邮件，一个明显的第一次尝试是简单地估计<math><mrow><mi>P</mi> <mo>(</mo> <msub><mi>X</mi> <mi>i</mi></msub> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow></math> 为仅仅是包含单词<math><msub><mi>w</mi> <mi>i</mi></msub></math> 的垃圾邮件的比例。

不过，这会造成一个很大的问题。想象一下，在我们的训练集中，词汇表中的单词*data*只出现在非垃圾邮件中。然后我们会估计<math alttext="upper P left-parenthesis quotation-mark data quotation-mark vertical-bar upper S right-parenthesis equals 0"><mrow><mi>P</mi> <mo>(</mo> <mtext>data</mtext> <mo>|</mo> <mi>S</mi> <mo>)</mo> <mo>=</mo> <mn>0</mn></mrow></math>。结果是，我们的朴素贝叶斯分类器将始终为包含单词*data*的*任何*消息分配垃圾邮件概率 0，即使是像“免费比特币和正品劳力士手表数据”这样的消息也是如此。为了避免这个问题，我们通常使用某种平滑技术。

特别是，我们将选择一个*伪计数*——*k*——并估计在垃圾邮件中看到第*i*个单词的概率为：

<math alttext="upper P left-parenthesis upper X Subscript i Baseline vertical-bar upper S right-parenthesis equals left-parenthesis k plus number of spams containing w Subscript i Baseline right-parenthesis slash left-parenthesis 2 k plus number of spams right-parenthesis" display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> <msub><mi>X</mi> <mi>i</mi></msub> <mo>|</mo> <mi>S</mi> <mo>)</mo></mrow> <mo>=</mo> <mrow><mo>(</mo> <mi>k</mi> <mo>+</mo> <mtext>number</mtext> <mtext>of</mtext> <mtext>spams</mtext> <mtext>containing</mtext> <msub><mi>w</mi> <mi>i</mi></msub> <mo>)</mo></mrow> <mo>/</mo> <mrow><mo>(</mo> <mn>2</mn> <mi>k</mi> <mo>+</mo> <mtext>number</mtext> <mtext>of</mtext> <mtext>spams</mtext> <mo>)</mo></mrow></mrow></math>

我们对<math><mrow><mi>P</mi> <mo>(</mo> <msub><mi>X</mi> <mi>i</mi></msub> <mo>|</mo> <mo>¬</mo> <mi>S</mi> <mo>)</mo></mrow></math>也是类似的。也就是说，当计算第*i*个单词的垃圾邮件概率时，我们假设我们还看到了包含该单词的*k*个额外的非垃圾邮件和*k*个额外的不包含该单词的非垃圾邮件。

例如，如果*data*出现在 0/98 封垃圾邮件中，如果*k*为 1，则我们将*P*(data|*S*)估计为 1/100 = 0.01，这使得我们的分类器仍然可以为包含单词*data*的消息分配一些非零的垃圾邮件概率。

# 实施

现在我们有了构建分类器所需的所有组件。首先，让我们创建一个简单的函数，将消息标记为不同的单词。我们首先将每条消息转换为小写，然后使用`re.findall`提取由字母、数字和撇号组成的“单词”。最后，我们将使用`set`获取不同的单词：

```py
from typing import Set
import re

def tokenize(text: str) -> Set[str]:
    text = text.lower()                         # Convert to lowercase,
    all_words = re.findall("[a-z0-9']+", text)  # extract the words, and
    return set(all_words)                       # remove duplicates.

assert tokenize("Data Science is science") == {"data", "science", "is"}
```

我们还将为我们的训练数据定义一个类型：

```py
from typing import NamedTuple

class Message(NamedTuple):
    text: str
    is_spam: bool
```

由于我们的分类器需要跟踪来自训练数据的标记、计数和标签，我们将其构建为一个类。按照惯例，我们将非垃圾邮件称为*ham*邮件。

构造函数只需要一个参数，即计算概率时使用的伪计数。它还初始化一个空的标记集合、计数器来跟踪在垃圾邮件和非垃圾邮件中看到每个标记的频率，以及它被训练的垃圾邮件和非垃圾邮件的数量：

```py
from typing import List, Tuple, Dict, Iterable
import math
from collections import defaultdict

class NaiveBayesClassifier:
    def __init__(self, k: float = 0.5) -> None:
        self.k = k  # smoothing factor

        self.tokens: Set[str] = set()
        self.token_spam_counts: Dict[str, int] = defaultdict(int)
        self.token_ham_counts: Dict[str, int] = defaultdict(int)
        self.spam_messages = self.ham_messages = 0
```

接下来，我们将为其提供一个方法，让它训练一堆消息。首先，我们增加`spam_messages`和`ham_messages`计数。然后我们对每个消息文本进行标记化，对于每个标记，根据消息类型递增`token_spam_counts`或`token_ham_counts`：

```py
    def train(self, messages: Iterable[Message]) -> None:
        for message in messages:
            # Increment message counts
            if message.is_spam:
                self.spam_messages += 1
            else:
                self.ham_messages += 1

            # Increment word counts
            for token in tokenize(message.text):
                self.tokens.add(token)
                if message.is_spam:
                    self.token_spam_counts[token] += 1
                else:
                    self.token_ham_counts[token] += 1
```

最终，我们将想要预测*P*(垃圾邮件 | 标记)。正如我们之前看到的，要应用贝叶斯定理，我们需要知道每个词汇表中的标记对于*垃圾邮件*和*非垃圾邮件*的*P*(标记 | 垃圾邮件)和*P*(标记 | 非垃圾邮件)。因此，我们将创建一个“私有”辅助函数来计算这些：

```py
    def _probabilities(self, token: str) -> Tuple[float, float]:
        """returns P(token | spam) and P(token | ham)"""
        spam = self.token_spam_counts[token]
        ham = self.token_ham_counts[token]

        p_token_spam = (spam + self.k) / (self.spam_messages + 2 * self.k)
        p_token_ham = (ham + self.k) / (self.ham_messages + 2 * self.k)

        return p_token_spam, p_token_ham
```

最后，我们准备编写我们的`predict`方法。如前所述，我们不会将许多小概率相乘，而是将对数概率相加：

```py
    def predict(self, text: str) -> float:
        text_tokens = tokenize(text)
        log_prob_if_spam = log_prob_if_ham = 0.0

        # Iterate through each word in our vocabulary
        for token in self.tokens:
            prob_if_spam, prob_if_ham = self._probabilities(token)

            # If *token* appears in the message,
            # add the log probability of seeing it
            if token in text_tokens:
                log_prob_if_spam += math.log(prob_if_spam)
                log_prob_if_ham += math.log(prob_if_ham)

            # Otherwise add the log probability of _not_ seeing it,
            # which is log(1 - probability of seeing it)
            else:
                log_prob_if_spam += math.log(1.0 - prob_if_spam)
                log_prob_if_ham += math.log(1.0 - prob_if_ham)

        prob_if_spam = math.exp(log_prob_if_spam)
        prob_if_ham = math.exp(log_prob_if_ham)
        return prob_if_spam / (prob_if_spam + prob_if_ham)
```

现在我们有了一个分类器。

# 测试我们的模型

让我们通过为其编写一些单元测试来确保我们的模型有效。

```py
messages = [Message("spam rules", is_spam=True),
            Message("ham rules", is_spam=False),
            Message("hello ham", is_spam=False)]

model = NaiveBayesClassifier(k=0.5)
model.train(messages)
```

首先，让我们检查它是否正确计数：

```py
assert model.tokens == {"spam", "ham", "rules", "hello"}
assert model.spam_messages == 1
assert model.ham_messages == 2
assert model.token_spam_counts == {"spam": 1, "rules": 1}
assert model.token_ham_counts == {"ham": 2, "rules": 1, "hello": 1}
```

现在让我们做出预测。我们还将（费力地）手动执行我们的朴素贝叶斯逻辑，并确保获得相同的结果：

```py
text = "hello spam"

probs_if_spam = [
    (1 + 0.5) / (1 + 2 * 0.5),      # "spam"  (present)
    1 - (0 + 0.5) / (1 + 2 * 0.5),  # "ham"   (not present)
    1 - (1 + 0.5) / (1 + 2 * 0.5),  # "rules" (not present)
    (0 + 0.5) / (1 + 2 * 0.5)       # "hello" (present)
]

probs_if_ham = [
    (0 + 0.5) / (2 + 2 * 0.5),      # "spam"  (present)
    1 - (2 + 0.5) / (2 + 2 * 0.5),  # "ham"   (not present)
    1 - (1 + 0.5) / (2 + 2 * 0.5),  # "rules" (not present)
    (1 + 0.5) / (2 + 2 * 0.5),      # "hello" (present)
]

p_if_spam = math.exp(sum(math.log(p) for p in probs_if_spam))
p_if_ham = math.exp(sum(math.log(p) for p in probs_if_ham))

# Should be about 0.83
assert model.predict(text) == p_if_spam / (p_if_spam + p_if_ham)
```

此测试通过，因此似乎我们的模型正在做我们认为的事情。如果您查看实际的概率，两个主要驱动因素是我们的消息包含*spam*（我们唯一的训练垃圾邮件消息包含）和它不包含*ham*（我们的两个训练非垃圾邮件消息均不包含）。

现在让我们尝试在一些真实数据上运行它。

# 使用我们的模型

一个流行的（尽管有些陈旧的）数据集是[SpamAssassin 公共语料库](https://spamassassin.apache.org/old/publiccorpus/)。我们将查看以*20021010*为前缀的文件。

这里是一个脚本，将下载并解压它们到您选择的目录（或者您可以手动执行）：

```py
from io import BytesIO  # So we can treat bytes as a file.
import requests         # To download the files, which
import tarfile          # are in .tar.bz format.

BASE_URL = "https://spamassassin.apache.org/old/publiccorpus"
FILES = ["20021010_easy_ham.tar.bz2",
         "20021010_hard_ham.tar.bz2",
         "20021010_spam.tar.bz2"]

# This is where the data will end up,
# in /spam, /easy_ham, and /hard_ham subdirectories.
# Change this to where you want the data.
OUTPUT_DIR = 'spam_data'

for filename in FILES:
    # Use requests to get the file contents at each URL.
    content = requests.get(f"{BASE_URL}/{filename}").content

    # Wrap the in-memory bytes so we can use them as a "file."
    fin = BytesIO(content)

    # And extract all the files to the specified output dir.
    with tarfile.open(fileobj=fin, mode='r:bz2') as tf:
        tf.extractall(OUTPUT_DIR)
```

文件的位置可能会更改（这在本书的第一版和第二版之间发生过），如果是这样，请相应调整脚本。

下载数据后，您应该有三个文件夹：*spam*，*easy_ham*和*hard_ham*。每个文件夹包含许多电子邮件，每封邮件都包含在单个文件中。为了保持*非常*简单，我们将仅查看每封电子邮件的主题行。

如何识别主题行？当我们浏览文件时，它们似乎都以“主题：”开头。因此，我们将寻找这个：

```py
import glob, re

# modify the path to wherever you've put the files
path = 'spam_data/*/*'

data: List[Message] = []

# glob.glob returns every filename that matches the wildcarded path
for filename in glob.glob(path):
    is_spam = "ham" not in filename

    # There are some garbage characters in the emails; the errors='ignore'
    # skips them instead of raising an exception.
    with open(filename, errors='ignore') as email_file:
        for line in email_file:
            if line.startswith("Subject:"):
                subject = line.lstrip("Subject: ")
                data.append(Message(subject, is_spam))
                break  # done with this file
```

现在我们可以将数据分为训练数据和测试数据，然后我们就可以构建分类器了：

```py
import random
from scratch.machine_learning import split_data

random.seed(0)      # just so you get the same answers as me
train_messages, test_messages = split_data(data, 0.75)

model = NaiveBayesClassifier()
model.train(train_messages)
```

让我们生成一些预测并检查我们的模型的表现：

```py
from collections import Counter

predictions = [(message, model.predict(message.text))
               for message in test_messages]

# Assume that spam_probability > 0.5 corresponds to spam prediction
# and count the combinations of (actual is_spam, predicted is_spam)
confusion_matrix = Counter((message.is_spam, spam_probability > 0.5)
                           for message, spam_probability in predictions)

print(confusion_matrix)
```

这给出了 84 个真正的阳性（被分类为“垃圾邮件”的垃圾邮件），25 个假阳性（被分类为“垃圾邮件”的非垃圾邮件），703 个真负（被分类为“非垃圾邮件”的非垃圾邮件）和 44 个假阴性（被分类为“非垃圾邮件”的垃圾邮件）。这意味着我们的精度是 84 /（84 + 25）= 77％，我们的召回率是 84 /（84 + 44）= 65％，对于如此简单的模型来说，这些数字并不差。（假设如果我们查看的不仅仅是主题行，我们可能会做得更好。）

我们还可以检查模型的内部，看看哪些单词最少和最具有指示性的垃圾邮件。

```py
def p_spam_given_token(token: str, model: NaiveBayesClassifier) -> float:
    # We probably shouldn't call private methods, but it's for a good cause.
    prob_if_spam, prob_if_ham = model._probabilities(token)

    return prob_if_spam / (prob_if_spam + prob_if_ham)

words = sorted(model.tokens, key=lambda t: p_spam_given_token(t, model))

print("spammiest_words", words[-10:])
print("hammiest_words", words[:10])
```

最垃圾的词包括像*sale*，*mortgage*，*money*和*rates*这样的词，而最不垃圾的词包括像*spambayes*，*users*，*apt*和*perl*这样的词。因此，这也让我们对我们的模型基本做出正确的判断有了一些直观的信心。

我们如何才能获得更好的性能？一个明显的方法是获得更多的训练数据。还有很多改进模型的方法。以下是您可以尝试的一些可能性：

+   查看消息内容，而不仅仅是主题行。您将需要小心处理消息头。

+   我们的分类器考虑了训练集中出现的每个单词，甚至是仅出现一次的单词。修改分类器以接受可选的`min_count`阈值，并忽略不至少出现这么多次的标记。

+   分词器没有类似词汇的概念（例如 *cheap* 和 *cheapest*）。修改分类器以接受可选的 `stemmer` 函数，将单词转换为 *等价类* 的单词。例如，一个非常简单的词干提取函数可以是：

    ```py
    def drop_final_s(word):
        return re.sub("s$", "", word)
    ```

    创建一个良好的词干提取函数很难。人们经常使用 [Porter stemmer](http://tartarus.org/martin/PorterStemmer/)。

+   尽管我们的特征都是形如“消息包含词 <math><msub><mi>w</mi> <mi>i</mi></msub></math>”，但这并非必须如此。在我们的实现中，我们可以通过创建虚假标记，如 *contains:number*，并在适当时修改 `tokenizer` 以发出它们来添加额外的特征，比如“消息包含数字”。

# 进一步探索

+   保罗·格雷厄姆的文章 [“一个垃圾邮件过滤计划”](http://www.paulgraham.com/spam.html) 和 [“更好的贝叶斯过滤”](http://www.paulgraham.com/better.html) 非常有趣，并深入探讨了构建垃圾邮件过滤器背后的思想。

+   [scikit-learn](https://scikit-learn.org/stable/modules/naive_bayes.html) 包含一个 `BernoulliNB` 模型，实现了我们这里实现的相同朴素贝叶斯算法，以及模型的其他变体。
