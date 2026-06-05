Для начала я выполнил сканирование через nmap. Увидел два открытых порта.
```
Host is up (0.0096s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up)
```

Уточнил более подробным сканированием что на них открыто. 22 ssh и 80 веб-сервер.
```
Host is up (0.0093s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Веб-сервер встречает стандартной страницей apache

![](../images/Pasted%20image%2020260605182146.png)

фаззер директорий и файлов показал несколько директорий, я решил изучить blog, но уже тут видно что это wordpress.

![](../images/Pasted%20image%2020260605182352.png)

Директория /blog/ - это шаблон блога для дальнейшей разработки.

![](../images/Pasted%20image%2020260605182846.png)

Следующим шагом я дальше изучил файлы, но уже в директории blog, но тут ничего интересного я не нашел.

![](../images/Pasted%20image%2020260605185538.png)

Я решил исследовать исходный код и нашел версию wordpress. Смотрим про нее информацию.
```
wp 5.4.2
```

У этой версии нет жестких CVE, поэтому идем дальше.

![](../images/Pasted%20image%2020260605190602.png)

Я решил попробовать перечислить пользователей.
Подставляя параметр ?author=1 удалось найти имя пользователя admin, теперь нужно разобраться с паролем.

![](../images/Pasted%20image%2020260605193643.png)

Проверил фаззингом, есть ли другие профили. Оказалось, что это единственный праофиль.

![](../images/Pasted%20image%2020260605193919.png)

Далее проверил, возможен ли быстрый брутфорс паролей через xml rpc. Понял что такая возможность есть.

![](../images/Pasted%20image%2020260605200914.png)

Запуск WP Scan на брут форс паролей быстро нашел пароль `my2boys`

![](../images/Pasted%20image%2020260605202758.png)

Далее зашел на wp, мы можем отредактировать шаблон и встроить туда реверс оболочку

![](../images/Pasted%20image%2020260605202926.png)

Открыл шаблон и закинул туда php реверс шел

![](../images/Pasted%20image%2020260605204523.png)

Принял входящее соединение.

![](../images/Pasted%20image%2020260605204548.png)

Зайти в директорию пользователя не удалось, доступ ограничен, значит должен быть другой путь.

![](../images/Pasted%20image%2020260605204918.png)

Среди файлов я нашел wp-config.php, там должны быть логи и пароль от БД. Смотрим.

Действительно тут есть имя и пароль от БД.

![](../images/Pasted%20image%2020260605205039.png)

С этими кредами получилось зайти на phpmyadmin. Но там ничего относительно другого пользователя нет.

![](../images/Pasted%20image%2020260605205234.png)

Изучил версию ядра и возможность использовать стандартный PwnKit.
```
-rwsr-xr-x 1 root root 22520 Mar 27  2019 /usr/bin/pkexec
pkexec version 0.105
Linux version 4.15.0-112-generic (buildd@lcy01-amd64-027) (gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)) #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020
```

Вроде бы всё подходит, поэтому скачал эксплоит к себе.
```
wget https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit -O PwnKit
```

Открыл сервер для передачи данных.

![](../images/Pasted%20image%2020260605220238.png)

Через реверс шел я скачал эксплоит на машину целевую и запустил.

Получил рут права.

![](../images/Pasted%20image%2020260605220311.png)

Далее читаем флаг пользователя и флаг рута.

![](../images/Pasted%20image%2020260605220404.png)

