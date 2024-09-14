# 第四十八章：深入：高斯混合模型

在前一章中探讨的 *k*-means 聚类模型简单且相对易于理解，但其简单性导致在实际应用中存在实际挑战。特别是，*k*-means 的非概率性质以及其使用简单的距离从聚类中心分配聚类成员导致在许多实际情况下性能不佳。在本章中，我们将介绍高斯混合模型，它可以被视为对 *k*-means 背后思想的扩展，同时也可以是一种超越简单聚类的强大工具。

我们从标准导入开始：

```py
In [1]: %matplotlib inline
        import matplotlib.pyplot as plt
        plt.style.use('seaborn-whitegrid')
        import numpy as np
```

# 激励高斯混合模型：*k*-means 的弱点

让我们来看一些 *k*-means 的弱点，并思考如何改进聚类模型。正如我们在前一章中看到的，对于简单而明确分离的数据，*k*-means 能够找到合适的聚类结果。

例如，如果我们有简单的数据块，*k*-means 算法可以快速地标记这些聚类，其结果与我们可能通过眼睛观察到的相似（参见 图 48-1）。

```py
In [2]: # Generate some data
        from sklearn.datasets import make_blobs
        X, y_true = make_blobs(n_samples=400, centers=4,
                               cluster_std=0.60, random_state=0)
        X = X[:, ::-1] # flip axes for better plotting
```

```py
In [3]: # Plot the data with k-means labels
        from sklearn.cluster import KMeans
        kmeans = KMeans(4, random_state=0)
        labels = kmeans.fit(X).predict(X)
        plt.scatter(X[:, 0], X[:, 1], c=labels, s=40, cmap='viridis');
```

![output 5 0](img/output_5_0.png)

###### 图 48-1：简单数据的 k-means 标签

从直觉上讲，我们可能期望某些点的聚类分配比其他点更加确定：例如，在两个中间聚类之间似乎存在非常轻微的重叠，因此我们可能对它们之间的点的聚类分配没有完全的信心。不幸的是，*k*-means 模型没有内在的概率或聚类分配不确定性的衡量方法（尽管可能可以使用自举方法来估计此不确定性）。为此，我们必须考虑模型的泛化。

关于 *k*-means 模型的一种思考方式是，它在每个聚类的中心放置一个圆（或在更高维度中，一个超球体），其半径由聚类中最远的点定义。这个半径作为训练集内聚类分配的硬截止：任何在这个圆外的点都不被视为聚类的成员。我们可以用以下函数可视化这个聚类模型（参见 图 48-2）。

```py
In [4]: from sklearn.cluster import KMeans
        from scipy.spatial.distance import cdist

        def plot_kmeans(kmeans, X, n_clusters=4, rseed=0, ax=None):
            labels = kmeans.fit_predict(X)

            # plot the input data
            ax = ax or plt.gca()
            ax.axis('equal')
            ax.scatter(X[:, 0], X[:, 1], c=labels, s=40, cmap='viridis', zorder=2)

            # plot the representation of the KMeans model
            centers = kmeans.cluster_centers_
            radii = [cdist(X[labels == i], [center]).max()
                     for i, center in enumerate(centers)]
            for c, r in zip(centers, radii):
                ax.add_patch(plt.Circle(c, r, ec='black', fc='lightgray',
                                        lw=3, alpha=0.5, zorder=1))
```

```py
In [5]: kmeans = KMeans(n_clusters=4, random_state=0)
        plot_kmeans(kmeans, X)
```

![output 8 0](img/output_8_0.png)

###### 图 48-2：k-means 模型暗示的圆形聚类

对于 *k*-means 的一个重要观察是，这些聚类模型必须是圆形的：*k*-means 没有内建的方法来处理椭圆形或椭圆形聚类。因此，例如，如果我们取同样的数据并对其进行转换，聚类分配最终变得混乱，正如你可以在 图 48-3 中看到的。

```py
In [6]: rng = np.random.RandomState(13)
        X_stretched = np.dot(X, rng.randn(2, 2))

        kmeans = KMeans(n_clusters=4, random_state=0)
        plot_kmeans(kmeans, X_stretched)
```

![output 10 0](img/output_10_0.png)

###### 图 48-3：*k*-means 对于非圆形聚类的性能不佳

凭眼观察，我们认识到这些转换后的聚类不是圆形的，因此圆形聚类会拟合效果差。然而，*k*-means 不足以解决这个问题，并试图强行将数据拟合为四个圆形聚类。这导致了聚类分配的混合，其中结果的圆形重叠：尤其是在图的右下角可见。可以想象通过使用 PCA 预处理数据来处理这种情况（参见第四十五章），但实际上不能保证这样的全局操作会使各个群体圆形化。

*k*-means 的这两个缺点——在聚类形状上的灵活性不足和缺乏概率聚类分配——意味着对于许多数据集（特别是低维数据集），其性能可能不如人们所期望的那样好。

您可以通过泛化*k*-means 模型来解决这些弱点：例如，可以通过比较每个点到*所有*聚类中心的距离来测量聚类分配的不确定性，而不是仅关注最近的距离。您还可以想象允许聚类边界为椭圆而不是圆形，以适应非圆形聚类。事实证明，这些是不同类型聚类模型——高斯混合模型的两个基本组成部分。

# 泛化 E-M：高斯混合模型

高斯混合模型（GMM）试图找到最适合模拟任何输入数据集的多维高斯概率分布混合物。在最简单的情况下，GMM 可以像*k*-means 一样用于查找聚类（参见图 48-4）。

```py
In [7]: from sklearn.mixture import GaussianMixture
        gmm = GaussianMixture(n_components=4).fit(X)
        labels = gmm.predict(X)
        plt.scatter(X[:, 0], X[:, 1], c=labels, s=40, cmap='viridis');
```

![output 13 0](img/output_13_0.png)

###### 图 48-4。数据的高斯混合模型标签

但是因为 GMM 在幕后包含一个概率模型，因此可以找到概率聚类分配——在 Scikit-Learn 中，这是通过`predict_proba`方法完成的。这将返回一个大小为`[n_samples, n_clusters]`的矩阵，用于测量任何点属于给定聚类的概率：

```py
In [8]: probs = gmm.predict_proba(X)
        print(probs[:5].round(3))
Out[8]: [[0.    0.531 0.469 0.   ]
         [0.    0.    0.    1.   ]
         [0.    0.    0.    1.   ]
         [0.    1.    0.    0.   ]
         [0.    0.    0.    1.   ]]
```

我们可以通过将每个点的大小与其预测的确定性成比例来可视化这种不确定性；查看图 48-5，我们可以看到恰好在聚类边界上的点反映了聚类分配的不确定性：

```py
In [9]: size = 50 * probs.max(1) ** 2  # square emphasizes differences
        plt.scatter(X[:, 0], X[:, 1], c=labels, cmap='viridis', s=size);
```

![output 17 0](img/output_17_0.png)

###### 图 48-5。GMM 概率标签：点的大小显示概率大小

在幕后，高斯混合模型与*k*-means 非常相似：它使用期望最大化方法，大致如下：

1.  选择位置和形状的初始猜测。

1.  直到收敛为止重复：

    1.  *E 步骤*：对于每个点，找到编码每个聚类成员概率的权重。

    1.  *M 步*：对于每个聚类，根据*所有*数据点更新其位置、归一化和形状，利用权重。

其结果是，每个聚类不再关联于硬边界的球体，而是与平滑的高斯模型相关联。就像*k*-means 期望最大化方法一样，这种算法有时会错过全局最优解，因此在实践中使用多个随机初始化。

让我们创建一个函数，通过根据 GMM 输出绘制椭圆来帮助我们可视化 GMM 聚类的位置和形状：

```py
In [10]: from matplotlib.patches import Ellipse

         def draw_ellipse(position, covariance, ax=None, **kwargs):
             """Draw an ellipse with a given position and covariance"""
             ax = ax or plt.gca()

             # Convert covariance to principal axes
             if covariance.shape == (2, 2):
                 U, s, Vt = np.linalg.svd(covariance)
                 angle = np.degrees(np.arctan2(U[1, 0], U[0, 0]))
                 width, height = 2 * np.sqrt(s)
             else:
                 angle = 0
                 width, height = 2 * np.sqrt(covariance)

             # Draw the ellipse
             for nsig in range(1, 4):
                 ax.add_patch(Ellipse(position, nsig * width, nsig * height,
                                      angle, **kwargs))

         def plot_gmm(gmm, X, label=True, ax=None):
             ax = ax or plt.gca()
             labels = gmm.fit(X).predict(X)
             if label:
                 ax.scatter(X[:, 0], X[:, 1], c=labels, s=40, cmap='viridis',
                            zorder=2)
             else:
                 ax.scatter(X[:, 0], X[:, 1], s=40, zorder=2)
             ax.axis('equal')

             w_factor = 0.2 / gmm.weights_.max()
             for pos, covar, w in zip(gmm.means_, gmm.covariances_, gmm.weights_):
                 draw_ellipse(pos, covar, alpha=w * w_factor)
```

有了这些基础，我们可以看看四分量 GMM 对我们的初始数据给出了什么结果（参见图 48-6）。

```py
In [11]: gmm = GaussianMixture(n_components=4, random_state=42)
         plot_gmm(gmm, X)
```

![output 21 0](img/output_21_0.png)

###### 图 48-6. 存在圆形聚类的四分量 GMM

同样地，我们可以使用 GMM 方法拟合我们的伸展数据集；允许完全协方差模型将适合甚至是非常椭圆形、拉伸的聚类，正如我们在图 48-7 中所看到的。

```py
In [12]: gmm = GaussianMixture(n_components=4, covariance_type='full',
                               random_state=42)
         plot_gmm(gmm, X_stretched)
```

![output 23 0](img/output_23_0.png)

###### 图 48-7. 存在非圆形聚类的四分量 GMM

这清楚地表明，GMM 解决了之前在*k*-means 中遇到的两个主要实际问题。

# 选择协方差类型

如果您查看前面拟合的细节，您会发现在每个拟合中设置了`covariance_type`选项。该超参数控制每个聚类形状的自由度；对于任何给定的问题，仔细设置这一点至关重要。默认值是`covariance_type="diag"`，这意味着可以独立设置每个维度上的聚类大小，生成的椭圆受限于与轴对齐。`covariance_type="spherical"`是一个稍微简单且更快的模型，它限制了聚类形状，使得所有维度相等。结果聚类将具有与*k*-means 类似的特征，尽管它并非完全等价。一个更复杂和计算开销更大的模型（特别是在维度增长时）是使用`covariance_type="full"`，它允许将每个聚类建模为带有任意方向的椭圆。图 48-8 表示了这三种选择对单个聚类的影响。

![05.12 协方差类型](img/05.12-covariance-type.png)

###### 图 48-8. GMM 协方差类型可视化¹

# 高斯混合模型作为密度估计

尽管 GMM 通常被归类为聚类算法，但从根本上讲，它是一种用于*密度估计*的算法。也就是说，对某些数据进行 GMM 拟合的结果在技术上不是聚类模型，而是描述数据分布的生成概率模型。

以 Scikit-Learn 的`make_moons`函数生成的数据为例，介绍在第四十七章中。

```py
In [13]: from sklearn.datasets import make_moons
         Xmoon, ymoon = make_moons(200, noise=.05, random_state=0)
         plt.scatter(Xmoon[:, 0], Xmoon[:, 1]);
```

![output 28 0](img/output_28_0.png)

###### 图 48-9\. GMM 应用于具有非线性边界的聚类

如果我们尝试用一个两组分的 GMM 作为聚类模型来拟合它，结果并不特别有用（见图 48-10）。

```py
In [14]: gmm2 = GaussianMixture(n_components=2, covariance_type='full',
                                random_state=0)
         plot_gmm(gmm2, Xmoon)
```

![output 30 0](img/output_30_0.png)

###### 图 48-10\. 对非线性聚类拟合的两组分 GMM

但是，如果我们使用更多组分并忽略聚类标签，我们会发现拟合结果更接近输入数据（见图 48-11）。

```py
In [15]: gmm16 = GaussianMixture(n_components=16, covariance_type='full',
                                 random_state=0)
         plot_gmm(gmm16, Xmoon, label=False)
```

![output 32 0](img/output_32_0.png)

###### 图 48-11\. 使用多个 GMM 组件来建模点分布

这里的 16 个高斯分量的混合并不是为了找到数据的分离聚类，而是为了对输入数据的整体*分布*进行建模。这是一个生成模型，意味着 GMM 给了我们一个生成新随机数据的方法，其分布类似于我们的原始输入数据。例如，这里有 400 个新点从这个 16 组分的 GMM 拟合到我们的原始数据中绘制出来（见图 48-12）。

```py
In [16]: Xnew, ynew = gmm16.sample(400)
         plt.scatter(Xnew[:, 0], Xnew[:, 1]);
```

![output 34 0](img/output_34_0.png)

###### 图 48-12\. 从 16 组分 GMM 中绘制的新数据

GMM 作为一种灵活的方法，方便地对数据的任意多维分布进行建模。

GMM 作为生成模型的事实给了我们一种自然的方法来确定给定数据集的最优组件数。生成模型本质上是数据集的概率分布，因此我们可以简单地在模型下评估数据的*似然性*，使用交叉验证来避免过度拟合。另一种校正过度拟合的方法是使用一些分析标准来调整模型的似然性，例如[阿卡奇信息准则（AIC）](https://oreil.ly/BmH9X)或[贝叶斯信息准则（BIC）](https://oreil.ly/Ewivh)。Scikit-Learn 的`GaussianMixture`估计器实际上包含内置方法来计算这两者，因此使用这种方法非常容易。

让我们看看我们的 moons 数据集的 GMM 组件数对应的 AIC 和 BIC（见图 48-13）。

```py
In [17]: n_components = np.arange(1, 21)
         models = [GaussianMixture(n, covariance_type='full',
                                   random_state=0).fit(Xmoon)
                   for n in n_components]

         plt.plot(n_components, [m.bic(Xmoon) for m in models], label='BIC')
         plt.plot(n_components, [m.aic(Xmoon) for m in models], label='AIC')
         plt.legend(loc='best')
         plt.xlabel('n_components');
```

![output 37 0](img/output_37_0.png)

###### 图 48-13\. AIC 和 BIC 的可视化，用于选择 GMM 组件数

最优的聚类数是能够最小化 AIC 或 BIC 的值，具体取决于我们希望使用哪种近似方法。AIC 告诉我们，我们之前选择的 16 组分可能太多了：选择大约 8-12 组分可能更合适。对于这类问题，BIC 通常推荐一个更简单的模型。

注意重要的一点：组件数量的选择衡量的是 GMM 作为密度估计器的工作效果，而不是作为聚类算法的工作效果。我鼓励您主要将 GMM 视为密度估计器，并仅在简单数据集内合适时用它进行聚类。

# 示例：使用 GMM 生成新数据

我们刚刚看到了使用 GMM 作为生成模型的简单示例，以便从定义为输入数据分布的模型中创建新的样本。在这里，我们将继续这个想法，并从之前使用过的标准数字语料库中生成*新的手写数字*。

首先，让我们使用 Scikit-Learn 的数据工具加载数字数据：

```py
In [18]: from sklearn.datasets import load_digits
         digits = load_digits()
         digits.data.shape
Out[18]: (1797, 64)
```

接下来，让我们绘制前 50 个样本，以确切回顾我们正在查看的内容（参见图 48-14）。

```py
In [19]: def plot_digits(data):
             fig, ax = plt.subplots(5, 10, figsize=(8, 4),
                                    subplot_kw=dict(xticks=[], yticks=[]))
             fig.subplots_adjust(hspace=0.05, wspace=0.05)
             for i, axi in enumerate(ax.flat):
                 im = axi.imshow(data[i].reshape(8, 8), cmap='binary')
                 im.set_clim(0, 16)
         plot_digits(digits.data)
```

![output 42 0](img/output_42_0.png)

###### 图 48-14\. 手写数字输入

我们有将近 1,800 个 64 维度的数字样本，我们可以在其上构建一个混合高斯模型（GMM）以生成更多数字。在这么高维度的空间中，GMM 可能会有收敛困难，因此我们将从数据中开始使用一个可逆的降维算法。这里我们将使用简单的 PCA，要求它在投影数据中保留 99%的方差：

```py
In [20]: from sklearn.decomposition import PCA
         pca = PCA(0.99, whiten=True)
         data = pca.fit_transform(digits.data)
         data.shape
Out[20]: (1797, 41)
```

结果是 41 个维度，几乎没有信息损失的减少了近 1/3。鉴于这个投影数据，让我们使用 AIC 来确定我们应该使用多少个 GMM 组件（参见图 48-15）。

```py
In [21]: n_components = np.arange(50, 210, 10)
         models = [GaussianMixture(n, covariance_type='full', random_state=0)
                   for n in n_components]
         aics = [model.fit(data).aic(data) for model in models]
         plt.plot(n_components, aics);
```

![output 46 0](img/output_46_0.png)

###### 图 48-15\. 选择适当的 GMM 组件数量的 AIC 曲线

看起来大约使用 140 个组件可以最小化 AIC；我们将使用这个模型。让我们快速将其拟合到数据上并确认它已经收敛：

```py
In [22]: gmm = GaussianMixture(140, covariance_type='full', random_state=0)
         gmm.fit(data)
         print(gmm.converged_)
Out[22]: True
```

现在我们可以在这个 41 维度的投影空间内绘制 100 个新点的样本，使用 GMM 作为生成模型：

```py
In [23]: data_new, label_new = gmm.sample(100)
         data_new.shape
Out[23]: (100, 41)
```

最后，我们可以使用 PCA 对象的逆变换来构造新的数字（参见图 48-16）。

```py
In [24]: digits_new = pca.inverse_transform(data_new)
         plot_digits(digits_new)
```

![output 52 0](img/output_52_0.png)

###### 图 48-16\. 从 GMM 估计器的基础模型中随机绘制的“新”数字

大多数结果看起来像数据集中合理的数字！

考虑我们在这里所做的：鉴于手写数字的抽样，我们已经模拟了该数据的分布，以便我们可以从数据中生成全新的样本：这些是“手写数字”，它们不会单独出现在原始数据集中，而是捕捉了混合模型建模的输入数据的一般特征。这样的手写数字的生成模型在贝叶斯生成分类器的组成部分中可以非常有用，这一点我们将在下一章看到。

¹ 生成此图的代码可以在[在线附录](https://oreil.ly/MLsk8)中找到。
