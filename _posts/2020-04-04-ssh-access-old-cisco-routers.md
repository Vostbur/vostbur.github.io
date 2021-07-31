---
layout: post
title: Подключение к устаревшим Cisco по SSH
---

Если при попытке подключиться выдает что-то похожее на "no matching key exchange method found":  

_На Cisco (ip 192.168.2.2)_:  
- длина ключа RSA должна быть больше 1024  
- создаем учетную запись пользователя (здесь - cisco)  

_В Ubuntu_:  
- создаем ~/.ssh/config  
  ```
  Host 192.168.2.2  
    KexAlgorithms +diffie-hellman-group1-sha1  
    Ciphers aes128-cbc,3des-cbc,aes192-cbc,aes256-cbc  
  ```

Алгоритмы могут быть другие, подходящие будут указаны в сообщении об ошибке.  
- проверяем  
`ssh 192.168.2.2 -l cisco`  

При подключении к роутеру в GNS3 иногда получаю ошибку _'connection refused'_. Помогает генерировать заново ключ:  
`crypto key generate rsa general-keys modulus 1024`  
В Ubuntu удаляю запись о старом ключе  
`ssh-keygen -f ~/.ssh/known_hosts -R "192.168.2.2"`  
и пробую поключиться  

_То же самое но для CLI в одну строку_

Пробуем

```
$ ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 user@host
```

Но потом мы можем получить следующее:

```
Unable to negotiate with host port 22: no matching cipher found. Their offer: aes128-cbc,3des-cbc,aes192-cbc,aes256-cbc
```

Проверим, какие типы шифрования поддерживает наш ssh клиент

```
$ ssh -Q cipher
3des-cbc
aes128-cbc
aes192-cbc
aes256-cbc
rijndael-cbc@lysator.liu.se
aes128-ctr
aes192-ctr
aes256-ctr
aes128-gcm@openssh.com
aes256-gcm@openssh.com
```

Видим, что можем использовать, например, **aes256-cbc**

```
$ ssh -c aes256-cbc -oKexAlgorithms=+diffie-hellman-group1-sha1 user@host
```

Еще почитать [здесь](https://4admin.info/legacy-ssh-device/).
