# 第十三章。使用 Flask 构建 RESTful 数据

正如在“使用 Flask 构建简单数据 API”一节中所见，我们看到如何使用 Flask 和 Dataset 构建一个非常简单的数据 API。对于许多简单的数据可视化来说，这种快速而简陋的 API 是可以接受的，但随着数据需求变得更加复杂，有一个遵循一些检索和有时创建、更新和删除的惯例的 API 会更有帮助。^(1) 在“使用 Python 从 Web API 消费数据”一章中，我们介绍了 Web API 的类型及为什么 RESTful^(2) API 正在获得应有的重视。在本章中，我们将看到将几个 Flask 库组合成一个灵活的 RESTful API 是多么简单。

# RESTful 作业工具

正如在“使用 Flask 构建简单数据 API”一节中所见，数据 API 的基础非常简单。它需要一个接受 HTTP 请求的服务器，例如 GET 请求用于检索数据或更高级的动词如 POST（用于添加）或 DELETE。这些请求位于诸如`api/winners`的路由上，然后由提供的函数处理。在这些函数中，数据从后端数据库中检索出来，可能会使用数据参数进行过滤（例如，像`?category=comic&name=Groucho`这样的字符串附加到 URL 调用中）。然后，这些数据需要以某种请求的格式返回或序列化，几乎总是基于 JSON。对于这种数据的往返，Flask/Python 生态系统提供了一些完美的库：

+   Flask 执行服务器工作

+   [Flask SQLAlchemy](https://oreil.ly/NVldl)是一个 Flask 扩展，将我们首选的 Python SQL 库与对象关系映射器（ORM）SQLAlchemy 集成。

+   [Flask-Marshmallow](https://oreil.ly/Vbgq3)是一个 Flask 扩展，添加了对[marshmallow](https://oreil.ly/AIySU)的支持，这是一个功能强大的 Python 对象序列化库。

您可以使用`pip`安装所需的扩展：

```py
$ pip install Flask-SQLALchemy flask-marshmallow marshmallow-sqlalchemy
```

# 创建数据库

在“保存清理后的数据集”一节中，我们看到使用`to_sql`方法将 pandas DataFrame 存储到 SQL 中是多么简单。这是一种非常方便的存储 DataFrame 的方式，但生成的表缺少一个[主键字段](https://oreil.ly/x4OM6)，这个字段唯一地指定表中的行。拥有主键是一个良好的习惯，而且对于通过我们的 Web API 创建或删除行来说几乎是必需的。因此，我们将通过另一种方式创建我们的 SQL 表。

首先，我们使用 SQLAlchemy 构建了一个 SQLite 数据库，并向获奖者表添加了主键 ID。

```py
from sqlalchemy import Column, Integer, String, Text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class Winner(Base):
    __tablename__ = 'winners'
    id = Column(Integer, primary_key=True) ![1](img/1.png)
    category = Column(String)
    country = Column(String)
    date_of_birth = Column(String) # string form dates
    date_of_death = Column(String)
    # ...

# create SQLite database and start a session
engine = sqlalchemy.create_engine('sqlite:///data/nobel_winners_cleaned_api.db')
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
session = Session()
```

![1](img/#co_restful_data_with_flask_CO1-1)

我们指定了至关重要的主键来消除获奖者之间的歧义。

请注意，我们将日期存储为字符串以限制序列化日期时间对象时可能出现的任何问题。在提交行到数据库之前，我们将转换这些 DataFrame 列：

```py
df['date_of_birth'] = df['date_of_birth'].astype(str)
df['date_of_death'] = dl['date_of_death'].astype(str)
df.date_of_birth
#0      1927-10-08
#4      1829-07-26
#5      1862-08-29
..
```

现在，我们可以迭代我们 DataFrame 的行，将它们作为字典记录添加到数据库中，然后将它们相对高效地作为一个事务提交：

```py
for d in df_tosql.to_dict(orient='records'):
    session.add(Winner(**d))
session.commit()
```

有了我们成形完善的数据库，让我们看看如何使用 Flask 轻松地提供它。

# Flask RESTful 数据服务器

这将是一个标准的 Flask 服务器，类似于 “使用 Flask 服务数据” 中看到的那个。首先，我们导入标准的 Flask 模块，以及 SQLALchemy 和 marshmallow 扩展，并创建我们的 Flask 应用程序：

```py
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow

# Init app
app = Flask(__name__)
```

现在一些特定于数据库的声明，使用 Flask 应用程序初始化 SQLAlchemy：

```py
app.config['SQLALCHEMY_DATABASE_URI'] =\
    'sqlite:///data/nobel_winners_cleaned_api_test.db'

db = SQLAlchemy(app)
```

现在我们可以使用 `db` 实例来定义我们的获胜者表，从基本声明模型继承而来。这与上一节中用来创建获胜者表的 schema 匹配：

```py
class Winner(db.Model):
    __tablename__ = 'winners'
    id = db.Column(db.Integer, primary_key=True)
    category = db.Column(db.String)
    country = db.Column(db.String)
    date_of_birth = db.Column(db.String)
    date_of_death = db.Column(db.String)
    gender = db.Column(db.String)
    link = db.Column(db.String)
    name = db.Column(db.String)
    place_of_birth = db.Column(db.String)
    place_of_death = db.Column(db.String)
    text = db.Column(db.Text)
    year = db.Column(db.Integer)
    award_age = db.Column(db.Integer)

    def __repr__(self):
        return "<Winner(name='%s', category='%s', year='%s')>"\
            % (self.name, self.category, self.year)
```

## 使用 marshmallow 进行序列化

marshmallow 是一个非常有用的小型 Python 库，做一件事并且做得很好。引用 [文档](https://oreil.ly/kLTVF) 的话说：

> marshmallow 是一个用于将复杂数据类型（如对象）转换为原生 Python 数据类型的 ORM/ODM/框架无关的库。

marshmallow 使用类似于 SQLAlchemy 的 schema，可以将输入数据反序列化为应用程序级别的对象，并验证该输入数据。在这里，它的关键优势是能够从由 SQLAlchemy 提供的我们的 SQLite 数据库中获取数据，并将其转换为符合 JSON 规范的数据。

要使用 Flask-Marshmallow，我们首先创建一个 marshmallow 实例 (`ma`)，并初始化 Flask 应用程序。然后，我们使用它创建一个 marshmallow schema，使用 SQLAlchemy 的 `Winner` 模型作为其基础。该 schema 还有一个 `fields` 属性，允许您指定要序列化的（数据库）字段：

```py
ma = Marshmallow(app)

class WinnerSchema(ma.Schema):
    class Meta:
        model = Winner
        fields = ('category', 'country', 'date_of_birth', 'date_of_death', ![1](img/1.png)
                  'gender', 'link', 'name', 'place_of_birth', 'place_of_death',
                  'text', 'year', 'award_age')

winner_schema = WinnerSchema() ![2](img/2.png)
winners_schema = WinnerSchema(many=True)
```

![1](img/#co_restful_data_with_flask_CO2-1)

要序列化的数据库字段。

![2](img/#co_restful_data_with_flask_CO2-2)

我们声明了两个 schema 实例，一个用于返回单个记录，另一个用于多个记录。

# 添加我们的 RESTful API 路由

现在骨架已经搭好，让我们创建一些 Flask 路由来定义一个小型的 RESTful API。作为第一次测试，我们将创建一个路由，返回我们数据库表中的所有诺贝尔获奖者：

```py
@app.route('/winners/')
def winner_list():
    all_winners = Winner.query.all() ![1](img/1.png)
    result = winners_schema.jsonify(all_winners) ![2](img/2.png)
    return result
```

![1](img/#co_restful_data_with_flask_CO3-1)

获得获胜者表中所有行的数据库查询。

![2](img/#co_restful_data_with_flask_CO3-2)

将许多行的 marshmallow schema 获取所有获胜者的结果并将其序列化为 JSON。

我们将使用一些命令行 curl 测试 API：

```py
$ curl http://localhost:5000/winners/
[
  {
    "award_age": 57,
    "category": "Physiology or Medicine",
    "country": "Argentina",
    "date_of_birth": "1927-10-08",
    "date_of_death": "2002-03-24",
    "gender": "male",
    "link": "http://en.wikipedia.org/wiki/C%C3%A9sar_Milstein",
    "name": "C\u00e9sar Milstein",
    "place_of_birth": "Bah\u00eda Blanca ,  Argentina",
    "place_of_death": "Cambridge , England",
    "text": "C\u00e9sar Milstein , Physiology or Medicine, 1984",
    "year": 1984
  },
  {
    "award_age": 80,
    "category": "Peace",  ...
  }...
]
```

现在我们有了一个 API 端点来返回所有的获胜者。那么，如何通过 ID（我们的获胜者表的主键）来检索个体呢？为此，我们从 API 调用中检索 ID，使用 Flask 的路由模式匹配，并将其用于进行特定的数据库查询。然后，我们使用我们的单行 marshmallow schema 将其序列化为 JSON：

```py
@app.route('/winners/<id>/')
def winner_detail(id):
    winner = Winner.query.get_or_404(id) ![1](img/1.png)
    result = winner_schema.jsonify(winner)
    return result
```

![1](img/#co_restful_data_with_flask_CO4-1)

Flask-SQLAlchemy 提供了一个默认的 404 错误消息，如果查询无效，则可以通过 marshmallow 序列化为 JSON。

使用 curl 测试显示预期的单个 JSON 对象返回：

```py
$ curl http://localhost:5000/winners/10/
{
  "award_age": 60,
  "category": "Chemistry",
  "country": "Belgium",
  "date_of_birth": "1917-01-25",
  "date_of_death": "2003-05-28",
  "gender": "male",
  "link": "http://en.wikipedia.org/wiki/Ilya_Prigogine",
  "name": "Ilya Prigogine",
  "place_of_birth": "Moscow ,  Russia",
  "place_of_death": "Brussels ,  Belgium",
  "text": "Ilya Prigogine ,  born in Russia , Chemistry, 1977",
  "year": 1977
}
```

能够在单个 API 调用中检索所有获奖者并不特别有用。让我们增加通过请求中提供的一些参数来过滤这些结果的功能。这些参数可以在 URL 查询字符串上找到，以`?`开头并以`&`分隔，跟随端点，例如`http://nobel.net/api/winners?category=Physics&year=1980`。Flask 提供了一个`request.args`对象，具有一个`to_dict`方法，返回 URL 参数的字典。^(3) 我们可以使用这个方法来指定我们的数据表过滤器，这可以作为键值对应用于 SQLAlchemy 的`to_filter`方法，这个方法可以应用于查询。这是一个简单的实现：

```py
@app.route('/winners/')
def winner_list():
    valid_filters = ('year', 'category', 'gender', 'country', 'name') ![1](img/1.png)
    filters = request.args.to_dict()

    args = {name: value for name, value in filters.items()
            if name in valid_filters} ![2](img/2.png)
    # This for loop does the same job as the dict
    # comprehension above
    # args = {}
    # for vf in valid_filters:
    #     if vf in filters:
    #         args[vf] = filters.get(vf)
    app.logger.info(f'Filtering with the fields: {args}')
    all_winners = Winner.query.filter_by(**args) ![3](img/3.png)
    result = winners_schema.jsonify(all_winners)
    return result
```

![1](img/#co_restful_data_with_flask_CO5-1)

这些是我们允许进行过滤的字段。

![2](img/#co_restful_data_with_flask_CO5-2)

我们遍历提供的过滤字段，并使用有效的字段创建我们的过滤器字典。在这里，我们使用 Python 的[字典推导](https://oreil.ly/wmy3c)构建`args`字典。

![3](img/#co_restful_data_with_flask_CO5-3)

使用 Python 的字典解包来指定方法参数。

让我们使用 curl 测试我们的过滤能力，使用`-d`（数据）参数来指定我们的查询参数：

```py
$ curl -d category=Physics -d year=1933 --get http://localhost:5000/winners/

[
  {
    "award_age": 31,
    "category": "Physics",
    "country": "United Kingdom",
    "date_of_birth": "1902-08-08",
    "date_of_death": "1984-10-20",
    "gender": "male",
    "link": "http://en.wikipedia.org/wiki/Paul_Dirac",
    "name": "Paul Dirac",
    "place_of_birth": "Bristol , England",
    "place_of_death": "Tallahassee, Florida , US",
    "text": "Paul Dirac , Physics, 1933",
    "year": 1933
  },
  {
    "award_age": 46,
    "category": "Physics",
    "country": "Austria",
    "date_of_birth": "1887-08-12",
    "date_of_death": "1961-01-04",
    "gender": "male",
    "link": "http://en.wikipedia.org/wiki/Erwin_Schr%C3%B6dinger",
    "name": "Erwin Schr\u00f6dinger",
    "place_of_birth": "Erdberg, Vienna, Austria",
    "place_of_death": "Vienna, Austria",
    "text": "Erwin Schr\u00f6dinger , Physics, 1933",
    "year": 1933
  }
]
```

现在我们对获奖者数据集进行了相当细粒度的过滤，对于许多数据可视化来说，这足以提供一个大的、用户驱动的数据集，根据需要从 RESTful API 获取数据。通过 API 或 Web 表单发布或创建数据条目是那些偶尔出现并且很好掌握的要求之一。这意味着您可以将 API 用作中央数据池，并从各个位置添加到其中。在 Flask 和我们的扩展中，这也很容易实现。

## 向 API 提交数据

Flask 路由接受一个可选的`methods`参数，指定接受的 HTTP 动词。GET 动词是默认的，但通过将其设置为 POST，我们可以使用`request`对象上可用的数据包将数据提交到此路由中，此处为 JSON 编码数据。

我们添加了另一个`/winners`端点，其中包含一个`methods`数组，其中包含`POST`，然后使用 JSON 数据创建一个`winner_data`字典，用于在获奖者表中创建条目。然后将其添加到数据库会话中，最后提交。使用 marshmallow 进行序列化返回新条目：

```py
@app.route('/winners/', methods=['POST'])
def add_winner():
    valid_fields = winner_schema.fields

    winner_data = {name: value for name,
                   value in request.json.items() if name in valid_fields}
    app.logger.info(f"Creating a winner with these fields: {winner_data}")
    new_winner = Winner(**winner_data)
    db.session.add(new_winner)
    db.session.commit()
    return winner_schema.jsonify(new_winner)
```

使用 curl 测试返回预期结果：

```py
$ curl http://localhost:5000/winners/ \
    -X POST \
    -H "Content-Type: application/json" \
    -d '{"category":"Physics","year":2021,
         "name":"Syukuro Manabe","country":"Japan"}' ![1](img/1.png)
{
  "award_age": null,
  "category": "Physics",
  "country": "Japan",
  "date_of_birth": null,
  "date_of_death": null,
  "gender": null,
  "link": null,
  "name": "Syukuro Manabe",
  "place_of_birth": null,
  "place_of_death": null,
  "text": null,
  "year": 2021
}
```

![1](img/#co_restful_data_with_flask_CO6-1)

输入数据是一个 JSON 编码的字符串。

对于数据管理可能更有用的是一个 API 端点，允许更新获奖者的数据。为此，我们可以使用 HTTP PATCH 动词调用单个 URL。与用于创建新获奖者的 POST 相同，我们会遍历 `request.json` 字典，并使用任何有效的字段，在本例中，所有字段都可以由 marshmallow 的序列化器使用，以更新获奖者的属性按 ID 排序：

```py
@app.route('/winners/<id>/', methods=['PATCH'])
def update_winner(id):
    winner = Winner.query.get_or_404(id)
    valid_fields = winner_schema.fields
    winner_data = {name: value for name, value
                    in request.json.items() if name in valid_fields}
    app.logger.info(f"Updating a winner with these fields: {winner_data}")
    for k, v in winner_data.items():
        setattr(winner, k, v)
    db.session.commit()
    return winner_schema.jsonify(winner)
```

在这里，我们使用此 API 修补点来更新，以演示为目的，一个诺贝尔奖获得者的姓名和奖励年份：

```py
$ curl http://localhost:5000/winners/3/ \
    -X PATCH \
    -H "Content-Type: application/json" \
    -d '{"name":"Morris Maeterlink","year":"1912"}'
{
  "award_age": 49,
  "category": "Literature",
  "country": "Belgium",
  "date_of_birth": "1862-08-29",
  "date_of_death": "1949-05-06",
  "gender": "male",
  "link": "http://en.wikipedia.org/wiki/Maurice_Maeterlinck",
  "name": "Morris Maeterlink",
  "place_of_birth": "Ghent ,  Belgium",
  "place_of_death": "Nice ,  France",
  "text": "Maurice Maeterlinck , Literature, 1911", ![1](img/1.png)
  "year": 1912
}
```

![1](img/#co_restful_data_with_flask_CO7-1)

原始细节。

到目前为止，我们已经构建了一个有用的、有针对性的 API，能够根据精细的过滤条件获取数据，并更新或创建获奖者。如果您想要添加更多覆盖几个数据库表的端点，Flask 路由的样板代码和相关方法可能会有些混乱。Flask MethodViews 提供了一种方式，将我们的端点 API 调用封装在单个类实例中，使得代码更清晰、更易于扩展。将现有的 API 转移到 MethodViews 并减少认知负荷，将在 API 变得更加复杂时获得回报。

# 使用 MethodViews 扩展 API

我们可以重复使用我们 API 的大部分代码，并将其转移到 MethodViews 中也减少了相当数量的样板。MethodViews 将端点及其关联的 HTTP 动词（GET、POST 等）封装在单个类实例中，可以轻松扩展和适应。要将我们的获奖者表转移到一个专用资源中，我们只需将现有的 Flask 路由方法提升到一个 `MethodView` 类中，并进行一些小的调整。首先，我们需要导入 `MethodView` 类：

```py
#...
from flask.views import MethodView
#...
```

SQLAlchemy 模型和 marshmallow schemas 无需更改。现在我们为获奖者集合创建一个 `MethodView` 实例，包含相关 HTTP 动词的方法。我们可以重用现有的路由方法。然后，我们使用 Flask 应用的 `add_url_rule` 方法提供一个端点，这个视图将处理它：

```py
class WinnersListView(MethodView):

    def get(self):
        valid_filters = ('year', 'category', 'gender', 'country', 'name')
        filters = request.args.to_dict()
        args = {name: value for name, value in filters.items()
                if name in valid_filters}
        app.logger.info('Filtering with the %s fields' % (str(args)))
        all_winners = Winner.query.filter_by(**args)
        result = winners_schema.jsonify(all_winners)
        return result

    def post(self):
        valid_fields = winner_schema.fields
        winner_data = {name: value for name,
                       value in request.json.items() if name in valid_fields}
        app.logger.info("Creating a winner with these fields: %s" %
                        str(winner_data))
        new_winner = Winner(**winner_data)
        db.session.add(new_winner)
        db.session.commit()
        return winner_schema.jsonify(new_winner)

app.add_url_rule("/winners/",
                 view_func=WinnersListView.as_view("winners_list_view"))
```

为每个表条目创建 HTTP 方法遵循相同的模式。我们会添加一个删除方法以确保万无一失。成功的 HTTP 删除应该返回 204（无内容）HTTP 代码和一个空的内容包：

```py
class WinnerView(MethodView):

    def get(self, winner_id):
        winner = Winner.query.get_or_404(winner_id)
        result = winner_schema.jsonify(winner)
        return result

    def patch(self, winner_id):
        winner = Winner.query.get_or_404(winner_id)
        valid_fields = winner_schema.fields
        winner_data = {name: value for name,
                       value in request.json.items() if name in valid_fields}
        app.logger.info("Updating a winner with these fields: %s" %
                        str(winner_data))
        for k, v in winner_data.items():
            setattr(winner, k, v)
        db.session.commit()
        return winner_schema.jsonify(winner)

    def delete(self, winner_id):
        winner = Winner.query.get_or_404(winner_id)
        db.session.delete(winner)
        db.session.commit()
        return '', 204

app.add_url_rule("/winners/<winner_id>",
                 view_func=WinnerView.as_view("winner_view")) ![1](img/1.png)
```

![1](img/#co_restful_data_with_flask_CO8-1)

所有命名的、模式匹配的参数都会传递给所有 MethodViews 方法。

让我们使用 curl 删除其中一位获奖者，指定详细输出：

```py
$ curl http://localhost:5000/winners/858 -X DELETE -v
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 5000 (#0)
> DELETE /winners/858 HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/7.47.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 204 NO CONTENT
< Content-Type: application/json
< Server: Werkzeug/2.0.2 Python/3.8.9
< Date: Sun, 27 Mar 2022 15:35:51 GMT
<
* Closing connection 0
```

通过在专用端点上使用 MethodViews，我们可以减少大量 Flask 路由的样板代码，并使代码库更易于使用和扩展。举个例子，让我们看看如何添加一个非常方便的 API 功能，即分页或分块数据的能力。

# 分页数据返回

如果你有大型数据集并预计有大量结果集，接收分页数据的能力是一个非常有用的 API 特性；对于许多用例来说，这是一个至关重要的特性。

SQLAlchemy 有一个方便的`paginate`方法，可以在查询上调用以返回指定页面大小的数据页面。 要将分页添加到我们的获奖者 API 中，我们只需添加几个查询参数来指定页面和页面大小。 我们将使用 `_page` 和 `_page-size`，并在前面加上下划线以区分它们与我们可能应用的任何过滤器查询。

这是调整过的`get`方法：

```py
class WinnersListView(MethodView):

    def get(self):
        valid_filters = ('year', 'category', 'gender', 'country', 'name')
        filters = request.args.to_dict()
        args = {name: value for name, value in filters.items()
                if name in valid_filters}

        app.logger.info(f'Filtering with the {args} fields')

        page = request.args.get("_page", 1, type=int) ![1](img/1.png)
        per_page = request.args.get("_per-page", 20, type=int)

        winners = Winner.query.filter_by(**args).paginate(page, per_page) ![2](img/2.png)
        winners_dumped = winners_schema.dump(winners.items)

        results = {
            "results": winners_dumped,
            "filters": args,
            "pagination": ![3](img/3.png)
            {
                "count": winners.total,
                "page": page,
                "per_page": per_page,
                "pages": winners.pages,
            },
        }

        make_pagination_links('winners', results) ![4](img/4.png)

        return jsonify(results)
    # ...
```

![1](img/#co_restful_data_with_flask_CO9-1)

具有合理默认值的分页参数。

![2](img/#co_restful_data_with_flask_CO9-2)

在这里，我们使用 SQLAlchemy 的`paginate`方法以及我们的`page`和`per_page`分页变量。

![3](img/#co_restful_data_with_flask_CO9-3)

我们返回我们的分页结果和其他有意义的内容。 在`pagination`字典中，我们提供有用的反馈 - 返回的页面和总数据集的大小。

![4](img/#co_restful_data_with_flask_CO9-4)

我们将使用此函数添加一些方便的 URL 以获取上一页或下一页。

按照惯例，返回上一页和下一页的 URL 端点非常有用，以便轻松访问整个数据集。 我们有一个小小的`make_pagination_links`函数来实现这一点，它将这些便捷的 URL 添加到分页字典中。 我们将使用 Python 的 *urllib* 库构建我们的 URL 查询字符串：

```py
#...
import urllib.parse
#...
def make_pagination_links(url, results):
    pag = results['pagination']
    query_string = urllib.parse.urlencode(results['filters']) ![1](img/1.png)

    page = pag['page']
    if page > 1:
        prev_page = url + '?_page=%d&_per-page=%d%s' % (page-1,
                                                        pag['per_page'],
                                                        query_string)
    else:
        prev_page = ''

    if page < pag['pages']: ![2](img/2.png)
        next_page = url + '?_page=%d&_per-page=%d%s' % (page+1,
                                                        pag['per_page'],
                                                        query_string)
    else:
        next_page = ''

    pag['prev_page'] = prev_page
    pag['next_page'] = next_page
```

![1](img/#co_restful_data_with_flask_CO10-1)

我们将从过滤查询重新生成我们的查询字符串，例如，`&category=Chemistry&year=1976`。 *urllib* 的解析模块将过滤器字典转换为正确格式的 URL 查询。

![2](img/#co_restful_data_with_flask_CO10-2)

在适用的情况下添加上一页和下一页的 URL，将任何过滤查询附加到结果中。

让我们使用 curl 来测试我们的分页数据。 我们将添加一个筛选器以获取所有诺贝尔物理学奖获得者：

```py
$ curl -d category=Physics  --get http://localhost:5000/winners/

{
  "filters": {
    "category": "Physics"
  },
  "pagination": {
    "count": 201,
    "next_page": "?_page=2&_per-page=20&category=Physics", ![1](img/1.png)
    "page": 1,
    "pages": 11,
    "per_page": 20,
    "prev_page": ""
  },
  "results":  ![2
    {
      "award_age": 81,
      "category": "Physics",
      "country": "Belgium",
      "date_of_birth": "1932-11-06",
      "date_of_death": "NaT",
      "gender": "male",
      "link": "http://en.wikipedia.org/wiki/Fran%C3%A7ois_Englert",
      "name": "Fran\u00e7ois Englert",
      "place_of_birth": "Etterbeek ,  Brussels ,  Belgium",
      "place_of_death": null,
      "text": "Fran\u00e7ois Englert , Physics, 2013",
      "year": 2013
    },
    {
      "award_age": 37,
      "category": "Physics",
      "country": "Denmark",
      "date_of_birth": "1885-10-07",
      "date_of_death": "1962-11-18",
      "gender": "male",
      "link": "http://en.wikipedia.org/wiki/Niels_Bohr",
      "name": "Niels Bohr",
    ...
    }]}
```

![1](img/#co_restful_data_with_flask_CO11-1)

这是第一页，因此没有可用的上一页，但提供了下一页的 URL，以便轻松消费数据。

![2](img/#co_restful_data_with_flask_CO11-2)

在获奖者表中包含前 20 位物理学家的结果数组。

借助像 marshmallow 这样强大的库作为 Flask 扩展集成，很容易制作自己的 API，而无需求助于专门的 RESTful Flask 库，根据经验表明，这可能不会长期存在。

# 使用 Heroku 远程部署 API

拥有像我们刚刚构建的本地开发数据服务器一样的服务器非常适合用于原型设计、测试数据流以及处理数据集过大以至于无法舒适地作为 JSON（或等效）文件消耗的各种数据可视化任务。但这意味着任何尝试可视化的人都需要运行一个本地数据服务器，这只是需要考虑的又一件事情。这就是将数据服务器作为远程资源放在网络上的非常有用之处。有各种方法可以做到这一点，但可能是 Pythonista 们最喜欢的方式之一（包括我自己），是使用[Heroku](https://oreil.ly/V4Q6h)，这是一个使得部署 Flask 服务器非常简单的云服务。让我们通过将我们的诺贝尔数据服务器放在网络上来演示这一点。

首先，您需要创建一个[免费的 Heroku 账号](https://signup.heroku.com)。然后，您需要为您的操作系统安装[Heroku 客户端工具](https://oreil.ly/wutXG)。

安装了这些工具后，您可以通过从命令行运行`login`来登录 Heroku：

```py
$ heroku login
heroku: Press any key to open up the browser to login or q to exit
 ›   Warning: If browser does not open, visit
 ›   https://cli-auth.heroku.com/auth/browser/***
heroku: Waiting for login...
Logging in... done
Logged in as me@example.com
```

现在您已登录，我们将创建一个 Heroku 应用程序并将其部署到网络上。首先，我们创建一个应用目录（*heroku_api*），并将我们的 Flask API *api_rest.py*文件放入其中。我们还需要一个 Procfile 文件，一个*requirements.txt*文件，以及`nobel_winners_cleaned_api.db` SQLite 数据库来提供服务：

```py
heroku_api
├── api_rest.py
├── data
│   ├── nobel_winners_cleaned_api.db
├── Procfile
└── requirements.txt
```

Procfile 用于告诉 Heroku 如何以及如何部署。在这种情况下，我们将使用 Python 的[Gunicorn WSGI HTTP 服务器](https://oreil.ly/yBTdb)来处理与我们的 Flask 应用程序的 Web 流量，并将其作为 Heroku 应用程序运行。Procfile 如下所示：

```py
web: gunicorn api_rest:app
```

除了 Procfile 外，Heroku 还需要知道要为应用程序安装的 Python 库。这些库在*requirements.txt*文件中找到：

```py
Flask==2.0.2
gunicorn==20.1.0
Flask-Cors==3.0.10
flask-marshmallow==0.14.0
Flask-SQLAlchemy==2.5.1
Jinja2==3.0.1
marshmallow==3.15.0
marshmallow-sqlalchemy==0.28.0
SQLAlchemy==1.4.26
Werkzeug==2.0.2
```

配置文件已就位后，我们可以通过从命令行运行`create`来创建一个 Heroku 应用程序。

现在，我们将使用`git`来初始化 Git 目录并添加现有文件：

```py
$ git init
$ git add .
$ git commit -m "First commit"
```

使用初始化的`git`，我们只需创建我们的 Heroku 应用程序：^(4)

```py
$ heroku create flask-rest-pyjs2
```

现在，部署到 Heroku 只需一次`git push`：

```py
$ git push heroku master
```

每次您对本地代码库进行更改时，只需将它们推送到 Heroku，网站就会更新。

让我们使用 curl 测试 API，获取物理学获奖者的第一页：

```py
 $ curl -d category=Physics --get
                       https://flask-rest-pyjs2.herokuapp.com/winners/
{"filters":{"category":"Physics"},"pagination":{"count":201,
"next_page":"winners/?_page=2&_per-page=20&category=Physics","page":1,
"pages":11,"per_page":20,"prev_page":""},"results":[{"award_age":81,
"category":"Physics","country":"Belgium","date_of_birth":"1932-11-06",
"date_of_death":"NaT","gender":"male","link":"http://en.wikipedia.org/wiki/
Fran%C3%A7ois_Englert","name":"Fran\u00e7ois Englert", ... }
```

## CORS

为了从 Web 浏览器中使用 API，我们需要处理[跨源资源共享（CORS）](https://oreil.ly/ECpu0)对服务器数据请求的限制。我们将使用 Flask CORS 扩展，并以默认方式运行，允许来自任何域的请求访问数据服务器。这只需要向我们的 Flask 应用程序添加几行代码：

```py
# ...
from flask_cors import CORS
# Init app
app = Flask(__name__)
CORS(app)
```

[Flask-CORS 库](https://oreil.ly/bCKME)可用于指定允许访问哪些资源的域。在这里，我们允许一般访问。

## 使用 JavaScript 消耗 API

要从网页应用/页面使用数据服务器，我们只需使用`fetch`请求数据。这个示例通过继续获取页面来消耗所有分页数据，直到`next_page`属性为空字符串为止：

```py
let data
async function init() {
  data = await getData('winners/?category=Physics&country=United States') ![1](img/1.png)
  console.log(`${data.length} US Physics winners:`, data)
  // Send the data to a suitable charting function
  drawChart(data)
}

init()

async function getData(ep='winners/?category=Physics'){ ![1](img/1.png)
  let API_URL = 'https://flask-rest-pyjs2.herokuapp.com/'
  let data = []
  while(true) {
    let response = await fetch(API_URL + ep) ![2](img/2.png)
    .then(res => res.json()) ![3](img/3.png)
    .then(data => {
      return data
    })

    ep = response.pagination.next_page
    data = data.concat(response.results) // add the page results
    if(!ep) break // no next-page so break out of the loop
  }
  return data
}
```

![1](img/#co_restful_data_with_flask_CO12-1)

从服务器传来的数据是异步的，因此我们使用异步函数来消费它。

![2](img/#co_restful_data_with_flask_CO12-3)

[`await`](https://oreil.ly/EWHec) 等待异步 Promise 自行解决，提供其值。

![3](img/#co_restful_data_with_flask_CO12-4)

我们将响应数据转换为 JSON，将其传递给下一个 `then` 调用，该调用返回服务器数据。

JS 调用将期望的结果输出到控制台：

```py
89 US Physics winners:
[{
    award_age: 42
    category: "Physics"
    country: "United States"
    date_of_birth: "1969-12-16"
    date_of_death: "NaT"
    gender: "male"
    link: "http://en.wikipedia.org/wiki/Adam_G._Riess"
    name: "Adam G. Riess"
    place_of_birth: "Washington, D.C., United States"
    place_of_death: null
    text: "Adam G. Riess , Physics, 2011"
    year: 2011
  }, ...
}]
```

我们现在拥有一个基于 Web 的 RESTful 数据 API，可以从任何地方访问（受到我们的 CORS 限制）。正如你所见，低开销和易用性使得 Heroku 难以匹敌。它已经运行了一段时间，已经成为一个非常精细的设置。

# 摘要

我希望这一章已经展示了，通过几个强大的扩展，很容易就能自己搭建 RESTful API。要使其达到工业标准，需要更多的工作和一些测试，但对于大多数数据可视化任务来说，该 API 将提供处理大型数据集并允许用户自由探索的能力。至少它展示了快速测试用户精细调整数据集的可视化方法有多么容易。仪表板是这种远程获取数据的预期应用之一。

通过 Heroku 轻松部署 API 的能力意味着大型数据集可以在不运行本地数据服务器的情况下切割和切块——非常适合向客户或同事演示雄心勃勃的数据可视化。

^(1) 这些创建、读取、更新和删除方法构成了[CRUD 首字母缩写](https://oreil.ly/0AkAw)。

^(2) 本质上，RESTful 意味着资源由无状态、可缓存的 URI/URL 标识，并由 GET 或 POST 等 HTTP 动词进行操作。参见[维基百科的解释](https://oreil.ly/l0QhB)和这个[Stack Overflow 的讨论](https://oreil.ly/6zxhv)。

^(3) 从技术上讲，URL 查询字符串形成了一个多字典，允许同一个键有多个出现。对于我们的 API，我们期望每个键只有一个实例，因此转换为字典是可以的。

^(4) 你可以从 Heroku 仪表板执行此操作，然后使用 `git remote - <app_name>` 将当前 Git 目录附加到应用上。
