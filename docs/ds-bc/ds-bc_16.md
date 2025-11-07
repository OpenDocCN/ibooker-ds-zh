# 第四部分. 案例研究 4：使用在线职位列表改进你的数据科学简历

## 问题陈述

我们已准备好扩展我们的数据科学职业生涯。六个月后，我们将申请一份新工作。为此，我们开始起草简历。早期的草稿粗糙且不完整。它还没有涵盖我们的职业目标或教育。尽管如此，简历涵盖了本书中的前四个案例研究，包括这个案例，我们将在寻求新工作之前完成。

我们的简历草稿远非完美。可能某些重要的数据科学技能尚未体现。如果是这样，这些缺失的技能是什么？我们决定通过分析来找出答案。毕竟，我们是数据科学家！我们使用严格的分析来填补知识空白，那么为什么我们不能将这种严格的分析应用于我们自己呢？

首先，我们需要一些数据。我们上网并访问一个流行的求职网站。该网站提供了数百万可搜索的职位列表，由人手不足的雇主发布。内置的搜索引擎允许我们通过关键词过滤职位，例如*分析师*或*数据科学家*。此外，搜索引擎可以将职位与上传的文档相匹配。此功能旨在根据简历内容搜索帖子。不幸的是，我们的简历仍在进行中。因此，我们转而搜索这本书的目录！我们将目录中列出的前 15 个部分复制粘贴到文本文件中。

接下来，我们将文件上传到求职网站。前四个案例研究的内容与数百万个职位列表进行了比较，并返回了数千个职位帖子。其中一些帖子可能比其他帖子更相关；我们无法保证搜索引擎的整体质量，但数据是受欢迎的。我们从每个帖子中下载 HTML。

我们的目标是从下载的数据中提取常见的数据科学技能。然后，我们将将这些技能与我们的简历进行比较，以确定哪些技能缺失。为了达到我们的目标，我们将按以下步骤进行：

1.  从下载的 HTML 文件中解析出所有文本。

1.  探索解析输出，了解在线帖子中通常如何描述职位技能。也许特定的 HTML 标签更常用于强调职位技能。

1.  尝试从我们的数据集中过滤掉任何不相关的职位帖子。搜索引擎并不完美。也许一些不相关的帖子被错误地下载了。我们可以通过将帖子与我们的简历和目录进行比较来评估相关性。

1.  在相关帖子中聚类职位技能，并可视化聚类。

1.  将聚类技能与我们的简历内容进行比较。然后，我们将制定计划更新我们的简历，以添加任何缺失的数据科学技能。

### 数据集描述

我们的简历草稿存储在文件 resume.txt 中。该草稿的全文如下：

```
Experience

1\. Developed probability simulations using NumPy
2\. Assessed online ad clicks for statistical significance using permutation testing
3\. Analyzed disease outbreaks using common clustering algorithms

Additional Skills

1\. Data visualization using Matplotlib
2\. Statistical analysis using SciPy
3\. Processing structured tables using Pandas
4\. Executing K-means clustering and DBSCAN clustering using scikit-learn
5\. Extracting locations from text using GeoNamesCache
6\. Location analysis and visualization using GeoNamesCache and Cartopy
7\. Dimensionality reduction with PCA and SVD using scikit-learn         ❶
8\. NLP analysis and text topic detection using scikit-learn
```

❶ 我们将在本案例研究的后续部分学习技能 7 和 8。

我们的初步草案简短且不完整。为了弥补任何缺失的材料，我们还使用了本书的部分目录，该目录存储在文件 table_of_contents.txt 中。它涵盖了本书的前 15 个章节，以及所有顶级子章节标题。目录文件已被用于搜索数千个相关的职位发布信息，这些信息已被下载并存储在 job_postings 目录中。目录中的每个文件都与一个单独的发布信息相关联。这些文件可以在您的网页浏览器中本地查看。

## 概述

为了解决当前的问题，我们需要知道如何做以下几件事：

+   测量文本之间的相似度

+   高效地对大量文本数据集进行聚类

+   可视化显示多个文本聚类

+   解析 HTML 文件以获取文本内容
