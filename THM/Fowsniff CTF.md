Небольшое описание задания
```
Вам нужно будет изучить эту машину, найдя открытые порты, провести небольшое исследование в интернете (удивительно, сколько информации может найти для вас Google), расшифровать хэши, подобрать логин pop3 методом перебора и сделать многое другое!
```

Сканируем порты через nmap. Открытые 22, 80, 110, 143. PLAIN у POP3 означает передачу данных в открытом виде. Если перехватить данные, то можно увидеть, что там передается, логин, пароль, возможно хэшированный.
```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)
|   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)
|_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Fowsniff Corp - Delivering Solutions
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) USER AUTH-RESP-CODE RESP-CODES CAPA UIDL PIPELINING TOP
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: listed IMAP4rev1 AUTH=PLAINA0001 ENABLE IDLE more have post-login LITERAL+ Pre-login capabilities SASL-IR OK ID LOGIN-REFERRALS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Далее я запустил ffuf для обнаружения скрытых директорий и файлов. Он обнаружил скрытый файл
```
robots                  [Status: 200, Size: 26, Words: 3, Lines: 3, Duration: 190ms]
security                [Status: 200, Size: 459, Words: 249, Lines: 22, Duration: 150ms]
```

В этом файле мы можем найти логин, судя по всему, администратора Fowsniff - B1gN1nj4
```
       WHAT SECURITY?

            ''~``
           ( o o )
+-----.oooO--(_)--Oooo.------+
|                            |
|          FOWSNIFF          |
|            got             |
|           PWN3D!!!         |
|                            |
|       .oooO                |
|        (   )   Oooo.       |
+---------\ (----(   )-------+
           \_)    ) /
                 (_/


Fowsniff Corp got pwn3d by B1gN1nj4!


No one is safe from my 1337 skillz!
```

Далее больше идти было некуда и я начал искать Fowsniff Corp в интернете и нашел [web.archive](https://web.archive.org/web/20200920053052/https://pastebin.com/NrAqVeeX) на котором были почты с паролями
```
1. mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
2. mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
3. tegel@fowsniff:1dc352435fecca338acfd4be10984009
4. baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
5. seina@fowsniff:90dc16d47114aa13671c697fd506cf26
6. stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
7. mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
8. parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
9. sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e
```

Почти все хэши легко ломаются

![](../images/Pasted%20image%2020260516004008.png)

Таким образом, у нас есть связки логинов и паролей. Переходим к брутфорсу pop3.
Запускаем гидру. И находим действующую связку логина и пароля.

![](../images/Pasted%20image%2020260516004625.png)

Зайдя в почту, смотрим письма и читаем их
```
LIST
+OK 2 messages:
1 1622
2 1280
.
RETR 1
+OK 1622 octets
Return-Path: <stone@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1000)
        id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
    mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
    tegel@fowsniff
Subject: URGENT! Security EVENT!
Message-Id: <20180313185107.0FA3916A@fowsniff>
Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
From: stone@fowsniff (stone)

Dear All,

A few days ago, a malicious actor was able to gain entry to
our internal email systems. The attacker was able to exploit
incorrectly filtered escape characters within our SQL database
to access our login credentials. Both the SQL and authentication
system used legacy methods that had not been updated in some time.

We have been instructed to perform a complete internal system
overhaul. While the main systems are "in the shop," we have
moved to this isolated, temporary server that has minimal
functionality.

This server is capable of sending and receiving emails, but only
locally. That means you can only send emails to other users, not
to the world wide web. You can, however, access this system via 
the SSH protocol.

The temporary password for SSH is "S1ck3nBluff+secureshell"

You MUST change this password as soon as possible, and you will do so under my
guidance. I saw the leak the attacker posted online, and I must say that your
passwords were not very secure.

Come see me in my office at your earliest convenience and we'll set it up.

Thanks,
A.J Stone


.
RETR 2
+OK 1280 octets
Return-Path: <baksteen@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1004)
        id 101CA1AC2; Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
To: seina@fowsniff
Subject: You missed out!
Message-Id: <20180313185405.101CA1AC2@fowsniff>
Date: Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
From: baksteen@fowsniff

Devin,

You should have seen the brass lay into AJ today!
We are going to be talking about this one for a looooong time hahaha.
Who knew the regional manager had been in the navy? She was swearing like a sailor!

I don't know what kind of pneumonia or something you brought back with
you from your camping trip, but I think I'm coming down with it myself.
How long have you been gone - a week?
Next time you're going to get sick and miss the managerial blowout of the century,
at least keep it to yourself!

I'm going to head home early and eat some chicken soup. 
I think I just got an email from Stone, too, but it's probably just some
"Let me explain the tone of my meeting with management" face-saving mail.
I'll read it when I get back.

Feel better,

Skyler

PS: Make sure you change your email password. 
AJ had been telling us to do that right before Captain Profanity showed up.
```

Исходя из данных письма, скорее всего, всем пользователям shh был поставлен временный пароль
```
S1ck3nBluff+secureshell
```

Брутим ssh логины под этот пароль. Находим данные для входа в систему.

![](../images/Pasted%20image%2020260516005344.png)

Далее я посмотрел sudo -l и SUID файлы, но ничего интересного там не нашел.
Поэтому я начал смотреть файлы, которые я, как участник группы могу менять и нашел интересный файл /opt/cube/cube.sh

![](../images/Pasted%20image%2020260516010001.png)

Этот файл является баннером для ssh. Если мы посмотрим на mot.d, можно найти скрипт, который запускает наш уже скрипт
```
#!/bin/sh
#
#    00-header - create the header of the MOTD
#    Copyright (C) 2009-2010 Canonical Ltd.
#
#    Authors: Dustin Kirkland <kirkland@canonical.com>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

#[ -r /etc/lsb-release ] && . /etc/lsb-release

#if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
#       # Fall back to using the very slow lsb_release utility
#       DISTRIB_DESCRIPTION=$(lsb_release -s -d)
#fi

#printf "Welcome to %s (%s %s %s)\n" "$DISTRIB_DESCRIPTION" "$(uname -o)" "$(uname -r)" "$(uname -m)"

sh /opt/cube/cube.sh
```
 
Это всё запускается пользователем рут и мы можем этот скрипт свой менять.
```
cat /opt/cube/cube.sh
printf "
                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions\n\n"
```

Поменяем его на реверс шел и перезаходим в систему по ssh.
Принимаем входящее подключение.

![](../images/Pasted%20image%2020260516012742.png)

От имени рута читаем флаг

```
# cat flag.txt
   ___                        _        _      _   _             _ 
  / __|___ _ _  __ _ _ _ __ _| |_ _  _| |__ _| |_(_)___ _ _  __| |
 | (__/ _ \ ' \/ _` | '_/ _` |  _| || | / _` |  _| / _ \ ' \(_-<_|
  \___\___/_||_\__, |_| \__,_|\__|\_,_|_\__,_|\__|_\___/_||_/__(_)
               |___/ 

 (_)
  |--------------
  |&&&&&&&&&&&&&&|
  |    R O O T   |
  |    F L A G   |
  |&&&&&&&&&&&&&&|
  |--------------
  |
  |
  |
  |
  |
  |
 ---

Nice work!

This CTF was built with love in every byte by @berzerk0 on Twitter.

Special thanks to psf, @nbulischeck and the whole Fofao Team.
```
