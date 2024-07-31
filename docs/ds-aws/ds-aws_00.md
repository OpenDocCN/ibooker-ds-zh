# 序言

通过这本实用书籍，AI 和机器学习（ML）从业者将学会如何在亚马逊网络服务（AWS）上成功构建和部署数据科学项目。亚马逊 AI 和 ML 堆栈统一了数据科学、数据工程和应用程序开发，帮助提升您的技能水平。本指南向您展示如何在云中构建和运行流水线，然后将结果集成到应用程序中，从而节省时间，只需几分钟而非几天。整本书中，作者 Chris Fregly 和 Antje Barth 演示了如何降低成本并提高性能。

+   将亚马逊 AI 和 ML 堆栈应用于自然语言处理、计算机视觉、欺诈检测、对话设备等真实用例。

+   使用自动化机器学习（AutoML）通过 Amazon SageMaker Autopilot 实现特定的用例子集。

+   深入探讨基于 BERT 的自然语言处理（NLP）用例的完整模型开发生命周期，包括数据摄入和分析等。

+   将所有内容整合成可重复使用的机器学习运营（MLOps）流水线。

+   利用 Amazon Kinesis 和 Amazon Managed Streaming for Apache Kafka（Amazon MSK）探索实时 ML、异常检测和流式分析的实时数据流。

+   学习数据科学项目和工作流程的安全最佳实践，包括 AWS 身份和访问管理（IAM）、认证、授权，包括数据摄入和分析、模型训练和部署。

# 章节概述

第一章概述了广泛而深入的亚马逊 AI 和 ML 堆栈，这是一个功能强大且多样化的服务、开源库和基础设施集，可用于任何复杂性和规模的数据科学项目。

第二章描述如何将亚马逊 AI 和 ML 堆栈应用于推荐、计算机视觉、欺诈检测、自然语言理解（NLU）、对话设备、认知搜索、客户支持、工业预测性维护、家庭自动化、物联网（IoT）、医疗保健和量子计算等真实用例。

第三章演示了如何使用 AutoML 通过 SageMaker Autopilot 实现这些用例的特定子集。

第四章至第九章深入探讨基于 BERT 的 NLP 用例的完整模型开发生命周期（MDLC），包括数据摄入和分析、特征选择和工程、模型训练和调优，以及与 Amazon SageMaker、Amazon Athena、Amazon Redshift、Amazon EMR、TensorFlow、PyTorch 和无服务器 Apache Spark 的模型部署。

第十章将所有内容整合到使用 SageMaker Pipelines、Kubeflow Pipelines、Apache Airflow、MLflow 和 TFX 的可重复流水线中的 MLOps。

第十一章演示了在实时数据流中使用 Amazon Kinesis 和 Apache Kafka 进行实时机器学习、异常检测和流分析。

第十二章介绍了数据科学项目和工作流的全面安全最佳实践，包括 IAM、身份验证、授权、网络隔离、数据静态加密、后量子网络传输加密、治理和可审计性。

本书始终提供了在 AWS 上减少成本、提高性能的数据科学项目的技巧。

# 适合阅读本书的人群

本书适合任何利用数据进行关键业务决策的人。本指南将帮助数据分析师、数据科学家、数据工程师、机器学习工程师、研究科学家、应用程序开发人员和 DevOps 工程师扩展其对现代数据科学技术栈的理解，并在云中提升其技能水平。

Amazon AI 和 ML 技术栈统一了数据科学、数据工程和应用开发，帮助用户超越当前角色的技能水平。我们展示了如何在云中构建和运行流水线，然后在几分钟内将结果集成到应用程序中，而不是几天。

理想情况下，为了从本书中获得最大收益，我们建议读者具备以下知识：

+   对云计算的基本理解

+   基本的 Python、R、Java/Scala 或 SQL 编程技能

+   基本的数据科学工具如 Jupyter Notebook、pandas、NumPy 或 scikit-learn 的基本了解

# 其他资源

本书从许多优秀的作者和资源中汲取了灵感：

+   Aurélien Géron 的 [*Hands-on Machine Learning with Scikit-Learn, Keras, and TensorFlow*](https://www.oreilly.com/library/view/hands-on-machine-learning/9781492032632/)（O’Reilly）是使用流行工具如 Python、scikit-learn 和 TensorFlow 构建智能机器学习系统的实用指南。

+   Jeremy Howard 和 Sylvain Gugger 的 [*Deep Learning for Coders with fastai and PyTorch*](https://www.oreilly.com/library/view/deep-learning-for/9781492045519/)（O’Reilly）是使用 PyTorch 构建深度学习应用的优秀参考，"无需博士学位"。

+   Hannes Hapke 和 Catherine Nelson 的 [*Building Machine Learning Pipelines*](https://www.oreilly.com/library/view/building-machine-learning/9781492053187/)（O’Reilly）是构建使用 TensorFlow 和 TFX 的 AutoML 流水线的绝佳易读参考。

+   Eric R. Johnston、Nic Harrigan 和 Mercedes Gimeno-Segovia 的 [*Programming Quantum Computers*](https://www.oreilly.com/library/view/programming-quantum-computers/9781492039679/)（O’Reilly）是介绍量子计算机的绝佳入门书籍，使用易于理解的示例展示了量子优势。

+   Micha Gorelick 和 Ian Ozsvald 的 [*High Performance Python*](https://www.oreilly.com/library/view/high-performance-python/9781492055013/) (O’Reilly) 是一本高级参考书，揭示了许多有价值的技巧和窍门，用于分析和优化 Python 代码，以实现高性能数据处理、特征工程和模型训练。

+   [*Data Science on AWS*](https://datascienceonaws.com) 有一个专门为本书提供高级研讨会、每月网络研讨会、聚会、视频和与本书内容相关的幻灯片的网站。

# 本书中使用的约定

本书使用以下排版约定：

*Italic*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落中引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

`**Constant width bold**`

显示用户应按照字面意思键入的命令或其他文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注意事项。

# 使用代码示例

可以从[*https://github.com/data-science-on-aws*](https://github.com/data-science-on-aws)下载附加材料（代码示例、练习等）。本书中显示的一些代码示例已缩短，以突出特定实现。该存储库包含本书未涵盖但对读者有用的额外笔记本。笔记本按书的章节组织，应易于跟随。

本书旨在帮助您完成工作。一般而言，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则无需联系我们请求许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发 O’Reilly 书籍中的示例代码需要许可。引用本书并引用示例代码回答问题不需要许可。将本书大量示例代码整合到产品文档中需要许可。

我们赞赏但不需要署名。通常，署名包括标题、作者、出版商和 ISBN。例如：“*Data Science on AWS* by Chris Fregly and Antje Barth (O’Reilly). Copyright 2021 Antje Barth and Flux Capacitor, LLC, 978-1-492-07939-2.”

如果您觉得您使用的代码示例超出了合理使用范围或上述许可，请随时通过*permissions@oreilly.com*联系我们。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](http://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章以及我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台让您随时访问现场培训课程、深入学习路径、交互式编码环境，以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

有关本书的评论和问题，请联系出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或当地）

+   707-829-0104（传真）

本书有一个网页，我们在那里列出勘误、示例和任何其他信息。您可以访问此页面：*[`oreil.ly/data-science-aws`](https://oreil.ly/data-science-aws)*。

发送电子邮件至 *bookquestions@oreilly.com* 对本书发表评论或提出技术问题。

获取有关我们的书籍和课程的新闻和信息，请访问*[`oreilly.com`](http://oreilly.com)*。

在 Facebook 上找到我们：*[`facebook.com/oreilly`](http://facebook.com/oreilly)*

在 Twitter 上关注我们：*[`twitter.com/oreillymedia`](http://twitter.com/oreillymedia)*

在 YouTube 上观看我们：*[`www.youtube.com/oreillymedia`](http://www.youtube.com/oreillymedia)*

作者经常在 Twitter 或 LinkedIn 上分享相关博客文章、会议演讲、幻灯片、社群邀请和研讨会日期。

在 Twitter 上关注作者：[*https://twitter.com/cfregly*](https://twitter.com/cfregly) 和 [*https://twitter.com/anbarth*](https://twitter.com/anbarth)

在 LinkedIn 上找到作者：[*https://www.linkedin.com/in/cfregly*](https://www.linkedin.com/in/cfregly) 和 [*https://www.linkedin.com/in/antje-barth*](https://www.linkedin.com/in/antje-barth)

# 致谢

我们要感谢我们的 O’Reilly 开发编辑 Gary O’Brien，在我们编写书籍过程中帮助我们导航，更重要的是，每次交谈时都让我们笑。谢谢你，Gary，让我们在第一章中包含源代码和低级硬件规格！我们还要感谢资深收购编辑 Jessica Haberman，在初步书籍提案到最终页数计算的各个阶段都提供了关键建议。在提交了七年的书籍提案之后，你帮助我们提高了水平，直到提案被接受！特别感谢 O’Reilly 的 Mike Loukides 和 Nicole Taché，在书写过程的早期，包括章节大纲、引言和总结，你们提供了周到的建议。

我们要衷心感谢那些不知疲倦地审阅——并重新审阅——这本书中的每一页的书评人员。按照名字的字母顺序排列，以下是审阅人员的名单：Ali Arsanjani，Andy Petrella，Brent Rabowsky，Dean Wampler，Francesco Mosconi，Hannah Marlowe，Hannes Hapke，Josh Patterson，Josh Wills，Liam Morrison，Noah Gift，Ramine Tinati，Robert Monarch，Roy Ben-Alta，Rustem Feyzkhanov，Sean Owen，Shelbee Eigenbrode，Sireesha Muppala，Stefan Natu，Ted Dunning 和 Tim O’Brien。你们深厚的技术专业知识和详尽的反馈不仅对这本书至关重要，也对我们今后呈现技术材料的方式产生了深远影响。你们帮助把这本书从好变得更好，并且我们非常享受与你们一同在这个项目上工作的时光。

## 克里斯

我要将这本书献给我已故的父亲，Thomas Fregly。爸爸：当我 8 岁时，你给我带回了我的第一台苹果电脑，从此改变了我的一生。你在我 10 岁时帮助我消化大学的微积分书，进一步坚定了我对数学的浓厚兴趣。你教会了我如何阅读，如何简洁地写作，如何有效地演讲，如何快速打字，以及如何及早提出问题。看着你在密歇根湖上被困修理船引擎时，我不断受到启发，深入理解支撑软件的硬件。在*芝加哥太阳报*的办公室里四处走动时，我学会了每个人都有一个有趣的故事，无论是前台人员、CEO 还是维护人员。你对每个人一视同仁，询问他们的孩子，倾听他们的故事，并通过你自己的有趣故事让他们开心。当我小的时候牵着你的手在你的大学校园里走来走去时，我学会了可以离开人行道，沿着草地开辟自己的道路。你说：“别担心，克里斯，他们最终会铺这条路，因为显然这是从工程楼到餐厅的最短路径。”爸爸，你是对的。多年后，我们走过那条新铺的路，从餐厅拿了你最喜欢的饮料，Diet Pepsi。从你身上，我学会了在生活中开辟自己的道路，而不是总是随波逐流。虽然你没能看到 Windows 95，老实说，你也没错过什么。而且，Mac OS 最终也转向了 Linux。在那方面，你也是对的。

我还要感谢我的合著者安特耶·巴特，因为她在许多深夜和周末为这本书的写作经历做出了巨大贡献。尽管我们身处旧金山和杜塞尔多夫之间有 8-9 小时的时差，你总是能够为虚拟白板会议、临时源代码改进以及牛津逗号讨论提供支持。因为这次经历，我们成为了更好的朋友，没有你的帮助，我无法创作出如此密集和高质量的书籍。我期待着与你在未来的许多项目中继续合作！

## 安特耶

我要感谢泰德·邓宁和艾伦·弗里德曼作为出色的导师，始终鼓励我接受新挑战。泰德，每次我们交谈时，你总能分享些智慧之言，帮助我从不同的角度看问题，无论是为演示竞赛做准备，还是在指导我们如何让读者从本书中获益最多时。艾伦，我依然记得你在我开始为 O'Reilly Strata 和 AI 会议提交演讲时，如何指导我创建引人入胜的会议议题提案。直到今天，我仍然会特别考虑如何想出吸引眼球的标题。不幸的是，O’Reilly 拒绝了我的建议，将这本书命名为*Alexa，请训练我的模型*。

你们两位在提到“帮助实现女孩们的梦想，展示她们的成就”时，不仅仅是身体力行。正因为如此，我想把这本书献给所有梦想着或者正在追求科技职业的女性和女孩们。只要你相信自己，没有什么能阻止你在这个行业实现梦想。

在我职业生涯中，有更多人支持和鼓励过我。感谢你们所有人。

我还要感谢克里斯作为一位有趣而富有洞见的合著者。从一开始，你总是坚持最高标准，推动我深入探索，鼓励我保持好奇心并多提问题。你帮助简化了我的代码，清晰地表达了我的思想，最终还接受了有争议的牛津逗号！
