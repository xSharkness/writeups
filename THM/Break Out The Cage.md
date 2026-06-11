Я провел первоначальную разведку и увидел 3 открытых порта.
```
Host is up (0.012s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up)
```

Утверждаю соответствие сервисом и смотрю их версии. На 21 порту открыт ftp с разрешенным анонимным входом, на 22 порту ssh и на 80 порту веб-сервер.
```
Host is up (0.0087s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             396 May 25  2020 dad_tasks
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.211.86
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dd:fd:88:94:f8:c8:d1:1b:51:e3:7d:f8:1d:dd:82:3e (RSA)
|   256 3e:ba:38:63:2b:8d:1c:68:13:d5:05:ba:7a:ae:d9:3b (ECDSA)
|_  256 c0:a6:a3:64:44:1e:cf:47:5f:85:f6:1f:78:4c:59:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Nicholas Cage Stories
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed.
```

Далее я начал изучать ftp сервис, заходим туда и смотрим что там за файлы.
Я скачал единственный файл, который там находился, также проверил, можно ли загрузить свои файлы на ftp, можно ли поменять директорию.

Единственное полезное, что получил оттуда - это файл. Смотрим его содержимое.

![](../images/Pasted%20image%2020260609202933.png)

В этом файле base64 строка, сейчас раскодирую её и посмотрю что там.
```
└─$ cat dad_tasks
UWFwdyBFZWtjbCAtIFB2ciBSTUtQLi4uWFpXIFZXVVIuLi4gVFRJIFhFRi4uLiBMQUEgWlJHUVJPISEhIQpTZncuIEtham5tYiB4c2kgb3d1b3dnZQpGYXouIFRtbCBma2ZyIHFnc2VpayBhZyBvcWVpYngKRWxqd3guIFhpbCBicWkgYWlrbGJ5d3FlClJzZnYuIFp3ZWwgdnZtIGltZWwgc3VtZWJ0IGxxd2RzZmsKWWVqci4gVHFlbmwgVnN3IHN2bnQgInVycXNqZXRwd2JuIGVpbnlqYW11IiB3Zi4KCkl6IGdsd3cgQSB5a2Z0ZWYuLi4uIFFqaHN2Ym91dW9leGNtdndrd3dhdGZsbHh1Z2hoYmJjbXlkaXp3bGtic2lkaXVzY3ds
```

Похоже тут используется ещё какой то способ шифрования. Проверю несколько вариантов.

![](../images/Pasted%20image%2020260609203144.png)

Посмотрев несколько вариантов, я не нашел расшифровки, поэтому будем держать в памяти это и перейдем к изучению веб-сервера.

![](../images/Pasted%20image%2020260609210306.png)

Фаззер быстро нашел несколлько интересных директорий. Наиболее яркими были contracts и auditions. Директория contracts оказалась пустой.

![](../images/Pasted%20image%2020260609205158.png)

Во второй директории можно найти мп3 файл. Прослушав его я сразу зааметил стандартные артефакты от информации внутри файла, которую можно прочитать по спектрограмме



![](../images/Pasted%20image%2020260609205216.png)



Посмотрев спектрограмму я нашел слово namelesstwo, единственным применением этому слову для меня было как ключ шифрования к тому тексту

![](../images/Pasted%20image%2020260609205241.png)

Попробовав расшифровать текст методом Вижинера и ключом в виде этого слова я увидел осмысленный текст

![](../images/Pasted%20image%2020260609210113.png)

Отсюда мы можем найти пароль к аккаунту и зайти в систему через ssh

```
weston:Mydadisghostrideraintthatcoolnocausehesonfirejokes
```

Далее я начал изучать систему на предмет повышения привилегий. Выполнив команду id, я обратил внимание, что пользователь weston имеет идентификатор 1001, а в системе присутствует группа cage с идентификатором 1000, причём я сам в эту группу тоже вхожу. Это натолкнуло на мысль, что, возможно, существуют файлы или процессы, принадлежащие пользователю cage, к которым я могу получить доступ.

![](../images/Pasted%20image%2020260609212916.png)

Я исследовал файлы и директории, к которым имею доступ в рамках группы и обнаружил несколько подозрительных файлов в каталоге /opt/.dads_scripts/. Внутри находился Python-скрипт и поддиректория .files с файлом .quotes.

![](../images/Pasted%20image%2020260609215113.png)

Я начал анализировать скрипт spread_the_quotes.py. В этот момент на экране неожиданно появилось сообщение – это выглядело как результат работы запланированной задачи

![](../images/Pasted%20image%2020260609215459.png)

Изучив код скрипта, я понял, что он открывает файл /opt/.dads_scripts/.files/.quotes, случайным образом выбирает из него строку и подставляет её в команду os.system("wall " + quote). Поскольку кавычки не экранируются, я могу внедрить дополнительные команды с помощью точки с запятой.

![](../images/Pasted%20image%2020260609215159.png)

Я решил воспользоваться этим. Перезаписал файл .quotes, добавив команду для копирования оболочки с установкой setuid-бита, возможно даже получим рут, а не просто cage.

После этого оставалось дождаться следующего выполнения скрипта spread_the_quotes.py. Через некоторое время автоматическая задача сработала, и в каталоге /tmp появился файл shell

![](../images/Pasted%20image%2020260609220645.png)

Я запустил этот файл с флагом -p (чтобы bash не сбрасывал эффективные права) и выполнил команду id. В выводе появилась запись euid=1000(cage) – это означало, что теперь я работаю с привилегиями пользователя cage (рисунок 4.28). Таким образом, мне удалось повысить свои права с weston до cage.

![](../images/Pasted%20image%2020260609220737.png)

Читаем флаг пользователя.

![](../images/Pasted%20image%2020260609220940.png)

Далее я посмотрел и увидел все признаки ,что должен сработать pwnkit, это нужная версия ядра и pkexec с SUID битом, поэтому на свою машину скачиваю скрип пайтона и переносим через Wget его на машину злоумышленника.

Запустив скрипт я получил рут

![](../images/Pasted%20image%2020260609221602.png)

Читаем рут флаг

![](../images/Pasted%20image%2020260609221700.png)
