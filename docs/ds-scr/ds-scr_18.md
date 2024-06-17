# 第17章\. 决策树

> 树是一个难以理解的神秘。
> 
> Jim Woodring

DataSciencester的人才副总裁已经面试了一些来自该网站的求职者，结果各不相同。他收集了一个数据集，其中包含每个候选人的几个（定性）属性，以及该候选人是否面试表现良好或不佳。他问道，你能否利用这些数据建立一个模型，识别哪些候选人会面试表现良好，这样他就不必浪费时间进行面试了？

这似乎非常适合*决策树*，这是数据科学家工具包中的另一种预测建模工具。

# 什么是决策树？

决策树使用树结构来表示多条可能的*决策路径*和每条路径的结果。

如果你玩过[Twenty Questions](http://en.wikipedia.org/wiki/Twenty_Questions)游戏，那么你就熟悉决策树了。例如：

+   “我在想一个动物。”

+   “它有五条腿以上吗？”

+   “不。”

+   “它好吃吗？”

+   “不。”

+   “它出现在澳大利亚五分硬币的背面吗？”

+   “是的。”

+   “它是针鼹吗？”

+   “是的，就是它！”

这对应于路径：

“不超过5条腿” → “不好吃” → “在5分硬币上” → “针鼹！”

在一个古怪（并不是很全面的）“猜动物”决策树中（[Figure 17-1](#guess_the_animal)）。

![猜动物。](assets/dsf2_1701.png)

###### 图17-1\. “猜动物”决策树

决策树有很多优点。它们非常容易理解和解释，它们达到预测的过程完全透明。与我们迄今所看到的其他模型不同，决策树可以轻松处理数值型（例如，腿的数量）和分类型（例如，好吃/不好吃）属性的混合数据，甚至可以对缺少属性的数据进行分类。

与此同时，为一组训练数据找到一个“最优”决策树在计算上是一个非常困难的问题。（我们将通过尝试构建一个足够好的树来避开这个问题，尽管对于大型数据集来说，这仍然可能是一项艰巨的工作。）更重要的是，很容易（也很糟糕）构建过度拟合训练数据的决策树，这些树在未见数据上的泛化能力很差。我们将探讨解决这个问题的方法。

大多数人将决策树分为*分类树*（生成分类输出）和*回归树*（生成数值输出）。在本章中，我们将专注于分类树，并通过ID3算法从一组标记数据中学习决策树，这将帮助我们理解决策树的实际工作原理。为了简化问题，我们将局限于具有二元输出的问题，例如“我应该雇佣这位候选人吗？”或“我应该向这位网站访客展示广告A还是广告B？”或“我在办公室冰箱里找到的这种食物会让我生病吗？”

# 熵

要构建决策树，我们需要决定提出什么问题以及顺序。在树的每个阶段，我们消除了一些可能性，还有一些没有。学到动物的腿不超过五条后，我们排除了它是蚱蜢的可能性。我们还没排除它是一只鸭子的可能性。每个可能的问题根据其答案将剩余的可能性进行分区。

理想情况下，我们希望选择的问题答案能够提供关于我们的树应该预测什么的大量信息。如果有一个单一的是/否问题，“是”答案总是对应于 `True` 输出，而“否”答案对应于 `False` 输出（反之亦然），那这将是一个很棒的问题选择。相反，如果一个是/否问题的任何答案都不能给出关于预测应该是什么的新信息，那可能不是一个好选择。

我们用*熵*来捕捉这种“信息量”概念。你可能听说过这个术语用来表示无序。我们用它来表示与数据相关的不确定性。

想象我们有一个数据集 *S*，其中每个成员都被标记为属于有限数量的类别 <math><mrow><msub><mi>C</mi> <mn>1</mn></msub> <mo>,</mo> <mo>...</mo> <mo>,</mo> <msub><mi>C</mi> <mi>n</mi></msub></mrow></math> 中的一种。如果所有数据点属于同一类，则没有真正的不确定性，这意味着我们希望熵很低。如果数据点均匀分布在各个类别中，就会有很多不确定性，我们希望熵很高。

从数学角度来看，如果 <math><msub><mi>p</mi> <mi>i</mi></msub></math> 是标记为类别 <math><msub><mi>c</mi> <mi>i</mi></msub></math> 的数据的比例，我们定义熵如下：

<math alttext="upper H left-parenthesis upper S right-parenthesis equals minus p 1 log Subscript 2 Baseline p 1 minus ellipsis minus p Subscript n Baseline log Subscript 2 Baseline p Subscript n" display="block"><mrow><mi>H</mi> <mrow><mo>(</mo> <mi>S</mi> <mo>)</mo></mrow> <mo>=</mo> <mo>-</mo> <msub><mi>p</mi> <mn>1</mn></msub> <msub><mo form="prefix">log</mo> <mn>2</mn></msub> <msub><mi>p</mi> <mn>1</mn></msub> <mo>-</mo> <mo>...</mo> <mo>-</mo> <msub><mi>p</mi> <mi>n</mi></msub> <msub><mo form="prefix">log</mo> <mn>2</mn></msub> <msub><mi>p</mi> <mi>n</mi></msub></mrow></math>

按照（标准）惯例，<math><mrow><mn>0</mn> <mo form="prefix">log</mo> <mn>0</mn> <mo>=</mo> <mn>0</mn></mrow></math> 。

不必过分担心可怕的细节，每个术语 <math><mrow><mo>-</mo> <msub><mi>p</mi> <mi>i</mi></msub> <msub><mo form="prefix">log</mo> <mn>2</mn></msub> <msub><mi>p</mi> <mi>i</mi></msub></mrow></math> 都是非负的，当 <math><msub><mi>p</mi> <mi>i</mi></msub></math> 接近于 0 或接近于 1 时，它接近于 0（[图 17-2](#p_log_p)）。

![–p log p 的图示。](assets/dsf2_1702.png)

###### 图 17-2\. -p log p 的图示

这意味着当每个 <math><msub><mi>p</mi> <mi>i</msub></math> 接近于 0 或 1 时（即大多数数据属于单一类别时），熵将很小，当许多 <math><msub><mi>p</mi> <mi>i</msub></math> 不接近于 0 时（即数据分布在多个类别中时），熵将较大。这正是我们期望的行为。

将所有这些内容整合到一个函数中是相当简单的：

```py
from typing import List
import math

def entropy(class_probabilities: List[float]) -> float:
    """Given a list of class probabilities, compute the entropy"""
    return sum(-p * math.log(p, 2)
               for p in class_probabilities
               if p > 0)                     # ignore zero probabilities

assert entropy([1.0]) == 0
assert entropy([0.5, 0.5]) == 1
assert 0.81 < entropy([0.25, 0.75]) < 0.82
```

我们的数据将由成对的`(输入，标签)`组成，这意味着我们需要自己计算类别概率。注意，我们实际上并不关心每个概率与哪个标签相关联，只关心这些概率是多少：

```py
from typing import Any
from collections import Counter

def class_probabilities(labels: List[Any]) -> List[float]:
    total_count = len(labels)
    return [count / total_count
            for count in Counter(labels).values()]

def data_entropy(labels: List[Any]) -> float:
    return entropy(class_probabilities(labels))

assert data_entropy(['a']) == 0
assert data_entropy([True, False]) == 1
assert data_entropy([3, 4, 4, 4]) == entropy([0.25, 0.75])
```

# 分区的熵

到目前为止，我们所做的是计算单一标记数据集的熵（想想“不确定性”）。现在，决策树的每个阶段都涉及提出一个问题，其答案将数据分成一个或多个子集。例如，我们的“它有超过五条腿吗？”问题将动物分成有超过五条腿的动物（例如，蜘蛛）和没有超过五条腿的动物（例如，针鼹）。

相应地，我们希望从某种程度上了解通过某种方式对数据集进行分区所产生的熵。如果分区将数据分成具有低熵（即高度确定）的子集，我们希望分区具有低熵；如果包含具有高熵（即高度不确定）的子集（即大且）分区具有高熵。

例如，我的“澳大利亚五分硬币”问题非常愚蠢（尽管非常幸运！），因为它将那时剩下的动物分成<math><msub><mi>S</mi> <mn>1</mn></msub></math> = {针鼹}和<math><msub><mi>S</mi> <mn>2</mn></msub></math> = {其他所有动物}，其中<math><msub><mi>S</mi> <mn>2</mn></msub></math>既大又高熵。 (<math><msub><mi>S</mi> <mn>1</mn></msub></math>没有熵，但它表示剩余“类别”的小部分。)

从数学上讲，如果我们将我们的数据*S*分成包含数据比例的子集<math><mrow><msub><mi>S</mi> <mn>1</mn></msub> <mo>,</mo> <mo>...</mo> <mo>,</mo> <msub><mi>S</mi> <mi>m</mi></msub></mrow></math>，那么我们将分区的熵计算为加权和：

<math alttext="upper H equals q 1 upper H left-parenthesis upper S 1 right-parenthesis plus period period period plus q Subscript m Baseline upper H left-parenthesis upper S Subscript m Baseline right-parenthesis" display="block"><mrow><mi>H</mi> <mo>=</mo> <msub><mi>q</mi> <mn>1</mn></msub> <mi>H</mi> <mrow><mo>(</mo> <msub><mi>S</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo>+</mo> <mo>...</mo> <mo>+</mo> <msub><mi>q</mi> <mi>m</mi></msub> <mi>H</mi> <mrow><mo>(</mo> <msub><mi>S</mi> <mi>m</mi></msub> <mo>)</mo></mrow></mrow></math>

我们可以实现为：

```py
def partition_entropy(subsets: List[List[Any]]) -> float:
    """Returns the entropy from this partition of data into subsets"""
    total_count = sum(len(subset) for subset in subsets)

    return sum(data_entropy(subset) * len(subset) / total_count
               for subset in subsets)
```

###### 注意

这种方法的一个问题是，使用具有许多不同值的属性进行分区会由于过拟合而导致熵非常低。例如，假设你在银行工作，试图建立一个决策树来预测哪些客户可能会违约他们的抵押贷款，使用一些历史数据作为你的训练集。进一步假设数据集包含每个客户的社会安全号码。在社会安全号码上进行分区将产生单个人的子集，每个子集的熵必然为零。但是依赖社会安全号码的模型*肯定*无法超出训练集的范围。因此，在创建决策树时，你应该尽量避免（或适当地分桶）具有大量可能值的属性。

# 创建决策树

副总裁为您提供了面试者数据，根据您的规定，每位候选人的相关属性是一个`NamedTuple`——她的级别、她偏爱的语言、她是否在 Twitter 上活跃、她是否有博士学位以及她是否面试表现良好：

```py
from typing import NamedTuple, Optional

class Candidate(NamedTuple):
    level: str
    lang: str
    tweets: bool
    phd: bool
    did_well: Optional[bool] = None  # allow unlabeled data

                  #  level     lang     tweets  phd  did_well
inputs = [Candidate('Senior', 'Java',   False, False, False),
          Candidate('Senior', 'Java',   False, True,  False),
          Candidate('Mid',    'Python', False, False, True),
          Candidate('Junior', 'Python', False, False, True),
          Candidate('Junior', 'R',      True,  False, True),
          Candidate('Junior', 'R',      True,  True,  False),
          Candidate('Mid',    'R',      True,  True,  True),
          Candidate('Senior', 'Python', False, False, False),
          Candidate('Senior', 'R',      True,  False, True),
          Candidate('Junior', 'Python', True,  False, True),
          Candidate('Senior', 'Python', True,  True,  True),
          Candidate('Mid',    'Python', False, True,  True),
          Candidate('Mid',    'Java',   True,  False, True),
          Candidate('Junior', 'Python', False, True,  False)
         ]
```

我们的树将包含*决策节点*（提出问题并根据答案引导我们不同路径）和*叶节点*（给出预测）。我们将使用相对简单的*ID3*算法构建它，该算法操作如下。假设我们有一些带标签的数据，并且有一个要考虑分支的属性列表：

+   如果所有数据都具有相同的标签，请创建一个预测该标签的叶节点，然后停止。

+   如果属性列表为空（即没有更多可能的问题可问），创建一个预测最常见标签的叶节点，然后停止。

+   否则，尝试按照每个属性对数据进行分割。

+   选择具有最低分区熵的分区。

+   根据选择的属性添加一个决策节点。

+   对剩余属性使用递归对每个分区的子集进行分割。

这被称为“贪婪”算法，因为在每一步中，它选择最即时的最佳选项。给定一个数据集，可能有一个看起来更差的第一步，但却会有一个更好的树。如果确实存在这样的情况，此算法将无法找到它。尽管如此，它相对容易理解和实现，这使得它成为探索决策树的一个很好的起点。

让我们手动按照面试者数据集中的这些步骤进行。数据集具有`True`和`False`标签，并且我们有四个可以分割的属性。因此，我们的第一步将是找到熵最小的分区。我们将首先编写一个执行分割的函数：

```py
from typing import Dict, TypeVar
from collections import defaultdict

T = TypeVar('T')  # generic type for inputs

def partition_by(inputs: List[T], attribute: str) -> Dict[Any, List[T]]:
    """Partition the inputs into lists based on the specified attribute."""
    partitions: Dict[Any, List[T]] = defaultdict(list)
    for input in inputs:
        key = getattr(input, attribute)  # value of the specified attribute
        partitions[key].append(input)    # add input to the correct partition
    return partitions
```

还有一个使用它计算熵的函数：

```py
def partition_entropy_by(inputs: List[Any],
                         attribute: str,
                         label_attribute: str) -> float:
    """Compute the entropy corresponding to the given partition"""
    # partitions consist of our inputs
    partitions = partition_by(inputs, attribute)

    # but partition_entropy needs just the class labels
    labels = [[getattr(input, label_attribute) for input in partition]
              for partition in partitions.values()]

    return partition_entropy(labels)
```

然后我们只需为整个数据集找到最小熵分区：

```py
for key in ['level','lang','tweets','phd']:
    print(key, partition_entropy_by(inputs, key, 'did_well'))

assert 0.69 < partition_entropy_by(inputs, 'level', 'did_well')  < 0.70
assert 0.86 < partition_entropy_by(inputs, 'lang', 'did_well')   < 0.87
assert 0.78 < partition_entropy_by(inputs, 'tweets', 'did_well') < 0.79
assert 0.89 < partition_entropy_by(inputs, 'phd', 'did_well')    < 0.90
```

最低熵来自于按`level`分割，因此我们需要为每个可能的`level`值创建一个子树。每个`Mid`候选人都标记为`True`，这意味着`Mid`子树只是一个预测`True`的叶节点。对于`Senior`候选人，我们有`True`和`False`的混合，因此我们需要再次分割：

```py
senior_inputs = [input for input in inputs if input.level == 'Senior']

assert 0.4 == partition_entropy_by(senior_inputs, 'lang', 'did_well')
assert 0.0 == partition_entropy_by(senior_inputs, 'tweets', 'did_well')
assert 0.95 < partition_entropy_by(senior_inputs, 'phd', 'did_well') < 0.96
```

这告诉我们我们接下来应该在`tweets`上进行分割，这会导致零熵分区。对于这些`Senior`级别的候选人，“是”推文总是导致`True`，而“否”推文总是导致`False`。

最后，如果我们对`Junior`候选人执行相同的操作，我们最终会在`phd`上进行分割，之后发现没有博士学位总是导致`True`，而有博士学位总是导致`False`。

[图 17-3](#hiring_decision_tree) 显示了完整的决策树。

![招聘决策树。](assets/dsf2_1703.png)

###### 图 17-3\. 招聘的决策树

# 把所有这些整合起来

现在我们已经了解了算法的工作原理，我们希望更普遍地实现它。这意味着我们需要决定如何表示树。我们将使用可能最轻量级的表示。我们将一个*树*定义为以下内容之一：

+   一个`Leaf`（预测单个值），或者

+   一个`Split`（包含要拆分的属性、特定属性值的子树，以及在遇到未知值时可能使用的默认值）。

    ```py
    from typing import NamedTuple, Union, Any

    class Leaf(NamedTuple):
        value: Any

    class Split(NamedTuple):
        attribute: str
        subtrees: dict
        default_value: Any = None

    DecisionTree = Union[Leaf, Split]
    ```

有了这个表示，我们的招聘树将如下所示：

```py
hiring_tree = Split('level', {   # first, consider "level"
    'Junior': Split('phd', {     # if level is "Junior", next look at "phd"
        False: Leaf(True),       #   if "phd" is False, predict True
        True: Leaf(False)        #   if "phd" is True, predict False
    }),
    'Mid': Leaf(True),           # if level is "Mid", just predict True
    'Senior': Split('tweets', {  # if level is "Senior", look at "tweets"
        False: Leaf(False),      #   if "tweets" is False, predict False
        True: Leaf(True)         #   if "tweets" is True, predict True
    })
})
```

还有一个问题，即如果我们遇到意外的（或缺失的）属性值该怎么办。如果我们的招聘树遇到`level`是`Intern`的候选人会怎么样？我们将通过用最常见的标签填充`default_value`属性来处理这种情况。

给定这样的表示，我们可以对输入进行分类：

```py
def classify(tree: DecisionTree, input: Any) -> Any:
    """classify the input using the given decision tree"""

    # If this is a leaf node, return its value
    if isinstance(tree, Leaf):
        return tree.value

    # Otherwise this tree consists of an attribute to split on
    # and a dictionary whose keys are values of that attribute
    # and whose values are subtrees to consider next
    subtree_key = getattr(input, tree.attribute)

    if subtree_key not in tree.subtrees:   # If no subtree for key,
        return tree.default_value          # return the default value.

    subtree = tree.subtrees[subtree_key]   # Choose the appropriate subtree
    return classify(subtree, input)        # and use it to classify the input.
```

剩下的就是从我们的训练数据中构建树表示：

```py
def build_tree_id3(inputs: List[Any],
                   split_attributes: List[str],
                   target_attribute: str) -> DecisionTree:
    # Count target labels
    label_counts = Counter(getattr(input, target_attribute)
                           for input in inputs)
    most_common_label = label_counts.most_common(1)[0][0]

    # If there's a unique label, predict it
    if len(label_counts) == 1:
        return Leaf(most_common_label)

    # If no split attributes left, return the majority label
    if not split_attributes:
        return Leaf(most_common_label)

    # Otherwise split by the best attribute

    def split_entropy(attribute: str) -> float:
        """Helper function for finding the best attribute"""
        return partition_entropy_by(inputs, attribute, target_attribute)

    best_attribute = min(split_attributes, key=split_entropy)

    partitions = partition_by(inputs, best_attribute)
    new_attributes = [a for a in split_attributes if a != best_attribute]

    # Recursively build the subtrees
    subtrees = {attribute_value : build_tree_id3(subset,
                                                 new_attributes,
                                                 target_attribute)
                for attribute_value, subset in partitions.items()}

    return Split(best_attribute, subtrees, default_value=most_common_label)
```

在我们构建的树中，每个叶子节点完全由`True`输入或完全由`False`输入组成。这意味着该树在训练数据集上的预测完全正确。但我们也可以将其应用于训练集中不存在的新数据：

```py
tree = build_tree_id3(inputs,
                      ['level', 'lang', 'tweets', 'phd'],
                      'did_well')

# Should predict True
assert classify(tree, Candidate("Junior", "Java", True, False))

# Should predict False
assert not classify(tree, Candidate("Junior", "Java", True, True))
```

也适用于具有意外值的数据：

```py
# Should predict True
assert classify(tree, Candidate("Intern", "Java", True, True))
```

###### 注意

由于我们的目标主要是演示如何构建树，所以我们使用整个数据集构建了树。一如既往，如果我们真的试图为某事创建一个良好的模型，我们会收集更多数据并将其分割成训练/验证/测试子集。

# 随机森林

鉴于决策树可以如此紧密地适应其训练数据，他们很容易出现过拟合的倾向并不奇怪。避免这种情况的一种方法是一种称为*随机森林*的技术，其中我们构建多个决策树并组合它们的输出。如果它们是分类树，我们可以让它们投票；如果它们是回归树，我们可以平均它们的预测。

我们的树构建过程是确定性的，那么我们如何获得随机树呢？

其中一部分涉及引导数据（参见[“插曲：自助法”](ch15.html#the_bootstrap)）。我们不是在整个训练集上训练每棵树，而是在`bootstrap_sample(inputs)`的结果上训练每棵树。由于每棵树都是使用不同的数据构建的，因此每棵树与其他每棵树都不同。（一个副作用是，使用未抽样数据来测试每棵树是完全公平的，这意味着如果在衡量性能时聪明地使用所有数据作为训练集，你就可以获得成功。）这种技术被称为*自助聚合*或*装袋*。

第二个随机性源涉及更改选择要拆分的`best_attribute`的方法。我们不是查看所有剩余属性，而是首先选择它们的随机子集，然后在其中最好的属性上进行拆分：

```py
    # if there are already few enough split candidates, look at all of them
    if len(split_candidates) <= self.num_split_candidates:
        sampled_split_candidates = split_candidates
    # otherwise pick a random sample
    else:
        sampled_split_candidates = random.sample(split_candidates,
                                                 self.num_split_candidates)

    # now choose the best attribute only from those candidates
    best_attribute = min(sampled_split_candidates, key=split_entropy)

    partitions = partition_by(inputs, best_attribute)
```

这是*集成学习*的一个例子，其中我们结合了多个*弱学习器*（通常是高偏差、低方差模型），以生成一个总体上强大的模型。

# 进一步探索

+   scikit-learn 包含许多[决策树](https://scikit-learn.org/stable/modules/tree.html)模型。它还有一个[`ensemble`](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.ensemble)模块，其中包括`RandomForestClassifier`以及其他集成方法。

+   [XGBoost](https://xgboost.ai/) 是一个用于训练*梯度提升*决策树的库，经常在许多 Kaggle 风格的机器学习竞赛中获胜。

+   我们只是浅尝了解决策树及其算法的表面。[维基百科](https://en.wikipedia.org/wiki/Decision_tree_learning)是一个更广泛探索的良好起点。
