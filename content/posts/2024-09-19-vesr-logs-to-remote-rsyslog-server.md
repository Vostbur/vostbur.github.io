---
categories:
- Tools
date: "2024-09-19"
slug: vesr-logs-to-remote-rsyslog-server
tags:
- Eltex
- vESR
title: Логирование vESR на удаленный сервер rsyslog
---

Настраиваем сервер rsyslog на прием логов по сети. На примере Ubuntu.

Редактируем `/etc/rsyslog.conf`:

Раскомментируем строки чтобы получать логи по TCP (для UDP - похожие строки, только модуль "imudp")

```
module(load="imtcp")
input(type="imtcp" port="514")
```

в конце файла добавить

```
global(
  parser.escapecontrolcharactertab="off"
)
```

иначе будет ругаться на какие-то символы табуляции в логах с vesr.
Для сохранения логов с разных ip в отдельные каталоги добавить строки

```
$template FILENAME,"/var/log/vesr/%fromhost-ip%/syslog.log"
*.* ?FILENAME
```

перезапускаем службу

```
sudo service rsyslog restart
```

Логи с vesr `192.168.0.99` будут складываться в `/var/log/vesr/192.168.0.99/syslog.log`

Теперь на vesr настраиваем отправку логов по адресу сервера, используя указанный в конфигурации сервера протокол и номер порта.

Я для примера включил логирование вводимых команд и поставил уровень сохраняемых сообщений - warning. Все эти парараметы, начинаая от названия хоста ubuntu, по которому логер vesr будет отличать одну секцию настройки от других, задаются исходя из своих данных

```
vesr(config)# syslog cli-commands
vesr(config)# syslog host ubuntu
vesr(config-syslog-host)# remote-address 192.168.0.104
vesr(config-syslog-host)# transport tcp
vesr(config-syslog-host)# port 514
vesr(config-syslog-host)# severity warning
vesr(config-syslog-host)# end
vesr# commit
vesr# confirm
```

Посмотреть настройки

```
vesr# show syslog configuration
```

![](/images/vesr-syslog.png)
