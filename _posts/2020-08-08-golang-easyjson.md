---
layout: post
title: 8 августа 2020 г.
---

Пакет Go [easyjson](https://github.com/mailru/easyjson) от [Mail.Ru](https://github.com/mailru) предоставляет быстрый и простой способ маршалировать/демаршалировать структуры Go в/из JSON основанный на рефлексии. В тестах производительности easyjson превосходит стандартный encoding/json пакет в несколько раз (сравнительный benchmark в конце).

### Установка и использование (OS Windows)

1. Должна быть установлена переменная окружения *GOPATH*. У меня:

    `echo %GOPATH%`

    `d:\projects-go\`

    И должен быть установлен [git](https://git-scm.com/download/win).

2. Cтавим easyjson:

    `go get -u github.com/mailru/easyjson/...`

    в *%GOPATH%\bin* появится *easyjson.exe*

3. В каталоге нашего проекта создаем подкаталог, он же пакет go, отличный от main (это обязательно), например
    `%GOPATH%\project\model`

    и создаем файл *model.go* с нашей моделью, например
    ```
    package model

    type User struct {
        Email    string   `json:"email"`
    }
    ```
4. Генерируем код для *easyjson*
   `%GOPATH%\project\model>%GOPATH%\bin\easyjson.exe -all model.go`
   в итоге в пакете *model* появляется файл с программным кодом *model_easyjson.go*

5. Импортируем пакет с моделью и сгенерированным кодом и   пользуемся
    ```
    import (
        ...
        "project/model"
        ...
    )

    ...
            user := &model.User{}
            err := user.UnmarshalJSON([]byte(line))
    ...
    ```

### Benchmark
Сравнение со стандартным *encoding/json* на реальном проекте
`%GOPATH%\project> go test -bench . -benchmem`
```
BenchmarkJSON-8                      247           4808229 ns/op         1577438 B/op     10546 allocs/op
BenchmarkEASYJSON-8              550           2148149 ns/op         1385620 B/op      8226 allocs/op
```
