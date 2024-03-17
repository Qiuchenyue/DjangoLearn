# 1. ORM-查询操作-1

## 1.1 查询简介

- 数据库的查询需要使用管理器对象进行
- 通过MyModel.objects管理器方法调用查询方法

| 方法        | 说明                    |
| --------- | --------------------- |
| all()     | 查询全部记录，返回QuerySet查询对象 |
| get()     | 查询符合条件的单一记录           |
| filter()  | 查询符合条件的多条记录           |
| exclude() | 查询符合条件之外的全部记录         |
| ....      |                       |

**all()方法**
- 用法：MyModel.objects.all()
- 作用：查询MyModel实体中的所有的数据
- 等同于select \* from table
- 返回值：QuerySet容器对象，内部存放MyModel实例
```python
from bookstore.models import Book 
books = Book.objects.all()
for book in books:
	print("书名", book.title, '出版社:', book.pub)
```

可以在模型类中定义__str__方法，自定义QuerySet中的输出格式
例如 在Book模型类下定义如下：
```python
def__str__(sefl):
	return '%s_%s_%s_%s'%(self.title,self.price,self.pub,self.market_price)
```
则在django shell中可得到如下显示输出
```
In [20]:Book.objects.all()
Out[20]:<QuerySet [<obj>,<obj>,<obj>]>
```

**values('列1', '列2'..)**
- 用法：MyModel.objects.values(...)
- 作用：查询部分列的数据并返回
- 等同于select列1，列2 from xxx
- 返回值：QuerySet，返回结果容器，容器内存字典，每个字典代表一条数据
- 格式为：{'列1': 值1，'列2': 值2}

**values_list('列1','列2'...)**
- 用法：MyModel.objects.values_list(...)
- 作用：返回元组形式的查询结果
- 等同于select 列1，列2 from xxx
- 返回值：QuerySet容器对象，内部存放'元组'
- 会将查询出来的数据封装到元组中，再封装到查询集合QuerySet中

**order_by()**
- 用法：MyModel.objects.order_by('-列','列')
- 作用：与all()方法不同，它会用SQL语句的ORDER BY子句对查询结果进行根据某个字段选择性的进行排序
- 说明：默认是按照升序排序，降序排序则需要再列前增加'-'表示

# 2. ORM-查询操作-2

## 2.1 条件查询-方法

**filter(条件)**
- 语法：MyModel.objects.filter(属性1=值1, 属性2=值2)
- 作用：返回包含此 条件 的 全部的数据集
- 返回值：QuerySet容器对象，内部存放MyModel实例
- 说明：当多个属性在一起时为“与”关系
filter样例
```python 
from bookstore.mdoels import Book
books = Book.objects.filter(pub="清华大学出版社")
for book in books:
	print("书名:",book.title)

# 查询Author实体中name为王老师并且age是28岁的
	authors = Author.objects.filter(name='王老师',age=28)
```

**exclude(条件)**
- 语法：MyModel.objects.exclude(条件)
- 作用：返回不包含此 条件 的 全部的数据集
- 示例
```python
books = models.Book.objects.exclude(pub="清华大学出版社",price=50)
for book in books:
	print(book)
```

**get(条件)**
- 语法：MyModel.objects.get(条件)
- 作用：返回满足条件的唯一一条数据
- 说明：该方法只能返回一条数据，查询多于一条数据则抛出Model.MultipleObjectsReturned异常，查询结果没有数据则抛出Model.DoesNotExist异常

## 2.2 查询谓词

定义：做更灵活的条件查询时需要使用查询谓词
说明：每一个查询谓词是一个独立的查询功能
**\_\_exact: 等值匹配**
示例：
```python
Author.objects.filter(id__exact=1)
# 等同于select * from author where id = 1
```

**__contains: 包含指定值**
示例：
```python
Author.objects.filter(name__contains='w')
# 等同于select * from author where name like '%w%'
```

**__startwith: 以xxx开始**
**__endswith: 以XXX结束**
**__gt: 大于指定值**
```python
Author.objects.filter(age__gt=50)
# 等同于select * from author where age > 50
```
**__gte: 大于等于**
**__lt: 小于**
**\_\_lte:  小于等于**
**__in: 查找数据是否在指定范围内**
```python
Author.objects.filter(country__in=['中国','日本','韩国'])
# 等同于select * from author where country in('中国','日本','韩国')
```
**__range: 查找数据是否在指定的范围内**
```python
# 查找年龄在某一区间内的所有作者
Author.objects.filter(age__range=(35,50))
# 等同于SELECT ... WEHRE Author BETWEEN 35 and 50
```

# 3. ORM-更新操作
## 3.1 更新单个数据
**修改单个实体的某些字段值的步骤**
1. 查-通过get()得到要修改的实体对象
2. 改-通过 对象.属性 的方式修改数据
3. 保存-通过 对象.save()保存数据
## 3.2 批量更新数据
- 直接调用QuerySet的Update(属性=值)实现批量修改
- 示例
```python
# 将id大于3的所有图书价格定位0元
books = Book.objects.filter(id__gt=3)
books.update(price=0)
# 将所有书的零售价定为100元
books = Book。objects.all()
books.update(market_price=100)
```

# 4. ORM-删除操作

## 4.1 单个数据删除
**步骤**
1. 查找查询结果对应的一个数据对象
2. 调用这个数据对象的delete() 方法实现删除
```python 
try:
	auth = Author.objects.get(id=1)
	auth.delete()
except:
	print(删除失败)
```
## 4.2 批量删除
**步骤**
1. 查找查询结果集中满足条件的全部QuerySet查询集合对象
2. 调用查询集合对象的delete()方法实现删除
```python
# 删除全部作者中，年龄大于65的全部信息
auth = Author.objects.filter(age__gt=65)
auth.delete()
```

**伪删除**
- 通常不会轻易在业务里把数据真正删掉，取而代之的是做伪删除，即在表中添加一个布尔类型字段(is_active)，默认是true；执行删除时，将欲删除数据的is_active字段置为False
- 注意：用伪删除时，确保显示数据的地方，均加了is_active=True的过滤查询

# 5. F对象和Q对象

## 5.1 F对象
- 一个F对象代表数据库中某条记录的字段的信息
- 作用：
	- 通常是对数据库中的字段值在不获取的情况下进行操作
	- 用于类属性（字段）之间的比较
- 语法
```python 
from django.db.models import F
F('列名')
```
示例1 更新Book实例中所有的零售价涨10元
```python 
Book.objects.all().update(market_price=F('market_price')+10)
# 以上做法好于如下代码
books = Book.objects.all()
for book in books:
	book.market_price = book.market_price+10
	book.save()
```
示例2 对数据库中的两个字段的值进行比较，列出哪些书的零售价高于定价？
```python
from django.db.models import F
from bookstore.models import Book
books = Book.objects.filter(market_price__gt=F('price'))
for book in books:
	print(book.title, '定价: ', book.price, '现价: ', book.market_price)
```

## 5.2 Q对象
当在获取查询结果集 使用复杂的逻辑或 |、逻辑非~等操作时可以借助于Q对象进行操作
如：想找出定价低于20元 或 清华大学出版社的全部书，可以写成
```python
Book.objects.filter(Q(price__lt=20)|Q(pub="清华大学出版社"))
```
Q对象在 数据包django.db.models中。需要先导入再使用
运算符：
& 与操作
| 或操作
~ 非操作
语法：
```python
from django.db.models import Q
Q(cond1)|Q(cond2)
Q(cond1)&Q(cond2)
Q(cond1)&~Q(cond2)
```
示例：
```python 
from django.db.models import Q
Book.objects.filter(Q(market_price__lt=50)|Q(pub_house='清华大学出版社'))
Book.objects.filter(Q(market_price__lt=50)&~Q(pub_house='机械工业出版社'))
```

# 6. 聚合查询和原生数据库操作

## 6.1 聚合查询

聚合查询是指对一个数据表中的一个字段的数据进行部分或全部进行统计查询，查bookstore_book数据表中的全部书的平均价格，查询所有书的总个数等，都要用到聚合查询

聚合查询分为：
1. 整表查询
2. 分组聚合

**整表查询**
不带分组的聚合查询是指导将全部数据进行集中统计查询
聚合函数\[需要导入\]
- 导入方法：from django.db.models.import \*
- 聚合函数：Sum，Avg，Count，Max，Min
语法：MyModel.objects.aggregate(结果变量名=聚合函数('列'))
- 返回结果：结果变量名和值组成的字典
- 格式为：{"结果变量名": 值}

整表聚合示例
```python
# 得到所有书的平均价格
from bookstore.models import Book
from django.db.models import Avg
result = Book.objects.aggregate(myAvg=Avg('price'))
print("平均价格是: ", result['myAvg'])
print("result=", result)

# 得到数据表里有多少本书
from django.db.models import Count
result = Book.objects.aggregate(mycnt=Count('title'))
print("数据记录总个数是: ",result['mycnt'])
print("result=", result) # {"mycnt": 10}
```

**分组聚合**
分组聚合是指通过计算查询结果中每一个对象所关联的对象集合，从而得出总计值（也可以是平均值或总和），即为查询集的每一项生成集合。

- 语法：QuerySet.annotate(结果变量名=聚合函数('列'))
- 返回值：QuerySet

1. 通过先用查询结果MyModel.objects.values查找查询要分组聚合的列
MyModel.objects.values('列1','列2')
如：
```python
pub_set = Book.objects.values('pub')
print(pub_set)
```
2. 通过返回结果的QuerySet.annotate方法分组聚合得到分组结果
	QuerySet.annotate(名=聚合函数('列'))
```python
pub_count_set = 
pub_set.annotate(myCount=count('pub'))
print(pub_count_set)
```

## 6.2 原生数据库操作
Django也可以支持 直接用sql语句的方式通信数据库
- 查询：使用MyModel.objects.raw()进行 数据库查询操作
- 语法：MyModel.objects.raw(sql语句, 拼接参数)
- 返回值：RawQuerySet集合对象【只支持基础操作，比如循环】
```python 
books = models.Book.objects.raw('select * from bookstore_book')
for book in books:
	print(book)
```
**SQL注入**
使用原生语句时小心SQL注入
定义：用户通过数据上传，将恶意的sql语句提交给服务器，从而达到攻击效果

**cursor**
完全跨过模型类操作数据库 - 查询/更新/删除
1. 导入cursor所在包
	from django.db import connection
2. 用创建cursor类的构造函数创建cursor对象，再使用cursor对象，为保证在出现异常时能释放cursor资源，通常使用with语句进行创建操作
```python
from django.db import connection
with connection.cursor() as cur:
	cur.execute('执行SQL语句', '拼接参数')
```

