# 附录 E. 本书使用的包

R 从无私的作者贡献中获得了其广度和力量的很大一部分。表 E.1 列出了本书中描述的用户贡献包，以及它们出现的章节。一些包的作者太多，无法在此列出。此外，许多包都得到了贡献者的增强。有关详细信息，请参阅包文档。

表 E.1 本书使用的贡献包

| 包 | 作者 | 描述 | 章节 |
| --- | --- | --- | --- |
| `AER` | Christian Kleiber 和 Achim Zeileis | 来自 Christian Kleiber 和 Achim Zeileis 的《使用 R 的应用计量经济学》一书中的函数、数据集、示例、演示和示例 | 13 |
| `boot` | S 原版由 Angelo Canty 提供，R 版由 Brian Ripley 提供。 | Bootstrap 函数 | 12 |
| `bootstrap` | S 原版由 Rob Tibshirani 提供，R 版由 Friedrich Leisch 转译。 | 软件和来自 B. Efron 和 R. Tibshirani 的《Bootstrap 简介》的数据（Chapman and Hall, 1993）（bootstrap、交叉验证、Jackknife） | 8 |
| `broom` | David Robinson, Alex Hayes 和 Simon Couch | 总结 tidy tibbles 中的关键信息 | 21 |
| `car` | John Fox, Sanford Weisberg 和 Brad Price | 伴随应用回归的函数 | 8, 9, 11 |
| `carData` | John Fox, Sanford Weisberg 和 Brad Price | 伴随应用回归的数据集 | 7 |

表 E.2 本书使用的贡献包

| 包 | 作者 | 描述 | 章节 |
| --- | --- | --- | --- |
| `cluster` | Martin Maechler, Peter Rousseeuw（Fortran 原版），Anja Struyf（S 原版）和 Mia Hubert（S 原版） | 聚类分析的方法 | 16 |
| `clusterability` | Zachariah Neville, Naomi Brownstein, Maya Ackerman 和 Andreas Adolfsson | 执行数据集聚类趋势的测试 | 16 |
| `coin` | Torsten Hothorn, Kurt Hornik, Mark A. van de Wiel 和 Achim Zeileis | 在排列检验框架中的条件推断程序 | 12 |
| `colorhcplot` | Damiano Fantini | 使用颜色突出显示的组构建树状图 | 16 |
| `corrgram` | Kevin Wright | 绘制 corrgram | 11 |
| `DALEX` | Przemyslaw Biecek, Szymon Maksymiuk 和 Hubert Baniecki | 用于探索和解释的模型无关语言 | 17 |
| `devtools` | Hadley Wickham, Jim Hester 和 Winston Chang | 使开发 R 包更容易的工具 | 22 |
| `directlabels` | Toby Dylan Hocking | 多颜色图的直接标签 | 15 |
| `doParallel` | Michelle Wallig, 微软公司和 Steve Weston | `foreach` 并行适配器，用于 `parallel` 包 | 20 |
| `dplyr` | Hadley Wickham, Romain François, Lionel Henry 和 Kirill Müller | 数据操作语法的语法 | 3, 5, 6, 7, 9, 16, 19, 21 |
| `e1071` | David Meyer, Evgenia Dimitriadou, Kurt Hornik, Andreas Weingessel 和 Friedrich Leisch | 维也纳科技大学统计学、概率论小组的杂项函数 | 17 |
| `effects` | John Fox 和 Jangman Hong | 线性、广义线性、多项 logit 和比例优势 logit 模型的效应显示 | 8, 9 |
| `factoextra` | Alboukadel Kassambara | 提取和可视化多元数据分析的结果 | 16 |
| `flexclust` | Friedrich Leish 和 Evgenia Dimnitriadou | 灵活的聚类算法 | 16 |
| `foreach` | Michelle Wallig, 微软公司, 和 Steve Weston | 为 R 提供 foreach 循环结构 | 20 |
| `forecast` | Rob J. Hyndman 和许多其他作者 | 时间序列和线性模型的预测函数 | 15 |
| `gapminder` | Jennifer Bryan | 来自 [Gapminder.org](http://gapminder.org/) 的数据摘录 | 19 |
| `Ggally` | Barret Schloerke 和许多其他作者 | 扩展 `ggplot2` 的功能 | 11 |
| `ggdendro` | Andrie de Vries 和 Brian D. Ripley | 使用 `ggplot2` 创建树状图和树形图 | 16 |
| `ggm` | Giovanni M. Marchetti, Mathias Drton 和 Kayvan Sadeghi | 用于边缘化、条件化和最大似然拟合的工具 | 7 |
| `ggplot2` | Hadley Wickam 和许多其他作者 | 图形语法的实现 | 4, 6, 9, 10, 11, 12, 15, 16, 18, 19, 20, 21 |
| `ggrepel` | Kamil Slowikowski | 使用 `ggplot2` 自动定位非重叠文本标签 | 19 |
| `gmodels` | Gregory R. Warnes；包括 Ben Bolker、Thomas Lumley 和 Randall C. Johnson 贡献的 R 源代码和/或文档。Randall C. Johnson 的贡献受版权保护（SAIC-Frederick, Inc., 2005）。 | 用于模型拟合的各种 R 编程工具 | 7 |
| `haven` | Hadley Wickham 和 Evan Miller | 导入和导出 SPSS、Stata 和 SAS 文件 | 2 |
| `Hmisc` | Frank E. Harrell, Jr. | 数据分析、高级图形、实用操作等方面的杂项函数 | 7 |
| `ISLR` | Gareth James, Daniela Witten, Trevor Hastie 和 Rob Tibshirani | 《统计学习导论及其在 R 中的应用》的数据 | 19 |
| `kableExtra` | Hao Zhu | 构建复杂的 HTML 和 LaTeX 表格 | 21 |
| `knitr` | Yihui Xie | R 中动态报告生成的通用包 | 21 |
| `leaps` | Thomas Lumley，使用 Alan Miller 的 Fortran 代码 | 回归子集选择，包括穷举搜索 | 8 |
| `lmPerm` | Bob Wheeler | 线性模型的排列检验 | 12 |
| `MASS` | 由 Venables 和 Ripley 创作的原版；R 版由 Brian Ripley 转换，遵循 Kurt Hornik 和 Albrecht Gebhardt 早期的工作 | 支持 Venables 和 Ripley 的《S 的现代应用统计》第四版（Springer，2003）的函数和数据集 | 7, 9, 12, 13, 14, 附录 D |
| `mice` | Stef van Buuren 和 Karin Groothuis-Oudshoorn | 通过链式方程进行多元插补 | 18 |
| `mosaicData` | Randall Pruim, Daniel Kaplan 和 Nicholas Horton | 来自 Project MOSAIC 的数据集 | 4 |
| `multcomp` | 托尔斯坦·霍恩、弗兰克·布雷茨·彼得·韦斯特法尔、理查德·M·海伯格和安德烈·舒特岑迈斯特 | 在参数模型中，包括线性、广义线性、线性混合效应和生存模型中的一般线性假设的同步测试和置信区间 | 9, 12, 21 |
| `MultiRNG` | 哈坎·德米塔斯、拉万·阿洛齐和兰·高 | 11 个多元分布的伪随机数生成 | 5 |
| `mvoutlier` | 莫里茨·格施万特纳和彼得·菲尔茨莫瑟 | 基于稳健方法的多元异常值检测 | 9 |
| `naniar` | 尼古拉斯·蒂尔尼、迪·库克、迈尔斯·麦克贝恩和科林·费 | 缺失数据的结构、摘要和可视化 | 18 |
| `NbClust` | 马利卡·查拉德、纳迪亚·加扎利、维罗尼克·博特和阿扎姆·尼卡法斯 | 确定聚类数量的指标研究 | 16 |
| `partykit` | 托尔斯坦·霍恩、海迪·索贝尔和阿奇姆·泽莱伊斯 | 递归分割工具包 | 17 |
| `pastecs` | 弗雷德里克·伊班内斯、菲利普·格罗斯让和米歇尔·埃蒂安 | 空间时间生态序列分析包 | 7 |
| `patchwork` | 托马斯·林·佩德森 | 由多个图表组成的组合 | 19 |
| `pkgdown` | 霍德尔·威克汉姆和杰伊·赫塞尔贝斯 | 为包创建吸引人的 HTML 文档和网站 | 22 |
| `plotly` | 卡森·西维特和许多其他作者 | 通过 plotly.js 创建交互式网络图形 | 19 |
| `psych` | 威廉·雷维尔 | 心理学、心理测量和人格研究程序 | 7, 14 |
| `pwr` | 斯蒂芬·尚佩利 | 功率分析的基本函数 | 10 |
| `qcc` | 卢卡·斯库尔卡 | 质量控制图 | 13 |
| `randomForest` | 最初由 Fortran 编写，作者为 Leo Breiman 和 Adele Cutler；R 语言版本由 Andy Liaw 和 Matthew Wiener 移植 | Breiman 和 Cutler 的随机森林用于分类和回归 | 17 |
| `rattle` | 格雷厄姆·威廉姆斯、马克·韦尔·卡尔普、埃德·科克斯、安东尼·诺兰、丹尼斯·怀特、达尼埃莱·梅德里、阿卡巴尔·瓦利杰（随机森林的 OOB AUC）和布莱恩·里普利（[print.summary.nnet](http://print.summary.nnet)的原始作者） | R 语言数据挖掘的图形用户界面 | 16, 17 |
| `readr` | 霍德尔·威克汉姆和吉姆·赫斯特 | 灵活导入矩形文本数据 | 2 |
| `readxl` | 霍德尔·威克汉姆和詹妮弗·布莱恩 | 导入 Excel 文件 | 2 |
| `rgl` | 丹尼尔·阿德勒和邓肯·默多克 | 3D 可视化设备系统（OpenGL） | 11 |
| `rmarkdown` | JJ·阿莱尔、叶辉和许多其他作者 | 将 R Markdown 文档转换为各种文档 | 21 |
| `robustbase` | 马丁·迈歇勒和许多其他作者 | 使用稳健方法分析数据的工具 | 13 |
| `rpart` | 特里·瑟诺、贝丝·阿特金森和布莱恩·里普利（R 语言初始版本作者） | 递归分割和回归树 | 17 |
| `rrcov` | 瓦伦丁·托多罗夫 | 具有高破坏点的稳健位置和散布估计以及稳健的多变量分析 | 9 |
| `scales` | 霍德尔·威克汉姆和达娜·塞德尔 | 数据可视化的缩放函数 | 6, 19 |
| `scatterplot3d` | Uwe Ligges | 绘制三维（3D）点云图 | 11 |
| `showtext` | Yixuan Qiu 和其他多位作者 | 在 R 图中使用各种类型的字体 | 19 |
| `sqldf` | G. Grothendieck | 使用 SQL 操作 R 数据框 | 3 |
| `tidyquant` | Matt Dancho 和 Davis Vaughan | 定量金融分析的整洁工具 | 21 |
| `tidyr` | Hadley Wickham | 整洁混乱数据的工具，包括数据旋转、嵌套和去嵌套 | 5, 16, 20 |
| `treemapify` | David Wilkins | 提供 `ggplot2` 地图元素用于绘制树状图 | 6 |
| `tseries` | Adrian Trapletti 和 Kurt Hornik | 时间序列分析和计算金融 | 15 |
| `usethis` | Hadley Wickham 和 Jennifer Bryan | 自动化包和项目设置任务 | 22 |
| `vcd` | David Meyer, Achim Zeileis 和 Kurt Hornik | 可视化分类数据的功能 | 1, 6, 7, 11, 12 |
| `VIM` | Matthias Templ, Andreas Alfons 和 Alexander Kowarik | 可视化和缺失值插补 | 18 |
| `xtable` | David B. Dahl 和其他多位作者 | 将表格导出到 LaTeX 或 HTML | 21 |
| `xts` | Jeffrey A. Ryan 和 Joshua M. Ulrich | 统一处理不同基于时间的数据类别 | 15 |
