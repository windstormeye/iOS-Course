这个文章主要是记录我在学习django过程中所遇到的问题，为后续其它个人项目做铺垫，之前陆陆续续的在使用djano，但是一直都没有好好的去记录一些内容，借此机会完整的记录下学习过程遇到的问题和注意点。django最适合用于做网站，从django官网上的slogan看的出来了。至于django有多适合进行网站开发估计得要接着后续的学习才能知道了，不过我用django主要还是用与做api服务，在考虑之中的还有flask，flask比django更加简单，其中[这个repo](https://github.com/windstormeye/watchDog)是基于flask做的api服务。

### 创建一个django项目
```django-admin startproject <项目名称>```
### __init__.py文件
python包文件必须包含的，只要有这个文件，说明该Python文件目录下的所有文件都是一个包。

### 启动本地服务
```python manage.py runserver```

### 通过域名访问（dev environment）
```python manager.py runserver 0.0.0.0:8000```并且需要把域名在setting.py中的`ALLOWED_HOSTS`字段中。

### 创建超级管理员
忘记命令`manage.py`提供的命令，可以采用`python manage.py help`进行查看。

可以通过`python manage.py  createsuperuser`创建创建超级管理员

### 创建一个django app
```django-admin startproject <项目名称>```

### 同步数据库
制造迁移（迁移数据库可以只保存这个）```python manage.py makemigrations``` 

迁移```python manage.py migrate```

### path
写url时最好是通过path方式，而不是用正则走`re_path/url`的方式，因为会更加直观一些。

### 异常
eg:
```python
try:
    article = Artcle.objects.get(id=article_id)
except Artcle.DoesNotExist:
    return HttpResponse("不存在”)
```

### 模板
前端页面和后端代码进行分离，降低耦合性。django中规定了模板文件（html）的存放位置和格式，在project下的setting.py文件中的`TEMPLATES`可以看到且修改，如果采用默认配置，需要在app中新建`templates`文件夹，在其中写好对应的模板文件，

eg:这是循环打印文章列表的模板例子，

```html
<html>
<head>
</head>
<body>
	{% for article in articles %}
		<a href="{% url 'article_details' article.pk %}">{{ article.title }}</a>
	{% endfor %}
	<h2>{{article_obj.title}}</h2>
	<p>{{article_obj.content}}</p>
</body>
</html>
```
### 模板渲染
使用`django.shortcuts`app中的`render/render_to_response/get_objects_or_404`都可以进行模板渲染和调起。

### response的部分写法
这是一种返回数据的写法：
```python
def article_details(request, article_id):
    try:
        article = Artcle.objects.get(id=article_id)
        context = {}
        context['article_obj'] = article
        return render(request, "article_details.html", context)
    except Artcle.DoesNotExist:
        raise Http404("not exist")
    return HttpResponse("文章标题：%s 
 文章内容：%s" % (article.title, article.content))
```
这是另外一种，使用from django.shortcuts import get_object_or_404
```python
def article_details(request, article_id):
    article = get_object_or_404(Artcle, pk=article_id)
    context = {}
    context['article_obj'] = article
    return render(request, "article_details.html", context)
```
这样做会简洁很多，这也是django所推崇的做法。

### 模板中列出所有文章
列出所有文章。模板中可以采用`{/article/{{ article.pk }}`或者`{% url ‘article_list’ article.pk %}`，用第二种方法需要在对应的`urls.py`文件中添加对应的path.name。

### 分离url
很多时候我们不应该把所有的路由设置都放在project下的`urls.py`文件中，最佳的做法应该是把对应的url放在各自的app`urls.py`文件中（需新建），做法如下所示：
```python
# project `urls.py`
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('article/', include('artcle.urls'))
]
```

```python
# article `urls.py`
from django.urls import path, include
from . import views

urlpatterns = [
    path('', views.article_list, name="article_list"),
    path('<int:article_id>', views.article_details, name="article_details"),
]
```
千万要注意对应文件的路径！！！

### Python定制类
通过使用定制类中的__str__给用户定制显示内容，当然python中的定制类不止只有这些，详情可以参考[廖雪峰](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0013946328809098c1be08a2c7e4319bd60269f62be04fa000)
```python
def __str__(self):
    return "Article: %s" % self.title
```

### 在admin页面中显示字段
想要在admin管理页面中显示字段，可以在对应app的`admin.py`中这么做：
```py
class ArticleAdmin(admin.ModelAdmin):
    list_display = ("title", "content”)
# admin注册（其中的一种方法）
admin.site.register(Artcle, ArticleAdmin)
```

### 给admin管理页面某个字段排序
```py
class ArticleAdmin(admin.ModelAdmin):
    list_display = ("id", "title", "content")
    ordering = ("id", )
```

ordering默认是升序，改为-id，则为降序排序

### 给现有模型添加时间字段。
`from django.utils import timezone`，第一第二种需要这个库。

1. 如果是直接写。`created_time = models.DateTimeField()`并在对应app中的models.py文件中添加进了新字段，运行时会告知模型和数据库匹配不上，需要去同步数据库，当同步数据库的时候，系统添加默认值，两种方法，第一种在terminal中直接添加，第二种退出然后再添加默认值。
2. 同上第二种，退出后再添加默认值。`created_time = models.DateTimeField(default=timezone.now)`
3.  
```py
created_time = models.DateTimeField(auto_now_add=True)
last_updated_time = models.DateTimeField(auto_now=True)
```

### 修改时间类型。
在project的`setting.py`文件中找到`TIME_ZONE`字段，`TIME_ZONE = 'Asia/Shanghai’`(不知道为啥没有北京时间

### 使用自带用户类型。
导入`from django.contrib.auth.models import User`。给每篇文章添加作者信息，外键。    `author = models.ForeignKey(User, on_delete=models.DO_NOTHING, default=1)`

### 文章删除。
最好不要真的删除，而是作为用户不可见。文章模型添加字段`is_deleted = models.BooleanField(default=False)`。文章`views.py`的添加数据筛选
```py
def article_list(request):
    articles = Artcle.objects.filter(is_deleted=False)
    context = {}
    context['articles'] = articles
    return render_to_response("article_list.html", context)
```

### 下载virtualenv
`pip install virtualenv`

下载过程中若出现如下信息：

```shell
Traceback (most recent call last):
  File "/usr/bin/pip3", line 11, in <module>
    sys.exit(main())
  File "/usr/lib/python3/dist-packages/pip/__init__.py", line 215, in main
    locale.setlocale(locale.LC_ALL, '')
  File "/usr/lib/python3.5/locale.py", line 594, in setlocale
    return _setlocale(category, locale)
locale.Error: unsupported locale setting
```

则是因为语言配置所导致的，执行如下命令即可：

```shell
$ export LC_ALL=C
```

目的是为了去除所有本地化的设置，让命令能够正确执行。`LC_ALL` 它是一个宏，如果该值设置了，则该值会覆盖所有LC_*的设置值。注意，LANG的值不受该宏影响。`C` 是系统默认的locale，"POSIX"是"C"的别名。所以当我们新安装完一个系统时，默认的locale就是C或POSIX。

### 创建虚拟环境
```py
$ cd env
$ source ./bin/activate
```

### 内容显示截断。        
`<p>{{ blog.content|truncatechars:30 }}</p>`，也可以使用`truncatewords`，但是要求词和词中间要有空格，针对的是一个个的词，英文可以直接用。

### 模板文件
模板文件（一般都是html）如果是跟着项目走，那就应该放到全局模板文件里，如果是跟着app走应该放到app的模板文件里。

### html中标签的id一般写在class前面

### django的命名空间
在app中新建的static文件夹中再新建一个跟app名字一样的文件夹，然后把需要的静态文件放进行。注意：要重启服务器（CSS没效果也可以重启）

### 加载静态文件的声明
{% load staticfiles %}要放在需要用到的静态文件之前，而不是拆开。
```python
{% load staticfiles %}
{% block header_extends %}
    <link rel="stylesheet" href="{% static 'blog/blog.css' %}">
{% endblock %}    
```

### bootstrap和jquery最好都down下来，毕竟也不是特别大。使用bootstrap推荐直接上官网查资料。www.bootcss.com

### python2的range()出来后是个list，而python3得到的是个生成器

### filter筛选符合条件，exclude筛选不符合条件。

### 条件中的双下划线：字段查询类型、外键拓展（以博客分类为例）、日期拓展（以月份分类为例）、支持链式查询：可以一直链接下去

### 去除该段文本中的h5比标签内容，只保留文本 
`<p>{{ blog.content|striptags|truncatechars:120 }}</p>。`

### 过滤器safa显示h5标签内容
`<div class="blog-content">{{ blog.content|safe }}</div>`

### 富文本编辑库。django-ckeditor

### admin中文简体——zh-hans

### django2.0后url的设置可以直接用path

### pypi.org。可以查到相关的pip库

### 框架引用顺序：先是python自带，后是django自带，最后才是自己写的

----

## 事物
`Django` 中实现事物主要有两种方式：一是通过 `Django ORM` 框架的事物处理；另外一种是基于原生执行 `SQL` 语句的 `transaction` 处理。

至于为什么需要做事物管理，简单来说就是需要对用户提交的本次操作做完整性保证，有可能用户提交的本次操作涉及10多条 `SQL` 语句，但如果执行到其中第七第八条时出现问题后，之前执行的却已经被写入数据库中了，按道理说，如果中途出现错误，应该把之前的以及执行完的语句全都撤回。具体细节可参考我的[另一篇文章](http://pjhubs.com/2018/03/25/软件开发项目实践（一）/)

据官方文档中所描述的内容[https://docs.djangoproject.com/zh-hans/2.0/topics/db/transactions/](https://docs.djangoproject.com/zh-hans/2.0/topics/db/transactions/) ，`Django` 中的默认事物级别为 `auto-commit` ，用文档中举的例子来说，当我们在 `Django` 中做的 `model().save()` 和 `model().delete` 操作所有的改动都会立即提交，没有 `rollback` 。


<!-- 通过这种配置，django在调每个view方法之前会开始一个事务，当且仅当该响应没有任何问题，django才会提交这个事务；如果view中出现了异常，则django会回滚该事务。这样做的好处显而易见，就是安全简便，但是随着网站的流量变大，如果每个view被调用时都打开一个事务就会变得有点繁重，从而会降低网站的效率。它对性能的影响取决于你的应用的查询效率以及你的数据库处理锁的能力。此外，使用这种方式，回退的只是数据库的状态，而不包括其他非数据库项的操作，例如发送email等。 -->

### `Django ORM` 框架的事物处理
1. **将 `http request` 数据库操作全都包括。** 这种做法相当的简单粗暴，具体配置如下：

```json
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '',
        'USER': '',
        'PASSWORD': '',
        'HOST': '',
        'PORT': '',
        'ATOMIC_REQUESTS': True, // !!!
    }
}
```

这种方法是在 `django` 调用每个视图方法前都启动一个事物，并且要保证该响应没有任何问题， `django` 最终才会提交这个事物；如果在这其中出现了其它问题， `django` 会自动的回滚该事物。 这样会非常的简单，但不适合流量大的服务，因为这要对每个视图函数都要开启事物，而且回滚的只是数据库操作，如果本次操作中途涉及到了一些其它操作，比如执行了到了一半发送了邮件，但是下一步操作后发现有错误要回滚，但邮件此时已经发送出去了，没法回滚除了数据库之外的操作。

2. **中间件拦截。** 这个办法是我最开始采用的，但无奈一直 `Run` 不起来，需要添加上这个中间件 `django.middleware.transaction.TransactionMiddleware` 。我在 django 官方文档中一直没找到这个中间件，估计在 2.0 被丢弃了吧。

3. **手动通过装饰器进行管理。** 这是官方文档中描述最清楚的内容，也是最方便的内容，具体的细节可以直接去看[文档](https://docs.djangoproject.com/zh-hans/2.0/topics/db/transactions/)。因为现在项目还有很多变化的地方，等后续业务逻辑稳定了再使用该方法。

### 原生 SQL 语句的处理
使用这种方法基本上就是存粹的手写 `SQL` 语句了，如果我们需要做更多细致的东西可以直接采用这种做法。通过连接定义一个游标 `cursor`，通过 `cursor` 执行sql语句，最后通过 `transaction.commit_unless_managed()` 来提交事务。

## 其它
需要注意的地方是，使用原生 SQL 处理事物和使用 django ORM 框架进行处理的区别在于，如果视图函数中出现的问题是视图函数本身而不是数据库的问题，那么使用 django ORM 框架也会回滚，而使用原生 SQL 处理却不会。

### 数据库表生成
写完 model 后，需要把对应的 app 放到 setting 配置文件中。


### 重新生成表结构
很多时候当我们在修改 django 的 model 结构时，会因为某些“特殊”情况（搞不懂为啥）没有更新数据库表结构，这个时候可以参考如下做法：
1. 把数据库中对应的表删除；
2. 把 `django_migrations` 表中 `app` 对应字段中的 model 删除；
3. 重新执行 `makemigrations` 和 `migrate` 。

### 让媒体文件（图片）可被访问
按照正常的 django 流程做图片（或其它资源文件）上传即可，如最终生成的 json 格式如下：
```json
{
    avatar = "/media/avatar/pjhubs.jpg";
    "masuser_id" = 7028492784;
}
```

此时在浏览器中直接访问 `http://hostname/media/avatar/pjhubs.jpg` ，是访问不到资源文件的，需要这么做：

1. 在 project app 下（与 settings.py 同级别）的 url.py 文件中添加：
```py
from . import settings
from django.conf.urls.static import static
```
2. 在末尾添加上：
```py
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

可根据自己的要去进行设置，此时就可通过 `http://hostname/media/avatar/pjhubs.jpg` 访问到资源文件了。