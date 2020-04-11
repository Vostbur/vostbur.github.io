---
layout: post
title: Запуск графического интерфейса Ubuntu в подсистеме Windows для Linux (WSL)
---

Как поставить Ubuntu в WSL написано [здесь](https://docs.microsoft.com/ru-ru/windows/wsl/install-win10).  

__В Windows__  
1. Ставим [VcXsrv Windows X Server](https://sourceforge.net/projects/vcxsrv/).  
2. Запускаем вручную или создаем конфиг для запуска - файл _config.xlaunch_:  

```
<?xml version="1.0" encoding="UTF-8"?>  
<XLaunch  
  WindowMode="Nodecoration"  
  ClientMode="NoClient"  
  LocalClient="False"  
  Display="0"  
  LocalProgram="xcalc"  
  RemoteProgram="xterm"  
  RemotePassword=""  
  PrivateKey=""  
  RemoteHost=""  
  RemoteUser=""  
  XDMCPHost=""  
  XDMCPBroadcast="False"  
  XDMCPIndirect="False"  
  Clipboard="True"  
  ClipboardPrimary="True"  
  ExtraParams=""  
  Wgl="True"  
  DisableAC="False"  
  XDMCPTerminate="False"  
> />  
```

__В Ubuntu__  

`sudo apt update && sudo apt dist-upgrade`  

_Для Xfce4 (минимум):_  
`sudo apt install -y xfce4-session xfce4-notifyd xfce4-appfinder xfce4-panel`  
`sudo apt install -y xfce4-quicklauncher-plugin xfce4-whiskermenu-plugin`  
`sudo apt install -y xfce4-xkb-plugin xfce4-settings xfce4-terminal xfce4-taskmanager`  
`sudo apt install -y mousepad`  

_Для GNOME:_  
`sudo apt install -y ubuntu-desktop`  

Затем  
`sudo service dbus start`  
`sudo service x11-common start`  

Локализация  
`sudo locale-gen ru_RU`  
`sudo locale-gen ru_RU.UTF-8`  
`sudo update-locale`  

Создаем start-desktop.sh  

_Для Xfce4:_  
`DISPLAY=:0 LANG=ru_RU.UTF-8 su alex -c xfce4-session`  

_Для GNOME:_  
`gnome-shell --x11 -r`  

Делаем исполняемым  
`chmod u+x start-desktop.sh`  

__Последовательность:__  
1. Запускаем _config.xlaunch_
2. Выполняем в Linux _./start-desktop.sh_
