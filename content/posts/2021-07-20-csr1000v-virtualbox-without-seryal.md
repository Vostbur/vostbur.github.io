---
categories:
- Network
- Tools
date: "2021-07-20"
description: Running CSR1000v in VirtualBox and VMware.
slug: csr-virtualbox-vmware
tags:
- cisco
- VirtualBox
- VMware
- emulation
- DevNet
- windows
title: Запуск CSR1000v в VirtualBox и VMware
---

*В [прошлой статье](https://vostbur.github.io/csr1000v-virtualbox-setup/) я рассказал как подготовить и запустить любую версию виртуального маршрутизатора Cisco - Cloud Services Router (СSR1000v) в VirtualBox под Windows. С версии Cisco IOS XE 3.13S можно обойтись без настройки последовательного интерфейса. Запускать будем всё тот же релиз 3.15.0S. С [официального сайта](https://software.cisco.com/download/home/284364978/type/282046477/release/3.15.0S) нужно скачать файлы **csr1000v-universalk9.03.15.00.S.155-2.S-std.iso** и **csr1000v-universalk9.03.15.00.S.155-2.S-std.ova** По-моему, сейчас это самая поздняя версия CSR, доступная с аккаунтом Cisco, версии новее требуют различные партнерские отношения с Cisco. Зарегистрироваться можно [по адресу](https://id.cisco.com/signin/register).*

## VirtualBox

Запустите [VirtualBox](https://www.virtualbox.org/) и импортируйте конфигурацию виртуальной машины из OVI-файла. Как и в прошлой статье в настройках SCSI-контроллера виртуальной машины подключите iso-образ дистрибутива CSR и стартуйте машину. Выберите пункт *"Auto Console"* (если ничего не делать, через 10 секунд начнется загрузка в этом режиме). Долго ждем когда закончится установка и машина один раз сама перегрузится.

Cisco CSR 1000v свяжет реальные интерфесы хоста с виртуальными сетевыми картами (vNIC). Посмотреть можно командой
    
    Router# show platform software vnic-if interface-mapping
    --------------------------------------------------------------------------
    Interface Name        Short Name     vNIC Name                Mac Addr
    --------------------------------------------------------------------------
    GigabitEthernet0       Gi0        eth0 (vmxnet3)           000c.2946.3f4d
    GigabitEthernet2       Gi2        eth2 (vmxnet3)           0050.5689.0034
    GigabitEthernet1       Gi1        eth1 (vmxnet3)           0050.5689.000b
    --------------------------------------------------------------------------

У меня в качестве сетевого моста в настройках адаптеров сети автоматически выбрался wi-fi адаптер как единственный активный сетевой интерфейс. Wi-fi роутер раздает адреса по dhcp. Всё что нужно, это включить интерфейс на CSR, перевеcти в режим dhcp-клиента  и получить адрес 

    Router#conf t
    Router(config)#int gi1
    Router(config-if)#no sh
    Router(config-if)#ip addr dhcp
    Router(config-if)#exit
    
Посыпятся уведомления о поднятии порта и получении адреса. Можно убедиться

    Router(config)#do sh ip int br

Wi-fi-роутер подключен к internet, можно указать dns-сервер и пропинговать какой-нибудь ресурс

    Router(config)#ip name-server 8.8.8.8
    Router(config)#end
    Router#ping ya.ru

Сохраняем конфигурацию

    Router#wr

Теперь мы знаем ip-адрес порта маршрутизатора и можно приступать к обычным начальным настройкам

![](/images/csr-virtualbox-vmware/ping-ya-ru-from-csr.PNG)

## VMware Workstation Player 

Для запуска в [VMware](https://www.vmware.com/ru/products/workstation-player/workstation-player-evaluation.html) достаточно так же импортировать конфигурацию из ovi-файла и запустить. Но я попробовал настроил свою конфигурацию (я пользуюсь VMware Workstation 15 Player, уже вышел VMware Workstation 16 Player).

- CPU 4 (минимум 1), memory 4 Gb (минимум 4 Gb), удалил звук, подключил iso-образ

![](/images/csr-virtualbox-vmware/csr-vmware-settings.PNG)

- настроил 4 адаптера в режиме Bridge с физическим wifi-адаптером

![](/images/csr-virtualbox-vmware/csr-vmware-net-settings.PNG)

после установки включил интерфейс так же как описал выше

![](/images/csr-virtualbox-vmware/csr-vmware-result.PNG)

## Итог

CSR под VMware работает существенно быстрее. Загрузка занимает меньше 2 минут (в Virtualbox около 8 минут). Я запустил в VMware две виртуальные машины с Cloud Services Router, можно играться

![](/images/csr-virtualbox-vmware/csrs-interconnection.PNG)

----
### Links:

- [1] [Официальный сайт Cisco с образами IOS](https://software.cisco.com/download/home/284364978/type/282046477/release/3.15.0S)
- [2] [Страница регистрации аккаунта Cisco](https://id.cisco.com/signin/register)
- [3] [Oracle VM VirtualBox](https://www.virtualbox.org/)
- [4] [VMware Workstation 16 Player](https://www.vmware.com/ru/products/workstation-player/workstation-player-evaluation.html)
- [5] [Cisco CSR 1000v and Cisco ISRv Software Configuration Guide](https://www.cisco.com/c/en/us/td/docs/routers/csr1000/software/configuration/b_CSR1000v_Configuration_Guide.html)
