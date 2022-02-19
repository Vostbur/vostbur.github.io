---
categories:
- OS
date: "2022-02-19"
description: Подключим СУБД PostgreSQL к Django в Astra Linux SE 1.5. Для написания многопользовательских web-приложений настроим работу с переменной окружения REMOTE_USER, через которую веб-сервер передает в веб-приложение идентификатор аутентифицированного пользователя.
slug: astra-linux-django-web-app-with-postgresql
tags:
- linux
- AstraLinux
- postresql
- django
title: Django с PostgreSQL в Astra Linux SE 1.5
---

Устанавливаем необходимые пакеты с зависимостями: postgresql, python-psycopg2. В ALSE 1.5 основная версия PostgreSQL 9.4.5.

Редактируем конфигурационный файл базы данных

    $ sudo vim /etc/postgresql/9.4/main/pg_hba.conf

Комментируем или удаляем все строки и дописываем новые, разделяя поля табуляцией 

    local               all                    postgres                    trust
    local               all                    all                   trust
    host                all                    all                   127.0.0.1/32  trust
    host                all                    all                   192.168.15.129/24  trust
    host                all                    all                   192.168.15.132/24  trust

Последние строки - адреса сервера и клиента, которым будем подключаться. Меняйте на свои или укажите всю сеть, например `192.168.15.0/24`.

Перезапускаем СУБД

    $ sudo pg_ctlcluster 9.4 main restart

Создаем базу, которую будем использовать в Django

    $ sudo su
    # su postgres -c psql postgres

    > ALTER USER postgres WITH PASSWORD 'postres';
    > CREATE DATABASE dbname;
    > \q

Редактируем раздел `settings.py` для настройки подключения к базе postgresql

    ...
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': 'dbname',
            'USER': 'postgres',
            'PASSWORD': 'postgres',
            'HOST': '',
            'PORT': '',
        }
    }
    ...

Создаем таблицы в базе командой `python manage.py syncdb`

Для проверки раскомментируем приложение админки в `settings.py` и маршрут к ней в `urls.py`. Админка должна заработать.

Теперь можно настроить django-приложение на работу с переменной окружения `REMOTE_USER`, через которую Apache передает в веб-приложение идентификатор аутентифицированного пользователя.

Редактируем `settings.py`

    ...
    MIDDLEWARE_CLASSES = (
        '...',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.auth.middleware.RemoteUserMiddleware',
        '...',
    )

    AUTHENTICATION_BACKENDS = (
        'django.contrib.auth.backends.RemoteUserBackend',
    )
    ...

Создаем django-приложение `app` командой `manage.py startapp app` и добавляем его в `settings.py`

    ...
    INSTALLED_APPS = (
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'django.contrib.admin',
        # Uncomment the next line to enable admin documentation:
        # 'django.contrib.admindocs',
        'app',
    )
    ...

В `app/views.py` создаем функцию, которое будет возвращать текущее время и имя авторизованного пользователя
    
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s. User: %s </body></html>" % (now, request.user.username)
        return HttpResponse(html)

В `django_site/urls.py` создаем маршрут к нашему `view`

    from django.conf.urls import patterns, url
    from django.contrib import admin
    
    urlpatterns = patterns('',
        url(r'^$', 'app.views.current_datetime'),
        url(r'^admin/', include(admin.site.urls)),
    )

Теперь при посещении главной страницы авторизированным в ALD-домене пользователем мы увидим его имя. Можно приступать к написанию многопользовательских django-приложений.

*PS К сожаланию, после включения авторизации по REMOTE_USER, не получается зайти в админку. Пока еще не разобрался почему.*