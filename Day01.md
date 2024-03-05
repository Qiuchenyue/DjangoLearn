# 1. 项目结构
## settings.py
---

 - settings.py 包含了Django项目启动的所有配置项
 - 配置项分为公有配置和自定义配置
 - 配置项格式为：BASE_DIR= 'xxxx'
 - 公有配置 - Django官方提供的基础配置
 
## 公有配置
---

**BASE_DIR**
- 用于绑定当前项目的绝对路劲（动态计算出来的），所有文件夹都可以依赖此路劲
**DEBUG**
- 用于配置Django项目的启动模式，取值：
- True表示开发环境中使用 开发模式（用于开发中）
- False表示当前项目运行在生产环境中
**ALLOWED_HOSTS**
- 设置允许访问到本项目的host头值
- \[\]空列表，表示请求头中host为127.0.0.1，locallhost能访问本项目 - DEGUB = True时有效
- \['\*'\], 表示任何请求头的host都能访问到当前项目
- \['192.168.1.3','127.0.0.1'\]表示只有当前两个host头的值能访问当前项目
示例：如果要在局域网其他主机也能访问此主机的Django服务，启动示例如下：
- python3 manage.py runserver 0.0.0.0:5000
- 指定网络设备如果内网环境下其他主机想正常访问该站点，需加ALLOWED_HOSTS = \['内网ip'\]
**INSTALLED_APPS**
- 指定当前项目中安装的应用列表
**MIDDLEWARE
- 用于注册中间件
**TEMPLATES**
- 用于指定模板的配置信息
**DATABASES**
- 用于指定数据库的配置信息
**LANGUAGE_CODE**
- 用于指定语言配置
- 英文：“en-us”
- 中文：“zh-Hans"
**TIME_ZONE**
- 用于指定当前服务器端时区
- 世界标准时间：”UTC“
- 中国时区：”Asia/Shanghai“
**ROOT_URLCONF
- 用于配置主url配置'mysite1.urls'
- ROOT_URLCONF = 'mysite1.urls'
## 自定义配置
---
- settings.py中也可以添加开发人员自定义的配置
- 配置建议：名字尽量个性化 - 以防覆盖掉公有配置
	例如：ALIPAY_KEY = 'xxxxxx'
settings.py中所有配置项，都可以按需的在代码中引入，引入方式：
from django.conf import settings

# 2. URL和视图函数
## URL - 结构
---
- 定义 - 即统一资源定位符 Uniform Resource Locator
- 作用 - 用来表示互联网上某个资源的地址
- URL的一般语法格式为（注：\[\]代表其中的内容可省略）：
	- protocol: //hostname\[:port\]/path\[?query\]\[\#fragment\]
	- http://tts.tmooc.cn/video/showVideo?menuld=657421&version=AID999#subject

**protocol(协议)** http://tts.tmooc.cn
- http通过HTTP访问该资源。格式 http://
- https 通过安全的HTTPS访问该资源。格式https://
- file 资源是本地计算机上的文件。格式: file:///
**hostname（主机名**）http://tts.tmooc.cn
- 是指存放资源的服务器的域名系统（DNS）主机名、域名或IP地址
**port(端口号) http://tts.tmooc.cn:80
- 整数，可选，省略时使用方案默认端口
- 各种传输协议都有默认的端口号，如http的默认端口为80
**path(路由选址) http://tts.tmooc.cn/video/showVideo
- 由零或多个”/“符号隔开的字符串，一般用来表示主机上的一个目录或文件地址。路由地址决定了服务器端如何处理这个请求
**query（查询）**
/video/showVideo?menuId=657421&version=AID999
- 可选，用于给动态网页传递参数，可有多个参数，用”&“符号隔开，每个参数的名和值用”=“符号隔开。
**fragment（信息片断）**
version=AID999#subject
- 字符串，用于指定网络资源中的片断。例如一个网页中有多个名词解释，可使用fragment直接定位到某一名词解释。
## Django如何处理URL的请求
---
**处理URL请求**
- 浏览器 地址栏--> http://127.0.0.1:8000/page/2003/
1. Django从配置文件中 根据ROOT_URLCONF 找到主路由文件；默认情况下，该文件在项目名目录下的urls；例如mysite1/mysite1/urls.py
2. Django加载 主路由文件中的urlpatterns变量\[包含很多路由的数组\]
3. 依次匹配urlpatterns中的path，匹配到第一个合适的中断后续匹配
4. 匹配成功 - 调用对应的视图函数处理请求，返回响应
5. 匹配失败 - 返回404响应
**主路由-urls.py样例**
```python
from django.urls import path
from . import views
urlpatterns = [
	path('admin/',admin.site.urls)
	path('page/2003/',views.page_2003),
	path('page/2004/',views.page_2004),
]
```

## 视图函数
---

- 视图函数时用于接收一个浏览器请求（HttpRequest对象）并通过HttpResponse对象返回响应的函数。此函数可以接收浏览器请求并根据业务逻辑返回相应的内容给浏览器
- 语法：
```python
def xxx_view(request[,其他参数...]):
	return HttpResponse对象
```
 - 样例
```python
# file:<项目同名文件夹下>/views.py
from django.http import HttpResponse
def page1_view(request):
	html = "<h1>这是第一个页面</h1>"
	return HttpResponse(html)
```

# 3. 路由配置
## path
---
- path()函数
- 导入 - from django.urls import path
- 语法 - path(rouute, views, name = None)
- 参数：
	1. route：字符串类型，匹配的请求路径
	2. views：指定路径所对应的视图处理函数的名称
	3. name：为地址起别名，在模板中地址反向解析时使用
## 路由转换器
---
- path转换器
- 语法：\<转换器类型：自定义名\>
- 作用：若转换器类型匹配到对应类型的数据，则将数据按照关键字传参的方式传递给视图函数
- 例子：path('page/\<int:page\>', views.xxx)

| 转换器类型 | 作用                             | 样例                                             |
| ----- | ------------------------------ | ---------------------------------------------- |
| str   | 匹配除了‘/’之外的非空字符串                | "v1/users/str:username>"匹配/v1/users/guoxiaonao |
| int   | 匹配0或任何正整数。返回一个int              | "page/<int:page>"匹配/page/100                   |
| slug  | 匹配任意有ASCII字母或数字以及连字符和下划线组成的短标签 | "detail/<slug:sl>"匹配/detail/this-is-django     |
| path  | 匹配非空字段，包括路径分隔符'/'              | "v1/users/<path:ph>"匹配/v1/goods/a/b/c          |
## re_path()
---
- re_path()函数
- 在url的匹配过程中可以使用正则表达式进行精确匹配
- 语法：
	- re_path(reg, view, name = xxx)
	- 正则表达式为命名分组模式(?P\<name\>pattern); 匹配提取参数后用关键字传参方式传递给视图函数。
