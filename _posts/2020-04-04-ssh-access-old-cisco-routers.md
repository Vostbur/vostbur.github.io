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
`ssh-keygen -f "~/.ssh/known_hosts" -R "192.168.2.2"`  
и пробую поключиться  

Еще почитать [здесь](https://4admin.info/legacy-ssh-device/).