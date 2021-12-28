---
categories:
- Network
- OS
date: "2020-09-19"
description: Start Ansible for Docker containers.
slug: start-ansible-for-docker-containers
tags:
- docker
- ansible
- DevNet
- linux
title: Start Ansible for Docker containers
---

For some pet projects, I need multiple instances of different servers.
The best way is to use Docker.
And the best way to automate configuration services that run in docker containers is Ansible.
This is the sequence of steps to run Ansible to configure containerized services in Ubuntu.

[Installing Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/) is very well described on the official website. A summary of the steps:

```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
```

The last command is used to verify successful installation.

Adding user credential to run docker (without sudo)

```
sudo usermod -aG docker "${USER}"
sudo newgrp docker
sudo service docker restart
```

Create a Dockerfile as described on the page [Dockerize an SSH service](https://docs.docker.com/engine/examples/running_ssh_service/). Then build the images using:

```
docker build -t server_one .
docker build -t server_two .
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
server_one          latest              f75f39181756        2 hours ago         220MB
server_two          latest              f75f39181756        2 hours ago         220MB
```

Run containers

``` 
docker run -p 52022:22 -d -P --name server_one server_one
docker run -p 53022:22 -d -P --name server_two server_two
docker ps

CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                   NAMES
b08de187afa7        server_one          "/usr/sbin/sshd -D"   27 minutes ago      Up 27 minutes       0.0.0.0:52022->22/tcp   server_one
9bb2d9c9fab5        server_two          "/usr/sbin/sshd -D"   27 minutes ago      Up 27 minutes       0.0.0.0:53022->22/tcp   server_two
```

Find out the ip address of the new containers and try ssh with the root password (from the configuration)

```
docker inspect -f ".NetworkSettings.IPAddress" server_one
docker inspect -f ".NetworkSettings.IPAddress" server_two
ssh root@172.17.0.2
ssh root@172.17.0.3
```

Create public key for user

```
ssh-keygen
ssh-copy-id root@172.17.0.2
ssh-copy-id root@172.17.0.3
```

Now you can get ssh access with the password for user pub key.

Install Ansible:

```
apt-add-repository ppa:ansible/ansible
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
ansible --version
```

Ansible local configuration

```
cp -R /etc/ansible . && cd ansible
sed -i 's/#*inventory *= \/etc\/ansible\/hosts/inventory = hosts/g' ansible.cfg
cat >> hosts << EOF
root@172.17.0.2
root@172.17.0.3
EOF
```

Checking

![](/images/test_ansible_docker_ubuntu.svg)
