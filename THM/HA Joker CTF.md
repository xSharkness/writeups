Выполняем сканирование портов nmap. Открыт ssh и два веб сервера.
```
Host is up (0.0092s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy
```

Рассмотрим порты поподробнее. Для второго веб-сервера необходима авторизация
```
Host is up (0.0087s latency).

PORT     STATE  SERVICE  VERSION
80/tcp   open   http     Apache httpd 2.4.41 ((Ubuntu))
|_http-title: HA: Joker
|_http-server-header: Apache/2.4.41 (Ubuntu)
8080/tcp open   http     Apache httpd 2.4.41
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Please enter the password.
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: 401 Unauthorized
Service Info: Host: localhost
```

Изучаем веб-сервер

![](../images/Pasted%20image%2020260517101743.png)

В исходном коде много разных комментариев. Из диалога можно взять два имени Brute и Pettit.

![](../images/Pasted%20image%2020260517101834.png)

Фаззер  директорий и файлов нашел скрытый файл

![](../images/Pasted%20image%2020260517102615.png)

Содержимое файла не дает особо никакой информации
```
Batman hits Joker.
Joker: "Bats you may be a rock but you won't break me." (Laughs!)
Batman: "I will break you with this rock. You made a mistake now."
Joker: "This is one of your 100 poor jokes, when will you get a sense of humor bats! You are dumb as a rock."
Joker: "HA! HA! HA! HA! HA! HA! HA! HA! HA! HA! HA! HA!"
```

Так как для всего абсолютно требуется авторизация, а сайт - одностраничный, я думаю, что там необходимо найти еще другие файлы через фаззер.

Реально со временем находится ещё один файл.

![](../images/Pasted%20image%2020260517103612.png)

Это полная информация о серверной стороне.

![](../images/Pasted%20image%2020260517103750.png)

Далее по заданию нас направляют на брутфорс 8080 имея имя пользователя.
Запускаем брутфорс.

Пароль найден.

![](../images/Pasted%20image%2020260517104349.png)

Далее также фаззером находим директорию администратора. И самую интересную директорию backup.

![](../images/Pasted%20image%2020260517105035.png)

Переходя по этому адресу автоматически скачивается архив.

![](../images/Pasted%20image%2020260517105014.png)

Архив запаролен, поэтому создаем его хэш в john и брутим пароль. Пароль от архива тот же.

![](../images/Pasted%20image%2020260517105514.png)

Внутри архива находится резервная копия сайта
```
└─$ ls         
backup

└─$ ls
db  site

└─$ ls
administrator  cache  components         htaccess.txt  includes   language  libraries    media    plugins     robots.txt  tmp
bin            cli    configuration.php  images        index.php  layouts   LICENSE.txt  modules  README.txt  templates   web.config.txt

└─$ ls
joomladb.sql
```

Самое интересное тут - это sql файл. Воссоздаем sql и смотрим содержимое БД.
Находим имя пользователя и пароль в виде хэша. Можем попробовать сбрутить пароль по хэшу.

![](../images/Pasted%20image%2020260517111207.png)

Хэш быстро подбирается
```
$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG:abcd1234

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 3200 (bcrypt $2*$, Blowfish (Unix))
Hash.Target......: $2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj...unOcbG
Time.Started.....: Sun May 17 11:14:05 2026 (3 secs)
Time.Estimated...: Sun May 17 11:14:08 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-72 bytes)
Guess.Base.......: File (rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:      377 H/s (17.90ms) @ Accel:1 Loops:32 Thr:16 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 1120/14344385 (0.01%)
Rejected.........: 0/1120 (0.00%)
Restore.Point....: 896/14344385 (0.01%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:992-1024
Candidate.Engine.: Device Generator
Candidates.#01...: hawaii -> hamster
Hardware.Mon.#01.: Temp: 55c Util: 96% Core:1785MHz Mem:6000MHz Bus:16
```

Теперь, мы можем зайти на сайт.

![](../images/Pasted%20image%2020260517111658.png)

На сайт можно загрузить файл. Попробуем загрузить туда reverse shell.

С загрузкой файлов не всё так просто. Приходится обходить расширение. Скорее всего внутри тоже сравнивает магические биты.

![](../images/Pasted%20image%2020260517113911.png)

Проверяются:
1) расширение
2) магические биты
3) тег php внутри файла

Не проверяются
1) Content-type
2) сокращенный тег php (но неизвестно работает ли это)
3) некоторые обходы расширения

Я провел ещё много проверок на обход загрузки. И пришел к тому, что надо двигаться не этим способом.

У нас есть раздел шаблонов, которые и так исполняются как php код, поэтому я могу его настроить и воспроизвести свой код.

![](../images/Pasted%20image%2020260517130115.png)

Меняем index.php и запускаем превью сайта

![](../images/Pasted%20image%2020260517130358.png)

Сразу же получаем входящее соединение. Также я сразу заметил, что этот пользователь находится в lxd группе, а значит мы можем контейнером повысить привилегии.

![](../images/Pasted%20image%2020260517130434.png)

Скачиваем архив и файловую систему и перекидываем на другую систему.

![](../images/Pasted%20image%2020260517131013.png)

Теперь мы можем создать контейнер, смонтировать туда файловую систему и получить рут.
