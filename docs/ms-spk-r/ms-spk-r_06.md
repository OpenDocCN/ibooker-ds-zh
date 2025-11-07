# 第五章：管道

> 你再也不会步行，但你会飞！
> 
> —三眼乌鸦

在第四章中，您学习了如何使用 Spark 提供的高级功能以及与 Spark 良好配合的知名 R 包来构建预测模型。您首先了解了监督方法，然后通过原始文本完成了该章节的无监督方法。

在本章中，我们深入探讨了 Spark Pipelines，这是支持我们在第四章中演示功能的引擎。例如，当您通过 R 中的公式接口调用 MLlib 函数，例如`ml_logistic_regression(cars, am ~ .)`，则在幕后为您构建了一个*pipeline*。因此，管道还允许您利用高级数据处理和建模工作流程。此外，管道还通过允许您将管道部署到生产系统、Web 应用程序、移动应用程序等，促进了数据科学和工程团队之间的协作。

该章节也恰好是鼓励您将本地计算机作为 Spark 集群使用的最后一章。您只需再阅读一章，就可以开始执行能够扩展到最具挑战性计算问题的数据科学或机器学习。

# 概述

管道的构建模块是称为*transformers*和*estimators*的对象，它们被统称为*pipeline stages*。*Transformer*用于对 DataFrame 应用转换并返回另一个 DataFrame；结果 DataFrame 通常包含原始 DataFrame 及其附加的新列。另一方面，*estimator*可用于根据一些训练数据创建 transformer。考虑以下示例来说明这种关系：一个“中心和缩放”的 estimator 可以学习数据的均值和标准差，并将统计信息存储在生成的 transformer 对象中；然后可以使用此 transformer 来规范化训练数据以及任何新的未见数据。

下面是定义 estimator 的示例：

```
library(sparklyr)
library(dplyr)

sc <- spark_connect(master = "local", version = "2.3")

scaler <- ft_standard_scaler(
  sc,
  input_col = "features",
  output_col = "features_scaled",
  with_mean = TRUE)

scaler
```

```
StandardScaler (Estimator)
<standard_scaler_7f6d46f452a1>
 (Parameters -- Column Names)
  input_col: features
  output_col: features_scaled
 (Parameters)
  with_mean: TRUE
  with_std: TRUE
```

现在我们可以创建一些数据（我们知道其均值和标准差），然后使用`ml_fit()`函数将我们的缩放模型拟合到数据中：

```
df <- copy_to(sc, data.frame(value = rnorm(100000))) %>%
  ft_vector_assembler(input_cols = "value", output_col = "features")

scaler_model <- ml_fit(scaler, df)
scaler_model
```

```
StandardScalerModel (Transformer)
<standard_scaler_7f6d46f452a1>
 (Parameters -- Column Names)
  input_col: features
  output_col: features_scaled
 (Transformer Info)
  mean:  num 0.00421
  std:  num 0.999
```

###### 注意

在 Spark ML 中，许多算法和特征转换器要求输入为向量列。函数`ft_vector_assembler()`执行此任务。您还可以使用该函数初始化一个用于 pipeline 中的 transformer。

我们看到均值和标准差非常接近 0 和 1，这是我们预期的结果。然后我们可以使用 transformer 来*transform*一个 DataFrame，使用`ml_transform()`函数：

```
scaler_model %>%
  ml_transform(df) %>%
  glimpse()
```

```
Observations: ??
Variables: 3
Database: spark_connection
$ value           <dbl> 0.75373300, -0.84207731, 0.59365113, -…
$ features        <list> [0.753733, -0.8420773, 0.5936511, -0.…
$ features_scaled <list> [0.7502211, -0.8470762, 0.58999, -0.4…
```

现在您已经看到了 estimators 和 transformers 的基本示例，我们可以继续讨论管道。

# 创建

*管道*只是一系列转换器和评估器，*管道模型*是在数据上训练过的管道，因此其所有组件都已转换为转换器。

在`sparklyr`中构建管道的几种方法，都使用`ml_pipeline()`函数。

我们可以用`ml_pipeline(sc)`初始化一个空管道，并将阶段追加到其中：

```
ml_pipeline(sc) %>%
  ft_standard_scaler(
    input_col = "features",
    output_col = "features_scaled",
    with_mean = TRUE)
```

```
Pipeline (Estimator) with 1 stage
<pipeline_7f6d6a6a38ee>
  Stages
  |--1 StandardScaler (Estimator)
  |    <standard_scaler_7f6d63bfc7d6>
  |     (Parameters -- Column Names)
  |      input_col: features
  |      output_col: features_scaled
  |     (Parameters)
  |      with_mean: TRUE
  |      with_std: TRUE
```

或者，我们可以直接将阶段传递给`ml_pipeline()`：

```
pipeline <- ml_pipeline(scaler)
```

我们像拟合评估器一样拟合管道：

```
pipeline_model <- ml_fit(pipeline, df)
pipeline_model
```

```
PipelineModel (Transformer) with 1 stage
<pipeline_7f6d64df6e45>
  Stages
  |--1 StandardScalerModel (Transformer)
  |    <standard_scaler_7f6d46f452a1>
  |     (Parameters -- Column Names)
  |      input_col: features
  |      output_col: features_scaled
  |     (Transformer Info)
  |      mean:  num 0.00421
  |      std:  num 0.999
```

```
pipeline
```

###### 注意

由于 Spark ML 的设计，管道始终是评估器对象，即使它们只包含转换器。这意味着，如果您有一个只包含转换器的管道，仍然需要对其调用`ml_fit()`来获取转换器。在这种情况下，“拟合”过程实际上不会修改任何转换器。

# 使用案例

现在您已经了解了 ML Pipelines 的基本概念，让我们将其应用于前一章节中的预测建模问题，我们试图通过查看其档案来预测人们当前是否就业。我们的起点是具有相关列的`okc_train` DataFrame。

```
okc_train <- spark_read_parquet(sc, "data/okc-train.parquet")

okc_train <- okc_train %>%
  select(not_working, age, sex, drinks, drugs, essay1:essay9, essay_length)
```

我们首先展示管道，包括特征工程和建模步骤，然后逐步介绍它：

```
pipeline <- ml_pipeline(sc) %>%
  ft_string_indexer(input_col = "sex", output_col = "sex_indexed") %>%
  ft_string_indexer(input_col = "drinks", output_col = "drinks_indexed") %>%
  ft_string_indexer(input_col = "drugs", output_col = "drugs_indexed") %>%
  ft_one_hot_encoder_estimator(
    input_cols = c("sex_indexed", "drinks_indexed", "drugs_indexed"),
    output_cols = c("sex_encoded", "drinks_encoded", "drugs_encoded")
  ) %>%
  ft_vector_assembler(
    input_cols = c("age", "sex_encoded", "drinks_encoded",
                   "drugs_encoded", "essay_length"),
    output_col = "features"
  ) %>%
  ft_standard_scaler(input_col = "features", output_col = "features_scaled",
                     with_mean = TRUE) %>%
  ml_logistic_regression(features_col = "features_scaled",
                         label_col = "not_working")
```

前三个阶段索引`sex`、`drinks`和`drugs`列，这些列是字符型，通过`ft_string_indexer()`将它们转换为数值索引。这对于接下来的`ft_one_hot_encoder_estimator()`是必要的，后者需要数值列输入。当所有预测变量都是数值类型时（回想一下`age`已经是数值型），我们可以使用`ft_vector_assembler()`创建特征向量，它将所有输入连接到一个向量列中。然后，我们可以使用`ft_standard_scaler()`来归一化特征列的所有元素（包括分类变量的独热编码 0/1 值），最后通过`ml_logistic_regression()`应用逻辑回归。

在原型设计阶段，您可能希望将这些转换*急切地*应用于数据的一个小子集，通过将 DataFrame 传递给`ft_`和`ml_`函数，并检查转换后的 DataFrame。即时反馈允许快速迭代想法；当您已经找到所需的处理步骤时，可以将它们整合成一个管道。例如，您可以执行以下操作：

```
okc_train %>%
  ft_string_indexer("sex", "sex_indexed") %>%
  select(sex_indexed)
```

```
# Source: spark<?> [?? x 1]
   sex_indexed
         <dbl>
 1           0
 2           0
 3           1
 4           0
 5           1
 6           0
 7           0
 8           1
 9           1
10           0
# … with more rows
```

找到数据集的适当转换后，您可以用`ml_pipeline(sc)`替换 DataFrame 输入，结果将是一个可以应用于具有适当架构的任何 DataFrame 的管道。在下一节中，我们将看到管道如何使我们更容易测试不同的模型规格。

## 超参数调整

回到我们之前创建的管道，我们可以使用 `ml_cross_validator()` 来执行我们在前一章中演示的交叉验证工作流，并轻松测试不同的超参数组合。在这个例子中，我们测试了是否通过中心化变量以及逻辑回归的各种正则化值来改进预测。我们定义交叉验证器如下：

```
cv <- ml_cross_validator(
  sc,
  estimator = pipeline,
  estimator_param_maps = list(
    standard_scaler = list(with_mean = c(TRUE, FALSE)),
    logistic_regression = list(
      elastic_net_param = c(0.25, 0.75),
      reg_param = c(1e-2, 1e-3)
    )
  ),
  evaluator = ml_binary_classification_evaluator(sc, label_col = "not_working"),
  num_folds = 10)
```

`estimator` 参数就是我们要调整的估算器，本例中是我们定义的 `pipeline`。我们通过 `estimator_param_maps` 参数提供我们感兴趣的超参数值，它接受一个嵌套的命名列表。第一级的名称对应于我们想要调整的阶段的唯一标识符（UID），这些 UID 是与每个管道阶段对象关联的唯一标识符（如果提供部分 UID，`sparklyr` 将尝试将其与管道阶段匹配），第二级的名称对应于每个阶段的参数。在上面的片段中，我们正在指定我们要测试以下内容：

标准缩放器

值 `TRUE` 和 `FALSE` 对于 `with_mean`，它表示预测值是否被居中。

逻辑回归

值 `0.25` 和 `0.75` 对于 <math><mi>α</mi></math> ，以及值 `1e-2` 和 `1e-3` 对于 <math><mi>λ</mi></math> 。

我们预计这将产生 <math><mrow><mn>2</mn> <mo>×</mo> <mn>2</mn> <mo>×</mo> <mn>2</mn> <mo>=</mo> <mn>8</mn></mrow></math> 种超参数组合，我们可以通过打印 `cv` 对象来确认：

```
cv
```

```
CrossValidator (Estimator)
<cross_validator_d5676ac6f5>
 (Parameters -- Tuning)
  estimator: Pipeline
             <pipeline_d563b0cba31>
  evaluator: BinaryClassificationEvaluator
             <binary_classification_evaluator_d561d90b53d>
    with metric areaUnderROC
  num_folds: 10
  [Tuned over 8 hyperparameter sets]
```

与任何其他估算器一样，我们可以使用 `ml_fit()` 来拟合交叉验证器

```
cv_model <- ml_fit(cv, okc_train)
```

然后检查结果：

```
ml_validation_metrics(cv_model) %>%
  arrange(-areaUnderROC)
```

```
  areaUnderROC elastic_net_param_1 reg_param_1 with_mean_2
1    0.7722700                0.75       0.001        TRUE
2    0.7718431                0.75       0.010       FALSE
3    0.7718350                0.75       0.010        TRUE
4    0.7717677                0.25       0.001        TRUE
5    0.7716070                0.25       0.010        TRUE
6    0.7715972                0.25       0.010       FALSE
7    0.7713816                0.75       0.001       FALSE
8    0.7703913                0.25       0.001       FALSE
```

现在我们已经看到管道 API 在操作中的样子，让我们更正式地讨论它们在各种上下文中的行为。

# 操作模式

到目前为止，您可能已经注意到管道阶段函数（如 `ft_string_indexer()` 和 `ml_logistic_regression()`）根据传递给它们的第一个参数返回不同类型的对象。表 5-1 展示了完整的模式。

表 5-1\. 机器学习函数中的操作模式

| 第一个参数 | 返回值 | 示例 |
| --- | --- | --- |
| Spark 连接 | 估算器或转换器对象 | `ft_string_indexer(sc)` |
| 管道 | 管道 | `ml_pipeline(sc) %>% ft_string_indexer()` |
| 不带公式的 DataFrame | DataFrame | `ft_string_indexer(iris, "Species", "indexed")` |
| 带公式的 DataFrame | sparklyr ML 模型对象 | `ml_logistic_regression(iris, Species ~ .)` |

这些函数是使用 [S3](https://adv-r.hadley.nz/s3.html) 实现的，它是 R 提供的最流行的面向对象编程范式。对于我们的目的，知道 `ml_` 或 `ft_` 函数的行为由提供的第一个参数的类别决定就足够了。这使我们能够提供广泛的功能而不引入额外的函数名称。现在我们可以总结这些函数的行为：

+   如果提供了 Spark 连接，则该函数返回一个转换器或估计器对象，可以直接使用`ml_fit()`或`ml_transform()`，也可以包含在管道中。

+   如果提供了管道，则该函数返回一个具有附加到其中的阶段的管道对象。

+   如果将 DataFrame 提供给特征转换器函数（带有前缀`ft_`）或不提供公式就提供 ML 算法，则该函数实例化管道阶段对象，如果必要的话（如果阶段是估计器），将其拟合到数据，然后转换 DataFrame 并返回一个 DataFrame。

+   如果将 DataFrame 和公式提供给支持公式接口的 ML 算法，`sparklyr`在后台构建一个管道模型，并返回一个包含附加元数据信息的 ML 模型对象。

公式接口方法是我们在第四章中学习的内容，也是我们建议刚接触 Spark 的用户从这里开始的原因，因为它的语法类似于现有的 R 建模包，并且可以摆脱一些 Spark ML 的特殊性。然而，要充分利用 Spark ML 的全部功能并利用管道进行工作流组织和互操作性，学习 ML 管道 API 是值得的。

掌握了管道的基础知识后，我们现在可以讨论在本章引言中提到的协作和模型部署方面的内容。

# 互操作性

管道最强大的一点之一是它们可以序列化到磁盘，并且与其他 Spark API（如 Python 和 Scala）完全兼容。这意味着您可以轻松地在使用不同语言的 Spark 用户之间共享它们，这些用户可能包括其他数据科学家、数据工程师和部署工程师。要保存管道模型，请调用`ml_save()`并提供路径：

```
model_dir <- file.path("spark_model")
ml_save(cv_model$best_model, model_dir, overwrite = TRUE)
```

```
Model successfully saved.
```

让我们看一下刚写入的目录：

```
list.dirs(model_dir,full.names = FALSE) %>%
  head(10)
```

```
 [1] ""
 [2] "metadata"
 [3] "stages"
 [4] "stages/0_string_indexer_5b42c72817b"
 [5] "stages/0_string_indexer_5b42c72817b/data"
 [6] "stages/0_string_indexer_5b42c72817b/metadata"
 [7] "stages/1_string_indexer_5b423192b89f"
 [8] "stages/1_string_indexer_5b423192b89f/data"
 [9] "stages/1_string_indexer_5b423192b89f/metadata"
[10] "stages/2_string_indexer_5b421796e826"
```

我们可以深入几个文件，看看保存了什么类型的数据：

```
spark_read_json(sc, file.path(
  file.path(dir(file.path(model_dir, "stages"),
                pattern = "1_string_indexer.*",
                full.names = TRUE), "metadata")
)) %>%
  glimpse()
```

```
Observations: ??
Variables: 5
Database: spark_connection
$ class        <chr> "org.apache.spark.ml.feature.StringIndexerModel"
$ paramMap     <list> [["error", "drinks", "drinks_indexed", "frequencyDesc"]]
$ sparkVersion <chr> "2.3.2"
$ timestamp    <dbl> 1.561763e+12
$ uid          <chr> "string_indexer_ce05afa9899"
```

```
spark_read_parquet(sc, file.path(
  file.path(dir(file.path(model_dir, "stages"),
                pattern = "6_logistic_regression.*",
                full.names = TRUE), "data")
))
```

```
# Source: spark<data> [?? x 5]
  numClasses numFeatures interceptVector coefficientMatr… isMultinomial
       <int>       <int> <list>          <list>           <lgl>
1          2          12 <dbl [1]>       <-1.27950828662… FALSE
```

我们看到已导出了相当多的信息，从`dplyr`转换器中的 SQL 语句到逻辑回归的拟合系数估计。然后（在一个新的 Spark 会话中），我们可以使用`ml_load()`来重建模型：

```
model_reload <- ml_load(sc, model_dir)
```

让我们看看是否可以从这个管道模型中检索逻辑回归阶段：

```
ml_stage(model_reload, "logistic_regression")
```

```
LogisticRegressionModel (Transformer)
<logistic_regression_5b423b539d0f>
 (Parameters -- Column Names)
  features_col: features_scaled
  label_col: not_working
  prediction_col: prediction
  probability_col: probability
  raw_prediction_col: rawPrediction
 (Transformer Info)
  coefficient_matrix:  num [1, 1:12] -1.2795 -0.0915 0 0.126 -0.0324 ...
  coefficients:  num [1:12] -1.2795 -0.0915 0 0.126 -0.0324 ...
  intercept:  num -2.79
  intercept_vector:  num -2.79
  num_classes:  int 2
  num_features:  int 12
  threshold:  num 0.5
  thresholds:  num [1:2] 0.5 0.5
```

请注意，导出的 JSON 和 parquet 文件与导出它们的 API 无关。这意味着在多语言机器学习工程团队中，您可以从使用 Python 的数据工程师那里获取数据预处理管道，构建一个预测模型，然后将最终管道交给使用 Scala 的部署工程师。在下一节中，我们将更详细地讨论模型的部署。

###### 注意

当为使用公式接口创建的`sparklyr` ML 模型调用`ml_save()`时，关联的管道模型将被保存，但不会保存任何`sparklyr`特定的元数据，如索引标签。换句话说，保存一个`sparklyr`的`ml_model`对象，然后加载它将产生一个管道模型对象，就像您通过 ML 流水线 API 创建它一样。这种行为对于在其他编程语言中使用流水线是必需的。

在我们继续讨论如何在生产环境中运行流水线之前，请确保断开与 Spark 的连接：

```
spark_disconnect(sc)
```

这样，我们可以从全新的环境开始，这也是在部署流水线时预期的情况。

# 部署

我们刚刚展示的值得强调的是：通过在 ML 流水线框架内协作，我们减少了数据科学团队中不同角色之间的摩擦。特别是，我们缩短了从建模到部署的时间。

在许多情况下，数据科学项目并不仅仅以幻灯片展示洞见和建议而告终。相反，手头的业务问题可能需要按计划或按需实时评分新数据点。例如，银行可能希望每晚评估其抵押贷款组合的风险或即时决策信用卡申请。将模型转化为其他人可以消费的服务的过程通常称为*部署*或*产品化*。历史上，在构建模型的分析师和部署模型的工程师之间存在很大的鸿沟：前者可能在 R 中工作，并且开发了关于评分机制的详尽文档，以便后者可以用 C++或 Java 重新实现模型。这种做法在某些组织中可能需要数月时间，但在今天已经不那么普遍，而且在 Spark ML 工作流中几乎总是不必要的。

上述的夜间投资组合风险和信用申请评分示例代表了两种机器学习部署模式，称为*批处理*和*实时*。粗略地说，批处理意味着同时处理许多记录，执行时间不重要，只要合理（通常在几分钟到几小时的范围内）。另一方面，实时处理意味着一次评分一个或几个记录，但延迟至关重要（在小于 1 秒的范围内）。现在让我们看看如何将我们的`OKCupid`流水线模型带入“生产”环境。

## 批量评分

对于批处理和实时两种评分方法，我们将以 Web 服务的形式公开我们的模型，通过超文本传输协议（HTTP）的 API 提供。这是软件进行通信的主要媒介。通过提供 API，其他服务或最终用户可以使用我们的模型，而无需了解 R 或 Spark。[`plumber`](https://www.rplumber.io/) R 包使我们可以通过注释我们的预测函数来轻松实现这一点。

您需要确保通过运行以下命令安装`plumber`、`callr`和`httr`包：

```
install.packages(c("plumber", "callr", "httr"))
```

`callr` 包支持在单独的 R 会话中运行 R 代码；虽然不是必需的，但我们将使用它来在后台启动 Web 服务。`httr` 包允许我们从 R 使用 Web API。

在批处理评分用例中，我们只需初始化一个 Spark 连接并加载保存的模型。将以下脚本保存为 *plumber/spark-plumber.R*：

```
library(sparklyr)
sc <- spark_connect(master = "local", version = "2.3")

spark_model <- ml_load(sc, "spark_model")

#* @post /predict
score_spark <- function(age, sex, drinks, drugs, essay_length) {
  new_data <- data.frame(
    age = age,
    sex = sex,
    drinks = drinks,
    drugs = drugs,
    essay_length = essay_length,
    stringsAsFactors = FALSE
  )
  new_data_tbl <- copy_to(sc, new_data, overwrite = TRUE)

  ml_transform(spark_model, new_data_tbl) %>%
    dplyr::pull(prediction)
}
```

然后我们可以通过执行以下操作来初始化服务：

```
service <- callr::r_bg(function() {
  p <- plumber::plumb("plumber/spark-plumber.R")
  p$run(port = 8000)
})
```

这会在本地启动 Web 服务，然后我们可以用新数据查询服务进行评分；但是，您可能需要等待几秒钟以便 Spark 服务初始化：

```
httr::content(httr::POST(
  "http://127.0.0.1:8000/predict",
  body = '{"age": 42, "sex": "m", "drinks": "not at all",
 "drugs": "never", "essay_length": 99}'
))
```

```
[[1]]
[1] 0
```

此回复告诉我们，这个特定的配置文件可能不会是失业的，即是受雇用的。现在我们可以通过停止 `callr` 服务来终止 `plumber` 服务：

```
service$interrupt()
```

如果我们计时这个操作（例如使用 `system.time()`），我们会发现延迟在数百毫秒的数量级上，这对于批处理应用可能是合适的，但对于实时应用来说不够。主要瓶颈是将 R DataFrame 序列化为 Spark DataFrame，然后再转换回来。此外，它需要一个活跃的 Spark 会话，这是一个重量级的运行时要求。为了改善这些问题，接下来我们讨论一个更适合实时部署的部署方法。

## 实时评分

对于实时生产，我们希望尽可能保持依赖项轻量化，以便可以针对更多平台进行部署。现在我们展示如何使用 [`mleap`](http://bit.ly/2Z7jgSV) 包，它提供了一个接口给 [MLeap](http://bit.ly/33G271R) 库，用于序列化和提供 Spark ML 模型。MLeap 是开源的（Apache License 2.0），支持广泛的 Spark ML 转换器，尽管不是所有。在运行时，环境的唯一先决条件是 Java 虚拟机（JVM）和 MLeap 运行时库。这避免了 Spark 二进制文件和昂贵的将数据转换为 Spark DataFrames 的开销。

由于 `mleap` 是 `sparklyr` 的扩展和一个 R 包，因此我们首先需要从 CRAN 安装它：

```
install.packages("mleap")
```

当调用 `spark_connect()` 时，必须加载它；所以让我们重新启动您的 R 会话，建立一个新的 Spark 连接，并加载我们之前保存的管道模型：

```
library(sparklyr)
library(mleap)
```

```
sc <- spark_connect(master = "local", version = "2.3")
```

```
spark_model <- ml_load(sc, "spark_model")
```

保存模型到 MLeap bundle 格式的方法与使用 Spark ML Pipelines API 保存模型非常相似；唯一的附加参数是 `sample_input`，它是一个具有我们期望对新数据进行评分的模式的 Spark DataFrame：

```
sample_input <- data.frame(
  sex = "m",
  drinks = "not at all",
  drugs = "never",
  essay_length = 99,
  age = 25,
  stringsAsFactors = FALSE
)

sample_input_tbl <- copy_to(sc, sample_input)

ml_write_bundle(spark_model, sample_input_tbl, "mleap_model.zip", overwrite =
TRUE)
```

现在我们可以在运行 Java 并具有开源 MLeap 运行时依赖项的任何设备上部署我们刚刚创建的文件 **mleap_model.zip**，而不需要 Spark 或 R！事实上，我们可以继续断开与 Spark 的连接：

```
spark_disconnect(sc)
```

在使用此 MLeap 模型之前，请确保安装了运行时依赖项：

```
mleap::install_maven()
mleap::install_mleap()
```

要测试此模型，我们可以创建一个新的 plumber API 来公开它。脚本 *plumber/mleap-plumber.R* 与前面的示例非常相似：

```
library(mleap)

mleap_model <- mleap_load_bundle("mleap_model.zip")

#* @post /predict
score_spark <- function(age, sex, drinks, drugs, essay_length) {
  new_data <- data.frame(
    age = as.double(age),
    sex = sex,
    drinks = drinks,
    drugs = drugs,
    essay_length = as.double(essay_length),
    stringsAsFactors = FALSE
  )
  mleap_transform(mleap_model, new_data)$prediction
}
```

启动服务的方式也完全相同：

```
service <- callr::r_bg(function() {
  p <- plumber::plumb("plumber/mleap-plumber.R")
  p$run(port = 8000)
})
```

我们可以运行与之前相同的代码来测试这个新服务中的失业预测：

```
httr::POST(
  "http://127.0.0.1:8000/predict",
  body = '{"age": 42, "sex": "m", "drinks": "not at all",
 "drugs": "never", "essay_length": 99}'
) %>%
  httr::content()
```

```
[[1]]
[1] 0
```

如果我们计时这个操作，我们会看到现在服务在几十毫秒内返回预测结果。

让我们停止这项服务，然后结束本章：

```
service$interrupt()
```

# 总结

在本章中，我们讨论了 Spark 管道，这是引擎在第四章介绍的建模功能背后的驱动力。您学会了通过将数据处理和建模算法组织到管道中来整理预测建模工作流程。您了解到管道还通过共享一种语言无关的序列化格式促进了多语言数据科学和工程团队成员之间的协作——您可以从 R 导出一个 Spark 管道，让其他人在 Python 或 Scala 中重新加载您的管道到 Spark 中，这使他们可以在不改变自己选择的语言的情况下进行协作。

您还学会了如何使用`mleap`部署管道，这是一个提供另一种将 Spark 模型投入生产的 Java 运行时——您可以导出管道并将其集成到支持 Java 的环境中，而不需要目标环境支持 Spark 或 R。

你可能已经注意到，一些算法，尤其是无监督学习类型的算法，对于可以加载到内存中的`OKCupid`数据集来说，速度较慢。如果我们能够访问一个合适的 Spark 集群，我们就可以花更多时间建模，而不是等待！不仅如此，我们还可以利用集群资源来运行更广泛的超参数调整作业和处理大型数据集。为了达到这个目标，第六章介绍了计算集群的具体内容，并解释了可以考虑的各种选项，如建立自己的集群或按需使用云集群。

¹ 截至本书撰写时，MLeap 不支持 Spark 2.4。
