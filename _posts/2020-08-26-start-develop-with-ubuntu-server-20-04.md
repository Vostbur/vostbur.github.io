---
layout: post
title: Start development with Ubuntu Server 20.04
---

Ubuntu and useful tools are constantly updated. I will update this article from time to time as well.

Install zsh
===========

```
sudo apt-get install zsh
zsh --version
whereis zsh
chsh -s /usr/bin/zsh user
logout
```

Install framework for delicate configuration *zsh*

`sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`

Install pip (globally)
======================

Installing *pip* as a system utility is not always necessary. If you aren't installing *pip*, it will still be available in the virtual environment. Except when running a virtual environment with a key *--without-pip*. But usually you need a *pip*

`sudo apt-get install python3-pip`

If you aren't using a virtual environment, it is better to install packages in the user's home directory without using *sudo*

`python3 -m pip install --user black`

Install & use pyvenv
====================

`sudo apt-get install python3-venv`

Usual workflow

```
mkdir dir && cd dir
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip setuptools
deactivate
```

Pyenv
=====

[Management of multiple versions of Python with pyenv](https://vostbur.github.io/pyenv/)

