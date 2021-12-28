---
categories:
- Network
date: "2020-09-24"
description: Cisco command 'history'.
slug: cisco-command-history
tags:
- cisco
- configure
title: Cisco command 'history'
---

Starting from IOS 15.1 cisco command `history` is appeared. It outputs a pretty nice ASCII diagram of interfaces loading.

For enable:

```
(config)# interface GigabitEthernet 0/1
(config-if)# history bps
```

Now you can see the result using the command:

```
# show interface GigabitEthernet 0/1 history 60sec input
```

The different periods (60sec, 60min, 72hour) and different traffic directions (input, output, both) are available.