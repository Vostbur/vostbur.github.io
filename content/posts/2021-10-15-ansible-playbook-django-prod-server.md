---
categories:
- Programming
date: "2021-10-15"
description: Starting a production Django server.
slug: starting-prod-django-server
tags:
- ansible
- django
- python
- emulation
- DevOps
title: Starting a production Django server
---

I wrote [ansible playbook](https://github.com/Vostbur/ansible-playbooks/tree/main/django-prod-server) for starting up a production Django server with PostgreSQL, Gunicorn and Nginx.

When I tested the script, I used the Ubuntu Server image from [this link](https://www.osboxes.org/ubuntu/#ubuntu-21-04-info).  All I needed to install additionally was an ssh server for ansible to work. 

```
sudo apt update && sudo apt upgrade
sudo apt install openssh-server
```

I used a VirtualBox image and set the network settings to bridge mode.  The IP address was obtained via the dhcp protocol. The command to display the ip address is `ip addr`.

----
### Links:

- [1] [OSBoxes offers you ready-to-use Linux/Unix guest operating systems](https://www.osboxes.org/)
- [2] [My ansible playbook](https://github.com/Vostbur/ansible-playbooks/tree/main/django-prod-server)