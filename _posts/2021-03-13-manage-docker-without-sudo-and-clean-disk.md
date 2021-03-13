---
layout: post
title: Manage Docker without sudo and clean up used disk space
---

Manage Docker without sudo
--------------------------

There are official [guidelines](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user) for managing Docker as a non-root user.

Summary:

add user to `docker` group

```
$ sudo gpasswd -a $USER docker
```

If testing on a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.

On a desktop Linux environment, log out of your session completely and then log back in.

On other cases for Linux, you can also run the following command to activate the changes to groups:

```
$ newgrp docker
```

The main thing for me is that I can now use the VSCode plugin for docker on Linux.

Clearing the disk space used by Docker
--------------------------------------

To find out how much space is being used by the docker

```
$ docker system df
```

delete all unused containers

```
$ docker container prune
```

delete all 'dangling' images. The images which doesn't linked to runned containers.

```
$ docker image prune
```

delete all unused volumes. For example, that were left after the containers were deleted.

```
$ docker volume prune
```
