# 第七章：出租车行程数据的地理空间和时间数据分析

地理空间数据指的是数据中嵌入有某种形式的位置信息。这类数据每天由数十亿来源（如手机和传感器）大规模生成。关于人类和机器移动的数据以及来自遥感的数据对我们的经济和整体福祉至关重要。地理空间分析可以为我们提供处理所有这些数据并将其用于解决面临问题的工具和方法。

在地理空间分析方面，PySpark 和 PyData 生态系统在过去几年中发生了显著发展。它们被各行各业用来处理富有位置信息的数据，并对我们的日常生活产生影响。地方交通是一个可以明显看到地理空间数据应用的日常活动领域。过去几年中数字打车服务的流行使我们更加关注地理空间技术。在本章中，我们将利用我们在该领域中的 PySpark 和数据分析技能来处理一个数据集，该数据集包含有关纽约市出租车行程的信息。

了解出租车经济的一个重要统计数据是*利用率*：出租车在道路上被一名或多名乘客占用的时间比例。影响利用率的一个因素是乘客的目的地：在正午将乘客送至联合广场附近的出租车很可能在一两分钟内找到下一个乘客，而在凌晨 2 点将乘客送至史泰登岛的出租车可能需要驾驶回曼哈顿才能找到下一个乘客。我们希望量化这些影响，并找出出租车在其将乘客卸下的区域（曼哈顿、布鲁克林、皇后区、布朗克斯、史泰登岛或其他地方，如纽瓦克自由国际机场）找到下一个乘客的平均时间。

我们将从设置数据集开始，然后深入进行地理空间分析。我们将学习关于 GeoJSON 格式的知识，并结合 PyData 生态系统中的工具与 PySpark 使用。我们将使用 GeoPandas 来处理*地理空间信息*，如经度和纬度点以及空间边界。最后，我们将通过执行会话化类型的分析来处理数据的时间特征，比如日期和时间。这将帮助我们了解纽约市出租车的利用情况。PySpark 的 DataFrame API 提供了处理时间数据的内置数据类型和方法。

让我们通过下载数据集并使用 PySpark 进行探索来开始吧。

# 准备数据

对于此分析，我们只考虑 2013 年 1 月的票价数据，解压后大约为 2.5 GB 数据。您可以访问[2013 年每个月的数据](https://oreil.ly/7m7Ki)，如果您有一个足够大的 PySpark 集群可供使用，可以对整年的数据重新进行以下分析。现在，让我们在客户机上创建一个工作目录，并查看票价数据的结构：

```
$ mkdir taxidata
$ cd taxidata
$ curl -O https://storage.googleapis.com/aas-data-sets/trip_data_1.csv.zip
$ unzip trip_data_1.csv.zip
$ head -n 5 trip_data_1.csv

...

medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,...
89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,...
0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,...
0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,...
DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,...
```

文件头后的每一行表示 CSV 格式中的单个出租车行程。对于每次行程，我们有一些有关出租车的属性（中介牌号的散列版本）以及驾驶员的信息（出租车驾驶执照的散列版本，这就是出租车驾驶许可证的称呼），有关行程何时开始和结束的一些时间信息，以及乘客上下车的经度/纬度坐标。

创建一个*taxidata*目录，并将行程数据复制到存储中：

```
$ mkdir taxidata
$ mv trip_data_1.csv taxidata/
```

在这里我们使用了本地文件系统，但您可能不是这种情况。现在更常见的是使用云原生文件系统，如 AWS S3 或 GCS。在这种情况下，您需要分别将数据上传到 S3 或 GCS。

现在开始 PySpark shell：

```
$ pyspark
```

一旦 PySpark shell 加载完成，我们就可以从出租车数据创建一个数据集，并检查前几行，就像我们在其他章节中所做的那样：

```
taxi_raw = pyspark.read.option("header", "true").csv("taxidata")
taxi_raw.show(1, vertical=True)

...

RECORD 0----------------------------------
 medallion          | 89D227B655E5C82AE...
 hack_license       | BA96DE419E711691B...
 vendor_id          | CMT
 rate_code          | 1
 store_and_fwd_flag | N
 pickup_datetime    | 2013-01-01 15:11:48
 dropoff_datetime   | 2013-01-01 15:18:10
 passenger_count    | 4
 trip_time_in_secs  | 382
 trip_distance      | 1.0
 pickup_longitude   | -73.978165
 pickup_latitude    | 40.757977
 dropoff_longitude  | -73.989838
 dropoff_latitude   | 40.751171
only showing top 1 row

...
```

乍看之下，这看起来是一个格式良好的数据集。让我们再次查看 DataFrame 的架构：

```
taxi_raw.printSchema()
...
root
 |-- medallion: string (nullable = true)
 |-- hack_license: string (nullable = true)
 |-- vendor_id: string (nullable = true)
 |-- rate_code: integer (nullable = true)
 |-- store_and_fwd_flag: string (nullable = true)
 |-- pickup_datetime: string (nullable = true)
 |-- dropoff_datetime: string (nullable = true)
 |-- passenger_count: integer (nullable = true)
 |-- trip_time_in_secs: integer (nullable = true)
 |-- trip_distance: double (nullable = true)
 |-- pickup_longitude: double (nullable = true)
 |-- pickup_latitude: double (nullable = true)
 |-- dropoff_longitude: double (nullable = true)
 |-- dropoff_latitude: double (nullable = true)
 ...
```

我们将`pickup_datetime`和`dropoff_datetime`字段表示为`Strings`，并将接送地点的个体`(x,y)`坐标存储在其自己的`Doubles`字段中。我们希望将日期时间字段转换为时间戳，因为这样可以方便地进行操作和分析。

## 将日期时间字符串转换为时间戳

如前所述，PySpark 提供了处理时间数据的开箱即用方法。

具体来说，我们将使用`to_timestamp`函数解析日期时间字符串并将其转换为时间戳：

```
from pyspark.sql import functions as fun

taxi_raw = taxi_raw.withColumn('pickup_datetime',
                                fun.to_timestamp(fun.col('pickup_datetime'),
                                                "yyyy-MM-dd HH:mm:ss"))
taxi_raw = taxi_raw.withColumn('dropoff_datetime',
                                fun.to_timestamp(fun.col('dropoff_datetime'),
                                                "yyyy-MM-dd HH:mm:ss"))
```

让我们再次查看架构：

```
taxi_raw.printSchema()
...

root
 |-- medallion: string (nullable = true)
 |-- hack_license: string (nullable = true)
 |-- vendor_id: string (nullable = true)
 |-- rate_code: integer (nullable = true)
 |-- store_and_fwd_flag: string (nullable = true)
 |-- pickup_datetime: timestamp (nullable = true)
 |-- dropoff_datetime: timestamp (nullable = true)
 |-- passenger_count: integer (nullable = true)
 |-- trip_time_in_secs: integer (nullable = true)
 |-- trip_distance: double (nullable = true)
 |-- pickup_longitude: double (nullable = true)
 |-- pickup_latitude: double (nullable = true)
 |-- dropoff_longitude: double (nullable = true)
 |-- dropoff_latitude: double (nullable = true)

 ...
```

现在，`pickup_datetime`和`dropoff_datetime`字段都是时间戳了。干得好！

我们提到这个数据集包含 2013 年 1 月的行程。不过，不要只听我们的话。我们可以通过对`pickup_datetime`字段进行排序来确认数据中的最新日期时间。为此，我们使用 DataFrame 的`sort`方法结合 PySpark 列的`desc`方法：

```
taxi_raw.sort(fun.col("pickup_datetime").desc()).show(3, vertical=True)
...

-RECORD 0----------------------------------
 medallion          | EA00A64CBDB68C77D...
 hack_license       | 2045C77002FA0F2E0...
 vendor_id          | CMT
 rate_code          | 1
 store_and_fwd_flag | N
 pickup_datetime    | 2013-01-31 23:59:59
 dropoff_datetime   | 2013-02-01 00:08:39
 passenger_count    | 1
 trip_time_in_secs  | 520
 trip_distance      | 1.5
 pickup_longitude   | -73.970528
 pickup_latitude    | 40.75502
 dropoff_longitude  | -73.981201
 dropoff_latitude   | 40.769104
-RECORD 1----------------------------------
 medallion          | E3F00BB3F4E710383...
 hack_license       | 10A2B96DE39865918...
 vendor_id          | CMT
 rate_code          | 1
 store_and_fwd_flag | N
 pickup_datetime    | 2013-01-31 23:59:59
 dropoff_datetime   | 2013-02-01 00:05:16
 passenger_count    | 1
 trip_time_in_secs  | 317
 trip_distance      | 1.0
 pickup_longitude   | -73.990685
 pickup_latitude    | 40.719158
 dropoff_longitude  | -74.003288
 dropoff_latitude   | 40.71521
-RECORD 2----------------------------------
 medallion          | 83D8E776A05EEF731...
 hack_license       | E6D27C8729EF55D20...
 vendor_id          | CMT
 rate_code          | 1
 store_and_fwd_flag | N
 pickup_datetime    | 2013-01-31 23:59:58
 dropoff_datetime   | 2013-02-01 00:04:19
 passenger_count    | 1
 trip_time_in_secs  | 260
 trip_distance      | 0.8
 pickup_longitude   | -73.982452
 pickup_latitude    | 40.77277
 dropoff_longitude  | -73.989227
 dropoff_latitude   | 40.766754
only showing top 3 rows
...
```

在确保数据类型正确后，让我们检查数据中是否存在任何不一致之处。

## 处理无效记录

任何在大规模、真实世界数据集上工作过的人都知道，这些数据集中必然包含至少一些不符合编写处理代码人员期望的记录。许多 PySpark 管道由于无效记录导致解析逻辑抛出异常而失败。在进行交互式分析时，我们可以通过关注关键变量来感知数据中潜在的异常。

在我们的案例中，包含地理空间和时间信息的变量存在不一致性是值得注意的。这些列中的空值肯定会影响我们的分析结果。

```
geospatial_temporal_colnames = ["pickup_longitude", "pickup_latitude", \
                                "dropoff_longitude", "dropoff_latitude", \
                                "pickup_datetime", "dropoff_datetime"]
taxi_raw.select([fun.count(fun.when(fun.isnull(c), c)).\
                            alias(c) for c in geospatial_temporal_colnames]).\
                show()
...

+----------------+---------------+-----------------
|pickup_longitude|pickup_latitude|dropoff_longitude
+----------------+---------------+-----------------
|               0|              0|               86
+----------------+---------------+-----------------
+----------------+---------------+----------------+
|dropoff_latitude|pickup_datetime|dropoff_datetime|
+----------------+---------------+----------------+
|              86|              0|               0|
+----------------+---------------+----------------+
```

让我们从数据中删除空值：

```
taxi_raw = taxi_raw.na.drop(subset=geospatial_temporal_colnames)
```

另一个常识检查是检查纬度和经度记录中值为零的情况。我们知道对于我们关注的地区，这些值是无效的：

```
print("Count of zero dropoff, pickup latitude and longitude records")
taxi_raw.groupBy((fun.col("dropoff_longitude") == 0) |
  (fun.col("dropoff_latitude") == 0) |
  (fun.col("pickup_longitude") == 0) |
  (fun.col("pickup_latitude") == 0)).\ ![1](img/1.png)
    count().show()
...

Count of zero dropoff, pickoff latitude and longitude records
+---------------+
| ...  |   count|
+------+--------+
| true |  285909|
| false|14490620|
+---------------+
```

![1](img/#co_geospatial_and_temporal_data_analysis___span_class__keep_together__on_taxi_trip_data__span__CO1-1)

如果任何记录的任一条件为 `True`，则多个`OR`条件将为真。

我们有很多这样的情况。如果看起来一辆出租车带乘客去了南极，我们可以相当有信心地认为该记录是无效的，并应从分析中排除。我们不会立即删除它们，而是在下一节结束时回顾它们，看看它们如何影响我们的分析。

在生产环境中，我们逐个处理这些异常，通过检查各个任务的日志，找出抛出异常的代码行，然后调整代码以忽略或修正无效记录。这是一个繁琐的过程，常常感觉像是在玩打地鼠游戏：就在我们修复一个异常时，我们发现分区内稍后出现的记录中又有另一个异常。

当数据科学家处理新数据集时，一个常用的策略是在他们的解析代码中添加 `try-except` 块，以便任何无效记录都可以被写入日志，而不会导致整个作业失败。如果整个数据集中只有少数无效记录，我们可能可以忽略它们并继续分析。

现在我们已经准备好我们的数据集，让我们开始地理空间分析。

# 地理空间分析

有两种主要类型的地理空间数据——矢量和栅格——以及用于处理每种类型的不同工具。在我们的案例中，我们有出租车行程记录的纬度和经度，以及以 GeoJSON 格式存储的矢量数据，表示纽约不同区域的边界。我们已经查看了纬度和经度点。让我们先看看 GeoJSON 数据。

## GeoJSON 简介

我们将用于纽约市各区域边界的数据以*GeoJSON*格式呈现。GeoJSON 中的核心对象称为*要素*，由一个*几何*实例和一组称为*属性*的键值对组成。几何是如点、线或多边形等形状。一组要素称为`FeatureCollection`。让我们下载纽约市区地图的 GeoJSON 数据，并查看其结构。

在客户端机器的*taxidata*目录中，下载数据并将文件重命名为稍短的名称：

```
$ url="https://nycdatastables.s3.amazonaws.com/\
 2013-08-19T18:15:35.172Z/nyc-borough-boundaries-polygon.geojson"
$ curl -O $url
$ mv nyc-borough-boundaries-polygon.geojson nyc-boroughs.geojson
```

打开文件并查看要素记录。注意属性和几何对象，例如表示区域边界的多边形和包含区域名称及其他相关信息的属性。

```
$ head -n 7 data/trip_data_ch07/nyc-boroughs.geojson
...
{
"type": "FeatureCollection",

"features": [{ "type": "Feature", "id": 0, "properties": { "boroughCode": 5, ...
,
{ "type": "Feature", "id": 1, "properties": { "boroughCode": 5, ...
```

## GeoPandas

在选择用于执行地理空间分析的库时，首先要考虑的是确定你需要处理的数据类型。我们需要一个可以解析 GeoJSON 数据并处理空间关系的库，比如检测给定的经度/纬度对是否包含在表示特定区域边界的多边形内。我们将使用[GeoPandas 库](https://geopandas.org)来完成这项任务。GeoPandas 是一个开源项目，旨在使 Python 中的地理空间数据处理更加简单。它扩展了 pandas 库中使用的数据类型，允许对几何数据类型进行空间操作，我们在之前的章节中已经使用过 pandas 库。

使用 pip 安装 GeoPandas 包：

```
pip3 install geopandas
```

现在让我们开始研究出租车数据的地理空间方面。对于每次行程，我们有经度/纬度对，表示乘客上车和下车的位置。我们希望能够确定每个经度/纬度对属于哪个区域，并识别没有在五个区域之一开始或结束的行程。例如，如果一辆出租车将乘客从曼哈顿送往纽瓦克自由国际机场，那将是一个有效的行程，也值得分析，尽管它不会在五个区域中的任何一个结束。

要执行我们的区域分析，我们需要加载我们之前下载并存储在*nyc-boroughs.geojson*文件中的 GeoJSON 数据：

```
import geopandas as gdp

gdf = gdp.read_file("./data/trip_data_ch07/nyc-boroughs.geojson")
```

在使用 GeoJSON 特性处理出租车行程数据之前，我们应该花点时间考虑如何组织这些地理空间数据以实现最大效率。一种选择是研究针对地理空间查找进行优化的数据结构，比如四叉树，然后找到或编写我们自己的实现。不过，我们将尝试提出一个快速的启发式方法，以便我们可以跳过那部分工作。

我们将通过`gdf`迭代，直到找到一个几何图形包含给定经度/纬度的`point`。大多数纽约市的出租车行程始发和结束在曼哈顿，因此如果代表曼哈顿的地理空间特征在序列中较早出现，我们的搜索会相对快速结束。我们可以利用每个特征的`boroughCode`属性作为排序键，曼哈顿的代码等于 1，斯塔滕岛的代码等于 5。在每个行政区的特征中，我们希望与较大多边形相关联的特征优先于与较小多边形相关联的特征，因为大多数行程将会发生在每个行政区的“主要”区域之间。

我们将计算与每个特征几何相关联的区域，并将其存储为一个新列：

```
gdf = gdf.to_crs(3857)

gdf['area'] = gdf.apply(lambda x: x['geometry'].area, axis=1)
gdf.head(5)
...

    boroughCode  borough        @id     geometry     area
0   5            Staten Island 	http://nyc.pediacities.com/Resource/Borough/St...
1   5            Staten Island 	http://nyc.pediacities.com/Resource/Borough/St...
2   5            Staten Island 	http://nyc.pediacities.com/Resource/Borough/St...
3   5            Staten Island 	http://nyc.pediacities.com/Resource/Borough/St...
4   4            Queens         http://nyc.pediacities.com/Resource/Borough/Qu...
```

将特征按照行政区代码和每个特征几何区域的组合排序应该能解决问题：

```
gdf = gdf.sort_values(by=['boroughCode', 'area'], ascending=[True, False])
gdf.head(5)
...
    boroughCode  borough    @id     geometry     area
72  1            Manhattan  http://nyc.pediacities.com/Resource/Borough/Ma...
71  1            Manhattan  http://nyc.pediacities.com/Resource/Borough/Ma...
51  1            Manhattan  http://nyc.pediacities.com/Resource/Borough/Ma...
69  1            Manhattan  http://nyc.pediacities.com/Resource/Borough/Ma...
73  1            Manhattan  http://nyc.pediacities.com/Resource/Borough/Ma...
```

请注意，我们基于面积值按降序排序，因为我们希望最大的多边形首先出现，而`sort_values`默认按升序排序。

现在我们可以将`gdf` GeoPandas DataFrame 中排序后的特征广播到集群，并编写一个函数，利用这些特征来找出特定行程结束在五个行政区中的哪一个（如果有的话）：

```
b_gdf = spark.sparkContext.broadcast(gdf)

def find_borough(latitude,longitude):
    mgdf = b_gdf.value.apply(lambda x: x['borough'] if \
                              x['geometry'].\
                              intersects(gdp.\
                                        points_from_xy(
                                            [longitude], \
                                            [latitude])[0]) \
                              else None, axis=1)
    idx = mgdf.first_valid_index()
    return mgdf.loc[idx] if idx is not None else None

find_borough_udf = fun.udf(find_borough, StringType())
```

我们可以将`find_borough`应用于`taxi_raw` DataFrame 中的行程，以创建一个按行政区划分的行程直方图：

```
df_with_boroughs = taxi_raw.\
                    withColumn("dropoff_borough", \
                              find_borough_udf(
                                fun.col("dropoff_latitude"),\
                                fun.col('dropoff_longitude')))

df_with_boroughs.groupBy(fun.col("dropoff_borough")).count().show()
...
+-----------------------+--------+
|     dropoff_borough   |   count|
+-----------------------+--------+
|                 Queens|  672192|
|                   null| 7942421|
|               Brooklyn|  715252|
|          Staten Island|    3338|
|              Manhattan|12979047|
|                  Bronx|   67434|
+-----------------------+--------+
```

我们预料到，绝大多数行程的终点在曼哈顿区，而只有相对较少的行程终点在斯塔滕岛。一个令人惊讶的观察是，有多少行程的终点在任何一个行政区外；`null`记录的数量远远大于在布朗克斯结束的出租车行程的数量。

我们之前讨论过如何处理这些无效记录，但并未将其删除。现在由你来移除这些记录并从清理后的数据中创建直方图。完成后，你会注意到`null`条目的减少，留下了更合理的在城市外进行下车的观察数据。

在处理了数据的地理空间方面，让我们现在通过使用 PySpark 对数据的时间特性进行更深入的挖掘来执行会话化。

# PySpark 中的会话化

在这种分析中，我们希望分析单个实体随时间执行一系列事件的类型被称为*会话化*，通常在 Web 日志中执行以分析网站用户的行为。 PySpark 提供了`Window`和聚合函数，可以用来执行这种分析。这些允许我们专注于业务逻辑，而不是试图实现复杂的数据操作和计算。我们将在下一节中使用这些功能来更好地理解数据集中出租车的利用率。

会话化可以是揭示数据洞察力并构建可帮助人们做出更好决策的新数据产品的强大技术。例如，谷歌的拼写纠正引擎是建立在每天从其网站属性上发生的每个事件（搜索、点击、地图访问等）的用户活动会话之上的。为了识别可能的拼写纠正候选项，谷歌处理这些会话，寻找用户输入一个查询后没有点击任何内容、几秒钟后再输入稍有不同的查询，然后点击一个结果且不返回谷歌的情况。然后，计算这种模式对于任何一对查询发生的频率。如果发生频率足够高（例如，每次看到查询“untied stats”后几秒钟后都跟随查询“united states”），则我们认为第二个查询是对第一个查询的拼写纠正。

这项分析利用事件日志中体现的人类行为模式来构建一个拼写纠正引擎，该引擎使用的数据比任何从字典创建的引擎更为强大。该引擎可用于任何语言的拼写纠正，并能纠正可能不包含在任何字典中的单词（例如新创企业的名称）或像“untied stats”这样的查询，其中没有任何单词拼写错误！谷歌使用类似的技术来显示推荐和相关搜索，以及决定哪些查询应返回一个 OneBox 结果，即在搜索页面本身给出查询答案，而无需用户点击转到不同页面。OneBox 可用于天气、体育比赛得分、地址以及许多其他类型的查询。

到目前为止，每个实体发生的事件集合的信息分散在 DataFrame 的分区中，因此，为了分析，我们需要将这些相关事件放在一起并按时间顺序排列。在接下来的部分中，我们将展示如何使用高级 PySpark 功能有效地构建和分析会话。

## 构建会话：PySpark 中的二次排序

在 PySpark 中创建会话的简单方法是对要创建会话的标识符执行`groupBy`，然后按时间戳标识符进行后续事件排序。如果每个实体只有少量事件，这种方法将表现得相当不错。但是，由于这种方法要求任何特定实体的所有事件同时在内存中以进行排序，所以随着每个实体的事件数量越来越大，它将无法扩展。我们需要一种构建会话的方法，不需要将特定实体的所有事件同时保留在内存中以进行排序。

在 MapReduce 中，我们可以通过执行*二次排序*来构建会话，其中我们创建一个由标识符和时间戳值组成的复合键，对所有记录按复合键排序，然后使用自定义分区器和分组函数确保同一标识符的所有记录出现在同一输出分区中。幸运的是，PySpark 也可以通过使用`Window`函数支持类似的模式：

```
from pyspark.sql import Window

window_spec = Window.partitionBy("hack_license").\
                      orderBy(fun.col("hack_license"),
                              fun.col("pickup_datetime"))
```

首先，我们使用`partitionBy`方法确保所有具有相同`license`列值的记录最终位于同一个分区中。然后，在每个分区内，我们按其`license`值对记录进行排序（使得同一驾驶员的所有行程出现在一起），然后再按其`pickupTime`排序，以使行程序列按排序顺序出现在分区内。现在，当我们聚合行程记录时，我们可以确保行程按照适合会话分析的方式进行排序。由于此操作触发了洗牌和相当数量的计算，并且我们需要多次使用结果，因此我们将它们缓存：

```
window_spec.cache()
```

执行会话化管道是一项昂贵的操作，而且会话化数据通常对我们可能要执行的许多不同分析任务都很有用。在可能希望稍后继续分析或与其他数据科学家合作的环境中，通过仅执行一次会话化大型数据集并将会话化数据写入诸如 S3 或 HDFS 之类的文件系统，可以分摊会话化成本，使其可用于回答许多不同的问题是一个好主意。仅执行一次会话化还是一种强制执行会话定义标准规则于整个数据科学团队的好方法，这对确保结果的苹果对苹果比较具有相同的好处。

此时，我们准备分析我们的会话数据，以查看司机在特定区域卸客后多长时间找到下一个乘客。我们将使用之前创建的`lag`函数以及`window_spec`对象来获取两次行程，并计算第一次行程的卸客时间和第二次行程的接客时间之间的持续时间（以秒为单位）：

```
df_ with_ borough_durations = df_with_boroughs.\
            withColumn("trip_time_difference", \
            fun.col("pickup_datetime") - fun.lag(fun.col("pickup_datetime"),
                                          1). \
            over(window_spec)).show(50, vertical=True)
```

现在，我们应该进行验证检查，确保大部分持续时间是非负的：

```
df_with_borough_durations.
  selectExpr("floor(seconds / 3600) as hours").
  groupBy("hours").
  count().
  sort("hours").
  show()
...
+-----+--------+
|hours|   count|
+-----+--------+
|   -3|       2|
|   -2|      16|
|   -1|    4253|
|    0|13359033|
|    1|  347634|
|    2|   76286|
|    3|   24812|
|    4|   10026|
|    5|    4789|
```

只有少数记录具有负持续时间，当我们更仔细地检查它们时，似乎没有任何我们可以用来理解错误数据来源的共同模式。如果我们从输入数据集中排除这些负持续时间记录，并查看按区域的接车时间的平均值和标准差，我们可以看到这个：

```
df_with_borough_durations.
  where("seconds > 0 AND seconds < 60*60*4").
  groupBy("borough").
  agg(avg("seconds"), stddev("seconds")).
  show()
...
+-------------+------------------+--------------------+
|      borough|      avg(seconds)|stddev_samp(seconds)|
+-------------+------------------+--------------------+
|       Queens|2380.6603554494727|  2206.6572799118035|
|           NA|  2006.53571169866|  1997.0891370324784|
|     Brooklyn| 1365.394576250576|  1612.9921698951398|
|Staten Island|         2723.5625|  2395.7745475546385|
|    Manhattan| 631.8473780726746|   1042.919915477234|
|        Bronx|1975.9209786770646|   1704.006452085683|
+-------------+------------------+--------------------+
```

正如我们预期的那样，数据显示，曼哈顿的下车时间最短，约为 10 分钟。在布鲁克林结束的出租车行程的空闲时间是这个的两倍以上，而在史泰登岛结束的相对较少的行程平均需要司机近 45 分钟才能到达下一个乘客。

正如数据所显示的那样，出租车司机有很大的经济激励来根据乘客的最终目的地进行歧视；特别是在史泰登岛下车，司机需要大量的空闲时间。多年来，纽约市出租车和豪华轿车委员会一直在努力识别这种歧视行为，并对因乘客目的地而拒载的司机处以罚款。有趣的是尝试分析数据，找出可能表明司机与乘客对于乘客要下车的地方存在分歧的异常短程出租车行程。

# 未来的发展方向

在本章中，我们处理了真实数据集的时间和空间特征。到目前为止，您所获得的地理空间分析的熟悉度可以用于深入研究诸如 Apache Sedona 或 GeoMesa 等框架。与使用 GeoPandas 和 UDF 相比，它们将具有更陡峭的学习曲线，但会更高效。在地理空间和时间数据的数据可视化方面也有很大的应用空间。

此外，想象一下使用相同的技术在出租车数据上构建一个应用程序，根据当前交通模式和历史记录的最佳下一个位置，推荐出租车在下车后最好去的地方。您还可以从需要打车的人的角度查看信息：根据当前时间、地点和天气数据，我能在接下来的五分钟内从街上拦到出租车的概率是多少？这种信息可以整合到应用程序中，例如谷歌地图，帮助旅行者决定何时出发以及选择哪种出行方式。
