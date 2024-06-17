# 附录 B. 关于 Git 的更多信息

大部分情况下，作为一个项目的唯一程序员使用 Git 是相当简单的：你对代码进行更改，提交它们，将它们推送到 GitHub（或其他远程存储库），就这样。

直到……不是。也许你在 GitHub 上更新了你的 *README.md* 文件，然后忘记在设备上对相同文件进行更改之前运行 `git pull`。也许你在运行 `git commit` 时忘记添加消息。虽然处理这些小问题并不是真的很复杂，但如果你以前从未遇到过这些问题，那么 Git 在终端中的某些错误消息和默认行为可能会让人感到棘手。虽然这个附录中的指导*远远*不是全面的，在我的经验中，对 Git 的基本工作知识已经足够了，除非你在一个大型组织的一个相对大的团队中工作。因此，虽然如果你计划与多个合作者在复杂的项目上使用 Git，你应该绝对超越这些简单的修复，但对于我们大多数人来说，这些示例涵盖了你经常会发现自己处于的情况，并且需要帮助摆脱的情况。

# 运行 git push/pull 后进入了一个奇怪的文本编辑器

像我一样，Git 在记录你的工作时是无情的，以至于*如果没有提交消息，它将不允许你提交更改*。这意味着，如果你运行 `commit` 命令而没有包含 `-m "提交消息在这里"`，例如：

```py
git commit *filename*
```

你很可能会发现你的终端窗口被接管，显示类似于 [图 B-1](#terminal_commit_editing) 中所示的文本。

![基于终端的提交消息编辑](assets/ppdw_ab01.png)

###### 图 B-1\. 基于终端的提交消息编辑

这可能相当令人不安，特别是第一次发生时。但因为它*会*发生，所以这里是如何快速解决的方法：

1.  从键入字母 **`i`** 开始。虽然并不是所有编辑器实际上都要求你键入 `i` 进入 `INSERT` 模式，但其中大多数会吞掉你键入的第一个字符，所以你最好从这个字符开始。你可以在 [图 B-2](#terminal_insert_mode) 中看到这个视图的样子。

1.  要退出 `INSERT` 模式，按下 Esc 键或 Escape 键。然后像 [图 B-3](#terminal_commit_save) 中所示输入 **`:x`**，然后按 Enter 键或 Return 键。这将按照要求保存你的提交消息，*并且*让你退出那个文本编辑器，回到你熟悉和喜爱的终端窗口。

请注意，你发现自己处于的编辑器可能看起来与 [图 B-2](#terminal_insert_mode) 和 [图 B-3](#terminal_commit_save) 所示的不同；如果不同，不要惊慌。现在是搜索在线编辑、保存和退出的时间，无论你被弹到了什么程序中。不管具体情况如何，目标都是一样的。

![`INSERT` 模式下的终端编辑器](assets/ppdw_ab02.png)

###### 图 B-2\. `INSERT` 模式下的终端编辑器

![带有“保存并退出”命令的终端编辑器](assets/ppdw_ab03.png)

###### 图 B-3\. 带有“保存并退出”命令的终端编辑器

# 您的git push/pull命令被拒绝了。

这种情况每个人都会遇到。你以为自己每次工作结束时都很勤奋地提交了代码，甚至为每个文件的更改编写了单独的提交消息，而不只是运行`git commit -a`。即便如此，有时你运行`git push`命令时会被拒绝。那么接下来该怎么办呢？

如果您的`git push`命令失败，您可能会看到如下错误：

```py
 ! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'https://github.com/your_user_name/your_r
epo_name.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

通常情况下，这种情况发生在您本地存储库中的至少一个文件自上次提交以来已更改，但*远程存储库中的版本也已更改*，因此Git不知道哪一个应该优先。不用担心！

## 运行git pull

如果文件的更改可以自动合并，Git将执行此操作。这将解决您的文件冲突，但可能会导致您进入命令行的内置文本编辑器，这本身可能会相当令人困惑。如果您运行`git pull`，突然看到像[图 B-1](#terminal_commit_editing)中显示的文本，参见[“您运行git push/pull并进入了一个奇怪的文本编辑器”](#editing_commits_in_vim)。

如果文件的更改*不能*自动合并，您将收到类似以下消息的提示（如果您只运行`git pull`而没有先运行`git push`，也会看到此消息）：

```py
Auto-merging *filename*
CONFLICT (content): Merge conflict in *filename*
Automatic merge failed; fix conflicts and then commit the result.

```

这基本上意味着由您作为人来决定哪个文件应优先，并通过直接编辑一个或两个文件或简单地强制其中一个文件覆盖另一个来将它们重新调整为一致。

### 手动解决冲突

假设您有一个包含*README.md*文件的存储库，在GitHub.com和您的设备上都对其进行了更改。您试图`git push`但遇到了错误，因此您尝试了`git pull`，但自动合并失败了。要手动解决冲突，请先在您首选的文本编辑器中打开*本地*文件副本。Markdown文件的冲突示例如[示例 B-1](#markdown_file_conflict)所示。

##### 示例 B-1\. Markdown文件中的一个冲突示例

```py
# This header is the same in both files

This content is also the same.

<<<<<<< HEAD

This content is what's currently in the local file.
=======
This content is what's currently in the remote file.
>>>>>>> 1baa345a02d7cbf8a2fcde164e3b5ee1bce84b1b
```

要解决冲突，只需根据您的喜好编辑文件内容，确保删除以`<<<<<<<`和`>>>>>>>`开头的行，以及`=======`；如果您留下这些行，它们将出现在最终文件中。现在像往常一样保存文件。

接下来，运行：

```py

git add *filename*

```

然后运行：

```py
git commit -m "How I fixed the conflict."
```

注意，在`git commit`命令中*不能*指定文件名，否则Git会抱怨你在合并过程中要求进行部分提交。

最后，运行：

```py
git push
```

然后您应该设置好一切！

### “修复”冲突是通过强制覆盖来完成的。

虽然查看冲突文件总是个好主意，但有时你要推送到一个明显过时的仓库，而你只是想要用本地版本覆盖远程仓库中的内容。在这种情况下，你*可以*运行以下命令推送更改，而不手动协调文件：

```py
git push --force
```

这将简单地用本地版本覆盖远程文件，但请记住，*所有覆盖版本的记录，包括提交历史*，都将丢失。这意味着一旦使用了`--force`，就无法回退——远程内容将会丢失。显然，这意味着如果其他人也在同一仓库上贡献，你*绝不*应该使用此选项——只有在你是唯一的工作人员，并且对远程仓库中的内容有(非常)充分的信心可以安全覆盖时，才应使用此选项。

# Git 快速参考

[表 B-1](#common_git_commands_table) 提供了最常用和有用的`git`命令的简要概述。更详细的列表可以在[GitHub网站](https://training.github.com/downloads/github-git-cheat-sheet)上找到。

表 B-1\. 最常用的`git`终端命令

| 命令文本 | 命令结果 |
| --- | --- |
| `git status` | 显示*仓库*的当前状态；不执行任何修改。列出仓库中所有文件^([a](app02.html#idm45143393092848))，按照它们是*新文件*、*未跟踪*还是*已修改*进行分组。 |
| `git add` *`filename`* | *将*当前*未跟踪*的特定文件*加入*到*暂存区*。必须通过`git add`命令*将文件暂存*后才能*提交到*仓库。之后，`git status`会将*已暂存*的文件标记为*新文件*。 |
| `git add -A` | 一次性*将*所有当前*未跟踪*的文件*加入*到*暂存区*。之后，`git status`会将*已暂存*的文件标记为*新文件*。 |
| `` git commit -m "Commit message here."` `` *`filename`* | 提交特定的已暂存文件，并附上双引号之间的消息。 |
| `git commit -a -m "Commit message here."` | 提交*所有*当前*已暂存*的文件，并附上相同的提交消息。 |
| `git push` | 将所有本地提交*推送*到远程仓库。添加`--force`命令将使用本地提交的文件*覆盖*远程仓库中的任何冲突提交。 |
| `git pull` | 将所有远程文件*拉取*到本地仓库。添加`--force`命令将使用远程提交的文件*覆盖*本地仓库中的任何冲突提交。 |
| ^([a](app02.html#idm45143393092848-marker)) 如果你的仓库中有一个活动的*.gitignore*文件，那么被忽略的文件将*不会*被`git status`列出。 |
