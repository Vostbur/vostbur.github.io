---
categories:
- Programming
date: "2020-04-08"
description: Saving passwords in Windows Credentials Locker from Python.
slug: saving-password-wcl-python
tags:
- windows
- wcl
- python
title: Сохранение паролей в Windows Credentials Locker из Python
---

1. Ставим [keyring](https://pypi.org/project/keyring/)  
`pip install keyring` 

2. Выполняем  
`import keyring`  
`keyring.set_password("system", "username", "password")`  

3. Результат  
![](/images/credentials.png)
  
4. Доступ к данным   
`keyring.get_password("system", "username")`  
`'password'` 

