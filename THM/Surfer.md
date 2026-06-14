Задание:
```
Мы просмотрели несколько веб-страниц и хотим, чтобы вы тоже присоединились! Говорят, в этом приложении есть функции, доступные только для внутреннего использования, но если вы поймаете нужную волну, то, возможно, найдете что-то интересное!

Раскройте флаг на скрытой странице приложения.
```

После стандартной разведки можно увидеть 1 неиндексируемую директорию в файле robots.txt. Посмотрим на сайт и затем посмотрим что в robots.txt
```
Host is up (0.0084s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 23:18:ae:f5:25:df:b7:09:5e:1a:04:91:24:84:22:54 (RSA)
|   256 17:25:6f:64:a6:31:d5:27:d4:35:3e:2a:75:80:c5:52 (ECDSA)
|_  256 eb:4d:06:55:36:b2:a9:5c:ab:4f:4b:2c:a9:6e:ea:54 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
| http-robots.txt: 1 disallowed entry
|_/backup/chat.txt
| http-title: 24X7 System+
|_Requested resource was /login.php
|_http-server-header: Apache/2.4.38 (Debian)
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed.
```

На сайте обычная форма входа, логина и пароля у меня нет поэтому смотрим сразу же в robots.txt

![](../images/Pasted%20image%2020260614135004.png)

Находим директорию и файл внутри неё, посмотрим что там.

![](../images/Pasted%20image%2020260614135149.png)

В файле представлен диалог о том, что на сайт внедрили новый инструмент для экспорта отчетов. Этот инструмент работает через запросы на внутренний сервер.
Кроме этого, из этого документа я узнал имя пользователя и пароль, стандартные admin:admin, поэтому теперь мы можем зайти на сайт.
```
Admin: I have finished setting up the new export2pdf tool.  
Kate: Thanks, we will require daily system reports in pdf format.  
Admin: Yes, I am updated about that.  
Kate: Have you finished adding the internal server.  
Admin: Yes, it should be serving flag from now.  
Kate: Also Don't forget to change the creds, plz stop using your username as password.  
Kate: Hello.. ?
```

Заходим на сайт и попадаем в dashboard. Справа, в разделе недавней активности мы видим упоминание внутренней страницы сервера /internal/admin.php, это та страница, которую нам надо открыть по заданию.
Чуть ниже есть кнопка экспорта отчета. Посмотрим как это всё работает.

![](../images/Pasted%20image%2020260614135753.png)

Генерируется pdf файл, теперь посмотрим как это происходит через веб-взаимодействие

![](../images/Pasted%20image%2020260614140049.png)

Я увидел, что на страницу отправляется внутренний url файла, который потом попадает к нам в pdf отчет.

![](../images/Pasted%20image%2020260614143557.png)

Далее я попробовал подменить ссылку и отправил новый запрос, уже с внутренним url до /internal/admin.php.
Так как оно автоматически не конвертируется, то было сложно понять, можно ли таким образом действительно получить этот файл.
Я воспользовался сравнением ответов и понял, что ответ приходит действительно разный в части, которая является pdf файлом.

![](../images/Pasted%20image%2020260614143638.png)

Далее я включил прерывание запросов OWASP ZAP и подменил ссылку, чтобы браузер открыл сам эту страницу.

![](../images/Pasted%20image%2020260614143711.png)

Страница загрузилась и нормально открылась. Флаг получен.

![](../images/Pasted%20image%2020260614143037.png)

