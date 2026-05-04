---
title: django
tags:
  - 'Django'
categories:
  - 'Python'
date: 2025-11-19 00:35:26
---

## 笔记

### Django 常用配置

1. 如果想用apps目录管理所以的app，需要在把之前所以用到app地方改为apps.app，注意是所有，着重检查`settings.py`、`app.py`、`views.py`、 `signals.py`、`forms.py`、`models.py` 这些文件。

2. 静态资源文件，比如图片、css、js等，需要放在`static`目录下，然后在`settings.py`中设置`STATIC_URL`和`STATICFILES_DIRS`。当app目录里没有static目录时，会从`STATICFILES_DIRS`中依次查找（从第一个开始）
静态资源配置还有STATIC_ROOT，其作用是在服务器上部署项目，实现服务器和项目之间的映射。
STATIC_ROOT 主要收集整个项目的静态资源并存放在一个新的文件夹，然后由该文件夹与服务器之间
构建映射关系。
STATIC_ROOT配置如下:
STATIC_ROOT = BASE_DIR / 'static'
当项目的配置属性 DEBUG 设为True的时候，Django 会自动提供静态文件代理服务，此时整个项目处于
开发阶段，因此无须使用STATIC_ROOT。当配置属性DEBUG 设为False的时候，意味着项目进入生产环
境，Django不再提供静态文件代理服务，此时需要在项目的配置文件中设置STATIC_ROOT。
设置STATIC_ROOT需要使用 Django操作指令collectstatic来收集所有静态资源，这些静态资源都会保存
在STATIC_ROOT所设置的文件夹里。

3. 媒体文件，比如上传的图片、文件等，需要放在`media`目录下，然后在`settings.py`中设置`MEDIA_URL`和`MEDIA_ROOT`。

4. 时区配置:

``` python
LANGUAGE_CODE = "zh-Hans"

TIME_ZONE = "Asia/Shanghai"
```

### 其他常用命令

* 常用的数据库配置:

``` python
MY_SQL_DATABASE = {
    "ENGINE": "django.db.backends.mysql",
    "NAME": "django_web",
    "USER": "root",
    "PASSWORD": "quhou",
    "HOST": "localhost",
    "PORT": "3306",
}

SQ_LITE_DATABASE = {
    "ENGINE": "django.db.backends.sqlite3",
    "NAME": BASE_DIR / "db.sqlite3",
}

DATABASES = {
    "default": MY_SQL_DATABASE
}
```

## Hello, Django

### 安装Django

``` bash
pip install django
```

### 创建项目

``` bash
django-admin startproject mysite
```

### 创建应用

``` bash
python manage.py startapp polls
```

### 运行项目

``` bash
python manage.py runserver
```

## 路由

``` python
from django.contrib import admin
from django.views.static import serve
from django.urls import path, re_path
from django_web import settings
from django.views.generic import RedirectView

from apps.helloWorld import views

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", views.index),
    path("redirect/", RedirectView.as_view(url="/blog/1")),
    path("blog/<int:id>", views.blog),
    # 媒体
    re_path(
        r"^media/(?P<path>.*)$",
        serve,
        {"document_root": settings.MEDIA_ROOT},
        name="media",
    ),
]

```

## 表单

### 创建一个继承表单

``` python
class UserRegisterCreationForm(UserCreationForm):
    # 自定义字段
    nickname = forms.CharField(
        max_length=100, 
        required=False,
        label='昵称',
        help_text='请输入您的昵称（可选）',
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': '请输入您的昵称'
        })
    )
    signature = forms.CharField(
        widget=forms.Textarea(attrs={
            'rows': 3,
            'class': 'form-control',
            'placeholder': '请输入您的个人签名'
        }),
        required=False,
        label='签名',
        help_text='请输入您的个人签名（可选）'
    )
    # Meta类中指定模型类和字段，labels和widgets是表单字段的显示信息只能写User存在的字段，不能写扩展字段
    # fields可以写扩展字段也可以不写，如果自己在html里form表单的话就不用写，如果用django的默认form表单的话就要写
    # fields的作用是指定表单字段的顺序，如果不写，默认是模型类中定义的字段顺序
    # 使用 form.as_p() 或者 form.as_table() 输出表单
    class Meta:
        model = User
        fields = ('username', 'password1', 'password2')
        labels = {
            'username': '用户名',
            'password1': '密码',
            'password2': '确认密码',
        }
        widgets = {
            'username': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': '请输入用户名'
            }),
            'password1': forms.PasswordInput(attrs={
                'class': 'form-control',
                'placeholder': '请输入密码'
            }),
            'password2': forms.PasswordInput(attrs={
                'class': 'form-control',
                'placeholder': '请再次输入密码'
            }),
        }
    # 重写save方法，保存用户信息和用户信息扩展信息
    def save(self, commit=True):
        user = super().save(commit=False)
        if commit:
            user.save()
        
        user_profile, created = UserProfile.objects.get_or_create(
            user=user,
            defaults={
                'nickname': self.cleaned_data['nickname'],
                'signature': self.cleaned_data['signature']
            }
        )
        
        if not created:
            user_profile.nickname = self.cleaned_data['nickname']
            user_profile.signature = self.cleaned_data['signature']
            user_profile.save()
        
        return user_profile

```

### 使用继承表单

``` python
def register_view(request):
    # 当前端有提交数据时，使用表单验证并把POST数据传入表单
    # 表单验证通过后，调用表单的save方法保存用户信息和用户信息扩展信息
    # 保存成功后，跳转到用户信息页面
    if request.method == 'POST':
        form = UserRegisterCreationForm(request.POST)
        if form.is_valid():
            user_profile = form.save()
            login(request, user_profile.user)
            return redirect('profile')
    else:
        # 初始页面，显示空表单，用户输入信息
        form = UserRegisterCreationForm()
    return render(request, 'user/register.html', {'form': form})
```

## 模型

### 自定义模型

``` python
from django.contrib.auth.models import User
from django.db import models

class UserProfile(models.Model):
    # 这句可以让UserProfile成为User的外键,从而实现一对一关系,使用request.user.userprofile可以获取到UserProfile对象
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    nickname = models.CharField(max_length=100, blank=True, null=True)
    signature = models.TextField(blank=True, null=True)

    def __str__(self):
        return self.user.username
```

所以个人信息页可以这样写:

``` python

# 只有登录用户才能访问个人信息页
@login_required
def profile_view(request):
    try:
        # 因为UserProfile是User的外键，所以可以通过request.user获取到UserProfile对象
        # 调用内置的login成功登录后，request.user会自动绑定到User对象上
        user_profile = request.user.userprofile
    except UserProfile.DoesNotExist:
        # 如果UserProfile不存在，则创建一个UserProfile对象，并与User关联
        user_profile = UserProfile.objects.create(user=request.user)
    if request.method == 'POST':
        form = UserProfileForm(request.POST, instance=user_profile)
        if form.is_valid():
            form.save()
            return redirect('profile')
    else:
        form = UserProfileForm(instance=user_profile)
    return render(request, 'user/profile.html', {
        'user': request.user,
        'user_profile': user_profile,
        'form': form
    })

```
