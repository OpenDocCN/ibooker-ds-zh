# 第七章：管道编排

前几章已经描述了数据管道的构建块，包括数据摄取、数据转换以及机器学习管道中的步骤。本章涵盖了如何“编排”或连接这些块或步骤。

编排确保管道中的步骤按正确顺序运行，并正确管理步骤之间的依赖关系。

当我在第二章介绍管道编排的挑战时，我还介绍了*工作流编排平台*（也称为*工作流管理系统*（WMSs）、*编排平台*或*编排框架*）的概念。本章将重点介绍 Apache Airflow，这是最流行的此类框架之一。尽管本章大部分内容都专注于 Airflow 中的示例，但这些概念也适用于其他框架。实际上，我稍后在本章中也提到了一些 Airflow 的替代方案。

最后，本章的后续部分讨论了管道编排中一些更高级的概念，包括在数据基础架构上协调多个管道。

## 有向无环图

虽然我在第二章介绍了 DAGs，但值得重复一下它们是什么。本章讨论了它们在 Apache Airflow 中的设计和实现，用于编排数据管道中的任务。

管道步骤（任务）始终是*有向*的，这意味着它们从一个或多个任务开始，并以特定的任务或任务结束。这是为了保证执行路径。换句话说，它确保任务在其所有依赖任务成功完成之前不会运行。

管道图必须也是*无环*的，这意味着一个任务不能指向先前已完成的任务。换句话说，它不能循环。如果可以的话，管道将无休止地运行！

您可能还记得来自第二章的 DAG 的以下示例，它在图 7-1 中有所说明。这是在 Apache Airflow 中定义的一个 DAG。

![dppr 0203](img/dppr_0203.png)

###### 图 7-1。一个具有四个任务的 DAG。在任务 A 完成后，任务 B 和任务 C 运行。当它们都完成时，任务 D 运行。

Airflow 中的任务可以代表从执行 SQL 语句到执行 Python 脚本的任何内容。正如您将在接下来的章节中看到的，Airflow 允许您定义、调度和执行数据管道中的任务，并确保它们按正确的顺序运行。

# Apache Airflow 设置和概述

Airflow 是一个由 Maxime Beauchemin 于 2014 年在 Airbnb 启动的开源项目。它于 2016 年 3 月加入了 Apache 软件基金会的孵化器计划。Airflow 的建立旨在解决数据工程团队面临的常见挑战：如何构建、管理和监控涉及多个具有相互依赖关系的任务的工作流程（特别是数据管道）。

自首次发布以来的六年里，Airflow 已成为数据团队中最受欢迎的工作流管理平台之一。其易于使用的 Web 界面、高级命令行实用程序、内置调度程序以及高度可定制化的特性意味着它几乎适用于任何数据基础设施。尽管是用 Python 构建的，但它可以执行任何语言或平台上运行的任务。事实上，尽管最常用于管理数据管道，但它实际上是一种用于编排任何类型依赖任务的通用平台。

###### 注意

本章中的代码示例和概述参考的是 Airflow 版本 1.x。Airflow 2.0 正在接近，承诺带来一些重大增强，如全新的 Web UI、改进的调度程序、全功能的 REST API 等。尽管本章具体内容涉及 Airflow 1.x，但这些概念在 Airflow 2.0 中仍然适用。此外，此处提供的代码旨在与 Airflow 2.0 兼容，几乎无需修改。

## 安装和配置

安装 Airflow 非常简单。您需要使用`pip`，这在“设置 Python 环境”中已介绍过。当您首次安装和启动 Airflow 时，会介绍其一些组件，如 Airflow 数据库、Web 服务器和调度程序。在接下来的章节中，我将定义每个组件及其如何进一步配置。

您可以按照官方[Airflow 快速入门指南](https://oreil.ly/_fGy8)的安装说明进行操作。这通常不到五分钟！

安装完 Airflow 并且 Web 服务器运行后，您可以在浏览器中访问[*http://localhost:8080*](http://localhost:8080)来查看 Airflow Web 界面。如果您想了解更多关于 Airflow 各组件及其配置的信息，本节其余部分将详细介绍每个组件。如果您准备构建您的第一个 Airflow DAG，可以直接跳转到“构建 Airflow DAGs”。

对于更高级的 Airflow 部署，建议查阅官方[Airflow 文档](https://oreil.ly/_VAXS)。

## Airflow 数据库

Airflow 使用数据库来存储与每个任务和 DAG 执行历史相关的所有元数据，以及您的 Airflow 配置。默认情况下，Airflow 使用 SQLite 数据库。在安装过程中运行`airflow initdb`命令时，Airflow 会为您创建一个 SQLite 数据库。对于学习 Airflow 或者小规模项目来说，这是可以接受的。然而，对于更大规模的需求，我建议使用 MySQL 或 Postgres 数据库。幸运的是，Airflow 在幕后使用备受推崇的`SqlAlchemy`库，并且可以轻松重新配置以使用这些数据库，而不是 SQLite。

要更改 Airflow 使用的数据库，请打开*airflow.cfg*文件，该文件位于安装期间用于`AIRFLOW_HOME`的路径中。在安装示例中，那是`~/airflow`。在文件中，您将看到一个用于`sql_alchemy_conn`配置的行。它看起来像这样：

```
# The SqlAlchemy connection string to the metadata database.
# SqlAlchemy supports many different database engine, more information
# their website
sql_alchemy_conn = sqlite:////Users/myuser/airflow/airflow.db
```

默认情况下，该值设置为本地 SQLite 数据库的连接字符串。在以下示例中，我将创建和配置一个 Postgres 数据库和用户供 Airflow 使用，然后配置 Airflow 以使用新的数据库而不是默认的 SQLite 数据库。

请注意，我假设您已经运行了一个 Postgres 服务器，并且有权限在`psql`中运行（Postgres 交互式终端）和创建数据库和用户。任何 Postgres 数据库都可以，但必须能够从安装 Airflow 的机器访问它。要了解有关安装和配置 Postgres 服务器的更多信息，请参阅[官方网站](https://www.postgresql.org)。您也可以在像 AWS 这样的平台上使用托管的 Postgres 实例。只要安装 Airflow 的机器能够访问它即可。

首先，在命令行上启动`psql`或以其他方式打开连接到您的 Postgres 服务器的 SQL 编辑器。

现在，创建一个用户供 Airflow 使用。为了简单起见，命名为`airflow`。此外，为用户设置密码：

```
CREATE USER airflow;
ALTER USER airflow WITH PASSWORD 'pass1';
```

接下来，为 Airflow 创建一个数据库。我将其命名为`airflowdb`：

```
CREATE DATABASE airflowdb;
```

最后，授予新用户在新数据库上的所有权限。Airflow 需要读取和写入数据库：

```
GRANT ALL PRIVILEGES
  ON DATABASE airflowdb TO airflow;
```

现在，您可以返回并修改*airflow.cfg*文件中的连接字符串。我假设您的 Postgres 服务器正在与 Airflow 运行在同一台机器上，但如果不是，则需要通过将`localhost`替换为 Postgres 运行的主机的完整路径来修改以下内容。完成后保存*airflow.cfg*：

```
sql_alchemy_conn = postgresql+psycopg2://airflow:pass1@localhost:5432/airflowdb
```

由于 Airflow 将通过 Python 连接到 Postgres 数据库，您还需要安装`psycopg2`库：

```
$ pip install psycopg2
```

最后，返回命令行，在 Postgres 中重新初始化 Airflow 数据库：

```
$ airflow initdb
```

从现在开始，您可以在 Postgres 服务器的`airflowdb`数据库中找到所有 Airflow 元数据。那里有大量信息，包括任务历史记录，可以查询。您可以直接从 Postrgres 数据库或 Airflow Web UI 中进行查询，如下一节所述。通过 SQL 查询数据使得分析和报告的机会无限。没有比使用默认情况下 Airflow 收集的数据更好的方法来分析您的流水线的性能！在本书的第十章中，我将讨论使用这些数据和其他数据来衡量和监控数据流水线的性能。

## Web 服务器和 UI

在安装后通过运行 `airflow webs erver -p 8080` 命令启动 Web 服务器后，您可能已经偷偷看过其内容。如果没有，请打开 Web 浏览器并导航至 *http://localhost:8080*。如果您使用的是全新安装的 Airflow，您将看到类似于 图 7-2 的页面。

![dppr 0702](img/dppr_0702.png)

###### 图 7-2\. Airflow Web UI。

Web UI 的首页显示了 DAG 列表。正如您所见，Airflow 包含一些示例 DAG。如果您对 Airflow 还不熟悉，这些示例是一个很好的起点。当您创建自己的 DAG 时，它们也会显示在这里。

页面上每个 DAG 都有许多链接和信息：

+   一个链接，用于打开 DAG 属性，包括源文件路径、标签、描述等信息。

+   切换按钮，用于启用和暂停 DAG。启用时，第四列中定义的计划决定运行时间。暂停时，计划被忽略，只能通过手动执行运行 DAG。

+   DAG 的名称，点击后会带您到 DAG 详细页面，如 图 7-3 所示。

+   DAG 在未暂停时运行的计划。以 [crontab 格式](https://oreil.ly/btt0G) 显示，并在 DAG 源文件中定义。

+   DAG 的所有者。通常情况下是 `airflow`，但在复杂的部署中，可能有多个所有者可供选择。

+   最近的任务，这是最新 DAG 运行的摘要。

+   最后一次 DAG 运行的时间戳。

+   先前 DAG 运行的摘要。

+   一组链接，指向各种 DAG 的配置和信息。如果点击 DAG 的名称，也会看到这些链接。

当您点击 DAG 的名称时，会跳转到 DAG 详细页面的树形视图，在 图 7-3 中展示。这是 Airflow 提供的示例 `example_python_operator` DAG。该 DAG 包含五个任务，全部是 `PythonOperators`（您将在本节后面学习有关运算符的内容）。在 `print_the_context` 任务成功完成后，会启动五个任务。任务完成后，DAG 运行完成。

![dppr 0703](img/dppr_0703.png)

###### 图 7-3\. Airflow DAG 的树形视图。

您也可以点击页面顶部的图形视图按钮，查看 DAG 的图形视图。我发现这个视图非常有用。您可以在 图 7-4 中看到这个特定 DAG 的图形表示。

在包含大量任务的复杂 DAG 中，图形视图可能在屏幕上显示时会有些难以看清。但请注意，您可以使用鼠标放大、缩小和滚动图形。

![dppr 0704](img/dppr_0704.png)

###### 图 7-4\. Airflow DAG 的图形视图。

屏幕上还有许多其他选项，其中许多是不言自明的。但我想重点介绍另外两个选项：代码和触发 DAG。

当您点击 Code 时，您当然会看到 DAG 背后的代码。您将注意到的第一件事是，DAG 是在 Python 脚本中定义的。在本例中，文件名为*example_python_operator.py*。稍后，您将了解有关 DAG 源文件结构的更多信息。目前，重要的是知道它保存了 DAG 的配置，包括其调度、每个任务的定义以及每个任务之间的依赖关系。

触发 DAG 按钮允许您按需执行 DAG。尽管 Airflow 是为了按计划运行 DAG 而构建的，在开发期间，在测试期间以及在生产环境中的非计划需求中，这是立即运行 DAG 的最简单方式。

除了管理 DAGs 之外，Web UI 的许多其他功能也会很有用。在顶部导航栏中，如果您点击数据分析，您将看到 Ad Hoc 查询、图表和已知事件的选项。在这里，您可以查询 Airflow 数据库中的信息，如果您不愿直接从其他工具连接到它。

在浏览下，您可以找到 DAG 的运行历史记录和其他日志文件，在管理员中，您可以找到各种配置设置。您可以在[官方 Airflow 文档](https://oreil.ly/OuUS_)中了解更多关于高级配置选项的信息。

## 调度器

Airflow 调度器是您在本章前面运行`airflow scheduler`命令时启动的服务。运行时，调度器不断监视 DAG 和任务，并运行已计划运行或已满足依赖关系的任务（对于 DAG 中的任务）。

调度器使用在*airflow.cfg*文件的`[core]`部分中定义的执行器来运行任务。您可以在以下部分了解更多关于执行器的信息。

## 执行器

*执行器*是 Airflow 用来运行调度器确定已准备运行的任务的工具。Airflow 支持多种类型的执行器。默认情况下，使用`SequentialExecutor`。您可以在*airflow.cfg*文件中更改执行器的类型。在文件的`core`部分下，您将看到一个`executor`变量，可以设置为此部分和 Airflow 文档中列出的任何执行器类型。正如您所看到的，当首次安装 Airflow 时设置了`SequentialExecutor`：

```
[core]
........

# The executor class that airflow should use. Choices include
# SequentialExecutor, LocalExecutor, CeleryExecutor, DaskExecutor, KubernetesExecutor
executor = SequentialExecutor
```

尽管默认情况下，`SequentialExecutor`并不适合生产环境，因为它只能同时运行一个任务。对于测试简单的 DAGs 是可以接受的，但仅限于此。然而，它是与 SQLite 数据库兼容的唯一执行器，因此如果您还没有配置 Airflow 与其他数据库，那么`SequentialExecutor`就是您唯一的选择。

如果你计划在任何规模上使用 Airflow，我建议你使用其他执行器，如`CeleryExecutor`、`DaskExecutor`或`KubernetesExecutor`。你的选择应该部分取决于你最熟悉的基础设施。例如，要使用`CeleryExecutor`，你需要设置一个 Celery 代理，使用 RabbitMQ、Amazon SQL 或 Redis。

配置每个执行器所需的基础设施超出了本书的范围，但本节中的示例即使在`SequentialExecutor`上也可以运行。你可以在[它们的文档](https://oreil.ly/YOplY)中了解更多关于 Airflow 执行器的信息。

## 运算符

请记住，DAG 中的每个节点都是一个任务。在 Airflow 中，每个任务都实现了一个*运算符*。运算符实际上执行脚本、命令和其他操作。这里有一些常见的运算符：

+   `BashOperator`

+   `PythonOperator`

+   `SimpleHttpOperator`

+   `EmailOperator`

+   `SlackAPIOperator`

+   `MySqlOperator`，`PostgresOperator`，以及其他特定于数据库的运算符，用于执行 SQL 命令。

+   `Sensor`

正如你将在接下来的章节中学到的那样，运算符被实例化并分配给 DAG 中的每个任务。

# 构建 Airflow 的 DAG。

现在你知道了 Airflow 的工作原理，是时候构建一个 DAG 了！虽然 Airflow 附带了一些示例 DAG，但我将按照本书之前的一些示例构建一个 DAG，执行一个示例 ELT 过程的步骤。具体来说，它将从数据库中提取数据，将其加载到数据仓库中，然后将数据转换为数据模型。

## 一个简单的 DAG

在构建示例 ELT DAG 之前，了解如何在 Airflow 中定义 DAG 是很重要的。DAG 在 Python 脚本中定义，其中结构和任务依赖关系都是用 Python 代码编写的。示例 7-1 是一个简单 DAG 的定义，包含三个任务。它被称为*DAG 定义文件*。每个任务都定义为一个`BashOperator`，第一个和第三个任务打印一些文本，第二个任务休眠三秒钟。虽然它没有做任何特别有用的事情，但它是完全功能的，代表了稍后你将编写的 DAG 定义。

##### 示例 7-1\. simple_dag.py

```
from datetime import timedelta
from airflow import DAG
from airflow.operators.bash_operator \
    import BashOperator
from airflow.utils.dates import days_ago

dag = DAG(
    'simple_dag',
    description='A simple DAG',
    schedule_interval=timedelta(days=1),
    start_date = days_ago(1),
)

t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag,
)

t2 = BashOperator(
    task_id='sleep',
    depends_on_past=False,
    bash_command='sleep 3',
    dag=dag,
)

t3 = BashOperator(
    task_id='print_end',
    depends_on_past=False,
    bash_command='echo \'end\'',
    dag=dag,
)

t1 >> t2
t2 >> t3
```

在你继续并运行 DAG 之前，我想指出 DAG 定义文件的关键特性。首先，像任何 Python 脚本一样，需要导入必要的模块。接下来，定义 DAG 本身并分配一些属性，如名称（`simple_dag`）、调度时间、开始日期等等。实际上，在这个简单示例中我没有定义的属性还有很多，你可能需要在本章节或官方 Airflow 文档中找到它们。

接下来，我定义了 DAG 中的三个任务。所有任务都是`BashOperator`类型，这意味着当执行时，它们将运行一个 bash 命令。每个任务还分配了几个属性，包括一个称为`task_id`的字母数字标识符，以及任务执行时运行的 bash 命令。正如稍后您将看到的那样，每种运算符类型都有其自定义属性，就像`BashOperator`有`bash_command`一样。

DAG 定义的最后两行定义了任务之间的依赖关系。阅读方法是，当任务`t1`完成时，`t2`运行。当`t2`完成时，`t3`运行。当您在 Airflow Web UI 中查看 DAG 时，您会在树形和图形视图中看到这一点。

要运行 DAG，您需要将其定义文件保存在 Airflow 查找 DAG 的位置。您可以在*airflow.cfg*文件中找到这个位置（或修改它）：

```
dags_folder = /Users/myuser/airflow/dags
```

将 DAG 定义保存在名为*simple_dag.py*的文件中，并将其放置在*dags_folder*位置。如果您已经运行 Airflow Web UI 和调度器，请刷新 Airflow Web UI，您应该在列表中看到名为`simple_dag`的 DAG。如果没有，请等待几秒钟然后重试，或者停止并重新启动 Web 服务。

接下来，点击 DAG 名称以查看更详细的信息。您将能够查看 DAG 的图形和树形视图，以及您刚刚编写的代码。准备好尝试了吗？在这个屏幕上或回到主页上，将切换按钮翻转，使 DAG 设置为 On，如图 7-5 所示。

![dppr 0705](img/dppr_0705.png)

###### 图 7-5。一个启用的 DAG。

请回忆一下，在代码中，DAG 的`schedule_interval`属性设置为`timedelta(days=1)`。这意味着 DAG 被设置为每天在 UTC 时间午夜运行一次。你会在 Airflow 首页和 DAG 详细页面上看到这个调度的反映。同时请注意，DAG 的`start_date`属性设置为`days_ago(1)`。这意味着 DAG 的第一次运行被设置为当前日期的前一天。当 DAG 被设置为启动时，第一次调度运行是在 UTC 时间前一天的 0:00:00，因此将在执行器有空闲时立即执行。

您可以在 DAG 详细页面上检查 DAG 运行的状态，或者通过导航到顶部菜单中的 Browse → DAG Runs 来查看。从那里，您应该能够看到 DAG 运行的可视状态，以及 DAG 中每个任务的状态。图 7-6 显示了一个`simple_dag`示例的运行，其中所有任务都成功完成。DAG 的最终状态标记为屏幕左上角的“success”。

如果你想按需运行 DAG，请在 DAG 详细页面点击触发 DAG 按钮。

![dppr 0706](img/dppr_0706.png)

###### 图 7-6。一个 Airflow DAG 的图形视图。

## 一个 ELT 管道的 DAG

现在你知道如何创建一个简单的 DAG 后，你可以构建一个用于数据管道的抽取、加载和转换步骤的功能性 DAG。这个 DAG 包含五个任务。

前两个任务使用`BashOperator`来执行两个不同的 Python 脚本，每个脚本从 Postgres 数据库表中提取数据，并将结果作为 CSV 文件发送到 S3 存储桶。虽然我不会在这里重新创建脚本的逻辑，但你可以在“从 PostgreSQL 数据库提取数据”中找到它。事实上，如果你想从 MySQL 数据库、REST API 或 MongoDB 数据库中提取数据，你可以使用该章节中的任何提取示例。

每当这些任务完成时，将执行相应的任务，从 S3 存储桶中加载数据到数据仓库中。再次，每个任务都使用`BashOperator`来执行包含加载 CSV 逻辑的 Python 脚本。你可以在“将数据加载到 Snowflake 数据仓库”或者“将数据加载到 Redshift 数据仓库”中找到相关示例代码，具体取决于你使用的平台。

DAG 中的最后一个任务使用`PostgresOperator`来执行一个 SQL 脚本（存储在*.sql*文件中），该脚本用于在数据仓库中创建数据模型。你可能还记得这一逻辑来自第六章。这五个任务组成了一个简单的流水线，遵循了首次在第三章中介绍的 ELT 模式。

图 7-7 显示了 DAG 的图形视图。

![dppr 0707](img/dppr_0707.png)

###### 图 7-7。ELT DAG 示例的图形视图。

示例 7-2 展示了 DAG 的定义。花点时间阅读它，即使我也会详细讲解。你可以将其保存到 Airflow 的*dags*文件夹中，但暂时不要启用它。

##### 示例 7-2。elt_pipeline_sample.py

```
from datetime import timedelta
from airflow import DAG
from airflow.operators.bash_operator \
  import BashOperator
from airflow.operators.postgres_operator \
  import PostgresOperator
from airflow.utils.dates import days_ago

dag = DAG(
    'elt_pipeline_sample',
    description='A sample ELT pipeline',
    schedule_interval=timedelta(days=1),
    start_date = days_ago(1),
)

extract_orders_task = BashOperator(
    task_id='extract_order_data',
    bash_command='python /p/extract_orders.py',
    dag=dag,
)

extract_customers_task = BashOperator(
    task_id='extract_customer_data',
    bash_command='python /p/extract_customers.py',
    dag=dag,
)

load_orders_task = BashOperator(
    task_id='load_order_data',
    bash_command='python /p/load_orders.py',
    dag=dag,
)

load_customers_task = BashOperator(
    task_id='load_customer_data',
    bash_command='python /p/load_customers.py',
    dag=dag,
)

revenue_model_task = PostgresOperator(
    task_id='build_data_model',
    postgres_conn_id='redshift_dw',
    sql='/sql/order_revenue_model.sql',
    dag=dag,
)

extract_orders_task >> load_orders_task
extract_customers_task >> load_customers_task
load_orders_task >> revenue_model_task
load_customers_task >> revenue_model_task
```

从示例 7-1 中，你可能还记得导入了一些必要的 Python 包并创建了一个`DAG`对象。这一次，为了在 DAG 的最后一个任务中使用`PostgresOperator`，还需要导入一个额外的包。

最后一个任务利用`PostgresOperator`执行存储在与 Airflow 同一台机器上的数据仓库上的目录中的 SQL 脚本。SQL 脚本的内容看起来类似于第六章中的数据模型转换。例如，考虑到 DAG 正在提取和加载`Orders`表和`Customers`表，我将使用来自第六章的以下示例。当然，你可以使用任何 SQL 查询来匹配你正在处理的数据。

```
CREATE TABLE IF NOT EXISTS order_summary_daily (
order_date date,
order_country varchar(10),
total_revenue numeric,
order_count int
);

INSERT INTO order_summary_daily
  (order_date, order_country,
  total_revenue, order_count)
SELECT
  o.OrderDate AS order_date,
  c.CustomerCountry AS order_country,
  SUM(o.OrderTotal) AS total_revenue,
  COUNT(o.OrderId) AS order_count
FROM Orders o
INNER JOIN Customers c
  ON c.CustomerId = o.CustomerId
GROUP BY
  o.OrderDate, c.CustomerCountry;
```

在启用 DAG 之前，还有一个步骤。那就是设置用于`PostgresOperator`的连接。正如在 DAG 定义中所见，有一个名为`postgres_conn_id`的参数，其值为`redshift_dw`。你需要在 Airflow 的 Web UI 中定义`redshift_dw`连接，以便`PostgresOperator`可以执行该脚本。

要做到这一点，请按照以下步骤操作：

1.  打开 Airflow web UI，从顶部导航栏选择 Admin → Connections。

1.  点击创建标签页。

1.  将 Conn ID 设置为 *redshift_dw*（或者你在 DAG 定义文件中想要使用的任何 ID）。

1.  选择 Conn Type 为 Postgres。

1.  设置数据库的连接信息。

1.  点击保存。

请注意，Amazon Redshift 兼容 Postgres 连接，这就是我选择该 Conn Type 的原因。你会在 Snowflake 和数十种其他数据库和平台（如 Spark）中找到连接。

现在，你可以准备启用 DAG。你可以返回主页或查看 DAG 详细页面，然后点击切换按钮将 DAG 设为启用状态。因为 DAG 的调度是每天午夜开始的前一天，所以会立即安排一个运行并执行 DAG。你可以在 DAG 详细页面查看 DAG 运行的状态，或者通过顶部菜单中的 Browse → DAG Runs 导航到 DAG 运行页面。如常，你也可以通过 DAG 详细页面上的 Trigger DAG 按钮触发 DAG 的单次运行。

虽然这个例子有些简化，但它整合了 ELT 管道的步骤。在更复杂的管道中，你会发现许多更多的任务。除了更多的数据抽取和加载之外，可能还会有许多数据模型，其中一些依赖于彼此。Airflow 可以轻松确保它们按正确的顺序执行。在大多数 Airflow 的生产部署中，你会发现许多 DAG 用于可能彼此有依赖关系的管道，或者一些外部系统或流程。查看 “高级编排配置” 获取有关管理此类挑战的一些提示。

# 其他管道任务

除了前一节样本 ELT 管道中的功能任务外，生产质量的管道还需要其他任务，例如在管道完成或失败时向 Slack 频道发送通知，以及在管道的各个点运行数据验证检查等。幸运的是，所有这些任务都可以由 Airflow DAG 处理。

## 警报和通知

虽然 Airflow web UI 是查看 DAG 运行状态的好地方，但当 DAG 失败（甚至成功）时，最好是收到电子邮件通知。有多种选项可用于发送通知。例如，如果你希望在 DAG 失败时收到电子邮件，可以在定义文件中实例化 `DAG` 对象时添加以下参数。如果你只想对特定任务收到通知，也可以将这些参数添加到任务而不是 DAG：

```
'email': ['me@example.com'],
'email_on_failure': True,
```

在 Airflow 可以发送邮件给你之前，你需要在 *airflow.cfg* 文件的 `[smtp]` 部分提供你的 SMTP 服务器的详细信息。

你还可以在 DAG 中的任务中使用 `EmailOperator` 发送电子邮件：

```
email_task = EmailOperator(
            task_id='send_email',
            to="me@example.com",
            subject="Airflow Test Email",
            html_content='some test content',
        )
```

除了`EmailOperator`之外，还有官方和社区支持的操作器，用于向 Slack、Microsoft Teams 和其他平台发送消息。当然，您也可以始终创建自己的 Python 脚本，将消息发送到您选择的平台，并使用`BashOperator`执行它。

## 数据验证检查

第八章详细讨论了数据验证和测试管道，但在您的 Airflow DAG 中添加任务以验证数据是一个良好的实践。正如您将在该章中了解到的那样，数据验证可以通过 SQL 或 Python 脚本实现，或者通过调用其他外部应用程序来实现。到目前为止，您已经知道 Airflow 可以处理它们全部！

# 高级编排配置

前一节介绍了一个简单的 DAG，它运行了一个完整的端到端数据管道，遵循了 ELT 模式。本节介绍了在构建更复杂的管道或需要协调具有共享依赖或不同调度的多个管道时可能遇到的一些挑战。

## 耦合与非耦合管道任务

尽管迄今为止的示例可能会让人觉得数据管道中的所有步骤（任务）都能够清晰地链接在一起，但情况并非总是如此。以流数据摄入为例。例如，假设使用 Kafka 将数据流式传输到 S3 存储桶，然后使用 Snowpipe 将数据连续加载到 Snowflake 数据仓库中（请参阅第四章和第五章）。

在这种情况下，数据持续流入数据仓库，但将数据转换的步骤仍然计划按照固定间隔运行，例如每 30 分钟一次。与示例 7-2 中的 DAG 不同，数据摄入的特定运行并不是将数据转换为数据模型任务的直接依赖项。在这种情况下，任务被称为*非耦合*，而不是 DAG 中的*耦合*任务。

鉴于这一现实，数据工程师必须在管道的编排方式上进行深思熟虑。虽然没有硬性规定，但在整个管道中进行一致和弹性的决策是管理解耦任务的必要条件。例如，在流数据导入和定期转换步骤的示例中，转换逻辑必须考虑来自两个不同来源（例如`Orders`和`Customers`表）的数据可能处于稍微不同的刷新状态。转换逻辑必须考虑存在仅具有订单记录而没有相应客户记录的情况，例如。

## 何时拆分 DAG

在设计管道时的一个关键决策点是确定哪些任务应该组成一个 DAG。虽然可能创建一个包含数据基础设施中所有的抽取、加载、转换、验证和警报任务的 DAG，但这将很快变得非常复杂。

决定何时将任务拆分成多个 DAG 以及何时将其保留在单个 DAG 中，有三个因素需要考虑：

当任务需要在不同的计划下运行时，拆分成多个 DAG

如果某些任务只需每天运行一次，而其他任务每 30 分钟运行一次，你可能应该将它们拆分为两个 DAG。否则，你将浪费时间和资源来额外运行某些任务 47 次！在计算成本经常基于实际使用的情况下，这是一个大问题。

当一个流水线是真正独立的时候，保持它们分开

如果管道中的任务仅相互关联，则保持它们在一个单独的 DAG 中。回顾示例 7-2，如果`Orders`和`Customer`表的摄取仅由该 DAG 中的数据模型使用，并且没有其他任务依赖于数据模型，则保持 DAG 独立是有意义的。

当一个 DAG 变得过于复杂时，确定是否可以逻辑上分解它

虽然这有些主观，但如果你发现自己看着一个具有数百个任务和依赖箭头的 DAG 图形视图，那么现在是考虑如何拆分 DAG 的时候了。否则，将来可能会难以维护。

尽管处理可能共享依赖的多个 DAG（例如数据摄取）可能看起来很头疼，但通常是必要的。在接下来的部分，我将讨论如何在 Airflow 中实现跨 DAG 的依赖关系。

## 使用 Sensor 协调多个 DAG

给定需要在 DAG 之间共享依赖的需求，Airflow 任务可以实现一种称为`Sensor`的特殊操作器。Airflow 的`Sensor`设计用于检查某些外部任务或进程的状态，一旦满足检查条件，就会继续执行其 DAG 中下游的依赖任务。

如果需要协调两个不同的 Airflow DAG，可以使用`ExternalTaskSensor`来检查另一个 DAG 中任务的状态或整个另一个 DAG 的状态。示例 7-3 定义了一个具有两个任务的 DAG。第一个任务使用`ExternalTaskSensor`来检查本章前面部分中`elt_pipeline_sample` DAG 的状态。当该 DAG 完成时，`Sensor`标记为“成功”，然后执行第二个任务（“task1”）。

##### 示例 7-3\. sensor_test.py

```
from datetime import datetime
from airflow import DAG
from airflow.operators.dummy_operator \
  import DummyOperator
from airflow.sensors.external_task_sensor \
  import ExternalTaskSensor
from datetime import timedelta
from airflow.utils.dates import days_ago

dag = DAG(
        'sensor_test',
        description='DAG with a sensor',
        schedule_interval=timedelta(days=1),
        start_date = days_ago(1))

sensor1 = ExternalTaskSensor(
            task_id='dag_sensor',
            external_dag_id = 'elt_pipeline_sample',
            external_task_id = None,
            dag=dag,
            mode = 'reschedule',
            timeout = 2500)

task1 = DummyOperator(
            task_id='dummy_task',
            retries=1,
            dag=dag)

sensor1 >> task1
```

图 7-8 显示了 DAG 的图形视图。

![dppr 0708](img/dppr_0708.png)

###### 图 7-8\. 示例 ELT DAG 的图形视图。

启用时，此 DAG 将首先启动`dag_sensor`任务。请注意其属性：

+   `external_dag_id`设置为`Sensor`将监视的 DAG 的 ID。在本例中，它是`elt_pipeline_sample` DAG。

+   在这种情况下，`external_task_id`属性设置为`None`，这意味着`Sensor`正在等待整个`elt_pipeline_sample` DAG 成功完成。如果你将其设置为`elt_pipeline_sample` DAG 中的特定`task_id`，那么一旦该`task_id`成功完成，`sensor1`将完成并启动`dummy_task`。

+   `mode` 属性设置为 `reschedule`。默认情况下，传感器以 `poke` 模式运行。在该模式下，传感器会阻塞一个工作槽位，同时`poke`以检查外部任务。根据您使用的执行器类型以及运行的任务数量，这并不理想。在 `reschedule` 模式下，通过重新调度任务释放工作槽位，从而使工作槽位空闲，直到再次设置为运行。

+   `timeout` 参数设置为 `ExternalTaskSensor` 在超时之前继续检查其外部依赖的秒数。在这里设置一个合理的超时时间是一个良好的实践；否则，DAG 将无限期地继续运行。

需要记住的一点是 DAG 按特定的时间表运行，因此 `Sensor` 需要检查特定的 DAG 运行。默认情况下，`ExternalTaskSensor` 将检查其所属 DAG 的当前时间表下的 `external_dag_id` 的运行。由于 `elt_pipeline_sample` 和 `sensor_test` DAG 每天午夜运行一次，使用默认设置就可以了。但是，如果两个 DAG 按不同的时间表运行，则最好指定 `Sensor` 应该检查哪个 `elt_pipeline_sample` 的运行。您可以使用 `execution_delta` 或 `execution_date_fn` 参数来实现这一点。`execution_date_fn` 参数定义了 DAG 运行的特定日期时间，但我发现它比 `execution_delta` 不太有用。

`execution_delta` 参数可用于查看 DAG 的特定运行。例如，要查看每 30 分钟调度一次的 DAG 的最近运行，您可以创建一个任务，定义如下：

```
sen1 = ExternalTaskSensor(
          task_id='dag_sensor',
          external_dag_id = 'elt_pipeline_sample',
          external_task_id = None,
          dag=dag,
          mode = 'reschedule',
          timeout = 2500,
          execution_delta=timedelta(minutes=30))
```

# 托管 Airflow 选项

虽然安装一个简单的 Airflow 实例相当简单，但在生产规模上就变得更具挑战性。处理更复杂的执行器以处理任务更大的并行性，保持您的实例最新，并扩展基础资源是不是每位数据工程师都有时间承担的工作。

与许多其他开源工具类似，Airflow 有几种完全托管的解决方案。其中两个最著名的是 [Cloud Composer](https://oreil.ly/ratu0) 在 Google Cloud 上和 [Astronomer](https://oreil.ly/yM7d8)。尽管您将支付月费，远远超过在自己的服务器上运行 Airflow 的成本，但 Airflow 的管理方面将得到照顾。

类似于本书中许多建设与购买的决策，根据您的具体情况，选择自托管 Airflow 还是选择托管解决方案。

+   您是否有系统运维团队可以帮助您进行自托管？

+   您的预算是否足以支持托管服务？

+   您的管道中有多少 DAG 和任务？您的运行规模是否足够高，需要更复杂的 Airflow 执行器？

+   您的安全和隐私需求是什么？您是否愿意允许外部服务连接到您的内部数据和系统？

# 其他编排框架

尽管本章重点讨论 Airflow，但这并不是唯一的选择。还有一些其他出色的编排框架，例如[Luigi](https://oreil.ly/QU2FZ)和[Dagster](https://docs.dagster.io)。而针对机器学习管道编排的[Kubeflow Pipelines](https://www.kubeflow.org)也受到了广泛支持，在机器学习社区中非常流行。

在数据模型转换步骤的编排方面，由 Fishtown Analytics 提供的[dbt](https://www.getdbt.com)是一个出色的选择。像 Airflow 一样，它是一个用 Python 构建的开源产品，因此您可以免费在自己的环境中运行，或选择付费的托管版本，称为*dbt Cloud*。一些组织选择使用 Airflow 或其他通用编排工具进行数据摄取，并运行诸如 Spark 作业之类的任务，但然后使用 dbt 来转换其数据模型。在这种情况下，dbt 作业运行由 Airflow DAG 中的任务触发，dbt 可以自行处理数据模型之间的依赖关系。有关使用 dbt 的示例，可以参考第九章。
