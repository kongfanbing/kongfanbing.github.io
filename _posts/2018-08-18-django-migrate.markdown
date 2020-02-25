---
layout: post
title:  Django之migrate记录
date:   2018-08-18
tags: [Django,Python]
---

>   不积跬步，无以至千里；不积小流，无以成江海。
>
>   ——荀子

# 前言

最近项目中使用到了Django，发现它的Model很好用，再加上migrate，配置好后，可以自动建表。由于项目是多人开发，且业务模块是不同的，所以存在多个数据库的情况，经过一番折腾终于配置好了支持多个APP和多个数据库。

# 多库支持

除了需要设置DATABASES中的数据库配置和DATABASES_APPS_MAPPING外，多个数据库还需要在settings.py文件中设置DATABASE_ROUTERS的值：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'default',
        'USER': 'user',
        'PASSWORD': 'passwd',
        'HOST': 'localhost',
        'PORT': '3306',
    },
    'myapp': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myapp',
        'USER': 'user',
        'PASSWORD': 'passwd',
        'HOST': 'localhost',
        'PORT': '3306',
    },
}

DATABASES_APPS_MAPPING = {
    'default': 'default',
    'myapp': 'myapp',
}
DATABASE_ROUTERS = ['project.database_app_router.DatabaseAppsRouter']
```

在project目录下创建database_app_router.py文件，代码如下：

```python
from django.conf import settings

class DatabaseAppsRouter(object):
    def db_for_read(self, model, **hints):
        app_label = model._meta.app_label
        if app_label in settings.DATABASES_APPS_MAPPING:
            res = settings.DATABASES_APPS_MAPPING[app_label]
            return res
        return None

    def db_for_write(self, model, **hints):
        app_label = model._meta.app_label
        if app_label in settings.DATABASES_APPS_MAPPING:
            return settings.DATABASES_APPS_MAPPING[app_label]
        return None

    def allow_relation(self, obj1, obj2, **hints):
        # 获取对应数据库的名字
        db_obj1 = settings.DATABASES_APPS_MAPPING.get(obj1._meta.app_label)
        db_obj2 = settings.DATABASES_APPS_MAPPING.get(obj2._meta.app_label)
        if db_obj1 and db_obj2:
            if db_obj1 == db_obj2:
                return True
            else:
                return False
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if db in settings.DATABASES_APPS_MAPPING.values():
            return settings.DATABASES_APPS_MAPPING.get(app_label) == db
        elif app_label in settings.DATABASES_APPS_MAPPING:
            return False
        return None

```



# 遇到的问题

正常的创建好model后，执行mirate即可创建好数据表。

```shell
python manage.py makemigrations myapp
python manage.py migrate myapp
```

不过多个数据库就会出现问题，一切都显示正常，但数据表并没有创建成功。需要添加--database参数，因为默认migrate执行时只会操作default数据库，没有使用default数据库的APP的更新将会被忽略。

```shell
python manage.py migrate --database=myapp myapp
#--database 的说明：
#Nominates a database to synchronize. Defaults to the "default" database.
```



# 结语

Django这个框架还是很不错的，其支持的功能也很丰富，但是网上的文档说明和教程等都是基于比较单个应用和数据库的，很少涉及多个数据库的情况，还是需要自己去探索。