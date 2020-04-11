---
layout: post
title: Начальное конфигурирование роутера Cisco (использую в GNS3)
---

Если осталась старая конфигурация, удалить и перезапустить  
`Cisco>enable`  
`Cisco#erase startup-config`  
`Cisco#reload`  
`Cisco>enable`  
  
Новая конфигурация  
`Cisco#configure terminal`  
  
Назвать роутер  
`Cisco(config)#hostname R1`  
  
Хранить пароли в файле конфигурации в зашифрованном виде  
`R1(config)#service password-encryption`  
  
Отключить управление по http, https, CDP  
`R1(config)#no ip http server`  
`R1(config)#no ip http secure-server`  
`R1(config)#no cdp run`  
  
Отключить интерпретацию неправильно введенных команд как DNS-запрос  
`R1(config)#no ip domain lookup`  

Пароль на подключение по консольному порту  
`R1(config)#line console 0`  
`R1(config-line)#password cisco`  
`R1(config-line)#login local`  
`R1(config-line)#exit`  
  
Сколько доступно виртуальных терминалов?  
`R1(config)#line vty ?`  
`<0-903>  First Line number`  
  
Пароль на подключение по telnet  
`R1(config)#line vty 0 903`  
`R1(config-line)#password cisco`  
`R1(config-line)#login local`  
`R1(config-line)#exit`  
  
Пароль на вход в режим enable  
`R1(config)#enable secret cisco`  
  
Настроить интерфейс внутренней сети  
`R1(config)#interface FastEthernet 0/0`  
`R1(config-if)#ip address 192.168.2.2 255.255.255.0`  
`R1(config-if)#description LAN`  
`R1(config-if)#no shutdown`  
`R1(config-if)#exit`  
  
DNS-сервер  
`R1(config)#ip name-server 192.168.2.1`  
  
Создать пользователя с максимальными привилегиями (если надо, чтобы запрашивался пароль на enable, вместо 15 ставить 1)   
`R1(config)#username cisco privilege 15 secret cisco`  
  
Теперь роутер доступен по telnet, если это достаточно, перейти к сохранению конфигурации  
  
__Настроить доступ по SSH__  
    
Проверить и установить точное время  
`R1(config)#do sh clock`  
`*00:19:58.371 UTC Fri Mar 1 2002`  
`R1(config)#do clock set 10:41:00 04 Apr 2020`  
  
Указать имя домена (любое)  
`R1(config)#ip domain name site.info`  
  
Генерировать ключ RSA, длину ключа ставить __не меньше 1024__  
`R1(config)#crypto key generate rsa`  
  
Включить SSH v.2 (с версии IOS 12.1(19)E)  
`R1(config)#ip ssh version 2`  
  
Включить доступ по SSH на всех виртуальных терминалах  
`R1(config)#line vty 0 903`  
`R1(config-line)#transport input ssh`  
Если нужен и telnet  
`R1(config-line)#transport input ssh telnet`  
Или  
`R1(config-line)#transport input all`   
  
Режим вывод сообщений в консоль, не мешающий вводу команд  
`R1(config-line)#logging synchronous`  
  
Тайм-аут до автоматического закрытия SSH-сессии, если надо отключить `exec-timeout 0 0`  
`R1(config-line)#exec-timeout 60 0`  
`R1(config-line)#exit`  
  
Сохранить конфигурацию  
`R1(config)#end`  
`R1#copy running-config startup-config`  
