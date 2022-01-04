---
categories:
- Programming
- Tools
date: "2022-01-04"
description: Docker для python. Первые шаги.
slug: docker-for-python-first-steps
tags:
- python
- docker
title: Docker для python. Первые шаги
---

Если не вдаваться в технические детали Docker ближе всего к VirtualBox, VMware или другим средствам виртуализации. Технические отличия заключаются в других способах изоляции запускаемой гостевой (guest) операционной системы и разделения ресурсов основной (host) операционной системы. Как правило каждый работающий в Docker экземпляр гостевой операционной системы предназначен для запуска одного единственного приложения. При этом задействуется меньше ресурсов, чем при запуске в виртуальной машине. Для этого создается своя файловая система, свои виртуальные сетевые интерфейсы - как бы контейнер внутри которого приложение работает. Docker это программное обеспечение для контейнеризации приложений.

Основные понятия, которые надо различать, это образ (image) и контейнер (container). Если совсем просто - образ это готовый для запуска в Docker пакет, контейнер это запущенный в Docker образ. Образ Docker можно представить как iso-образ дистрибутива операционной системы, а контейнер - запущенный экземпляр операционной системы в виртуальной машине. Из одного образа можно запустить несколько контейнеров.

Образ создается из конфигурационного текстового файла. Новый образ можно создать на основе уже существующего. Как правило конфигурационный файл начинается с команды, которая указывает на существующий образ, взятый за основу и дополняется командами для установки дополнительного программного обеспечения или изменения конфигурации.

Для опубликования своих образов существуют репозитории, самый популярный из которых Docker Hub.

Начнем с самого простого примера. Выполним команду

    $ docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    2db29710123e: Pull complete
    Digest: sha256:cc15c5b292d8525effc0f89cb299f1804f3a725c8d05e158653a563f15e4f685
    Status: Downloaded newer image for hello-world:latest

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
    3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

    For more examples and ideas, visit:
    https://docs.docker.com/get-started/

Команда `run` запускает контейнер. Разберем каждую строку

    Unable to find image 'hello-world:latest' locally

Сначала Docker ищет запрашиваемый образ на локальной машине, если не находит - продолжает поиск в репозитории. По умолчанию это Docker Hub. Название образа состоит из двух частей: название и версия. Docker сам добавляет номер версии `latest` к названию, если пользователь не указал что-то другое.

    latest: Pulling from library/hello-world

Docker нашел подходящий образ в репозитории и готов его скачать.

    2db29710123e: Pull complete
    Digest: sha256:cc15c5b292d8525effc0f89cb299f1804f3a725c8d05e158653a563f15e4f685
    Status: Downloaded newer image for hello-world:latest

Образ скачан, в выводе указан результат вычисления хэш-функции, подсчитанный по алгоритму sha256 для проверки образа.
Всё остальное это результат выполнения приложения в запущенном контейнере. Результат работы этого примера сам по себе интересен - в выводе информация об этапах запуска контейнера. Там же еще одна интересная команда

    $ docker run -it ubuntu bash

По этой команде запускается контейнер из образа `ubuntu:latest` и управление передается выполняемой в контейнере командной оболочке bash.

Опции `-i` или `--interactive` и `-t` или `--tty` (или вместе `-it`) позволяют запустить в контейнере интерактивные (взаимодействующие с пользователем) приложение. Они указывают Docker оставить стандартный вход (stdin) в открытом состоянии и выделить псевдо-терминал, который соединяет используемый терминал с потоками stdin и stdout контейнера. Если это сложно, то достаточно запомнить, что опции `-it` позволяют перейти в контейнер и запустить команду, ожидающую ввода пользователя.

Например команда

    $ docker run -it --rm python

Запустит REPL python последней версии. Опция `--rm` автоматически удаляет контейнер после того, как его выполнение завершится.

Теперь создадим свой первый образ для запуска контейнера, который будет выполнять скрипт на python. Скрипт самый простой

    print("Hello, world!")

сохраним его в файл `hello.py`. Конфигурационный файл docker-образа называется `Dockerfile`. Создадим Dockerfile такого содержания:

    FROM python
    COPY hello.py /
    ENTRYPOINT [ "python", "./example.py" ]

Каждая строчка файла начинается с команды. Команды принято записывать заглавными буквами, но это не обязательно. Команды выполняются последовательно, в том порядке, в каком они указаны в Dockerfile.

`FROM python` - указываем, что будем делать свой образ на основе базового образа python последней версии (latest) из репозитория Docker Hub.

`COPY hello.py /` - скопируем наш скрипт `hello.py` из текущего каталога (тот же каталог, где находится и Dockerfile) в корневой каталог файловой системы образа.

`ENTRYPOINT [ "python", "/hello.py" ]` - запустим наш скрипт командой python `/hello.py`

Теперь соберем наш образ командой `build`

    $ docker build -t hello-py:1.0 .

Точка в конце команды указывает, что Dockerfile для сборки образа находится в текущем каталоге.

Опция `-t` (tag) команды `build` позволяет задать имя и версию нашему образу. Если версию не указывать, Docker автоматически добавить latest. Если не указывать имя, к образу можно будет обратиться по значению `IMAGE ID`. Узнать, какие образы у нас уже есть можно командой

    $ docker images

или 

    $ docker image ls

Это идентичные команды, но лучше пользоваться вторым вариантом. Он введен для приведения команд Docker к некоторому унифицированному виду. Все команды, которые управляют образами находятся в разделе `docker image`, контейнерами - `docker container`.

    REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
    <none>       <none>    6af91789bf03   20 minutes ago   917MB
    python       latest    a42e2a4f3833   5 days ago       917MB

Здесь видно, что при создании образа не было указано имя. Контейнер в таком случае можно запустить командой

    $ docker run 6af91789bf03

Если указано имя, команда для запуска контейнера может быть более удобочитаемой

    $ docker image ls
    REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
    hello-py     1.0       6af91789bf03   22 minutes ago   917MB
    python       latest    a42e2a4f3833   5 days ago       917MB

    $ docker run hello-py:1.0

Контейнер мы запустили без опции `--rm`, значит после завершения выполнения, он не удалился. Посмотреть существующие контейнеры можно командой

    $ docker ps -a

или

    $ docker container ls -a

Команды идентичны, как в случает с `docker images` и `docker image ls`. Опция `-a` указывает выводить все контейнеры, запущенные и уже остановленные

    $ docker container ls -a
    CONTAINER ID   IMAGE          COMMAND              CREATED         STATUS                     PORTS     NAMES
    9839a6c57209   hello-py:1.0   "python /hello.py"   5 seconds ago   Exited (0) 4 seconds ago             upbeat_noyce

Каждому контейнеру присваивается случайное имя. В этом случае это `upbeat_noyce`. Но при создании контейнера можно указать и свое значение имени используя опцию `--name`.

    $ docker run --name hello-world hello-py:1.0

в этом случае список контейнеров будет такой

    $ docker container ls -a
    CONTAINER ID   IMAGE          COMMAND              CREATED          STATUS                      PORTS     NAMES
    11794828bf57   hello-py:1.0   "python /hello.py"   29 seconds ago   Exited (0) 29 seconds ago             hello-world
    9839a6c57209   hello-py:1.0   "python /hello.py"   3 minutes ago    Exited (0) 3 minutes ago              upbeat_noyce

Удалить остановленные контейнеры можно командой `docker container rm <имя_контейнера>`

    $ docker container rm hello-world

Удалить локальные образы можно командой `docker image rm <имя образа>`

    $ docker image rm hello-py:1.0

Усложним наш скрипт. Запустим в контейнере web-приложение выводящее на главную страницу `Hello, world!` В качестве сервера будем использовать Flask.

Наш новый файл `hello.py`

    from flask import Flask

    app = Flask(__name__)


    @app.route("/")
    def index():
        return "Hello, world!"


    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000)

Изменим Dockerfile, добавим только одну команду, с помощью которой установим flask.

    FROM python
    RUN python -m pip install Flask
    COPY hello.py /
    ENTRYPOINT ["python", "/hello.py"]

Немного о порядке команд в Dockerfile. Docker выполняет команды по очереди, но, если команда не меняет состояние образа, она пропускается. Выполняется первая и последующие команды, которые приведут образ в состояние отличное от прошлой сборки. Для этого в результате выполнения каждой команды создается новый слой (layer), для которого вычисляется значение хэш-функции. Это значение будет сравниваться с новым значением при каждой последующей сборке, и, если хэши совпадают, текущий шаг сборки будет пропущен. Поэтому имеет смысл в каждом слое объединять как можно больше команд, которые будут редко меняться. Например установку дополнительных пакетов лучше объединить в одну команду `RUN`. А те команды, которые будут приводить к изменению образа, лучше переносить в конец Dockerfile насколько это возможно. В нашем случае предполагается, что мы можем часто менять файл `hello.py`, поэтому копирование нашего скрипта в контейнер выполняется после остальных команд, непосредственно перед запуском.

Если интересует, почему использую команду `python -m pip install Flask` вместо `pip install Flask` советую прочитать [Зачем использовать python -m pip](https://habr.com/ru/company/otus/blog/475392/).

Соберем новый образ

    $ docker build -t hello-py .

Запустим контейнер с именем `hello-flask` из образа `hello-py:latest`

    $ docker run -d -p 80:5000 --name hello-flask hello-py

Опция `-d` (или `--detach`) запускает контейнер в фоновом режиме. Это позволяет использовать терминал, из которого запущен контейнер, для выполнения других команд во время работы контейнера.

Опция `-p` (или `--port`) открывает порт контейнера для взаимодействия с внешним миром. В нашем случае запросы на порт с номером 80 (дефолтный порт для http-сервера) на локальном компьютере будут перенаправлены на порт 5000 запущенного контейнера. 

Теперь мы можем открыть в браузере адрес http://localhost и увидеть наше приветствие `Hello, world!`

Убедимся, что наш контейнер продолжает работать

    $ docker container ps
    CONTAINER ID   IMAGE      COMMAND              CREATED         STATUS         PORTS                  NAMES
    3371c56c3c3a   hello-py   "python /hello.py"   3 minutes ago   Up 3 minutes   0.0.0.0:80->5000/tcp   hello-flask

В колонке `STATUS` видим, что контейнер запущен, а в колонке `PORTS` запись о перенаправлении запросов на порт с номером 5000.
Прежде чем удалить работающий контейнер, его надо остановить

    $ docker container stop hello-flask

Бывает, что команда `stop` не срабатывает, тогда можно воспользоваться командой `kill`

    $ docker container kill hello-flask

Теперь можно контейнер удалять.

Продолжим изменять Dockerfile

    FROM python
    WORKDIR /app
    RUN python -m pip install Flask
    COPY hello.py .
    CMD ["python", "./hello.py"]

Команда `WORKDIR` устанавливает рабочий каталог. Если такого каталога нет, он создается. Теперь команда `COPY` копирует скрипт в текущий каталог, которым стал `/app` и скрипт выполняет по относительному пути `./hello.py`.

Вместо команды `ENTRYPOINT` мы используем команду `CMD`. Разница в том, что параметры команды `CMD` можно подменить непосредственно из команды запуска контейнера. Например, следующая команда запустит наш контейнер, но http-сервер в нем запущен не будет, а вместо него выведется информация об установленных пакетах python

    $ docker run -it --rm --name hello-flask hello-py pip freeze
    click==8.0.3
    Flask==2.0.2
    itsdangerous==2.0.1
    Jinja2==3.0.3
    MarkupSafe==2.0.1
    Werkzeug==2.0.2

При отладке удобно использовать команду `CMD`, но в конечном Dockerfile лучше использовать `ENTRYPOINT`, чтобы никто не смог сломать наш контейнер, передав из командной строки на выполнение незапланированную команду.

Давайте создадим файл `requirement.txt` с нашими зависимостями. Пока нам нужен только Flask

    Flask==2.0.2

Изменим Dockerfile

    FROM python
    WORKDIR /app
    COPY requirement.txt .
    RUN python -m pip install -r requirement.txt
    COPY . .
    ENTRYPOINT ["python"]
    CMD ["./hello.py"]

Файл `requirement.txt` с перечисленными зависимостями будет меняться не так часто, как скрипт, поэтому его копирование вынесли в отдельную команду. Так же я разделил запуск скрипта на две части. Позже я покажу зачем, а пока проверим, что все по-прежнему работает.

Собираем образ

    $ docker build -t hello-py .

Запускаем контейнер

    $ docker run -d --rm -p 80:5000 --name hello-flask hello-py

Проверяем результат в браузере. Если все нормально контейнер можно остановить

    $ docker container stop hello-flask

У нас указана опция `--rm`, поэтому после остановки контейнер сам удалится.

Создадим новый файл `hello-cmd.py`

    print("Hello from command line")

Пересоберем образ. На этот раз образ собрался намного быстрее, чем в прошлый. Это связано с тем, что изменился только один слой, в котором мы выполняем операцию копирования всех файлов нашего каталога в текущий каталог контейнера `COPY . .`

Запустим контейнер, но в качестве дополнительного параметра укажем название нашего нового скрипта. Этот параметр заменит тот, что указан в команде `CMD`.

    $ docker run --rm --name hello-flask hello-py hello-cmd.py
    Hello from command line

Проверим, что контейнер с web-сервером так же запускается.

Таким образом, можно писать новые программы и передавать их как параметры для запуска нашему контейнеру. Только для этого придется каждый раз пересобирать образ, иначе новые программы не попадут в рабочий каталог контейнера. Это удобно, когда мы хотим распространять образ с готовой программной для запуска на других компьютерах. Но для разработки или тестирования наших программ это решение не удобно. Для этого можно смонтировать локальный каталог в качестве тома (volume) контейнера Docker. Наш локальный каталог будет доступен в файловой системе контейнера.

Удалим команду `COPY` из Dockerfile

    FROM python
    WORKDIR /app
    COPY requirement.txt .
    RUN python -m pip install -r requirement.txt
    ENTRYPOINT ["python"]
    CMD ["./hello.py"]

Пересоберем образ

    $ docker build -t hello-volume .

Запустим контейнер с опцией `-v`. Параметром этой опции служит полный путь к каталогу, который мы ходим сделать доступным из контейнера и после `:` каталог в файловой системе контейнера, обращаясь к которому мы будем получать доступ к нашему локальному каталогу. В моем случае это `/home/alex/projects/docker-projects/starting-docker:/app` у вас путь до локального каталога, где лежат наши программы скорее всего другой.

    $ docker run -d --rm -p 80:5000 -v /home/alex/projects/docker-projects/starting-docker:/app --name hello-flask hello-volume

Запустим еще один контейнер, но передадим для выполнения скрипт `hello-cmd.py`

    $ docker run -v /home/alex/projects/docker-projects/starting-docker:/app --name hello-cmd hello-volume hello-cmd.py
    Hello from command line

Проверим что наши контейнеры существуют

    $ docker container ls -a
    CONTAINER ID   IMAGE          COMMAND                 CREATED              STATUS                     PORTS                  NAMES
    de7a891e4e81   hello-volume   "python hello-cmd.py"   5 seconds ago        Exited (0) 4 seconds ago                          hello-cmd
    c012bc603b88   hello-volume   "python ./hello.py"     About a minute ago   Up About a minute          0.0.0.0:80->5000/tcp   hello-flask

Напишем еще одну программу `new-hello.py`

    for i in range(10):
        print("Hello!", end=" === ")

И запустим ее в третьем контейнере 

    $ docker run -v $(pwd):/app --name new-hello hello-volume
    new-hello.py
    Hello! === Hello! === Hello! === Hello! === Hello! === Hello! === Hello! === Hello! === Hello! === Hello! ===

Я подключаю к контейнеру текущий каталог, поэтому вместо ввода полного пути я воспользовался командой подстановка `$(pwd)`. Эта запись означает, что результат выполнения команды `pwd` - вывод полного пути к текущему каталогу, подставится в строку нашей команды на запуск контейнера.  

Еще раз убедимся, что все три контейнера существуют

    $ docker container ls -a
    CONTAINER ID   IMAGE          COMMAND                 CREATED          STATUS                      PORTS                  NAMES
    c4c103819fb0   hello-volume   "python new-hello.py"   16 seconds ago   Exited (0) 15 seconds ago                          new-hello
    de7a891e4e81   hello-volume   "python hello-cmd.py"   12 minutes ago   Exited (0) 12 minutes ago                          hello-cmd
    c012bc603b88   hello-volume   "python ./hello.py"     14 minutes ago   Up 14 minutes               0.0.0.0:80->5000/tcp   hello-flask

Как видите, на этот раз пересобирать образ не пришлось.

В заключении расскажу как использовать в docker-контейнере виртуальное окружение python. Хотя Docker сам по себе изолирует среду выполнения иногда возникает необходимость воспользоваться модулем `python venv`. Изменим наш Dockerfile

    FROM python
    WORKDIR /app
    RUN python -m venv /venv
    ENV PATH="/venv/bin:$PATH"
    COPY requirement.txt .
    RUN python -m pip install -r requirement.txt
    ENTRYPOINT ["python"]
    CMD ["./hello.py"]

В добавленных строках мы создаем в каталоге `/venv` виртуальное окружение и добавляем путь к каталогу, откуда будет запускаться python в самое начало значения переменной окружения `PATH`.

Для проверки того, что наша программа действительно выполнится в виртуальном окружении, напишем небольшой скрипт `check-venv.py`

    import sys
    print(sys.prefix)
    print(sys.executable)

Соберем образ и запустим контейнер

    $docker build -t hello-volume .
    $docker run --rm -v $(pwd):/app --name venv hello-volume check-venv.py
    /venv
    /venv/bin/python

Всё работает.