Базовое сканирование показало непонятные открытые порты

![](../images/Pasted%20image%2020260508213350.png)

Запустим более подробное сканирование, вывод получился огромным

```
# Nmap 7.95 scan initiated Fri May  8 17:34:52 2026 as: nmap -sV -sC -T4 --stats-every=5s -p 22,3306,9999,15065,16109,46969 -oA result 10.112.130.220
Nmap scan report for 10.112.130.220
Host is up (0.0085s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 28:0c:0c:d9:5a:7d:be:e6:f4:3c:ed:10:51:49:4d:19 (RSA)
|   256 17:ce:03:3b:bb:20:78:09:ab:76:c0:6d:8d:c4:df:51 (ECDSA)
|_  256 07:8a:50:b5:5b:4a:a7:6c:c8:b3:a1:ca:77:b9:0d:07 (ED25519)
3306/tcp  open  mysql   MySQL 5.7.29-0ubuntu0.18.04.1
| ssl-cert: Subject: commonName=MySQL_Server_5.7.29_Auto_Generated_Server_Certificate
| Not valid before: 2020-03-19T17:21:30
|_Not valid after:  2030-03-17T17:21:30
|_ssl-date: TLS randomness does not represent time
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.29-0ubuntu0.18.04.1
|   Thread ID: 11
|   Capabilities flags: 65535
|   Some Capabilities: Support41Auth, FoundRows, LongPassword, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, SupportsLoadDataLocal, Speaks41ProtocolOld, DontAllowDatabaseTableColumn, ODBCClient, SupportsTransactions, IgnoreSigpipes, SupportsCompression, InteractiveClient, SwitchToSSLAfterHandshake, Speaks41ProtocolNew, LongColumnFlag, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: [Y4\x04\x13\l\x15\x1Fhe\x06S\x7F`]>+\x1C	
|_  Auth Plugin Name: mysql_native_password
9999/tcp  open  http    Golang net/http server
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 200 OK
|     Date: Fri, 08 May 2026 17:34:58 GMT
|     Content-Length: 4
|     Content-Type: text/plain; charset=utf-8
|     king
|   GenericLines, Help, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, Socks5: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   OfficeScan: 
|     HTTP/1.1 400 Bad Request: missing required Host header
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request: missing required Host header
15065/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Host monitoring
16109/tcp open  http    Golang net/http server
|_http-title: Site doesn't have a title (image/jpeg).
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Date: Fri, 08 May 2026 17:34:58 GMT
|     Content-Type: image/jpeg
|     JFIF
|     #*%%*525EE\xff
|     #*%%*525EE\xff
|     $3br
|     %&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
|     &'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
|     Y$?_
|     qR]$Oyk
|_    |$o.
46969/tcp open  telnet  Linux telnetd
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9999-TCP:V=7.95%I=7%D=5/8%Time=69FE1EC3%P=x86_64-pc-linux-gnu%r(Get
SF:Request,78,"HTTP/1\.0\x20200\x20OK\r\nDate:\x20Fri,\x2008\x20May\x20202
SF:6\x2017:34:58\x20GMT\r\nContent-Length:\x204\r\nContent-Type:\x20text/p
SF:lain;\x20charset=utf-8\r\n\r\nking")%r(HTTPOptions,78,"HTTP/1\.0\x20200
SF:\x20OK\r\nDate:\x20Fri,\x2008\x20May\x202026\x2017:34:58\x20GMT\r\nCont
SF:ent-Length:\x204\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\n\r
SF:\nking")%r(FourOhFourRequest,78,"HTTP/1\.0\x20200\x20OK\r\nDate:\x20Fri
SF:,\x2008\x20May\x202026\x2017:34:58\x20GMT\r\nContent-Length:\x204\r\nCo
SF:ntent-Type:\x20text/plain;\x20charset=utf-8\r\n\r\nking")%r(GenericLine
SF:s,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain
SF:;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request
SF:")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type
SF::\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x2
SF:0Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCont
SF:ent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r
SF:\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x
SF:20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnectio
SF:n:\x20close\r\n\r\n400\x20Bad\x20Request")%r(LPDString,67,"HTTP/1\.1\x2
SF:0400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8
SF:\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SIPOptions,67
SF:,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x2
SF:0charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r
SF:(Socks5,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text
SF:/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20R
SF:equest")%r(OfficeScan,A3,"HTTP/1\.1\x20400\x20Bad\x20Request:\x20missin
SF:g\x20required\x20Host\x20header\r\nContent-Type:\x20text/plain;\x20char
SF:set=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request:\x20miss
SF:ing\x20required\x20Host\x20header");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port16109-TCP:V=7.95%I=7%D=5/8%Time=69FE1EC3%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,2D28,"HTTP/1\.0\x20200\x20OK\r\nDate:\x20Fri,\
SF:x2008\x20May\x202026\x2017:34:58\x20GMT\r\nContent-Type:\x20image/jpeg\
SF:r\n\r\n\xff\xd8\xff\xe0\0\x10JFIF\0\x01\x01\x01\0H\0H\0\0\xff\xdb\0C\0\
SF:x02\x03\x03\x03\x04\x03\x04\x05\x05\x04\x06\x06\x06\x06\x06\x08\x08\x07
SF:\x07\x08\x08\r\t\n\t\n\t\r\x13\x0c\x0e\x0c\x0c\x0e\x0c\x13\x11\x14\x11\
SF:x0f\x11\x14\x11\x1e\x18\x15\x15\x18\x1e#\x1d\x1c\x1d#\*%%\*525EE\\\xff\
SF:xdb\0C\x01\x02\x03\x03\x03\x04\x03\x04\x05\x05\x04\x06\x06\x06\x06\x06\
SF:x08\x08\x07\x07\x08\x08\r\t\n\t\n\t\r\x13\x0c\x0e\x0c\x0c\x0e\x0c\x13\x
SF:11\x14\x11\x0f\x11\x14\x11\x1e\x18\x15\x15\x18\x1e#\x1d\x1c\x1d#\*%%\*5
SF:25EE\\\xff\xc0\0\x11\x08\x03\x84\x05F\x03\x01\"\0\x02\x11\x01\x03\x11\x
SF:01\xff\xc4\0\x1f\0\0\x01\x05\x01\x01\x01\x01\x01\x01\0\0\0\0\0\0\0\0\x0
SF:1\x02\x03\x04\x05\x06\x07\x08\t\n\x0b\xff\xc4\0\xb5\x10\0\x02\x01\x03\x
SF:03\x02\x04\x03\x05\x05\x04\x04\0\0\x01}\x01\x02\x03\0\x04\x11\x05\x12!1
SF:A\x06\x13Qa\x07\"q\x142\x81\x91\xa1\x08#B\xb1\xc1\x15R\xd1\xf0\$3br\x82
SF:\t\n\x16\x17\x18\x19\x1a%&'\(\)\*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxy
SF:z\x83\x84\x85\x86\x87\x88\x89\x8a\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x
SF:a2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\
SF:xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda
SF:\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf
SF:8\xf9\xfa\xff\xc4\0\x1f\x01\0\x03\x01\x01\x01\x01\x01\x01\x01\x01\x01\0
SF:\0\0\0\0\0\x01\x02\x03\x04\x05\x06\x07\x08\t\n\x0b\xff\xc4\0\xb5\x11\0\
SF:x02\x01\x02\x04\x04\x03\x04\x07\x05\x04\x04\0\x01\x02w\0\x01\x02\x03\x1
SF:1\x04\x05!1\x06\x12AQ\x07aq\x13\"2\x81\x08\x14B\x91\xa1\xb1\xc1\t#3R\xf
SF:0\x15br\xd1\n\x16\$4\xe1%\xf1\x17\x18\x19\x1a&'\(\)\*56789:CDEFGHIJSTUV
SF:WXYZcdefghijstuvwxyz\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x92\x93\x94\x9
SF:5\x96\x97\x98\x99\x9a\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xb2\xb3\xb4\x
SF:b5\xb6\xb7\xb8\xb9\xba\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xd2\xd3\xd4\
SF:xd5\xd6\xd7\xd8\xd9\xda\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xf2\xf3\xf4
SF:\xf5\xf6\xf7\xf8\xf9\xfa\xff\xda\0\x0c\x03\x01\0\x02\x11\x03\x11\0\?\0\
SF:xfa\x96F\xf3/\x0f\xcd\xc0\xdcp\x7f\*\x97!\x1e\xd4p\x7f\|\x83\xdf\x8c\xb
SF:7\xf4\xa4\xb4\x8e=\x92\xc9\xce\xec\xe2\x90\xc6Zks\x91\x85Y\$\?_\xba\+\x
SF:81\x1e\xa9E\xees31\xe0\x02\xccA\xfe\x20\xa35\x90\x1c\xff\0fC\x95\x1b\x8
SF:8\x047L\xe4\xf4\x1f\x9d\^\x92=\xdez\?\xded!~\x8eqR\]\$Oyk\x02\x81\x85\x
SF:c1\xc9\xe8\0\xed\xfaS\x11\|\x05q\x20\xee\xbbT\x0fM\xc6\xa3i\xb2\x97\x93
SF:7\x18\xca\xae:\xd6t\x0e\xdb\xe3\xf4/#\x96\xf4\t\x92\)\xad\xb7\xca\x89\x
SF:03}\xf9@l\xfbsLh\xcb\xba@\xb7d\x86%\x96\xdc\*\xfb\x175\x8b\|\$o\.\xd9N\
SF:xe1\xf2n\xfa\x97\x15\xbdrA\x86G\r\x9c\xce\xaa9\xfe\xe7ZM2\x08");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May  8 17:35:35 2026 -- 1 IP address (1 host up) scanned in 42.68 seconds

```

Интересно, ssh, mysql, 3 веб-сервера и telnet

Пока что самыми интересными являются веб-сервера, изучим их поближе

На порту 9999 одностраничник с одним единственным словом king, думаю это и есть один из флагов, но потом всё проверим. Это единственная реакция от взаимодействия с этой страницей.

![](../images/Pasted%20image%2020260508215111.png)

Далее идем на следующий веб сервер. Он тоже встречает только одной страницей с текстом:

```
# Site down for maintenance

Blame Dan, he keeps messing with the prod servers.
```

Отсюда мы узнаем имя пользователя - dan, возможно пригодиться дальше.

На третьем сайте лежит изображение

![](../images/Pasted%20image%2020260508215753.png)

Проанализируем это изображение

![](../images/Pasted%20image%2020260508220520.png)

binwalk нашел скрытые данные внутри файла, экспортируем их

![](../images/Pasted%20image%2020260508220704.png)

Отлично, найдены какие-то креды
```
pasta:pastaisdynamic
```

По идее только 3 пути, где эти креды могут пригодиться, это ssh, mysql и telnet.
Проверим их по очереди.

![](../images/Pasted%20image%2020260508221141.png)

Отлично, получилось зайти с этими кредами через ssh.

В папке bread находим первый flag. Теперь мне понятен формат флагов.

![](../images/Pasted%20image%2020260509005937.png)

```
1) thm{7baf5aa8491a4b7b1c2d231a24aec575}
```

Изучаем директорию /home/bread

![](../images/Pasted%20image%2020260509010041.png)

Логикой я дошел до того, что эта папка - это директория сайта, открытого на порте 15065, там есть директория monitor, перейдем туда.

![](../images/Pasted%20image%2020260508221625.png)

Написав туда легитимный ip адрес я нашел api эндпоинт

![](../images/Pasted%20image%2020260508221803.png)

Потестируем его на уязвимости.

Как я и предполагал, api уязвим к CMDi и мы можем выполнять команды от имени пользователя bread, но пока не вижу в этом большого смысла, когда мы уже в системе. Возможно у них разные права.

Я ещё немного походил по системе. Перейдя в директорию food можно найти скрытый файл.

![](../images/Pasted%20image%2020260508222426.png)

```
1) thm{7baf5aa8491a4b7b1c2d231a24aec575}
2) thm{58a3cb46855af54d0660b34fd20a04c1}
```

Далее я задался вопросом, откуда запускаются все веб-сервера. Ресурсы одного веб сервера мы нашли, а остальные - нет. Для этого я открыл все процессы и нашел:

bread      801  0.4  1.0 108896 10372 ?        Ssl  17:27   0:19 /home/bread/main
root       809  0.3  0.9 108432  9544 ?        Ssl  17:27   0:17 /root/koth
tryhack+   786  0.0  1.3 108432 13556 ?        Ssl  17:27   0:00 /home/tryhackme/img

первое - это монитор с пингом, второе - это сайт со словом king и третье это изображение со стеганографией.

Но, к сожалению, нельзя открыть их исходники, не хватает прав.

Дальше я попытался повысить привилегии до рута, запустил линпис и нашел интересный вектор

```
Pkexec binary found at: /usr/bin/pkexec

Pkexec binary has SUID bit set!
-rwsr-xr-x 1 root root 22520 Mar 27  2019 /usr/bin/pkexec
pkexec version 0.105
Potentially vulnerable to CVE-2021-4034 (PwnKit) - check distro patches
```

Проверим его.

![](../images/Pasted%20image%2020260508235010.png)

ОТЛИЧНО, всё сработало. Ну а теперь начинается охота за флагами, разбросанными по системе. Так как у нас права рута, можно не переживать, что куда-то нет доступа))

Просто переходим сразу же и читаем флаг

![](../images/Pasted%20image%2020260508235046.png)

```
1) thm{7baf5aa8491a4b7b1c2d231a24aec575}
2) thm{58a3cb46855af54d0660b34fd20a04c1}
3) thm{5a926ab5d3561e976f4ae5a7e2d034fe}
```

Переходим в рут директорию и находим еще один флаг

![](../images/Pasted%20image%2020260508235432.png)

```
1) thm{7baf5aa8491a4b7b1c2d231a24aec575}
2) thm{58a3cb46855af54d0660b34fd20a04c1}
3) thm{5a926ab5d3561e976f4ae5a7e2d034fe}
4) thm{9f1ee18d3021d135b03b943cc58f34db}
```

Далее читаем скрытый файл .mysql_history и находим флаг там

![](../images/Pasted%20image%2020260509000647.png)

```
1) thm{7baf5aa8491a4b7b1c2d231a24aec575}
2) thm{58a3cb46855af54d0660b34fd20a04c1}
3) thm{5a926ab5d3561e976f4ae5a7e2d034fe}
4) thm{9f1ee18d3021d135b03b943cc58f34db}
5) thm{2f30841ff8d9646845295135adda8332}
```

Далее я решил найти все файлы в названии которых встречается flag и нашел ещё один флаг

![](../images/Pasted%20image%2020260509001142.png)

```
1) thm{7baf5aa8491a4b7b1c2d231a24aec575}
2) thm{58a3cb46855af54d0660b34fd20a04c1}
3) thm{5a926ab5d3561e976f4ae5a7e2d034fe}
4) thm{9f1ee18d3021d135b03b943cc58f34db}
5) thm{2f30841ff8d9646845295135adda8332}
6) thm{0c48608136e6f8c86aecdb5d4c3d7ba8}
```

Следующий флаг лежит в логах аутентификации

![](../images/Pasted%20image%2020260509005315.png)

```
1) thm{7baf5aa8491a4b7b1c2d231a24aec575}
2) thm{58a3cb46855af54d0660b34fd20a04c1}
3) thm{5a926ab5d3561e976f4ae5a7e2d034fe}
4) thm{9f1ee18d3021d135b03b943cc58f34db}
5) thm{2f30841ff8d9646845295135adda8332}
6) thm{0c48608136e6f8c86aecdb5d4c3d7ba8}
7) thm{4675c55160bb806ef39172976bc0aa5f}
```
И последний флаг я нашел в файле /root/.profile

![](../images/Pasted%20image%2020260509005413.png)

```
1) thm{7baf5aa8491a4b7b1c2d231a24aec575}
2) thm{58a3cb46855af54d0660b34fd20a04c1}
3) thm{5a926ab5d3561e976f4ae5a7e2d034fe}
4) thm{9f1ee18d3021d135b03b943cc58f34db}
5) thm{2f30841ff8d9646845295135adda8332}
6) thm{0c48608136e6f8c86aecdb5d4c3d7ba8}
7) thm{4675c55160bb806ef39172976bc0aa5f}
8) thm{237741b0835c77a30a4a7ef3393f8a7d}
```
