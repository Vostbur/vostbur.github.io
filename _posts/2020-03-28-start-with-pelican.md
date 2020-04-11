---
layout: post
title: Блог на GitHub Pages с Pelican
---

1. [Создаем](https://github.com/new) два новых репозитория *username.github.io-src* и *username.github.io*.
Первый для исходников, второй - сам сайт. Подробности [здесь](https://pages.github.com/).

2. Делаем клон репозитория для исходников, подключаем репозиторий с сайтом как сабмодуль.  
  `git clone https://github.com/username/username.github.io-src.git`  
  `cd username.github.io-src`  
  проверям, что склонировали правильно  
  `git remote -v`  
  `git submodule add https://github.com/username/username.github.io.git output`  
  
3. Устанавливаем pelican, markdown  
  `pip install pelican, markdown`  
  
4. Выполняем настройку:  
  `pelican-quickstart`  
  
    Ответы на основные вопросы, остальные не важны:  
    - Где вы хотите создать свой новый веб–сайт? **(Enter)**  
    - URL префикс:  **http://username.github.io**  
    - Сформировать Fabfile/Makefile: **Да**  
    - Авто-обновление и сценарий simpleHTTP: **Да**  
    - Загрузить механизмы: **Нет для всех, кроме GitHub Pages**  
    - Это ваша персональная страница (username.github.io)? **да**  
  
    Сообщение об ошибке (каталог output уже существует) игнорируйте.  
    В файле *publishconf.py* нужно установить переменную *DELETE_OUTPUT_DIRECTORY* в значение *False*.
    Иначе, после публикации, Pelican удалит output с сабмодулем.  
  
5. Создаем первую запись, пример из [документации](https://docs.getpelican.com/en/stable/quickstart.html).
Генерируем html-страницу и проверяем в локальном web-сервере на http://localhost:8000/  
  `pelican content`  
  `pelican --listen`  
  
6. Выкладывем на GitHub Pages:  
  `cd output`  
  `git add .`  
  `git commit -m "message"`  
  `git push -u origin master`  
  `cd ..`  
  `git add .`  
  `git commit -m "message"`  
  `git push -u origin master`  

**Полезные ссылки:**  

1. [Темы](http://pelicanthemes.com/). Подключение:
    - в username.github.io-src создаем каталог theme куда клонируем выбранную тему.
    - в pelicanconf.py прописываем переменную THEME = 'theme/blue-penguin' где blue-penguin каталог с темой.
2. [Плагины](https://github.com/getpelican/pelican-plugins).
3. [Советы](https://github.com/getpelican/pelican/wiki/Tips-n-Tricks).