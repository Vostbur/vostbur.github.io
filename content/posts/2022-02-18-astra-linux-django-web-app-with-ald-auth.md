---
categories:
- OS
date: "2022-02-18"
description: Запустим в Astra Linux SE 1.5 написанное на Python (Django) web-приложение, возвращающее имя пользователя. Web-сервер - входящий в дистрибутив apache2 с модулем mod_python, авторизация через Astra Linux Directory. Проверим работу мандатного разграничения доступа.
slug: astra-linux-django-web-app-with-ald-auth
tags:
- linux
- AstraLinux
- ald
- django
- apache
title: Web-приложение на Python в Astra Linux с ALD авторизацией
---

Как настроить сервер - контроллер домена ALD и подключиться к нему клиентом в статье [Настройка входа в систему с Astra Linux Directory](https://vostbur.github.io/posts/ald-auth-astra-linux-se-15/).

Устанавливаем web-сервер (если еще не установлен), нужные нам модули apache и пакет с django

    $ sudo apt-get install apache2 libapache2-mod-auth-kerb libapache2-mod-python python-django

Включаем модули apache, с которыми будем работать, отключаем лишний auth_pam, если ранее с ним работали, затем запускаем web-сервер

    $ sudo a2dismod auth_pam
    $ sudo a2enmod python
    $ sudo a2enmod auth_kerb
    $ sudo service apache2 reload

Создаем принципала в ALD и добавляем его в группу `mac`

    $ sudo ald-admin service-add HTTP/ald-server.example.ru
    $ sudo ald-admin sgroup-svc-add HTTP/ald-server.example.ru --sgroup=mac

Создаем файл ключа Kerberos и даем на него права пользователю `www-data`

    $ KEYTAB="/etc/apache2/keytab"
    $ sudo ald-client update-svc-keytab HTTP/ald-server.example.ru --ktfile=$KEYTAB
    $ sudo chown www-data $KEYTAB
    $ sudo chmod 644 $KEYTAB

Блокируем дефолтный виртуальный сайт, конфигурация которого создается по-умолчанию и создаем в `/etc/apache2/sites-available/` свой конфигурационный файл `astra-django-project` для виртуального хоста

    $ sudo a2dissite default
    $ sudo vim /etc/apache2/sites-available/astra-django-project
    
Файл `astra-django-project`
    
    <VirtualHost *:80>
        ServerName   ald-server.example.ru
        ServerAdmin  useradmin@example.ru
        DocumentRoot /var/www/django_site

        AddDefaultCharset utf-8

        <Directory /var/www/django_site>
            Options -Indexes FollowSymLinks -MultiViews
            AllowOverride None

            AuthType Kerberos
            KrbAuthRealms EXAMPLE.RU
            KrbServiceName HTTP/ald-server.example.ru
            Krb5Keytab /etc/apache2/keytab
            KrbMethodNegotiate on
            KrbMethodK5Passwd off
            KrbSaveCredentials on
            require valid-user

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
        LogLevel debug
        SetEnvIf Request_URI "\.jpg$|\.gif$|\.css$|\.js" is_static
        CustomLog /var/www/django_site/log/access.log combined env=!is_static
    </VirtualHost>

Добавляем созданную конфигурацию

    $ sudo a2ensite astra-django-project

Но перезапускать apache еще рано, теперь надо создать наше django-приложение

    $ cd /var/www/
    $ sudo django-admin startproject django_site
    $ cd django_site
    $ sudo python manage.py startapp app

Создаем каталог для log-файлов, назначаем мандатные атрибуты каталогу с виртуальным сервером и перезапускаем web-сервер

    $ cd django_site
    $ sudo pdpl-file 3:0:0xffffffffffffffff:ccnr /var/www/  
    $ sudo pdpl-file 3:0:0xffffffffffffffff:ccnr /var/www/django_site/
    $ sudo service apache2 restart

На любом компьютере, который должен выступать в роли клиента, в firefox включаем negotiate авторизацию. Для чего открыть страницу `about:config` и добавить `http://` в параметр `network.negotiate-auth.trusted-uris`. 

![](/images/astra-linux-firefox-aboute-config.PNG)

Если все сделано правильно, то в браузере клиента, авторизованного через ald при обращении к `http://ald-server.example.ru` покажется стандартная начальная страница django.

Данные пользователя, в отличии от работы с PAM, передаются в зашифрованном виде, на сервере они расшифровываются и становятся доступны из стандартных http-заголовков. Напишем простое приложение, которое показываем имя пользователя, зашедшего на сервер.

В `/var/www/django_site/django_site/settings.py` добавляем наше приложение `app`

    ...
    INSTALLED_APPS = (
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        # Uncomment the next line to enable the admin:
        # 'django.contrib.admin',
        # Uncomment the next line to enable admin documentation:
        # 'django.contrib.admindocs',
        'app',
    )
    ...

В `/var/www/django_site/app/views.py` создаем функцию, которое будет возвращать текущее время и авторизованного пользователя из `request.META['REMOTE_USER']`
    
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s. User: %s </body></html>" % (now, request.META['REMOTE_USER'])
        return HttpResponse(html)

В `/var/www/django_site/django_site/urls.py` создаем маршрут к нашему `view`

    from django.conf.urls import patterns, url

    urlpatterns = patterns('',
        url(r'^$', 'app.views.current_datetime'),
    )

Результат

![](/images/astra-linux-ald-username-django-result.PNG)

**Для проверки работы мандатных ограничений** можно провести эксперимент на конфигурации виртуального сервера `default`. Деактивируем виртуальный сервер с django, добавим проверочную конфигурацию и перезапустим web-сервер

    $ sudo a2dissite astra-django-project
    $ sudo a2ensite default
    $ sudo service apache2 reload

На контроллере домена нужно создать доменного пользователя, и выставить ему ненулевой `"максимальный уровень конфиденциальности"` (например, 1). На компьютере - web-сервере в папке виртуального сервера например `/var/www/` создать 2 файла и установить ненулевую мандатную метку на файл `1.html`

    echo "Hello world! 0" | sudo tee /var/www/0.html  
    echo "Hello world! 1" | sudo tee /var/www/1.html  
    sudo pdpl-file 1:0:0 /var/www/1.html

После чего зайти на клиентский компьютер под пользовательской доменной учетной записью с нулевым уровнем конфиденциальности и попытаться получить доступ к странице с ненулевым уровнем конфиденциальности `http://ald-server.example.ru/1.html`.

Результатом должен стать отказ в доступе. При этом в диагностическом сообщении об ошибке будет указано, что объект не найден (`ошибка 404`), и не будет указана `ошибка 403` (недостаточно прав).  Такая диагностика исключает передачу информации о существовании объектов с высоким уровнем конфиденциальности, перекрывая возможный канал утечки информации.

Доступ к странице с нулевым уровнем конфиденциальности `http://ald-server.example.ru/0.html` должен предоставляться.

При входе на клиентский компьютер под пользовательской доменной учетной записью с ненулевым уровнем конфиденциальности должны быть доступны обе страницы.

----
### Links:

- [1] [Настройка входа в систему с Astra Linux Directory](https://vostbur.github.io/posts/ald-auth-astra-linux-se-15/)