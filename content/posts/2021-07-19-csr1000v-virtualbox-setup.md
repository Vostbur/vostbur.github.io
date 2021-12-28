---
categories:
- Network
- Tools
date: "2021-07-19"
description: Running CSR1000v in VirtualBox (Windows) with serial port emulation.
slug: csr-virtualbox-serial-port-emulation
tags:
- cisco
- VirtualBox
- emulation
- DevNet
- windows
title: Запуск CSR1000v в VirtualBox (Windows) с эмуляцией serial port
---

*Я расскажу как подготовить и запустить виртуальный маршрутизатор Cisco - Cloud Services Router (СSR1000v) в VirtualBox под Windows с эмуляцией serial port, настроить доступ по telnet, ssh и написать скрипт на python для автоматизации конфигурирования.*

## Необходимый софт

С [официального сайта](https://software.cisco.com/download/home/284364978/type/282046477/release/3.15.0S) нужно скачать файлы **csr1000v-universalk9.03.15.00.S.155-2.S-std.iso** и **csr1000v-universalk9.03.15.00.S.155-2.S-std.ova**.

По-моему, сейчас это самая поздняя версия CSR, доступная с аккаунтом Cisco, версии новее требуют различные партнерские отношения с Cisco. Зарегистрироваться можно [по адресу](https://id.cisco.com/signin/register).

Если еще не установлен VirtualBox, [скачайте](https://www.virtualbox.org/) и установите.

Скачайте и установите [Named Pipe TCP Proxy](http://shvechkov.tripod.com/nptp.html). Эта утилита перенаправляет на последовательный интерфейс (serial port) всё, что приходит на определенный TCP порт localhost. Так мы сможем подключиться через Putty к CSR и выполнить первоначальные настройки для дальнейшей работы через telnet или ssh.

Ну и конечно нужен будет [PuTTY](https://www.putty.org/).

## Подготовка к запуску

Запустите **piped.exe**, создайте новый канал с настройками:

    Pipe-Name “\\.\pipe\csr1000v”
    Port 1999

![](/images/csr-virtualbox-setup/add-pipe.PNG)

Запустите VirtualBox и импортируйте конфигурацию виртуальной машины из OVI-файла.

![](/images/csr-virtualbox-setup/import-ovi.PNG)

После импорта изменим некоторые настройки:

- В настройках SCSI-контроллера подключите iso-образ дистрибутива CSR.

![](/images/csr-virtualbox-setup/scsi-iso-setup.PNG)

- В настройках сети я установил все интерфейсы в тип подключения *"Виртуальный адаптер хоста"*

![](/images/csr-virtualbox-setup/network-setup.PNG)

- В настройках COM-портов включил первый порт, выбрал режим *"Хост-канал"* и указал адрес "`\\.\pipe\csr1000v`".

![](/images/csr-virtualbox-setup/serial-setup.PNG)

Поле *"Подключиться к существующему каналу/сокету"* оставил не активным. При его включении у меня виртуальная машина не запускалась, выдавая ошибку.

Убедившись еще раз, что **Named Pipe Proxy TCP Proxy** запущен, запускаем Putty, в настройках **Connection->Telnet** выключаем **Return key sends Telnet New Line instead of ^M** и устанавливаем telnet-соединение по адресу localhost (**127.0.0.1**), порт - **1999**.

![](/images/csr-virtualbox-setup/putty-connect.PNG)

Проверяем, что соединение установлено

![](/images/csr-virtualbox-setup/proxy-connection.PNG)

## Запуск виртуальной машины

Стартуем созданную виртуальную машину. В окне Putty нажимаем любую клавишу в ответ на *'Press any key to continue'* и выбираем в меню *GRUB* пункт *‘Serial Console’*.

Первая загрузка идет очень долго, у меня заняло больше 10 минут и виртуальная машина один раз перегрузится. Ничего делать не надо.

После успешного запуска настраиваем доступ по **telnet**.
Адрес виртуального сетевого адаптера VirtualBox по умолчанию находится в сети **192.168.56.0/24**.

![](/images/csr-virtualbox-setup/virtual-adapter-network.PNG)

Интерфейсу CSR даем адрес из этой же сети. Вводим команды в putty:

    en
    conf t
    int gi1
    ip addr 192.168.56.2 255.255.255.0
    no sh
    exit
    enable secret cisco1
    line vty 0 4
    password cisco0
    exec-timeout 0 0
    end
    wr

Проверяем доступность CSR

![](/images/csr-virtualbox-setup/check-ping.PNG)

Пробуем подключиться по **telnet**. Пароль - **cisco0**, пароль на enable - **cisco1**

![](/images/csr-virtualbox-setup/setup-ready.PNG)

## Доступ по ssh

В заключении настроим доступ по **ssh**

    router(config)#hostname csr
    csr(config)#ip domain name site.info
    csr(config)#crypto key generate rsa
    ...
    How many bits in the modulus [512]: 1024
    % Generating 1024 bit RSA keys, keys will be non-exportable...
    [OK] (elapsed time was 0 seconds)

    csr(config)#ip ssh version 2
    csr(config)#line vty 0 ?
    <1-98>  Last Line number
    <cr>

    csr(config)#line vty 0 98
    csr(config-line)#transport input all
    csr(config-line)#login local
    csr(config-line)#exit
    csr(config)#username admin privilege 1 secret cisco0
    csr(config)#end
    csr#wr

В Putty подключаемся по ssh с именем пользователя - **admin**, паролем - **cisco0**, enable - **cisco1**.

## Зачем это надо?

Виртуальный маршрутизатор Cisco это полноценный роутер с Cisco IOS XE на котором можно пробовать различные варианты конфигурации или тестировать devops-скрипты. 

Например скрипт на Python с библиотекой [netmiko](https://github.com/ktbyers/netmiko)

    from netmiko import (
        ConnectHandler,
        NetmikoTimeoutException,
        NetmikoAuthenticationException,
    )

    cisco_router = {
        'device_type': 'cisco_ios',
        'host': '192.168.56.2',
        'username': 'admin',
        'password': 'cisco0',
        'secret': 'cisco1',
        'port': 22,
    }

    commands_list = ['int gi1',
                    'desc SSH_INTERFACE',
                    'do sh ip int br',
                    'do sh int desc']


    def send_commands(commands: list) -> str:
        try:
            with ConnectHandler(**cisco_router) as ssh:
                ssh.enable()
                res = ssh.send_config_set(commands)
        except (NetmikoTimeoutException, NetmikoAuthenticationException) as error:
            res = error
        return res


    if __name__ == '__main__':
        result = send_commands(commands_list)
        print(result)

Выведет

![](/images/csr-virtualbox-setup/netmiko-output.PNG)

----
### Links:

- [1] [Официальный сайт Cisco с образами IOS](https://software.cisco.com/download/home/284364978/type/282046477/release/3.15.0S)
- [2] [Страница регистрации аккаунта Cisco](https://id.cisco.com/signin/register)
- [3] [Oracle VM VirtualBox](https://www.virtualbox.org/)
- [4] [Named Pipe TCP Proxy](http://shvechkov.tripod.com/nptp.html)
- [5] [PuTTY](https://www.putty.org/)
- [6] [Installing Cisco CSR 1000V in VirtualBox](https://techandtrains.com/2014/01/09/installing-cisco-csr-1000v-in-virtualbox/)
- [7] [github.com/ktbyers/netmiko](https://github.com/ktbyers/netmiko)
- [8] [Модуль netmiko](https://pyneng.readthedocs.io/ru/latest/book/18_ssh_telnet/netmiko.html)
