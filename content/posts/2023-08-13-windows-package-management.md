---
categories:
- Tools
date: "2023-08-13"
description: Решение проблем с менеджером пакетов Windows
slug: windows-package-management
tags:
- windows
title: Решение проблем с менеджером пакетов Windows
---

> В апреле 2014 года Microsoft представила OneGet (позже переименованный в PackageManagement) вместе с PowerShell 5.
> Это бесплатный менеджер поставщиков пакетов с открытым исходным кодом, который позволяет интегрировать другие менеджеры пакетов в PowerShell.
> *— Wikipedia*

## Команды PackageManagement

```
Get-Command -Module PackageManagement
```

- Find-Package — поиск пакета (программы) в доступных репозиториях
- Find-PackageProvider — поиск провайдеров распространения пакетов
- Get-Package — показывает список установленных пакетов
- Get-PackageProvider — выводит список поставщиков пакетов, доступных на компьютере
- Get-PackageSource — выводит список доступных источников пакетов
- Import-PackageProvider — добавляет поставщиков пакетов управления пакетами в текущий сеанс
- Install-Package — устанавливает пакет (программу) на компьютер
- Install-PackageProvider — устанавливает одного или нескольких поставщиков пакетов управления пакетами.
- Register-PackageSource — добавляет источник пакета для поставщика
- Save-Package — сохраняет пакет локально, не устанавливая его
- Set-PackageSource — устанавливает поставщика в качестве источника пакета
- Uninstall-Package — удаляет программу (пакет)
- Unregister-PackageSource — удаляет провайдера из списка источников пакетов


## Возникшие проблемы на примере утановки golang из репозитория Chocolatey

```
PS C:\WINDOWS\system32> Find-Package -Name *golang* -Source Chocolatey
ПРЕДУПРЕЖДЕНИЕ: NuGet: Запрос был прерван: Не удалось создать защищенный канал SSL/TLS.
```

В PowerShell по умолчанию используется TLS 1.0. Меняем версию TLS по умолчанию в .NET Framework на 1.2

```
PS C:\WINDOWS\system32> reg add HKLM\SOFTWARE\Microsoft\.NETFramework\v4.0.30319 /v SystemDefaultTlsVersions /t REG_DWORD /d 1 /f /reg:64
Операция успешно завершена.
PS C:\WINDOWS\system32> reg add HKLM\SOFTWARE\Microsoft\.NETFramework\v4.0.30319 /v SystemDefaultTlsVersions /t REG_DWORD /d 1 /f /reg:32                                                                                                       Операция успешно завершена.
Операция успешно завершена.
```


```
PS C:\WINDOWS\system32> Find-Package -Name *golang* -Source Chocolatey                                                                                                                                                                          

Name                           Version          Source 		 Summary
----                           -------          ------           -------                                                
golang                         1.21.0           chocolatey       The Go programming language is an open source proje... 
golangci-lint                  1.54.1           chocolatey       golangci-lint is a Go linters aggregator.
```

Пробуем установить пакет...

```
PS C:\WINDOWS\system32> Install-Package -Name golang -ProviderName Chocolatey
```

...ничего не происходит. Меняем политику выполнения

```
PS C:\WINDOWS\system32> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```

Всё работает
