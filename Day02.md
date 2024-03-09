# 1. 请求和响应

## 1.1 Http协议

- 请求和响应是指浏览器通过HTTP协议发送给服务器端的数据
- 响应是指服务器端接收到请求后做相应的处理后再回复给浏览器端的数据
## 1.2 请求中的方法

- 根据HTTP标准，HTTP请求可以使用多种请求方法
- HTTP1.0定义了三种请求方法：GET，POST和HEAD方法（最常用）
- HTTP1.1新增了五种请求方法：OPTIONS，PUT，DELETE，TRACE和CONNECT方法

| 序号  | 方法      | 描述                                                                     |
| --- | ------- | ---------------------------------------------------------------------- |
| 1   | GET     | 请求指定的页面信息，并返回实体主体                                                      |
| 2   | HEAD    | 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头                                       |
| 3   | POST    | 向指定资源提交数据进行处理 请求（例如提交表单或上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。 |
| 4   | PUT     | 从客户端向服务器传送的数据取代指定的文档的内容                                                |
| 5   | DELETE  | 请求服务器删除指定的页面                                                           |
| 6   | CONNECT | HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。                                       |
| 7   | OPTIONS | 允许客户端查看服务器的性能                                                          |
| 8   | TRACE   | 回显服务器收到的请求，主要用于测试或诊断                                                   |

**Django中的请求**
- 请求再django中实则就是 视图函数的第一个参数，即HttpRequest对象
- Django接收到http协议的请求后，会根据请求数据报文创建HttpRequest对象
- HttpRequest对象 通过属性 描述了 请求的 所有相关信息
- path_info：URL字符串
- method：字符串，表示HTTP请求方法，常用值：'GET'、’POST‘
- GET：QueryDict查询字典的对象，包含get请求方式的所有数据
- POST：QueryDict查询字典的对象，包含post请求方式的所有数据
- FILES：类似于字典的对象，包含所有的上传文件信息
- COOKIES：python字典，包含所有的cookie，键和值都为字符串
- session：似于字典的对象，表示当前的会话
- body：字符串，请求体的内容（POST或PUT）
- scheme：请求协议（’http‘/’https）
- request.get_full_path()：请求的完整路径
- request.META：请求中的元数据（消息头）
	-request.META\['REMOTE_ADDER'\]：客户端IP地址


## 1.3 响应状态码

HTTP状态码的英文为HTTP Status Code
下面时常见的HTTP状态码：
- -200- 请求成功
- -301-永久重定向-资源（网页等）被永久转移到其他URL
- -302-临时重定向
- -404-请求的资源（网页等）不存在
- -500-内部服务器错误

| 分类  | 分类描述                    |
| --- | ----------------------- |
| 1** | 信息，服务器收到请求，需要请求者继续执行操作  |
| 2** | 成功，操作被成功接收并处理           |
| 3** | 重定向，需要进一步的操作以完成请求       |
| 4** | 客户端错误，请求包含语法错误或无法完成请求   |
| 5** | 服务器错误，服务器在处理请求的过程中发生了错误 |

**Django中的响应对象**
*构造函数格式*：
	HttpResponse（content=响应体，content_type=响应体数据类型，status=状态码）
	作用：向客户端浏览器返回响应，同时携带响应体内容
*常用的Content-type如下*
	- 'text/html'（默认的，html文件）
	- 'text/plain'（纯文本）
	- 'text/css'（css文件）
	- 'text/javascript'（js文件）
	- 'multipart/form-data'（文件提交）
	- 'application/json'（json传输）
	- 'application/xml'（xml文件）
*HttpResponse子类*

| 类型                      | 作用      | 状态码 |
| ----------------------- | ------- | --- |
| HttpResponseRedirect    | 重定向     | 302 |
| HttpResponseNotModified | 未修改     | 304 |
| HttpResponseBadRequest  | 错误请求    | 400 |
| HttpResponseNotFound    | 没有对应的资源 | 404 |
| HttpResponseForbidden   | 请求被禁止   | 403 |
| HttpResponseServerError | 服务器错误   | 500 |


# 2. GET请求和POST请求

- 无论是GET还是POST，统一都由视图函数接收请求，通过判断request.method区分具体的请求动作
- 样例
```python 
if request.method == 'GET':
	处理GET请求时的业务逻辑
elif request.method == 'POST':
	处理POST请求的业务逻辑
else:
	其他请求业务逻辑
```

## 2.1 GET处理

- GET请求动作，一般用于向服务器获取数据
- 能够产生GET请求的场景：
	-浏览器地址栏中输入URL，回车后
	-\<a href="地址?参数=值&参数=值"\>
	-form表单中的method为get
- GET请求方法中，如果有数据需要传递给服务器，通常会用查询字符串（Query String）传递【注意：不要传递敏感数据】
- URL格式：xxx?参数名1=值1&参数名2=值2...
	-如：http://127.0.0.1:8000/page1?a=100&b=200
- 服务器端接收参数，获取客户端请求GET请求提交的数据
- 方法示例：
```python
request.GET['参数名']
request.GEt.get('参数名','默认值')
request.GET.getlist('参数名')
```

## 2.2 POST处理

- POST请求动作，一般用于向服务器提交大量/隐私数据
- 客户端通过表单等POST请求将数据传递给服务器端，如：
```python
<form method='post' action="/login">
	姓名:<input type="text" name="username">
	<input type='sunmit' value='登录'>
</form>
```
- 服务端接收参数
	通过request.method来判断是否为POST请求，如：
```python
if request.method == 'POST':
	处理POST请求的数据并响应
else:
	处理非POST 请求的响应
```
- 使用post方式接收客户端数据
```pyhton
request.POST['参数名']
request.POST.get('参数名','')
request.POST.getlist('参数名')
```
取消csrf验证，否则Django将会拒绝客户端发来的POST请求，报403响应。
	禁止掉settings.py中MIDDLEWARE中的CsrfViewsMiddleWare的中间件

# 3. Django的设计模式及模板层

## 3.1 MVC和MTV

**MVC代表Model-View-Controller（模型-视图-控制器）模式**
- M模型层（Model），主要用于对数据库层的封装
- V视图层（View），用户向用户展会结果（What+How）
- C控制（Controller，用于处理请求、获取数据、返回结果（重要）
**作用：降低模块间的耦合度（解耦）**
![[Pasted image 20240304125238.png]]
**MTV代表Model-Template-View（模型-模板-视图）模式
- M模型层（Model）负责与数据库交互
- T模板层（Template）负责呈现内容到浏览器（How）
- V视图层（View）是核心，负责接收请求、获取数据、返回结果（What）
**作用：降低模块间的耦合度（解耦）**
![[Pasted image 20240304125528.png]]

## 3.2 模板层

### 什么是模板

1. 模板时可以根据字典数据动态变化的html网页
2. 模板可以根据视图中传递的字典数据动态生成相应的HTML网页
### 模板配置

创建模板文件夹<项目名>/templates
在settings.py中TEMPLATES配置项
1. BACKEND：指定模板的引擎
2. DIRS：模板的搜索目录（可以是一个或多个）
3. APP_DIRS：是否要在应用中的templates文件夹中搜索模板文件
4. OPTIONS：有关模板的选项
- 配置中 需要修改的部分：
	设置DIRS- 'DIRS'：\[os.path.join(BASE_DIR, 'templates')\],
### 模板的加载方式

方案1 - 通过loader获取模板，通过HttpResponse进行响应
在视图函数中
```python
from django.template import loader
# 1.通过loader加载模板
t = loader.get_template("模板文件名")
# 2.将t转化成HTML字符串
html = t.render(字典数据)
# 3.用响应对象将转换的字符串内容返回给浏览器
return HttpResponse(html)
```

方案2 - 使用render()直接加载并响应模板
在视图函数中：
```python
from django.shortcuts import render
return render(request,'模板文件名',字典数据)
```

### 视图层与模板层之间的交互

1. 视图函数中可以将python变量封装到字典中传递到模板
```python
def xxx_view(request)
	dic = {
		"变量1":"值1",
		"变量2":"值2",
	}
	return render(request,'xxx.html',dic)
```
2. 模板中，我们可以用{{变量名}}的语法 调用视图函数传进来的变量

### 模板层 - 变量

**1. 模板的变量**
视图函数中可以将Python变量封装到字典中

**2. 能传递到模板中的数据类型：**
str - 字符串     int - 整型
list - 数组        tuple - 元组
dict - 字典       func - 方法
obj - 类实例化的对象

**3. 在模板中使用变量语法**
-{{变量名}}
-{{变量名.index}}
-{{变量名.key}}
-{{对象.方法}}
-{{函数名}}

### 模板标签

**作用**：将一些服务器端的功能嵌入到模板中，例如流程控制等
**标签语法**：
```
{% 标签 %}
···
{% 结束标签 %}
```
*if 标签*
```
{% if condition1 %}
...
{% elif condition2 %}
...
{% elif condition3 %}
...
{% else %}
...
{% endif %}
```
注意：
1. if条件表达式里可以用的运算符 \==, !=, <, >, <=, >=, in, not in, is,  is not, not, and, or
2. 在if标记中使用实际括号是无效的语法。如果你需要它们指示优先级，则应使用嵌套的if标记
*for标签*
```
{% for 变量 in 可迭代对象 %}
	...循环语句
{% empty %}
	...可迭代对象无数据时填充的语句
{% endfor %}
```
*内置变量 - forloop*

| 变量                 | 描述                     |
| ------------------ | ---------------------- |
| forloop.counter    | 循环的当前迭代（从1开始索引）        |
| forloop.counter0   | 循环的当前迭代（从0开始索引）        |
| forloop.recounter  | counter值的倒序            |
| forloop.recounter0 | recounter值的倒序          |
| forloop.first      | 如果这是第一次通过循环，则为真        |
| forloop.last       | 如果这是最后一次循环，则为真         |
| forloop.parentloop | 当嵌套循环，parentloop表示外层循环 |

## 3.3 模板层-过滤器和继承

### 模板过滤器

定义：在变量输出时对变量的值进行处理
作用：可以通过使用 过滤器来改变变量的输出显示
语法：{{变量 | 过滤器1:'参数值1' | 过滤器2:'参数值2'...}}
常用过滤器

| 过滤器                | 说明                                                  |
| ------------------ | --------------------------------------------------- |
| lower              | 将字符串转换为全部小写                                         |
| uppeer             | 将字符串转换为大写形式                                         |
| safe               | 默认不对变量内的字符串进行html转义                                 |
| add: "n"           | 将value的值增加n                                         |
| truncatechars: 'n' | 如果字符串字符多于指定的字符数量，那么会被截断。截断的字符串将以可翻译的省略号序列("...")结尾。 |
### 模板的继承

模板继承可以使父模板的内容重用，子模版直接继承父模板的全部内容并可以覆盖父模板中相应的块
**语法 - 父模板中**：
- 定义父模板中的块block标签
- 标识出哪些在子模块中是允许被修改的
- block标签：在父模板中定义，可以在子模版中覆盖
**语法 - 子模版中**：
- 继承模板extends标签（写在模板文件的第一行）
	例如{% extends 'base.html' %}
- 子模版 重写父模板中的内容块
	{% block block_name %}
	子模版块用来覆盖父模板中block_name块的内容
	{% endblock blockname %}
**重写的覆盖规则**
- 不重写，将按照父模板的效果显示
- 重写，则按照重写效果显示
注意：
- 模板继承时，服务器端的动态内容无法继承

# 4. url反向解析

## 4.1 再谈url

### 代码中url出现位置

1. 模板【html中】
	- \<a href='url'\>超链接\</a\>点击后 页面跳转至url
	- \<form action = 'url'  method='post'\>form 表单中的数据 用post方法提交至url
2. 视图函数中 - 302跳转 HttpResponseRedirect('url')
	将用户地址栏中的地址跳转到url

**url书写规范**
1. 绝对地址
	http://127.0.0.1:8000/page/1
2. 相对地址
	1，'/page/1' - '/'开头的相对地址，浏览器会把当前地址栏里的协议，ip和端口加上这个地址，作为最终访问地址，即如果当前页面地址栏为http://127.0.0.1:8000/page/3; 当前相对地址最终结果为http://127.0.0.1:8000 + /page/1
	2, 'page/1' - 没有’/‘开头的相对地址，浏览器会根据当前url最后一个/之前的内容 加上 该相对地址 作为最终访问地址，例如当前地址栏地址为http://127.0.0.1:8000/topic/detail; 则该相对地址最终为http://127.0.0.1:8000/topic/+page/1

## 4.2 url反向解析

url反向解析是指在视图或模板中，用path定义的名称来动态查找或计算出相应的路由
path函数的语法
- path(route, views, name='别名')
- path('page', views.page_view, name="page_url")
根据path中的’name=‘关键字传参给url确定了个唯一确定的名字，在模板或视图中，可以通过这个名字反向推断出此url信息

**模板中 - 通过url标签实现地址的反向解析**
```
{% url '别名' %}
{% url '别名' '参数1' '参数2' %}
ex：
{% url 'pagen' '400' %}
{% url 'person' age='18' name='gxn' %}
```
在视图函数中 -> 可调用django中的reverse方法进行反向解析
```python
from django.urls import reverse
reverse('别名', args=[], kwargs={})
ex:
print(reverse('pagen',args=[300]))
print(reverse('person',kwargs={'name':'xixi','age':18}))
```
