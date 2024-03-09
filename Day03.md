# 1. 静态文件

**什么是静态文件**
- 如：图片，css，js，音频，视频

**静态文件配置-settings.py中**
1. 配置静态文件的访问路径【该配置默认存在】
	- 通过哪个url地址找静态文件
	- STATIC_URL = '\/static\/'
	- 说明：
		指定访问静态文件时是需要通过/static/xxx或
		http://127.0.0.1:8000/static/xxx
		\[xxx表示具体的静态资源位置\]
2. 配置静态文件的存储路径STATICFILES_DIRS
STATICFILES_DIRS保存的是静态文件在服务器端的存储位置
```
# file: setting.py
STATICFILES_DIRS = (
	os.path.join(BASE_DIR, "static"),
)
```

**静态文件访问**
模板中访问静态文件 - img标签为例
**方案1** - 直接拼接访问路径
\<img src = "/static/images/lena.jpg\>或者
\<img src = "http://127.0.0.1:8000/static/images/lena.jpg>
**方案2** - 通过{% static %}访问静态文件
1. 加载static - {% load static %}
2. 使用静态资源 - {% static'静态资源路径' %}
3. 样例
- \<img src="{% static 'images/lena.jpg' %}"\>

# 2. Django应用及分布式路由

## 2.1 应用

**什么是应用**
- 应用在Django项目中是一个独立的业务模块，可以包含自己的路由，视图，模板，模型
![[Pasted image 20240309161427.png]]

**创建应用**
1. 用manage.py中的子命令startapp创建应用文件夹
- python3 manage.py startapp music 
2. 在settings.py的INSTALLED_APPS列表中配置安装此应用
- settings.py配置样例
```
INSTALLED_APPS = [
	# ....
	'user',  # 用户信息模块
	'music', # 音乐模块
]
```

## 2.2 分布式路由

Django中，主路由配置文件（urls.py）可以不处理用户具体路由，主路由配置文件的可以做请求的分支（分布式请求处理）。具体的请求可以由各自的应用来进行处理

![[Pasted image 20240309161950.png]]

**配置分布式路由**
1. 主路由中调用include函数
	语法：include('app名字.url模块名')
	作用：用于将当前路由转到各个应用的路由配置文件的urlpatterns进行分布式处理
	以http://127.0.0.1:8000/music/index为例：
```python
from django.urls import path, include
from . import views

urlpatterns = [
	path('admin/', admin.site.urls),
	path('test_static', views.test_static),
	path('music/',include('music.urls'))
]
```
2. 应用下配置urls.py
应用下手动创建urls.py文件，内容结构同主路由完全一致
```python
from django.urls import path
from . import views

urlpatterns = [
	# http://127.0.0.1:8000/music/index
	path('index', views.index_view)
]
```

## 2.3 应用下的模板

**应用内部可以配置模板目录**
1. 应用下手动创建templates文件夹
2. settings.py中 开启 应用模板功能
- TEMPLATES配置项中的'APP_DIRS'值为True即可
**应用下templates 和 外层templates 都存在时，django得查找模板规则**
1. 优先查找外层templates目录下的模板
2. 按INSTALLED_APPS配置下的 应用顺序 逐层查找

# 3. 模型层及ORM介绍

## 3.1 模型层-定义

- 回顾Django MTV
![[Pasted image 20240309163424.png]]
- 模型层 - 负责和数据库之间进行通信

**Django配置mysql**
- 创建数据库
- 进入mysql数据库 执行
	- create database数据库名 default charset utf8
	- 通常数据库名和项目名保持一致
- 在settings.py中进行数据库的配置
	- 修改DATABASES配置项的内容，由sqlite3变为mysql

## 3.2 什么是模型

- 模型是一个Python类，它由django.db.models.Model派生出的子类
- 一个模型类别代表数据库中的一张数据表
- 模型类中的每一个类属性都代表数据库中的一个字段
- 模型是数据交互的接口，是表示和操作数据库的方法和方式


## 3.3 ORM框架

- **定义**：ORM（Object Relational Mapping）即对象关系映射，它是一种程序技术，它允许你使用类和对象对数据库进行操作，从而避免通过SQL语句操作数据库
- **作用**：
	1. 建立模型类和表之间的对应关系，允许我们通过面向对象的方式来操作数据库
	2. 根据设计的模型类生成数据库中的表格
	3. 通过简单的配置就可以进行数据库的切换
- **优点**：
- 只需要面向对象编程，不需要面向数据库编写代码
	- 对数据库的操作都转化为对类属性和方法的操作
	- 不用编写各种数据库sql语句
- 实现了数据模型与数据库的解耦，屏蔽了不同数据库操作上的差异
	- 不在关注用的是mysql、oracle...等数据库的内部细节
	- 通过简单的配置就可以轻松更换数据库，而不需要修改代码
- **缺点**
- 对于复杂业务，使用成本较高
- 根据对象的操作转化成SQL语句，根据查询的结果转化为对象，在映射过程中由性能损失
- **映射图**
![[Pasted image 20240309164810.png]]

### 模型示例
- 此示例为添加一个bookstore_book数据表来存放图书馆中书目信息
1. **添加一个bookstore的app**
python3 manage.py startapp bookstore
2. **添加模型类并注册app**
- 模型类代码示例
```python
# file: bookstore/models.py
from django.db import models

class Book(model.Model):
	title = models.CharField("书名", max_length=50, default='')
	price = models.DecimalField('定价', max_digits=7, decimal_places=2, default=0.0)
```
3. **数据库迁移**
迁移是Django同步你对模型所做更改（添加字段，删除模型等）到你的数据库模式的方式
- 生成迁移文件 - 执行 python3 manage.py makemigrations 将应用下的models.py文件生成一个中间文件，并保存在migration文件夹中
- 执行迁移脚本程序 - 执行 python3 manage.py migrate 执行迁移程序实现迁移。将每个应用下的migration目录中的中间文件同步回数据库

## 3.4 模型类-创建

- 模型类 - 创建
```python
from django.db import models
class 模型类名(models.Model):
	字段名 = models.字段类型(字段选项)
```


# 4. ORM-基础字段及选项1
## 4.1 创建模型类流程

- 创建应用
- 在应用下的models.py中编写模型类
```python
from django.db import models
class 模型类名(model.Model):
	字段名 = models.字段类型(字段选项)
```
- 迁移同步 makemigrations & migrate
- 任何关于表结构的修改，务必在对应模型类上修改
- 例：为bookstore_book表 添加一个 名为info的字段 varchar(100)
	解决方案->
	- 模型类中添加 对应 类属性
	- 执行数据库迁移

## 4.2 字段类型

**模型类 - 字段类型**
- BooleanField()
	- 数据库类型：tinyint(1)
	- 编程语言中：使用True或False来表示值
	- 在数据库中：使用1或0来表示具体的值
- CharField()
	- 数据库类型：varchar
	- 注意：必须要指定max_length参数值
- DateField()
	- 数据库类型：date
	- 作用：表示日期
	- 参数：
	1. auto_now：每次保存对象时，自动设置该字段为当前时间（取值：True/False）
	2. auto_now_add：当对象第一次被创建时自动设置当前时间（取值：True/False）
	3. default：设置当前时间（取值：字符串格式时间如：'2019-6-1'）
	- 以上参数只能多选一
- DateTimeField()
	- 数据库类型：datetime(6)
	- 作用：表示日期和时间
	- 参数同DateField
- FloatField()
	- 数据库类型：double
	- 编程语言中和数据库中都使用小数表示值
- DecimalField()
	- 数据库类型：decimal(x, y)
	- 编程语言中：使用小数表示该列的值
	- 在数据库中：使用小数
	- 参数：
		max_digits：位数总数，包括小数点后的位数。该值必须大于等于decimal_places：小数点后的数字数量
- EmailField()
	- 数据库类型：varchar
	- 编程语言和数据库中使用字符串
- IntegerField()
	- 数据库类型：int
	- 编程语言和数据库中使用整数
- ImageField()
	- 数据库类型：varchar(100)
	- 作用：在数据库中为了保存图片的路径
	- 编程语言和数据库中使用字符串
- TextField()
	- 数据库类型：longtext
	- 作用：表示不定长的字符串数据

# 5. ORM-基础字段及选项2

## 5.1 模型类 - 字段选项

- 字段选项，指定创建的列的额外的信息
- 允许出现多个字段选项，多个选项之间使用，隔开
- primary_key
	- 如果设置为True，表示该列为主键，如果指定一个字段为主键，则此数据表不会创建id字段
- blank
	- 设置为True时，字段可以为空。设置为False时，字段是必须填写的
- null
	- 如果设置为True，表示该列允许为空
	- 默认为False，如果此选项为False建议加入default选项来设置默认值
- default
	- 设置所在列的默认值，如果字段选项null=False建议添加此项
- db_index
	- 如果设置为True，表示为该列增加索引
- unique
	- 如果设置为True，表示该字段在数据库中的值必须是唯一（不能重复出现的）
- db_column
	- 指定列的名称，如果不指定的话则采用属性名作为列名
- verbose_name
	- 设置此字段在admin界面上的显示名称
**字段选项样例**
```python
# 创建一个属性，表示用户名称，长度30个字符，必须是唯一的，不能为空，添加索引
name = models.CharField(max_length=30, unique=True, null=False, db_idnex=True)
```

好习惯：修改过字段选项【添加或更改】均要执行makemigrations 和 migrate

## 5.2 模型类-Meta类

**Meta类-定义**

使用内部Meta类 来给模型赋予属性，Meta类下有很多内建的类属性，可对模型类做一些控制
示例：
```python
# file:bookstore/models.py
from django.db import models

class Book(models.Model):
	title = models.CharField("书名", max_length=50, default='')
	price = models.DecimalField('定价', max_digits=7, decimal_places=2, default=0.0)
	class Meta:
		db_table = 'book' # 可改变当前模型类对应的表名
```

# 6. ORM-基本操作-创建数据

## 6.1 ORM-操作

- 基本操作包括增删改查，即（CRUD操作）
- CRUD是指在做计算处理时的增加，读取查询，更新和删除
- ORM CRUD核心 -> 模型类.管理器对象

## 6.2 管理器对象
- 每个继承自models.Model的模型类，都会有个objects对象被同样继承下来。这个对象叫管理器对象
- 数据库的增删改查可以通过模型的管理器实现
```python
class MyModel(models.Model):
	...
MyModel.objects.create(...) # objects 是管理对象
```

## 6.3 创建数据
- Django ORM使用一种直观的方法把数据库表中的数据表示成Python对象
- 创建数据中每一条记录就是创建一个数据对象

**方案1**
MyModel.objects.create(属性1=值1, 属性2=值1，....)
- 成功：放回创建好的实体对象
- 失败，抛出异常
**方案2**
创建MyModel示例对象，并调用save()进行保存
```python
obj = MyModel(属性=值，属性=值)
obj.属性=值
obj.save()
```

## Django Shell

在Django提供了一个交互式的操作项目叫Django Shell 它能在交互式模式用项目工程的代码执行相应的操作

利用Django Shell可以代替编写view的代码来进行直接操作
注意：项目代码发生变化时，重新进入Django shell

**启动方式**：
- python3 manage.py shell
