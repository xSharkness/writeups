nmap разведка. видим ssh, веб-сервер и smb.
Изучим вбе-сервер и посмотрим потом smb.
```
Nmap scan report for 10.113.186.231
Host is up (0.0093s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
|_http-generator: WordPress 5.0
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time:
|   date: 2026-05-18T18:55:07
|_  start_date: N/A
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2026-05-18T18:55:07+00:00
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.55 seconds
```

На веб-сервере какой-то блог

![](../images/Pasted%20image%2020260518231114.png)

Теперь пробежимся по smb. Посмотрим какие там есть общие папки.
```
Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        BillySMB        Disk      Billy's local SMB Share
        IPC$            IPC       IPC Service (blog server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------
        Workgroup            Master
        ---------            -------
        WORKGROUP            BLOG
```

Внутри общей папки лежит несколько файлов. Проверим их.

![](../images/Pasted%20image%2020260518234913.png)

Также надо запомнить, что я могу положить любой файл на сервер.

Файл check-this.png - это qr код, который ведет на видео на ютубе
"Billy Joel - We Didn't Start The Fire (Official HD Video)"

![](../images/Pasted%20image%2020260518235324.png)

Файл Alice-White_Rabbit.jpg )))
```
└─$ steghide extract -sf Alice-White-Rabbit.jpg -p ''
wrote extracted data to "rabbit_hole.txt".

└─$ ls                                               
Alice-White-Rabbit.jpg  check-this.png  rabbit_hole.txt  tswift.mp4

└─$ cat rabbit_hole.txt                
You've found yourself in a rabbit hole, friend.
```

Никаких подсказок нету, поэтому переходим к перечислению пользователей wordpress.

Я увидел, что ссылки на авторов имеют вид:
```
http://blog.thm/?author=1
```

Мы можем перебрать число 1, чтобы понять какие существуют авторы.

ffuf быстро с этим справился
```
3                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 259ms]
1                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 267ms]
```

У нас есть два id, которые соответствуют двум пользователям сайта

Первый ник bjoel

![](../images/Pasted%20image%2020260519134949.png)

А второй ник kwheel

![](../images/Pasted%20image%2020260519135027.png)

Активируем двойной брутфорс этих пользователей, потому что неизвестно какого надо взламывать.

Первым завершилась работа у kwheel, поэтому переходим к эксплуатации уязвимостей.

![](../images/Pasted%20image%2020260519140807.png)

Из nmap помним версию wordpress - 5.0
Далее я нашел интересную уязвимость CVE 2019-8943 и CVE 2019-8942. С помощью этих уязвимостей мы можем получить доступ к системе через обработчик фотографий.

![](../images/Pasted%20image%2020260519141055.png)

Заходим в metasploit и настраиваем его

![](../images/Pasted%20image%2020260519141513.png)

Запускаем эксплоит и принимаем входящее соединение

![](../images/Pasted%20image%2020260519144030.png)

Наиболее легкий user.txt оказался неправильным

![](../images/Pasted%20image%2020260519144402.png)

Тогда сначала попробуем эскалировать до рута, а потом найдем user.txt
Я стабилизировал оболочку и затем выполнил поиск файлов SUID.
Нашел нестандартный файл.

![](../images/Pasted%20image%2020260519175324.png)

Смотрим через ltrace что происходит во время запуска его.
Нам надо установить admin = 1

![](../images/Pasted%20image%2020260519175438.png)

Ставим admin в переменной окружения в значение 1, запускаем и получаем рута

![](../images/Pasted%20image%2020260519174953.png)

Читаем флаг root.txt 

![](../images/Pasted%20image%2020260519175021.png)

Находим через общий поиск где лежит еще один user.txt и читаем его тоже

![](../images/Pasted%20image%2020260519175125.png)
