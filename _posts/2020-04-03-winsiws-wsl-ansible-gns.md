---
layout: post
title: Доступ по telnet к роутеру в GNS3 (Windows) из Ansible
---

В Windows запущен GNS3 с роутером, доступным по telnet (_ip 192.168.2.2_), установлена Ubuntu в WSL. Как управлять роутером через Ansible?
 
- Ставим Ansible  
`sudo apt install ansible`  

- Создаем файлы в домашнем каталоге  
 
   __myhosts__  
   ```
   [cisco-routers]  
   192.168.2.2  
   ```

   __ansible.cfg__  
   ```
   [defaults]  
   inventory = ./myhosts  
   gathering = explicit
   ```

   __telnet_command_show_ver.yml__  
   _(логин без ввода имени пользователя)_
   
   ```
   ---  
   - name: Run show command on routers  
   hosts: cisco-routers  
   tasks:  
     - name: run show commands  
     telnet:  
       login_prompt: "Password: "  
       password: cisco  
       prompts:  
         - "[>#]"  
       command:  
         - terminal length 0  
         - show version
   ```

- Выполняем  
`ansible-playbook telnet_command_show_ver.yml`  

![](/images/ansible-wsl-gns.jpg)