---
layout: post
title: Логгирование команд на Cisco с помощью Event Manager
---

Настроить Event Manager для логгирования всех выполненных на роутере команд:  
```
event manager applet CLIaccounting  
event cli pattern ".*" sync no skip no  
action 1.0 syslog priority informational msg "$_cli_msg"  
set 2.0 _exit_status 1  
```

Включить логгирование:  
```
archive  
log config  
 logging enable  
 logging size 1000  
 notify syslog  
 hidekeys  
```

По умолчанию __logging size 100__ если этого достаточно, команду можно не вводить. Посмотреть лог:  
`show archive log config all`  
  
![](/images/sh-archive-log-config.jpg)

Все выполненные команды будут дублироваться на консоль. Если это не надо, отключить вывод:  
`no logging console`  
Включить:  
`logging console`  
  
Дальше можно настроить syslog, отправку лога на ftp и т.д.  
