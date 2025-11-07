# 附录 A. 安装 Conda

Conda 包管理器在第五章中介绍。按照以下说明在您的系统上安装 Conda：

1.  打开 *Miniconda* 的主页 [`mng.bz/BM5r`](http://mng.bz/BM5r)。

1.  下载适用于 Mac OS X 或 Linux 的 Miniconda 安装程序（Metaflow 目前不支持 Windows 原生）。

1.  下载完包后，在终端中按照以下方式执行安装程序：

    ```
    bash Miniconda3-latest-MacOSX-x86_64.sh
    ```

    将包名替换为您实际下载的包名。

1.  当安装程序询问“您希望安装程序初始化 Miniconda3 吗？”时，回答是。

1.  安装完成后，重新启动您的终端以使更改生效。

1.  在新的终端中运行以下命令：

    ```
    conda config --add channels conda-forge
    ```

    这将使 conda-forge.org 上的社区维护包在您的安装中可用。

就这样！Metaflow 中的 @conda 装饰器将负责使用新安装的 Conda 安装包。
