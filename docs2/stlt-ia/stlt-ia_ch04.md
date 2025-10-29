# 5 在全球范围内分享您的应用程序

### 本章涵盖了

+   可用于与用户分享应用程序的各种选项

+   免费将应用程序部署到 Streamlit 社区云

+   将应用程序连接到外部服务，如 API

+   在生产中保护您的 API 密钥和其他机密信息

+   管理您的应用程序依赖项

当您第一次成功运行您从头开始构建的应用程序时，这是一种神奇的时刻——这是所有花费在设计、开发和改进上的时间最终得到回报的时刻。您已经引导它经历了多次迭代，解决了错误，并微调了每个功能。

但接下来是什么？您是否将其隐藏在本地机器上？除非您只为自己的使用构建了某些东西，否则答案可能是否定的。要使您的应用程序真正有用，您需要将应用程序交给您的目标受众。

本章是关于从本地开发到全球部署的飞跃。我们将简要讨论您可用于分享应用程序的各种路径。然后我们将确定其中之一，并指导您将应用程序投入生产，让全世界体验。

在这个过程中，我们将讨论使您的应用程序公开时涉及的关键考虑因素，例如保护 API 密钥等机密信息以及管理您的代码依赖项。一如既往，我们将采取实用方法，让您直接亲身体验我们讨论的每一件事。

## 5.1 部署您的应用程序

自我们从 Streamlit 开始以来已经走了很长的路。在过去的三个章节中，您已经创建了三个完全功能性的——我甚至可以说，*有用的*——应用程序。然而，您一直将您的才华隐藏在俗语所说的灌木丛下；没有人体验过您的技艺。是时候改变这一点了！

### 5.1.1 什么是部署？

部署应用程序松散地意味着使其可供其他人使用。更具体地说，这意味着将您的应用程序托管在您的目标用户可以轻松访问的地方。

回想一下第四章，Streamlit 应用程序由一个后端服务器和一个在网页浏览器上运行的客户端组成。虽然前端向服务器发送请求并显示结果，但实际上是服务器在真正运行。

要建立前端和服务器之间的连接并加载应用程序，用户必须导航到服务器运行的 URL 和端口。您之前已经体验过这个过程；当您使用`streamlit run`命令启动应用程序时，这个命令最终实际上做的就是为您打开一个网页浏览器并导航到一个类似于"`https://localhost:8501`"的 URL。

您也可以手动完成这项工作。事实上，只要您的 Streamlit 服务器正在运行，在新的浏览器标签页或窗口中打开 URL 就会创建到服务器的新的连接和您应用程序的新实例。

因此，部署你的应用程序涉及启动并保持 Streamlit 服务器运行，以便接受新的连接。只是，而不是你通过 `localhost` URL 访问自己的应用程序，其他人将通过不同的 URL 访问它。

部署应用程序有几种方法。我们将在下一节中简要讨论这些方法。

### 5.1.2 部署选项

根据你的需求、你愿意花费的金额以及你愿意投入的努力，你可能需要考虑多种部署选项。让我们简要地考虑一些：

#### 在本地网络中运行服务器

部署你的应用程序最简单的方法就是每次运行 Streamlit 应用程序时你已经做过的。回想一下，当你使用 `streamlit run` 命令这样做时，Streamlit 服务器会启动，你可以在终端窗口中看到类似以下输出：

```py
You can now view your Streamlit app in your browser.

  Local URL: http://localhost:8502
  Network URL: http://192.168.50.68:8502
```

就像我们在这里多次看到的那样，这里的“本地 URL”允许你从运行应用程序的计算机上访问你的应用程序。

但如果你的机器连接到本地网络或甚至你的家庭 Wi-Fi，网络上的其他设备可以通过 *网络 URL* 访问它。

好吧，试试看！如果你在 Wi-Fi（或局域网）上，并且有另一台设备——比如智能手机或另一台电脑——连接到相同的 Wi-Fi / LAN，尝试运行你创建的一个应用程序，记下网络 URL，然后在第二台设备的网页浏览器中打开它。

例如，图 5.1 展示了我从连接到同一 Wi-Fi 网络的手机打开我的待办事项应用程序时看到的内容：

![image](img/05__image001.png)

##### 图 5.1 使用 Streamlit 应用程序的网络 URL 从同一网络中的另一台设备访问它

##### 注意

使其工作取决于你的网络如何设置。例如，你的防火墙可能阻止来自其他设备的传入流量，从而阻止它到达你的 Streamlit 服务器，或者可能存在其他类似的规则。修复此类问题超出了本书的范围，但你应该能够通过一些 Google 搜索或从你的网络管理员那里获得帮助来解决这个问题。

这种部署方法的一个优点是，更改代码并让用户看到这些更改就像编辑你的代码一样简单；没有额外的步骤！

然而，有一些明显的局限性：

+   它仅在运行 Streamlit 服务器的计算机开启并连接到网络时才有效。

+   它只允许连接到你的本地网络的设备访问你的应用程序，而不是公众。

然而，它在很多地方都可能很有用。例如，你可以为你的家庭创建应用程序，并与家人分享链接。甚至根据你的网络和安全策略的宽松程度，你还可以在工作场所使用这种类型的部署来运行基本的非业务关键应用程序。

#### 设置专用服务器

如果你希望让你的 Streamlit 应用程序对更广泛的受众可用，设置一个专用服务器可能是在本地部署之外的一个合理的步骤。这涉及到使用一个独立于你个人电脑的单独的物理或虚拟机器。通过这样做，你可以确保你的应用程序全天候可用，并且对本地网络之外的用户可访问。

在这种设置中，你首先选择一个合适的服务器——这可能是你拥有的一个重新利用的额外电脑，或者是一个专门为此目的设置的新机器。在选择服务器并在其上安装 Python 和 Streamlit 之后，你将启动你的应用程序的 Streamlit 服务器，并将正确的端口（例如，端口 8501）暴露给外部流量。你还需要处理网络配置以允许访问，例如，如果服务器位于防火墙后面，你需要在路由器上设置端口转发。

运营一个专用服务器可能是一项艰巨的任务，伴随着许多责任，尤其是与安全相关。你将负责配置防火墙，维护和更新服务器的操作系统和软件等。

选择这种方式的优点是你可以完全控制你的部署，但另一方面，它需要大量的技术知识，也许更重要的是，你将投入大量的时间。

如果你只是想让公众使用你开发的应用程序，我会推荐我们接下来要讨论的剩余选项之一。

#### 部署到云端

为了实现更大的可扩展性、可靠性和易于访问，你可以使用基于云的平台来部署你的应用程序。这种方法利用了公共云服务提供商的基础设施——例如亚马逊网络服务（AWS）、微软 Azure 或谷歌云——允许你无需物理硬件即可托管你的应用程序。它提供了许多好处，包括自动扩展以处理不同水平的流量，强大的安全措施以保护你的数据，以及高可用性以确保你的应用程序始终可访问。

云部署的关键好处是云服务提供商管理了与维护应用程序相关的许多基础设施责任。这包括服务器维护和安全更新，让你可以专注于应用程序的开发和改进。

许多公司已经将或正在将他们的内部应用程序迁移到云端。如果你考虑让你的应用程序在组织内部或更广泛的受众中可用，云部署可以是一个高效且有效的解决方案。与你的云管理员或 IT 团队合作将有助于确保顺利的设置和集成过程。

然而，重要的是要注意，随着你的应用程序变得流行，使用云提供商可能会很昂贵，因为成本通常基于你的应用程序使用的资源，这随着访问它的用户数量的增加而增加。

第十三章将详细讨论如何将您的应用程序部署到公共云平台，如 AWS 和 Google Cloud。

#### Streamlit 社区云

这就留下了我们将在这本书的大部分内容中使用的选项——Streamlit 社区云，这是一种将您的应用程序发布给任何感兴趣的人的方式，*完全免费*。

Streamlit 社区云由 Streamlit 的所有者 Snowflake 公司运营。它优先考虑易用性，正如其名称所暗示的，是专门为运行 Streamlit 应用程序而构建的。

社区云确实有一些资源限制，例如您的应用程序可以使用多少计算能力、内存和存储空间。如果您超过了这些限制——比如说，如果您的应用程序因受欢迎而变得非常流行——您可能需要考虑其他选项，例如部署到付费云提供商（见第十三章）。

然而，鉴于我们正处于学习 Streamlit 的过程中，社区云非常适合我们的需求。在本章的其余部分，我们将介绍如何将应用程序部署到社区云。

## 5.2 将我们的待办事项应用程序部署到 Streamlit 社区云

如前所述，Streamlit 社区云完美地满足了我们的部署需求，因为它免费、专门为 Streamlit 定制，并且极其易于使用。

在本节中，我们将部署我们之前构建的一个应用程序——第四章中的待办事项应用程序——到社区云上，这样任何有互联网连接的人都可以使用它。

### 5.2.1 前提条件

除了 Python 和 Streamlit 本身之外，将应用程序部署到 Streamlit 社区云还需要以下内容：

+   `git`，我们在第二章中简要讨论的流行版本控制工具

+   GitHub 账户

+   Streamlit 社区云账户

+   将您的 GitHub 账户连接到社区云

如果您之前从未使用过`git`并且想了解它，请参阅附录 B，其中简要介绍了如何使用它。

#### 创建和设置 GitHub 账户

您可能听说过 GitHub，这是一个用于版本控制和协作软件开发的基于网络的平台。它使用`git`，一个分布式版本控制系统，帮助开发者跟踪其代码的变化，在项目上进行协作，并管理其软件的版本。

##### 注意

GitHub 和 git 不要混淆。git 是版本控制系统的名称，而 Git*Hub*是使用 git 创建的仓库最流行的托管平台。您可以使用 git 与其他托管平台一起使用，例如 Bitbucket，尽管 Streamlit 社区云确实需要 GitHub。

GitHub 账户的链接现在是大多数开发者简历上的一个相当标准的固定项目。对我们来说，重要的是 Streamlit 社区云期望您的应用程序代码存储在 GitHub *仓库* 中，该仓库包含一组文件和目录及其修订历史。

首先，请访问`github.com`并注册一个新账户。注册过程与其他网站上的预期非常相似——您需要输入您的电子邮件并验证它，创建用户名，并选择一个强大的密码。

一旦您创建了账户，您需要启用您的命令行以进行身份验证并将代码推送到您创建的任何仓库。有几种方法可以做到这一点，但我们将使用个人访问令牌（PATs），这是密码的替代品，用于通过命令行或 API 访问 GitHub。

在撰写本文时，要到达 PAT 创建屏幕，您可以按照以下步骤操作：

+   点击您的个人头像然后“设置”

+   在侧面板中找到并点击“开发者设置”

+   点击“个人访问令牌”>“令牌（经典）”

+   选择“生成新令牌”

图 5.2 以可视方式显示了此路径（尽管当然 GitHub 可能会更改其配置方式）。

![image](img/05__image002.png)

##### 图 5.2 如何在 GitHub 上到达个人访问令牌生成页面

在打开的屏幕上，请确保选择“repo”范围，这会为您提供对仓库的完全控制权。

您还需要输入一个描述令牌用途的备注（您可以输入类似“用于推送 Streamlit 代码”的内容）以及一个有效期。

一旦 PAT 过期，您将无法再使用它，需要创建一个新的，因此请相应选择。较短的过期时间更安全——因为它在泄露的情况下有效期限更短——但也意味着您需要更频繁地更改它。

图 5.3 显示了您可以做出的选择。

![image](img/05__image003.png)

##### 图 5.3 GitHub 上的 PAT 创建屏幕；请确保选择“repo”范围

点击“生成令牌”以实际创建它。GitHub 现在会向您展示您创建的令牌。请复制并安全存储它，因为您将再也看不到它了！

我们将在推送我们的代码时使用 PAT。

#### 创建 Streamlit 社区云账户

要创建社区云账户，请访问 Streamlit 网站，`https://streamlit.io/`，点击“注册”，并按照指示操作。我建议使用您的 GitHub 账户注册。

完成后，它应该会带您到一个“仪表板”页面。

如果您没有使用 GitHub 账户注册，您会在左上角的“工作区”旁边看到一个感叹号。

如果是这样，您需要单独将您的 GitHub 账户连接到社区云。截至撰写本文时，您可以通过点击“工作区”然后“连接 GitHub 账户”来完成此操作。

如果您已经登录 GitHub，您不需要做任何事情。如果您还没有，您需要输入您的 GitHub 账户凭据。

### 5.2.2 部署步骤

现在您的账户已经设置好了，部署您的应用程序是一个三步过程：

+   创建 GitHub 仓库

+   将您的代码推送到 GitHub

+   告诉社区云在哪里查找它

如果您的应用需要连接到外部服务，或者如果它需要专用库，我们将在本章后面探索一些额外的步骤，但上述步骤将适用于我们在第四章中构建的现有待办事项应用。

如果您已经像我们在第二章中建议的那样，将 `git` 作为您作为开发者的常规工作流程的一部分，您可能已经创建了一个仓库并将代码推送到其中，因此可以跳过到“告诉社区云您的应用位置”这一部分。

如果还没有，按照下面的部分顺序阅读。

#### 创建 GitHub 仓库

首先，登录您的 GitHub 账户。创建新仓库的按钮应该相当明显。如果您在此账户中之前从未创建过，您应该会看到一个“创建仓库”按钮，如图 5.4 左侧所示。如果您已经有了一些仓库，它将显示您的顶级仓库列表，您可以通过点击“新建”，如图 5.4 右侧所示来创建一个新的。

![图片](img/05__image004.png)

##### 图 5.4 在 GitHub 上创建新仓库的按钮

这应该会带您到一个询问您新仓库详细信息的页面（见图 5.5）

![图片](img/05__image005.png)

##### 图 5.5 GitHub 上的仓库创建界面

您可以忽略这里的大多数设置——只需确保您给仓库起一个容易记住的名字，并选择“公开”作为可见性。

完成后，点击“创建仓库”。您将被带到包含一些说明和重要的是，您的仓库 URL 的界面，如图 5.6 所示。

![图片](img/05__image006.png)

##### 图 5.6 您的仓库 URL

记下这个 URL，因为您在下一节中需要它。如果需要，您可以在以后通过导航到您的仓库再次找到这个 URL。

您的仓库现在已准备好供您存放一些代码了！

#### 将代码推送到 GitHub

到目前为止，您有一个*远程* GitHub 仓库，但您的 Streamlit 应用代码是存储在*本地*，在您的计算机上。我们现在将使此代码在您的远程仓库中可用，这个过程称为*推送*您的代码。

打开一个终端窗口，导航到包含第四章中待办事项应用（由两个文件组成：`todo_list.py` 和 `task.py`）的目录。

输入以下命令以将此目录初始化为*本地* Git 仓库（与 GitHub 上的远程仓库相对）：

```py
git init
```

`git init` 命令创建一个空的 Git 仓库，在名为 `.git` 的隐藏子目录中设置所有必需的文件。

接下来，输入：

```py
git add .
```

这会将当前目录的内容添加到 Git 的*暂存区*，这是一个临时存放您对代码所做的更改的地方。您使用暂存区来准备您想要保存的代码的确切快照。

是时候*提交*您的更改了，所以输入：

```py
git commit -m "Commit Streamlit to-do list app"
```

此命令捕获您代码当前状态的快照，并将其保存到本地仓库中，附有关于更改的描述性消息。

此处的最后一步是将您的代码复制到 GitHub。为此，我们首先需要让您的本地 Git 仓库知道您在 GitHub 上的远程仓库。

获取您在 5.1.1 节中创建的 PAT 以及您远程仓库的 URL，并将它们组合起来创建一个 PAT 嵌入的 GitHub URL：

```py
https://<Your PAT>@<Repo URL without the https://>
```

例如，如果您的 PAT 是`ghp_fLbbSwjMlw3gUs7fgRux8Ha3PIlG9w3ZY3zY`（这不是一个真实的 PAT）并且您的仓库是`https://github.com/omnom-code2/streamlit-todo-list-app.git`，您的 PAT 嵌入 URL 将是：

```py
https://ghp_fLbbSwjMlw3gUs7fgRux8Ha3PIlG9w3ZY3zY@github.com/omnom-code2/streamlit-todo-list-app.git
```

您现在可以通过输入以下命令将此 URL 添加为本地仓库的*远程*：

```py
git remote add origin <PAT-embedded URL>
```

或者在我们的例子中：

```py
git remote add origin https://ghp_fLbbSwjMlw3gUs7fgRux8Ha3PIlG9w3ZY3zY@github.com/omnom-code2/streamlit-todo-list-app.git
```

这告诉 Git 将一个带有别名“origin”的远程仓库添加到您的本地 Git 配置中，并将其与指定的 PAT 嵌入 URL 关联。这允许您在未来的 Git 命令中使用该别名与远程仓库交互，并自动使用 PAT 进行身份验证。

最后，运行以下命令以执行代码推送：

```py
git push -u origin master
```

这做了两件事：

+   将您当前所在的本地分支（默认称为“master”）推送到您指定的“origin”远程仓库，从而使您的代码在远程仓库的“master”分支中可用。

+   将您的本地仓库的默认*上游分支*（`-u`是`-–set-upstream`的缩写）设置为远程仓库上的“master”分支，这样在将来您只需使用`git push`即可推送代码，无需`-u origin main`。

##### 注意

我们在这里假设 Git 在您的仓库中创建的默认分支名为“master”。一些 Git 版本使用“main”这个名字。如果您在“master”上使用时遇到错误，请尝试将其替换为“main”。您的命令将变为：“git push -u origin main”。或者，您可以通过输入“git branch”来确定您所在的分支名称（当前分支将被突出显示），并使用它。

如果您导航到您在 GitHub 上创建的仓库，您现在应该能够看到您的代码，如图 5.7 所示。

![image](img/05__image007.png)

##### 图 5.7 推送代码后您的 GitHub 仓库

Git 是一个相当复杂的工具，与它一起工作可能会有些复杂。我在本节中描述的只是您部署代码到 Streamlit 社区云所需了解的 Git 的最基本知识，但理想情况下，您应该将其作为常规开发工作流程的一部分来使用，每次完成有意义的工作后都提交代码。这要求您熟悉各种 Git 命令和选项。如果您想为使用 Git 建立一个良好的心理模型，请查看附录 B 中的教程。

#### 告诉社区云在哪里找到您的代码

我们已经在 GitHub 上设置了 Streamlit 代码。我们现在需要告诉 Streamlit 社区云在哪里查找它。

登录到您的社区云账户`streamlit.io`，然后点击右上角的“创建应用”按钮。如果它询问您是否已有应用，请选择表示您已有的选项。

这应该会带您到如图 5.8 所示的“部署应用”页面。

![image](img/05__image008.png)

##### 图 5.8 Streamlit 社区云上的应用程序部署屏幕

填写你推送代码的详细信息，包括：

+   你创建的 GitHub 仓库

+   你推送代码到的分支（例如，从`git push -u origin master`推送到的`master`）

+   你应用程序的路径（这将是`todo_list.py`，因为这是你在`streamlit run`命令中使用的文件，并且它位于你的仓库根目录中）

在应用程序 URL 字段中，你可以选择人们可以使用来访问你的应用程序的地址。Streamlit 会建议一个默认 URL，但你也可以将其覆盖为更有意义的内容。在图 5.8 中，我选择了`stmlit-todo-list-app-1.streamlit.app`。

一些高级设置将在本章的后面部分进行探讨，但你现在可以忽略它们。

就这些了！继续点击“部署！”按钮。

一分钟左右，你的应用程序应该准备好了！任何有互联网连接的人现在都可以访问你选择的地址（在这个例子中是`https://stmlit-todo-list-app-1.streamlit.app/`）来运行你的待办事项列表应用程序并整理自己的生活。你，改变生活的人！

## 5.3 部署使用外部服务的应用程序

正如我们在上一节中看到的，将一个简单的应用程序部署到 Streamlit 社区云遵循一个逻辑路径，并且相当直接。我们的待办事项列表应用程序在“简单”这个意义上是相当自包含的。

首先，除了 Streamlit 和 Python 本身之外，它没有其他库或软件依赖于它。其次，它不与任何外部服务或 API 交互。

这对于你创建的大多数实际应用程序来说并不成立。在现实世界中，你将在其他人构建的软件之上构建软件。因此，你的业务逻辑很可能需要第三方库，而这些库并不是 Python 预先安装的。

你还会发现，通常，你的应用程序需要连接到互联网上的服务以执行某些有用的操作。实际上，这是几乎不可避免的。在某个时候，你的应用程序将需要访问英语词典以检查用户输入的单词是否有效，或者货币汇率以显示不同货币的价格，或者来自全球的新闻头条。在这些情况下，你通常需要注册并连接到某种类型的应用程序编程接口（API），以提供你需要的特定服务。

在本节中，我们将向现有的待办事项列表应用程序添加一些复杂性，并查看如何正确部署更改。

### 5.3.1 一句每日名言来激励用户

想象一下你的待办事项应用程序的用户开始他们的一天。天空晴朗，鸟儿在鸣叫，他们面前有一张干净的画布。*diem*属于他们去*carpe*。因此，哼唱维瓦尔第的《四季》中的春之部分，他们开始添加他们的任务。

快进五分钟，他们又添加了第十八件他们刚刚想起今天必须做的事情。肩膀垂下，你的用户现在在脑海中模糊地哼着《星球大战》中的达斯·维达曲调，慢慢地意识到这一天将会是多么艰难的攀登。

好吧，我们不能让这种情况发生！如果我们的应用能在用户需要的时候提供一些安慰，一些激励他们的名言呢？我们难道不能找到一些值得分享的智慧来激励他们——比如像“成功的关键在于开始行动”这样的名言——来鼓励他们应对挑战吗？

当然，你们的一些更愤世嫉俗的用户可能会把咖啡杯扔向屏幕，但嘿，你不可能取悦每一个人。

不论如何，让我们考虑一下在我们的应用中添加每日名言将涉及什么。我们可以在代码中硬编码一些名言，但这似乎很浪费，而且不太可扩展。相反，我们将使用公共 API 来获取名言。

#### API 及其调用方法

API 实际上只是一个术语，指的是一组指令，允许不同的软件组件相互通信。你可能记得在第三章中，我们为我们的单位转换应用的后端定义了一个“API”，这是一个*合约*，定义了前端如何与之交互。

一般而言，“API”意味着几乎相同的意思，只不过我们不是指同一应用两个部分之间的合约，而是指定义任何软件如何与特定外部服务交互的合约。

API 可以按照其开发者的意愿进行结构化，而唯一完美理解如何使用它的方法就是阅读文档。话虽如此，你会在实践中遇到一些常见的*传统*API 模式。

一般来说，你可以通过向以下形式的 URL 发送 HTTP（或 Web）请求来调用此类 API：

```py
https://<base address>/<endpoint>
```

基础地址是 API 请求中每个请求都包含的常见地址，而端点是针对你请求的类型特定的。例如，一个与天气相关的 API 可能有`api.weathersite.com`作为基础地址，`forecast`和`history`作为天气预测和过去天气数据的端点。

你通常会通过传递键值参数来定制你的 API 请求，这些参数可以是 URL 中的（在 GET 请求的情况下）或者作为请求负载（在 POST 请求中）。你可能还需要通过 HTTP 头传递其他信息，例如 API 密钥。请参阅侧边栏了解 HTTP 工作原理的更多信息。

在我们上面的天气示例中，我们可能需要将日期作为 URL 参数传递给 API，以获取该日期的预报，因此 URL 变为：

```py
https://api.weathersite.com/forecast?date=2024-08-01
```

然后，API 会返回一个响应，通常是以 JSON（JavaScript 对象表示法）格式返回，你可以在代码中解析它以理解其含义。

在我们的例子中，响应可能如下所示：

```py
{
  "high_temp": 72,
  "low_temp": 60,
  "forecast_text": "Nice and sunny, a good day for the park"
}
```

##### 侧边栏：HTTP 请求的工作原理

HTTP，即超文本传输协议，是一组定义如何在网络上发送消息的规则。把它想象成计算机在网络上通信时使用的语法结构。

要使用 HTTP 进行通信（例如，当网页浏览器想要与网页服务器通信以检索网页时），一个 HTTP 客户端——比如浏览器——会向网页服务器发送格式正确的 HTTP 请求。然后服务器返回一个 HTTP 响应，这可能包括一个 HTML 文件、一张图片、一些文本，或者几乎任何其他内容。

HTTP 请求包括：

+   一个请求行（在我们的用例中，我们可以将其视为一个 URL）

+   一些 HTTP 头部，它们是键值对，提供了关于如何处理请求的额外信息

+   一个可选的主体，其中包含客户端想要发送到服务器的数据

HTTP 请求有几种不同的类型，但最常见的是 GET 和 POST。

通俗地说，GET 请求是没有主体的“轻量级”请求。关于请求的所有信息都包含在 URL 中。需要发送到服务器的数据被编码为键值对，并附加到 URL 的末尾，如下所示：`?param1=value1&param2=value2...`

当你在搜索引擎中输入查询时，这通常是一个 GET 请求。事实上，当你在你浏览器的地址栏中输入一个 URL 并按回车时，你正在向该 URL 发送一个 GET 请求。

POST 请求确实有一个包含额外信息的主体。这个主体通常被称为*负载*，可能包含诸如输入到网页表单中的信息或上传的文件。

#### API Ninjas 的报价 API

我们将要使用的 API 来自一个名为 API Ninjas 的网站（`https://api-ninjas.com/`）。API Ninjas 提供各种服务的免费 API，例如实时商品价格、汇率、URL 信息查找、图像中的面部检测等等。

他们通过他们 API 的付费版本赚钱，如果你每月发出超过 10,000 次调用，就必须使用这个版本。由于我们只是在学习，我们可以用得少一些，所以我们只会使用免费版本。具体来说，我们对其报价 API 感兴趣，根据他们的文档，“提供历史上著名人物的几乎无尽的引言。”

要访问它，你需要在他们那里创建一个账户。前往`https://api-ninjas.com/`并注册。像往常一样，你需要提供并验证你的电子邮件并创建一个密码。

一旦你登录，找到你的 API 密钥（你将有一个可以访问所有 API 的密钥），这是一个当你连接到 API 时用来识别你的字符集。当我写这篇文章时，在“我的账户”下有一个“显示 API 密钥”按钮。记下这个密钥。

查看在`https://api-ninjas.com/api/quotes`上的文档，以了解如何连接到报价 API。

这里的基础 URL 是`api.api-ninjas.com`，这是他们所有 API 中共享的一个常见 URL。Quotes API 的端点是`/v1/quotes`，它“返回一个（或多个）随机名言”。

文档还提到，你可以传递一个类别参数，以获取特定列表类别中的一句话。还有一个限制参数，允许你指定你想要多少句话，但这是一个付费功能，我们不会使用。默认情况下，API 将返回一句话，这是我们需要的所有内容。

假设我们想要“友谊”类别中的一句话。我们将向以下 URL 发送 GET 请求

```py
https://api.api-ninjas.com/v1/quotes?category=friendship
```

作为实验，尝试在你的浏览器中访问这个 URL（正如 HTTP 请求侧边栏中提到的，这会发送一个 GET 请求）。如果一切按预期工作，你应该得到一个认证错误消息：

```py
{"error": "Missing API Key."}
```

这是因为我们还需要提供之前记下的 API 密钥。文档提到，这需要以名为`X-Api-Key`的 HTTP 头的形式传递。

为了做到这一点，我们将使用一个名为`requests`的 Python 库。

### 5.3.2 使用`requests`库连接到 API

正如我们所见，我们将使用的生成励志名言的 API 是一个基于 Web 的 API，调用它需要我们的代码通过 HTTP 发送和接收消息。

我们*可以*使用纯 Python 来做这件事，但我们将使用`requests`，这是 Python 世界中用于 HTTP 通信的首选库。`requests`提供了大量处理 HTTP`requests`的内置功能，并且比其他替代方案具有更友好的工具集。

首先，你需要使用以下命令安装`requests`：

```py
pip install requests
```

完成后，使用`pip show requests`命令验证是否已安装，该命令应显示你拥有的`requests`版本以及其他一些信息。

使用`requests`模块发送 HTTP 请求相当简单：我们有简单的`get`和`post`方法可以使用。

为了尝试一下，打开 Python shell（在命令行中输入`python`或`python3`）并尝试以下操作：

```py
>>> import requests
>>> response = requests.get("https://api.api-ninjas.com/v1/quotes?category=friendship")
>>> response
<Response [400]>
>>> response.text
'{"error": "Missing API Key."}'

```

在这里，我们只是使用`requests.get`方法发送我们之前通过浏览器发送的相同请求。此方法返回一个`Response`类的实例，它封装了 HTTP 服务器对我们请求的响应。

然后，我们访问`response`的`text`属性，它以字符串的形式获取响应的主体。正如你所见，字符串与我们之前在浏览器中看到的一模一样。

正如我们从文档中学到的，我们可以使用`X-Api-Key`头传递我们的 API 密钥。`requests.get`方法接受一个名为`headers`的参数，其形式为常规 Python 字典，所以让我们设置一下：

```py
>>> headers={"X-Api-Key": "+4VJR..."}   #A
>>> response = requests.get("https://api.api-ninjas.com/v1/quotes?category=friendship", headers=headers)
>>> response.text
'[{"quote": "People come in and out of our lives, and the true test of friendship is whether you can pick back up right where you left off the last time you saw each other.", "author": "Lisa See", "category": "friendship"}]'
```

#A 将之前记下的 API 密钥“+4VJR…”替换

看起来现在它正在工作！当我们通过`X-Api-Key`头传递从 API Ninjas 获得的 API 密钥时，我们得到了 API 的实际名言！

`requests` 库的功能远不止这些。在这本书的剩余部分，我们将多次遇到这个模块，但如果你想了解更多，请查看 `https://requests.readthedocs.io/` 上的文档。

### 5.3.3 在我们的应用中整合名言

现在我们已经拥有了添加每日名言功能所需的一切，到我们的待办事项应用中。由于这与管理待办事项的核心功能有很大不同，因此创建一个新的文件 `quotes.py` 可能是有意义的。

具体来说，我们需要一个函数，比如 `generate_quote`，它接受一个 API 密钥并从名言 API 中提取名言。

列表 5.1 展示了 `quotes.py` 可能的样子。

##### 列表 5.1 quotes.py

```py
import requests

API_URL = "https://api.api-ninjas.com/v1/quotes"
QUOTE_CATEGORY = "inspirational"

def generate_quote(api_key):
    params = {"category": QUOTE_CATEGORY}
    headers = {"X-Api-Key": api_key}
    response = requests.get(API_URL, params=params, headers=headers)
    quote_obj = response.json()[0]
    if response.status_code == requests.codes.ok:
        return f"{quote_obj['quote']} -- {quote_obj['author']}"
    return f"Just do it! -- Shia LaBeouf"

```

我们将 `API_URL` 和 `CATEGORY`（设置为“inspirational”，这是 API Ninjas 文档告诉我们可以使用的一个类别）放置在文件顶部，作为易于配置的常量，以便如果以后需要更改它们，可以轻松地做到这一点，而无需修改函数本身。

在函数内部，你可能注意到我们没有直接使用 `?category=` 符号将类别直接传递到 URL 中，而是做了不同的处理；我们创建了一个类似这样的字典：

```py
params = {"category": QUOTE_CATEGORY}
```

并将这个字典传递给 `requests.get` 的 `params` 参数：

```py
response = requests.get(API_URL, params=params, headers=headers)
```

另一种方法同样有效，但这种方法更易读。此外，如果我们需要发送 POST 请求（使用 `requests.post`），`params` 参数在那里也适用，而你不能使用 URL 方法。

接下来，我们不再使用之前在 Python shell 中使用的 `response.text`，而是使用 `response.json()`。这是因为，从我们在 shell 中的早期实验中，我们知道 API 的响应是 JSON 格式：

```py
>>> response.text
'[{"quote": "People come in and out of our lives, and the true test of friendship is whether you can pick back up right where you left off the last time you saw each other.", "author": "Lisa See", "category": "friendship"}]'
```

JSON 是一种格式，允许你使用文本创建任意层次的数据结构。对于 Python 开发者来说，读取 JSON 数据块实际上相当容易，因为你在 JSON 中存储数据的方式几乎与你在 Python 字面量中编码相同的数据方式相同。

所以在 `response.text` 的上述值中，你可能已经看出数据被编码为一个列表（因为方括号），它包含一个元素：一个包含三个键：“quote”、“author”和“category”的字典。

然而，`response.text` 是一个字符串。为了能够在 Python 中轻松访问数据，我们需要将其解析成一个 Python 对象。这就是 `response.json()` 方法的作用：它将响应的文本主体解析成一个 Python 对象，假设文本是 JSON。

```py
quote_obj = response.json()[0]
```

一旦我们有了 `response.json()` 的结果，我们可以将其视为一个正常的 Python 对象。在这种情况下，我们知道数据是一个单元素列表，因此我们使用 `response.json()[0]` 来获取列表的唯一元素，它是一个包含三个键的字典。

下一个部分检查请求是否顺利通过：

```py
if response.status_code == requests.codes.ok:
        return f"{quote_obj['quote']} -- {quote_obj['author']}"
```

每个 HTTP 响应都附带一个状态码，用于分类请求的结果。你可能之前见过这些。状态码 200 表示“OK”，代码 404 表示“未找到”等。

在上面的代码片段中，`if` 语句验证状态码对应于成功的结果，或者在这种情况下，返回了一个引用。`requests.codes` 提供了一种更易读的方式来引用特定的 HTTP 状态码。`requests.codes.ok` 仅表示代码 200。

如果一切正常，我们构建一个包含引用本身以及作者（用"--"分隔）的字符串。我们使用 `quote_obj['quote']` 和 `quote_obj['author']` 从我们解析 JSON 结果的 `quote_obj` 字典中提取这些信息。

```py
return f"Just do it! -- Shia LaBeouf"

```

最后，如果 API 调用出现错误且状态码不是 200，我们将优雅地失败，并返回 Shia LaBeouf 的永恒励志名言，他是我们这个时代伟大的哲学家之一。

一旦 `quotes.py` 完成，使用它来创建在主应用程序中显示的引用非常简单。在 `todo_list.py` 中，在顶部导入 `generate_quotes` 函数：

```py
from quotes import generate_quote
```

然后，要显示引用，请在显示任务完成度指标的正上方调用 `generate_quote` 函数并使用您的 API 密钥，并将结果放入一个 st.info 框中：

```py
st.info(generate_quote("+4VJR..."))
```

和以前一样，别忘了将 `generate_quote` 的参数替换为您的实际 API 密钥。

如果您此时使用 `streamlit run` 运行应用程序，您应该会看到如图 5.9 所示的每日名言出现。

![image](img/05__image009.png)

##### 图 5.9 我们待办事项应用程序中的每日名言

太好了！这似乎表明我们的 API 调用完全按照我们的预期工作！我们可以放心，如果我们的用户在组织他们的一天中间想要放弃，他们会有一个每日励志名言来激励他们保持坚强。

但等等！回想一下，Streamlit 每次发生任何交互时都会重新运行您的 *全部* 代码。这包括通过 `generate_quote` 进行 API 调用。基本上，每次您的用户添加任务、标记某事为完成或基本上在应用程序中做任何事情时，都会进行一次 API 调用并生成一个新的引用。忘掉每日名言吧，我们在这里处理的是点击名言。

通常，您可能会认为这是“功能，而不是错误”。但请记住，我们每月只有 10,000 次免费的 API 调用。如果我们每次用户点击任何东西时都拉取一个引用，我们很快就会用完这个庞大的配额。

相反，让我们利用我们第四章的老朋友 `st.session_state`。就像我们保存任务列表以便每次运行时不会重置一样，我们也会保存引用，这样我们只调用 API 一次：

```py
if "quote" not in st.session_state:
    st.session_state.quote = generate_quote("+4VJR...")    #A
```

#A 再次，将 "+4VJR…" 替换为您之前记下的 API 密钥

我们现在可以用这个来替换我们之前的 st.info 行：

```py
st.info(st.session_state.quote)
```

如果现在尝试该应用程序，您会看到当您在应用程序中添加任务或执行其他任何操作时，引用不会改变，只有当您刷新页面时才会改变。

那么，我们完成了吗？嗯，还不完全...

### 5.3.4 使用 st.secrets 安全地访问您的 API 密钥

我们的应用程序现在运行得很好，但它有一个明显的安全漏洞。我们只是直接将 API 密钥嵌入到代码中。

如果我们现在将此代码提交到公共 GitHub 仓库，任何人都可以看到它，因此可以找到我们的 API 密钥！在这种情况下，使用免费 API 的风险相对较低，但想象一下，如果你在付费 API 上——无论是用完了免费配额，还是因为你正在使用一个需要付费的 API！

任何访问你的 GitHub 仓库的人——可能是无意中或出于恶意意图——都可能对你的 API 账户造成破坏，可能花费你数百甚至数千美元。

##### 注意

现在值得记住这个规则：永远不要在代码中存储任何秘密信息，无论是 API 密钥、密码还是任何其他类型的凭证——特别是如果代码将要公开访问。

那么，替代方案是什么？如果我们不能将其放入代码中，我们如何让我们的应用访问 API 密钥？一般来说，有几种方法可以做到这一点：你可以将你的密钥放入一个存储在服务器上的环境变量中，该服务器上运行着你的应用。另一个选择是使用像 AWS Secrets Manager 或 HashiCorp Vault 这样的秘密管理服务，这些服务允许你安全地检索机密信息。

幸运的是，Streamlit 提供了一个简单的方法来保护你的秘密：`st.secrets`。

与`st.session_state`类似，`st.secrets`是一个类似字典的结构，你可以用它来存储作为键值对的秘密信息。

要使一个值可用于`st.secrets`，你将其放入一个名为`secrets.toml`的特殊文件中。当你本地开发时，Streamlit 会从`secrets.toml`文件中提取值。当你推送代码时，请确保不要推送`secrets.toml`文件。

相反，你将直接在 Streamlit 社区云中配置这些信息，这样你的部署应用就可以在生产环境中访问它。

#### `secrets.toml`文件

在你本地仓库的根目录（你之前在这里运行了`git init`），创建一个名为`.streamlit`的空目录，并在其中创建一个名为`secrets.toml`的空文本文件。

接下来，按照列表 5.2 所示，在文件中输入你的 API 密钥。

##### 列表 5.2 secrets.toml

```py
[quotes_api]
api_key = "+4VJR..."
```

如前所述，在这里使用你的实际 API 密钥。

`.toml`格式可能对你来说很新。TOML 代表 Tom's Obvious Minimal Language。它是一种用于配置文件的格式，旨在简洁且易于人类阅读，最初由一个名叫——信不信由你——Tom 的人创建。

`.toml`文件主要由键值对组成，支持一系列简单和复杂的数据类型，包括数组和表格。它们可以通过方括号分隔的节来划分。

列表 5.2 中显示的文件有一个单独的节，`quotes_api`，其中有一个键值对（键为`api_key`）。

如果我们在 Python 中读取它（在 Streamlit 之外，你通常会使用`toml`模块来做这件事），它将映射到以下嵌套字典：

```py
{'quotes_api': {'api_key': '+4VJR...'}}
```

在 Streamlit 中，`st.secrets`大致也会包含这些内容。

因此，我们现在可以将`todo_list.py`中调用`generate_quote`的部分替换为以下内容：

```py
if "quote" not in st.session_state:
    api_key = st.secrets["quotes_api"]["api_key"]
    st.session_state.quote = generate_quote(api_key)
```

在运行时，Streamlit 将读取我们的`secrets.toml`文件，如上所示，填充`st.secrets`，我们可以使用`st.secrets["quotes_api"]["api_key"]`来引用 API 密钥。试一试——你会看到你的应用仍然能够从 API 中获取引言。

##### 注意

技术上，.streamlit 目录应该位于你运行“streamlit run <file_path>”命令的目录中。例如，如果你目前在一个名为“apps”的目录中，你的实际代码位于一个名为“todo_list_app”的文件夹中，你用来运行应用的命令是“streamlit run todo_list_app/todo_list.py”，那么.streamlit 文件夹应该位于“apps”文件夹中，而不是“todo_list_app”文件夹中。当你将代码部署到 Streamlit 社区云时，“streamlit run”实际上会从你的仓库根目录运行，这就是为什么我建议将其放置在那里。在本地开发时，如果你的本地仓库根目录与您通常执行“streamlit run”的文件夹不同，你可能需要“cd”到仓库根目录才能使其工作。

在此之后，你的`todo_list.py`文件应该如列表 5.3 所示

##### 列表 5.3 `todo_list.py`的最终状态

```py
import streamlit as st
from task import Task
from quotes import generate_quote

if "task_list" not in st.session_state:
    st.session_state.task_list = []
task_list = st.session_state.task_list

if "quote" not in st.session_state:
 api_key = st.secrets["quotes_api"]["api_key"]
 st.session_state.quote = generate_quote(api_key)

def add_task(task_name: str):
    task_list.append(Task(task_name))

def delete_task(idx: int):
    del task_list[idx]

def mark_done(task: Task):
    task.is_done = True

def mark_not_done(task: Task):
    task.is_done = False

with st.sidebar:
    task = st.text_input("Enter a task")
    if st.button("Add task", type="primary"):
        add_task(task)

st.info(st.session_state.quote)

total_tasks = len(task_list)
completed_tasks = sum(1 for task in task_list if task.is_done)
metric_display = f"{completed_tasks}/{total_tasks} done"
st.metric("Task completion", metric_display, delta=None)

st.header("Today's to-dos:", divider="gray")
for idx, task in enumerate(task_list):
    task_col, delete_col = st.columns([0.8, 0.2])
    label = f"~~{task.name}~~" if task.is_done else task.name
    checked = task_col.checkbox(label, task.is_done, key=f"task_{idx}")
    if checked and not task.is_done:
        mark_done(task)
        st.rerun()
    elif not checked and task.is_done:
        mark_not_done(task)
        st.rerun()
    if delete_col.button("Delete", key=f"delete_{idx}"):
        delete_task(idx)
        st.rerun()
```

## 5.4 将我们的更改部署到 Streamlit 社区云

通常来说，对已经部署的 Streamlit 应用进行更改很简单。你所需要做的就是提交你的更改，并使用`git push`将它们推送到你的 GitHub 仓库，Streamlit 社区云将自动获取这些更改。

然而，鉴于我们对应用所做的最新更改，在我们的情况下，这个过程将稍微复杂一些。有三个复杂因素：

+   我们现在使用了一个额外的第三方库，requests，并且我们必须确保在生产环境中安装了正确的版本

+   我们需要确保永远不要将`secrets.toml`推送到 GitHub，即使是不小心也是如此

+   由于我们不会将 API 密钥存储在 GitHub 仓库中，我们需要直接在社区云中配置它

### 5.4.1 使用`requirements.txt`文件管理 Python 依赖项

当你刚开始将你的应用发布到世界时，可能会让你感到困惑的一件事是，你现在需要管理两个*环境*：你的开发环境（通常简称为*dev*），它位于你的笔记本电脑或你用来编写应用的任何计算机上，以及你的生产环境（称为*prod*），它位于 Streamlit 社区云中。

为了确保你在开发环境中编写的应用在生产环境中按预期工作，你必须确保两个环境配置得完全相同，或者至少确保*开发环境和生产环境之间的任何差异都不会影响你的应用*。

显然，这意味着在生产环境中运行的 Python 代码应该与你在开发环境中运行的代码完全相同，但这还不够；你的 Python 代码可能依赖于你未编写的软件，例如在我们的待办事项应用程序中，`requests` 模块，或者实际上，Streamlit 本身——这意味着我们还需要确保这些 *依赖项* 在生产环境和开发环境中是相同的，或者至少它们在影响应用程序功能的方式上没有差异。

例如，如果你使用了 requests 库在 2.1.1 版本中引入的功能，但你在生产环境中安装的 `requests` 版本是 2.0.5，你的应用程序在生产环境中将会遇到错误。为了确保你的代码不会出错，你需要在生产环境中安装 `requests` 版本 2.1.1 或更高版本。技术上，为了确保你的代码不会出错，你需要安装确切的 2.1.1 版本，因为总有可能你使用的功能在未来的版本中被弃用或删除。

在 Python 世界中，我们传统上使用一个名为 `requirements.txt` 的文件来防止这类差异。`requirements.txt` 的前提很简单：它中的每一行代表一个 Python 模块和一个字符串，该字符串指定了该模块的版本或一系列版本，这些版本与你的代码兼容。

当将文件输入到 `pip install` 命令（如 `pip install -r requirements.txt`）时，`pip` 将会自动遍历该文件并安装其中每个模块的正确版本。

Streamlit 社区云知道如何处理 `requirements.txt` 文件，所以如果你在你的 GitHub 仓库中添加一个，它将自动安装使你的应用程序工作的库。

#### 创建一个 `requirements.txt` 文件

我们的应用程序只有两个外部依赖项——`streamlit` 和 `requests`（你可以通过检查所有 `.py` 文件中的导入语句来确定这一点）。因此，我们的 `requirements.txt` 文件的内容可以非常简单，如列表 5.4 所示。

##### 列表 5.4 一个简单的 requirements.txt 文件

```py
requests
streamlit

```

虽然这是一个有效的 `requirements.txt` 文件，但它并没有说明我们想要安装的版本。

处理这个问题的一个安全方法就是确定我们在开发环境中安装的这些库的版本，并在 `requirements.txt` 中指定这些确切的版本。

有几种方法可以做到这一点：

+   你可以输入 `pip show streamlit` 和 `pip show requests` 命令来查看（以及其他信息）你拥有的每个库的版本。

+   你也可以输入命令 `pip freeze > requirements.txt` 以自动从你在开发环境中安装的模块创建 `requirements.txt` 文件——然而，这将会包括你安装的 *所有* 模块，无论你是否在代码中导入它们。如果你选择这种方式，可能需要删除文件中除了与 `streamlit` 或 `requests` 对应的行之外的所有行。

无论哪种方式，一旦你使用`pip show`确定了正确的版本，或者已经使用`pip freeze`创建了文件，你的`requirements.txt`文件应该看起来像列表 5.5。

##### 列表 5.5 严格的 requirements.txt

```py
requests==2.31.0
streamlit==1.34.0

```

我们**不必**对版本这么严格。我们也可以允许这些库的任何版本，只要它们与我们安装的版本相同或更高。这样，我们就可以让我们的应用程序从这些库可能在未来做出的任何底层改进中受益，同时仍然可以**合理地**确信它不会崩溃。在这种情况下，我们的文件可能看起来像列表 5.6。

##### 列表 5.6 允许此版本或更高版本的 requirements.txt

```py
requests>=2.31.0
streamlit>=1.34.0

```

一旦你的文件准备好了，将其保存到本地 Git 仓库的根目录下。

##### 注意

你可能会注意到，即使你没有创建 requirements.txt 文件，你的应用程序在生产环境中也能正常工作。这是因为 Streamlit Community Cloud 预先安装了我们在这里使用的两个库，即 streamlit 和 requests。显然，streamlit 库对于任何应用程序的正常运行都是必需的。由于 streamlit 模块实际上依赖于 requests 模块，因此 requests 也被预先安装了！然而，你无疑会在未来遇到需要安装除这两个库之外的其他库的情况，那时你需要一个 requirements.txt 文件。即使在当前情况下，你也需要这个文件来限制在生产环境中安装的这些模块的确切版本。

### 5.4.2 使用.gitignore 保护 secrets.toml

你可能会想知道使用`secrets.toml`文件实际上是如何保护你的 API 密钥不被窥视的——密钥可能技术上不再在你的 Python 代码中，但它仍然存储在你实际代码旁边的文件中。好吧，关键是要在本地保留`secrets.toml`文件，但永远不要将其提交到我们的 Git 仓库中。

为了确保我们不会意外提交`secrets.toml`文件，我们将使用一个名为`.gitignore`的文件。

`.gitignore`是一个简单的文本文件，位于你的仓库根目录，它告诉 Git 忽略某些文件，永远不要提交它们。

如果文件还不存在，在仓库根目录下创建一个名为`.gitignore`的空文本文件。然后添加你的`.streamlit`文件夹的路径。由于`.streamlit`文件夹也应该在仓库根目录下，你只需将以下内容添加到`.gitignore`中：

```py
.streamlit/
```

如果你现在执行`git add`或`git commit`，你的`secrets.toml`文件（以及实际上`.streamlit`目录中的任何文件）都不会被提交。

##### 注意

Git 可能已经在你将其添加到.gitignore 之前跟踪了你的`.streamlit`文件夹。在这种情况下，为了将其从 Git 的索引中删除，你可能需要输入以下命令：

git rm -r --cached .streamlit/

到目前为止，你的目录结构应该看起来像以下这样：

```py
Root of your Git repo
├── .streamlit (folder)
│   └── secrets.toml
├── .gitignore
├── requirements.txt
├── quotes.py
├── task.py
└── todo_list.py
```

将你的更改添加到 Git 中，用消息提交，然后将它们推送到 GitHub。你可以使用以下命令：

```py
git add .
git commit -m "Add quote-of-the-day functionality"
git push

```

注意，你可以使用上面的简单 `git push` 命令，因为你之前已经使用 `git push -u` 将分支的上游设置为远程仓库。

你的更改现在应该反映在你的生产 Streamlit 应用程序中。或者，应该是这样吗？现在导航到你的应用程序的公共 URL，看看图 5.10 中显示的内容。

![image](img/05__image010.png)

##### 图 5.10 由于无法访问 API 密钥，我们的应用程序抛出了错误

看起来在我们应用程序正常工作之前，我们还需要进行一些最后的配置。

### 5.4.3 在社区云中配置秘密

我们已经尽力保护我们的 API 密钥不被泄露，但我们的应用程序在运行时仍然需要它。为了启用此功能，我们需要在部署应用程序的 Streamlit 社区云中进行配置。

我们可以在社区云的 *应用设置* 屏幕上完成这项操作。你可以通过两种方式访问此屏幕：

+   当你在已部署应用程序的公共 URL 上时，点击右下角的 "管理应用程序"（如图 5.11 所示），然后点击三个垂直点，再点击 "设置"。

+   当你在 streamlit.io 上的社区云登录时，点击应用程序列表中你应用程序旁边的三个垂直点，然后从那里点击 "设置"。

无论哪种方式，一旦你打开了应用程序设置页面，点击侧面板上的 "秘密"，就可以进入图 5.11 中显示的屏幕。

![image](img/05__image011.png)

##### 图 5.11 Streamlit 社区云中的秘密配置屏幕

在 "秘密" 下的文本框中，只需复制并粘贴你在开发机器上的 `secrets.toml` 文件中的内容，然后点击 "保存"。

这样，你就完成了所有操作！再次访问你应用程序的公共 URL。现在它应该完全可用，如图 5.12 所示。

![image](img/05__image012.png)

##### 图 5.12 已部署到生产环境的完整功能应用程序

到现在为止，你应该已经准备好构建和与公众分享有用的 Streamlit 应用程序了。在下一章中，我们将探讨一个更高级的应用程序，该应用程序专注于数据处理。

## 5.5 总结

+   部署是将你的应用程序托管并设置起来，以便目标用户可以访问的过程。

+   部署应用程序有多种方式——你可以在本地网络中简单地运行一个服务器，设置一个专用服务器，或者使用云服务提供商。

+   你可以免费将无限数量的应用程序部署到 Streamlit 社区云，但受一些资源使用限制的约束。

+   在社区云中部署涉及创建 GitHub 账户，将本地代码推送到远程 GitHub 仓库，并告诉社区云你的代码位置。

+   你可以通过名为 `requests` 的库使用 HTTP 协议连接到外部服务。

+   结合位于 `.streamlit` 文件夹中的 `secrets.toml` 文件，可以使用 `st.secrets` 来保护你的凭证安全。

+   确保永远不要提交你的 `secrets.toml` 文件——通过将其添加到 `.gitignore` 来防止意外，而是在 Streamlit 社区云的应用设置页面上保存你在本地 `secrets.toml` 中存储的信息。

+   您可以使用`requirements.txt`文件来指定您的应用程序所依赖的各种 Python 库的版本。

+   在社区云上部署您对已部署应用程序所做的任何更改，就像将更改推送到您的远程 GitHub 仓库一样简单。
