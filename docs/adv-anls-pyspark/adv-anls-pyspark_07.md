# 第七章。纽约市出租车行程数据的地理空间和时间数据分析

地理空间数据指的是数据中嵌入了某种形式的位置信息。这类数据目前以每天数十亿源的规模产生。这些源包括移动电话和传感器等，涉及人类和机器的移动数据，以及来自遥感的数据，对我们的经济和总体福祉至关重要。地理空间分析能够为我们提供处理这些数据并解决相关问题所需的工具和方法。

过去几年来，PySpark 和 PyData 生态系统在地理空间分析方面有了显著发展。它们被各行各业用于处理富含位置信息的数据，从而影响我们的日常生活。一个日常活动中，地理空间数据以显著方式展现出来的例子是本地交通。过去几年间数字打车服务的流行使得我们更加关注地理空间技术。在本章中，我们将利用我们的 PySpark 和数据分析技能，处理一个包含纽约市出租车行程信息的数据集。

了解出租车经济学的一个重要统计量是*利用率*：出租车在路上并有一名或多名乘客的时间比例。影响利用率的一个因素是乘客的目的地：白天在联合广场附近下车的出租车更有可能在一两分钟内找到下一单，而凌晨2点在史泰登岛下车的出租车可能需要驱车返回曼哈顿才能找到下一单。我们希望量化这些影响，并找出出租车在各区域下车后找到下一单的平均时间，例如曼哈顿、布鲁克林、皇后区、布朗克斯、史泰登岛，或者在城市之外（如纽瓦克自由国际机场）下车的情况。

我们将从设置数据集开始，然后深入地理空间分析。我们将学习 GeoJSON 格式，并结合 PyData 生态系统中的工具和 PySpark 使用。我们将使用 GeoPandas 处理*地理空间信息*，如经度和纬度点和空间边界。最后，我们将通过进行会话化分析处理数据的时间特征，如日期和时间。这将帮助我们了解纽约市出租车的使用情况。PySpark 的 DataFrame API 提供了处理时间数据所需的数据类型和方法。

让我们开始通过下载数据集并使用 PySpark 进行探索。

# 准备数据

对于这个分析，我们只考虑 2013 年 1 月的车费数据，解压后大约为 2.5 GB 数据。你可以访问 [2013 年每个月的数据](https://oreil.ly/7m7Ki)，如果你有足够大的 PySpark 集群，可以对整年的数据进行类似分析。现在，让我们在客户端机器上创建一个工作目录，并查看车费数据的结构：

```py
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

每个文件头之后的行代表 CSV 格式中的单个出租车行程。对于每次行程，我们有一些关于出租车的属性（车牌号的哈希版本）以及司机的信息（*hack license* 的哈希版本，这是驾驶出租车的许可证称呼），以及有关行程开始和结束的时间信息，以及乘客上车和下车的经度/纬度坐标。

让我们创建一个 *taxidata* 目录，并将行程数据复制到存储中：

```py
$ mkdir taxidata
$ mv trip_data_1.csv taxidata/
```

我们这里使用的是本地文件系统，但你可能不是这种情况。现在更常见的是使用像 AWS S3 或 GCS 这样的云原生文件系统。在这种情况下，你需要分别将数据上传到 S3 或 GCS。

现在启动 PySpark shell：

```py
$ pyspark
```

一旦 PySpark shell 加载完毕，我们可以从出租车数据创建一个数据集，并查看前几行，就像我们在其他章节中做的一样：

```py
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

乍看之下，这看起来是一个格式良好的数据集。让我们再次查看 DataFrame 的模式：

```py
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

我们将 `pickup_datetime` 和 `dropoff_datetime` 字段表示为 `Strings`，并将乘客上车和下车位置的个别 `(x,y)` 坐标存储在自己的 `Doubles` 字段中。我们希望将日期时间字段作为时间戳，因为这样可以方便地进行操作和分析。

## 将日期时间字符串转换为时间戳

如前所述，PySpark 提供了处理时间数据的开箱即用方法。

具体来说，我们将使用 `to_timestamp` 函数来解析日期时间字符串并将其转换为时间戳：

```py
from pyspark.sql import functions as fun

taxi_raw = taxi_raw.withColumn('pickup_datetime',
                                fun.to_timestamp(fun.col('pickup_datetime'),
                                                "yyyy-MM-dd HH:mm:ss"))
taxi_raw = taxi_raw.withColumn('dropoff_datetime',
                                fun.to_timestamp(fun.col('dropoff_datetime'),
                                                "yyyy-MM-dd HH:mm:ss"))
```

再次查看一下模式（schema）：

```py
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

现在，`pickup_datetime` 和 `dropoff_datetime` 字段已经是时间戳了。做得好！

我们提到过，这个数据集包含了 2013 年 1 月的行程。不过，不要仅仅听我们的话。我们可以通过对 `pickup_datetime` 字段进行排序来确认数据中的最新日期时间。为此，我们使用 DataFrame 的 `sort` 方法结合 PySpark 列的 `desc` 方法：

```py
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

数据类型就位后，让我们检查一下数据中是否存在任何不一致之处。

## 处理无效记录

所有在大规模、实际数据集上工作的人都知道，这些数据不可避免地会包含一些不符合编写处理代码期望的记录。许多 PySpark 管道因为无效记录导致解析逻辑抛出异常而失败。在进行交互式分析时，我们可以通过关注关键变量来感知数据中潜在的异常情况。

在我们的情况下，包含地理空间和时间信息的变量值得关注是否存在不一致。这些列中的空值肯定会影响我们的分析。

```py
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

```py
taxi_raw = taxi_raw.na.drop(subset=geospatial_temporal_colnames)
```

我们还可以进行一个常识性检查，即检查纬度和经度记录中值为零的情况。我们知道对于我们关心的区域，这些将是无效值：

```py
print("Count of zero dropoff, pickup latitude and longitude records")
taxi_raw.groupBy((fun.col("dropoff_longitude") == 0) |
  (fun.col("dropoff_latitude") == 0) |
  (fun.col("pickup_longitude") == 0) |
  (fun.col("pickup_latitude") == 0)).\ ![1](assets/1.png)
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

[![1](assets/1.png)](#co_geospatial_and_temporal_data_analysis___span_class__keep_together__on_taxi_trip_data__span__CO1-1)

对于任何记录，如果多个 `OR` 条件为真，则任何一个条件为真时都为真。

我们有相当多这样的记录。如果看起来一辆出租车把乘客带到了南极点，我们可以合理地认为该记录是无效的，并应该从我们的分析中排除。我们不会删除它们，而是在下一节结束时回来看看它们如何影响我们的分析。

在生产环境中，我们逐个检查这些异常，查看各个任务的日志，找出引发异常的代码行，然后调整代码以忽略或纠正无效记录。这是一个繁琐的过程，通常感觉就像我们在打地鼠：刚解决一个异常，我们又发现了下一个异常记录。

当数据科学家在处理新数据集时，常用的一种策略是在其解析代码中添加 `try-except` 块，以便任何无效记录可以写入日志而不会导致整个作业失败。如果整个数据集中只有少数无效记录，我们可能可以忽略它们并继续分析。

现在我们已经准备好我们的数据集，让我们开始地理空间分析。

# 地理空间分析

有两种主要的地理空间数据类型——矢量和栅格——每种类型都有不同的工具用于处理。在我们的情况下，我们有出租车行程记录的纬度和经度，以及以 GeoJSON 格式存储的矢量数据，该数据表示了纽约不同区域的边界。我们已经查看了纬度和经度点。让我们首先看看 GeoJSON 数据。

## GeoJSON 简介

我们将使用的纽约市各区边界的数据以*GeoJSON*格式编写。 GeoJSON中的核心对象称为*feature*，由*geometry*实例和一组称为*properties*的键值对组成。 几何体是指点、线或多边形等形状。 一组要素称为`FeatureCollection`。 让我们下载纽约市各区地图的GeoJSON数据并查看其结构。

在客户端机器上的*taxidata*目录中，下载数据并将文件重命名为更短的名称：

```py
$ url="https://nycdatastables.s3.amazonaws.com/\
 2013-08-19T18:15:35.172Z/nyc-borough-boundaries-polygon.geojson"
$ curl -O $url
$ mv nyc-borough-boundaries-polygon.geojson nyc-boroughs.geojson
```

打开文件并查看要素记录。 注意属性和几何对象 - 在本例中，多边形表示区域的边界和包含区域名称和其他相关信息的属性。

```py
$ head -n 7 data/trip_data_ch07/nyc-boroughs.geojson
...
{
"type": "FeatureCollection",

"features": [{ "type": "Feature", "id": 0, "properties": { "boroughCode": 5, ...
,
{ "type": "Feature", "id": 1, "properties": { "boroughCode": 5, ...
```

## GeoPandas

在选择执行地理空间分析的库时，首先要考虑的是确定您需要处理哪种类型的数据。 我们需要一个可以解析GeoJSON数据并处理空间关系的库，例如检测给定的经度/纬度对是否包含在表示特定区域边界的多边形内。 我们将使用[GeoPandas库](https://geopandas.org)来执行此任务。 GeoPandas是一个开源项目，旨在使Python中的地理空间数据处理更加简单。 它扩展了pandas库使用的数据类型，我们在之前的章节中使用过，以允许在几何数据类型上进行空间操作。

使用pip安装GeoPandas包：

```py
pip3 install geopandas
```

现在让我们开始检查出租车数据的地理空间方面。 对于每次行程，我们都有表示乘客上车和下车位置的经度/纬度对。 我们希望能够确定这些经度/纬度对中的每一个属于哪个区，并确定任何未在五个区域中的经纬度对。 例如，如果一辆出租车将乘客从曼哈顿送到纽瓦克自由国际机场，那将是一次有效的行程，我们很有兴趣分析，即使它并没有在五个区域之一结束。

要执行我们的区域分析，我们需要加载我们之前下载并存储在*nyc-boroughs.geojson*文件中的GeoJSON数据：

```py
import geopandas as gdp

gdf = gdp.read_file("./data/trip_data_ch07/nyc-boroughs.geojson")
```

在使用GeoJSON功能在出租车行程数据上之前，我们应该花点时间考虑如何为最大效率组织这些地理空间数据。 一个选择是研究针对地理空间查找进行优化的数据结构，例如四叉树，然后找到或编写我们自己的实现。 相反，我们将尝试提出一个快速的启发式方法，使我们能够跳过那一部分工作。

我们将通过 `gdf` 迭代，直到找到一个几何图形包含给定经度/纬度点的要素为止。大多数纽约市的出租车行程始发和结束在曼哈顿，因此如果表示曼哈顿的地理空间要素在序列中较早出现，我们的搜索将相对较快结束。我们可以利用每个要素的 `boroughCode` 属性作为排序键，曼哈顿的代码为1，斯塔滕岛的代码为5。在每个区的要素中，我们希望与较大多边形相关联的要素优先于与较小多边形相关联的要素，因为大多数行程将在每个区的“主要”区域内进行。

我们将计算与每个要素几何图形相关联的面积，并将其存储为新列：

```py
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

按照区号和每个要素几何图形的面积的组合进行排序应该可以解决问题：

```py
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

注意，我们基于面积值的降序排序，因为我们希望最大的多边形首先出现，而`sort_values`默认按升序排序。

现在我们可以将 `gdf` GeoPandas DataFrame 中排序后的要素广播到集群，并编写一个函数，该函数使用这些要素来找出特定行程结束在哪个（如果有的话）五个区中的哪一个：

```py
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

我们可以将 `find_borough` 应用于 `taxi_raw` DataFrame 中的行程，以创建一个按区划分的直方图：

```py
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

正如我们预期的那样，绝大多数行程都在曼哈顿区结束，而在斯塔滕岛结束的行程相对较少。一个令人惊讶的观察是，结束在任何区域之外的行程的数量；`null`记录的数量远远超过结束在布朗克斯的出租车行程的数量。

我们之前谈论过如何处理这些无效记录，但没有将它们删除。现在留给你的练习是删除这些记录并从清理后的数据中创建直方图。一旦完成，您将注意到`null`条目的数量减少，剩下的观察数量更加合理，这些观察是在城市外部结束的行程。

在处理了数据的地理空间方面后，现在让我们通过使用 PySpark 执行会话化来深入了解我们数据的时间特性。

# PySpark 中的会话化

在这种分析中，我们希望分析单个实体随着时间执行一系列事件的情况，称为*会话化*，通常在网站的日志中执行，以分析用户的行为。 PySpark 提供了窗口和聚合函数，可以直接用于执行此类分析。这些函数允许我们专注于业务逻辑，而不是试图实现复杂的数据操作和计算。我们将在下一节中使用它们来更好地理解数据集中出租车的使用情况。

会话化可以是发现数据见解和构建新数据产品的强大技术。例如，Google 的拼写校正引擎是建立在每天从其网站属性上发生的每个事件（搜索、点击、地图访问等）的记录构成的会话之上的。为了识别可能的拼写校正候选项，Google 处理这些会话，寻找用户输入查询但没有点击任何内容，然后几秒钟后输入稍微不同的查询，然后点击结果并且不再返回 Google 的情况。然后，它统计这种模式对于任何一对查询发生的频率。如果发生频率足够高（例如，如果我们每次看到查询“解开的统计数据”，它后面都跟着几秒钟后的查询“美国”），那么我们就假定第二个查询是第一个查询的拼写校正。

此分析利用了事件日志中所表示的人类行为模式，从而构建了一个比可以从字典中创建的任何引擎更强大的数据拼写校正引擎。该引擎可用于执行任何语言的拼写校正，并且可以纠正可能不包含在任何字典中的单词（例如，新创业公司的名称）或查询，如“解开的统计数据”，其中没有任何单词拼写错误！Google 使用类似的技术显示推荐和相关搜索，以及决定哪些查询应返回 OneBox 结果，该结果在搜索页面本身上给出查询的答案，而不需要用户点击转到不同页面。有关天气、体育比分、地址和许多其他类型查询的 OneBoxes。

到目前为止，每个实体发生的事件集合的信息分布在 DataFrame 的分区中，因此，为了分析，我们需要将这些相关事件放在一起，并按时间顺序排列。在下一节中，我们将展示如何使用高级 PySpark 功能有效地构建和分析会话。

## 构建会话：PySpark 中的次要排序

在 PySpark 中创建会话的天真方式是对我们想要为其创建会话的标识符执行 `groupBy`，然后在洗牌后按时间戳标识符对事件进行排序。如果每个实体只有少量事件，这种方法会工作得相当不错。然而，由于这种方法需要将任何特定实体的所有事件同时保存在内存中进行排序，随着每个实体的事件数量越来越多，它将不会扩展。我们需要一种构建会话的方法，不需要将某个特定实体的所有事件同时保存在内存中进行排序。

在 MapReduce 中，我们可以通过执行*二次排序*来构建会话，其中我们创建一个由标识符和时间戳值组成的复合键，对所有记录按照复合键排序，然后使用自定义分区器和分组函数确保相同标识符的所有记录出现在同一个输出分区中。幸运的是，PySpark 也可以通过使用`Window`函数来支持类似的模式：

```py
from pyspark.sql import Window

window_spec = Window.partitionBy("hack_license").\
                      orderBy(fun.col("hack_license"),
                              fun.col("pickup_datetime"))
```

首先，我们使用`partitionBy`方法确保所有具有相同`license`列值的记录最终进入同一个分区。然后，在每个分区内部，我们按照它们的`license`值（以便同一司机的所有行程出现在一起）和`pickupTime`排序这些记录，以便在分区内按排序顺序显示行程序列。现在，当我们聚合行程记录时，我们可以确保行程按照会话分析最佳顺序排序。因为这个操作会触发重分区和相当多的计算，并且我们需要多次使用结果，所以我们将其缓存：

```py
window_spec.cache()
```

执行会话化管道是一个昂贵的操作，会话化数据通常对我们可能要执行的许多不同分析任务都很有用。在可能需要稍后进行分析或与其他数据科学家合作的设置中，通过仅执行一次会话化大型数据集并将会话化数据写入诸如 S3 或 HDFS 的文件系统中，可以分摊会话化成本，以便用于回答许多不同的问题。只执行一次会话化还是确保整个数据科学团队会话定义标准规则的好方法，这对于确保结果的苹果与苹果之间的比较具有相同的好处。

此时，我们已经准备好分析我们的会话数据，以查看司机在特定区域下车后找到下一个行程所需的时间。我们将使用`lag`函数以及之前创建的`window_spec`对象来获取两次行程之间的持续时间（以秒为单位）：

```py
df_ with_ borough_durations = df_with_boroughs.\
            withColumn("trip_time_difference", \
            fun.col("pickup_datetime") - fun.lag(fun.col("pickup_datetime"),
                                          1). \
            over(window_spec)).show(50, vertical=True)
```

现在，我们应该进行验证检查，确保大部分持续时间是非负的：

```py
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

只有少数记录持续时间为负，当我们更仔细地检查它们时，似乎没有任何共同的模式可以帮助我们理解错误数据的来源。如果我们从输入数据集中排除这些负持续时间记录，并查看按区域分组的接驳时间的平均值和标准偏差，我们会看到这样的结果：

```py
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

正如我们所预期的那样，数据显示，在曼哈顿的乘车等待时间最短，大约为10分钟。在布鲁克林结束的出租车行程的等待时间是这个的两倍多，而在史泰登岛结束的相对较少的行程则需要司机平均将近45分钟才能找到下一个乘客。

正如数据所表明的，出租车司机有很大的经济激励来根据乘客的最终目的地进行区别对待；尤其是在史泰登岛的乘客下车后，司机需要花费大量时间等待下一个订单。多年来，纽约市出租车和豪华轿车委员会一直在努力识别这种歧视，并对因乘客目的地而拒载的司机进行罚款。有趣的是，试图检验数据中异常短的出租车行程，可能是司机和乘客关于乘客想下车的位置存在争执的迹象。

# 下一步该何去何从

在本章中，我们处理了一个真实世界数据集的时间和空间特征。到目前为止，你已经获得了地理空间分析的熟练度，可以用于深入研究如Apache Sedona或GeoMesa等框架。与使用GeoPandas和UDFs相比，它们的学习曲线会更陡峭，但效率更高。在使用地理空间和时间数据进行数据可视化方面，还有很大的发展空间。

此外，想象一下，使用相同的技术来分析出租车数据，构建一个能够根据当前交通模式和包含在这些数据中的历史最佳位置记录，推荐出租车在落客后去的最佳地点的应用程序。你还可以从需要打车的人的角度来看待这些信息：在当前时间、地点和天气数据下，我能在接下来的五分钟内从街上拦到一辆出租车的概率是多少？这种信息可以整合到诸如Google地图之类的应用程序中，帮助旅行者决定何时出发以及选择哪种出行方式。
