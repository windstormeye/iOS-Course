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

### 创建虚拟环境
```py
$ cd env
$ source ./bin/activate
```