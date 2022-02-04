---
categories:
- OS
date: "2022-02-04"
description: Запуск Django-приложений через mod_python в Astra Linux SE 1.5
slug: django-starting-astra-linux-se-15
tags:
- linux
- web-server
- apache2
- AstraLinux
- django
title: Запуск Django-приложений через mod_python в Astra Linux SE 1.5
---

Считаем, что Apache уже установлен и настроена аутентификация и авторизация через PAM (см. [Настройка web-сервера на Astra Linux SE 1.5](https://vostbur.github.io/posts/apache2-astra-linux-se-15/))

Устанавливаем необходимые пакеты:

    # apt-get install libapache2-mod-python python-django

Включаем модуль:

    # a2enmod python

Создаем проект Django и каталог для log-файлов:

    # cd /var/www
    # django-admin startproject django_site
    # mkdir django_site/log

В `/etc/apache2/sites-available/` создаем файл `django_site`. Это файл конфигурации виртуального хоста для нашего Django-проекта, лежащего в каталоге `/var/www/django_site/`.

    <VirtualHost *:80>
        ServerAdmin webmaster@domain.name
        ServerName server.domain.name

        DocumentRoot /var/www/django_site
        <Directory "/var/www/django_site/">
            AuthPAM_Enabled on
            AuthType Basic
            AuthName "PAM authentication"
            require valid-user

            Options -Indexes FollowSymLinks -MultiViews
            AllowOverride None

            Order deny,allow
            Allow from all
        </Directory>

        <Location "/">
            SetHandler    python-program
            PythonHandler django.core.handlers.modpython
            SetEnv        DJANGO_SETTINGS_MODULE django_site.settings
            PythonOption  diango.root /var/www/django_site
            PythonPath    "['/var/www/django_site/',] + sys.path"
            PythonAutoReload On
        </Location>

        <Location "/media/”>
            SetHandler None
        </Location>

        <Location "/static/">
            SetHandler None
        </Location>

        <LocationMatch "\.(jpg|gif|png)$">
            SetHandler None
        </LocationMatch>

        ErrorLog /var/www/django_site/log/error.log
        LogLevel warn
        CustomLog /var/www/django_site/log/access.log combined
    </VirtualHost>


Включаем наш сайт в список разрешенных и перезапускаем web-сервер
    
    # a2ensite astra-django-project
    # service apache2 reload

После ввода имени пользователя и пароля в браузере должна показаться стандартная начальная страница Django:

![](/images/django-start-page-astralinux.png)

----
### Links:

- [1] [Настройка web-сервера на Astra Linux SE 1.5](https://vostbur.github.io/posts/apache2-astra-linux-se-15/)