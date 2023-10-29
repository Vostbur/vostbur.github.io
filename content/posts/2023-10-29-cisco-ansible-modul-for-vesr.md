---
categories:
- Tools
date: "2023-10-29"
description: Управление Eltex vESR через модули Ansible от Cisco
slug: cisco-ansible-modul-for-vesr
tags:
- ansible
- DevNet
- Eltex
- vESR
title: Управление Eltex vESR через модули Ansible от Cisco
---

В июне 2023 года российский разработчик и производитель телекоммуникационного оборудования Eltex представил виртуальный сервисный маршрутизотор [vESR](https://eltex-co.ru/catalog/service_gateways/virtualnyy_servisnyy_marshrutizator_vesr/), доступный для бесплатного тестированя в базовой версии.

Конфигурационные команды схожи с Cisco, и при небольших изменениях кода модули Ansible _ios_command_ и _ios_config_ тоже успешно работают.

Для этого надо поменять (у меня Ansible установлен в домашнем каталоге пользователя):

- в файле `~/.local/lib/python3.10/dist-packages/ansible_collections/cisco/ios/plugins/cliconf/ios.py` в строке `self.send_command("configure terminal")` выражение `"configure terminal"` на `"configure"`

- в файле `~/.local/lib/python3.10/dist-packages/ansible_collections/cisco/ios/plugins/terminal/ios.py` в строке `self._exec_cli_command(b"terminal lenght 0")` выражение `"terminal lenght 0"` на `"terminal datadump"`


Пример playbook _test.yaml_ с выводом результата на экран

```
---
- name: Run show commands
  hosts: vesr
  gather_facts: false

  vars:
    ansible_user: admin
    ansible_ssh_pass: admin
    ansible_connection: ansible.netcommon.network_cli
    ansible_network_os: cisco.ios.ios

  tasks:
    - name: run sh ver
      ansible.netcommon.cli_command:
        command: show version
      register: ps
    - debug: var=ps.stdout_lines
```

inventory-файл _hosts_

```
[vesr]
192.168.0.99
```

команда для запуска

```
$ ansible-playbook test.yaml -i hosts
```