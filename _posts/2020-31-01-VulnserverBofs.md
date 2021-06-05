---
layout: post
title: Vulnserver
date: 2020-01-31
author: Mojo
categories: ["Vulnserver"]
---
# **Some walk throughs on [Vulnserver](https://github.com/stephenbradshaw/vulnserver)**

#### I will add a guide on how to set up Vulnserver #soonTM

Vulnserver exposes a bunch of vulnerable commands via an open TCP socket.
There are a bunch of commands which you can fuzz to get the server to crash.
Below is a list of POCs which are ordered by the command that will trigger the BOF.

I created a little script to fuzz for buffer overflows [here](https://gist.github.com/mojodojo101/9df5012a0928f824d158e50d91305435)

For windows debugging i will be using [Immunity Debugger](https://www.immunityinc.com/products/debugger/) and [Mona](https://github.com/corelan/mona.git)


This is what the interface looks like when you connect to vulnserver, just to give you an idea of the application.

```bash
root@kali:~/redteam/vulnserver/trun# nc -nv 192.168.138.139 9999
(UNKNOWN) [192.168.138.139] 9999 (?) open
Welcome to Vulnerable Server! Enter HELP for help.
HELP
Valid Commands:
HELP
STATS [stat_value]
RTIME [rtime_value]
LTIME [ltime_value]
SRUN [srun_value]
TRUN [trun_value]
GMON [gmon_value]
GDOG [gdog_value]
KSTET [kstet_value]
GTER [gter_value]
HTER [hter_value]
LTER [lter_value]
KSTAN [lstan_value]
EXIT

```


## [Vulnserver TRUN ](/bofs/vulnserver/trun.html)