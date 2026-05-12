Видим несколько открытых портов, изучим их попоробнее.

![](../images/Pasted%20image%2020260512115922.png)

ftp и ssh требуют учетных данных, поэтому глянем на веб-сервер

![](../images/Pasted%20image%2020260512120201.png)

Куча текста разного, смотрим чо там

![](../images/Pasted%20image%2020260512120558.png)

В конце заметка о том, что этот текст не относится к решению CTF, а просто для атмосферы задания. Пока я читал это всё, фазер обнаружил скрытую директорию.

![](../images/Pasted%20image%2020260512121048.png)

Смотря исходный код страницы можно прочитать Code Word

![](../images/Pasted%20image%2020260512121205.png)

Предположительно это учетные данные от чего-то, либо просто логин, а пароль будет позже?

```
Lian_Yu:vigilante
```

Данные никуда не подошли, ни в каком формате и я запустил фаззинг директорий в /island/
Спустя некоторое время я нашел директорию 2100

![](../images/Pasted%20image%2020260512123855.png)

В исходном коде страницы 
```
<!-- you can avail your .ticket here but how?   -->
```

Далее я запустил фаззинг уже файлов с разрешением .ticket

```
green_arrow.ticket      [Status: 200, Size: 71, Words: 10, Lines: 7, Duration: 183ms]
```

Переходя по этому адресу находим

![](../images/Pasted%20image%2020260512130158.png)

```
RTy8yhBQdscX
```

Попробовав все возможные комбинации я понял, что это не пароль в его исходном виде и надо его как-то раскодировать.
После долгих поисков я нашел, что это строка в base58

![](../images/Pasted%20image%2020260512133252.png)

Теперь, вводя имя пользователя и пароль, мы можем зайти на ftp сервер!
```
vigilante:!#th3h00d
```

![](../images/Pasted%20image%2020260512133424.png)

Заходим и скачиваем файлы, смотрим что там. Также я чуть не забыл посмотреть, есть ли скрытые файлы!

```
ftp> ls -la
229 Entering Extended Passive Mode (|||57573|).
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 May 05  2020 .
drwxr-xr-x    4 0        0            4096 May 01  2020 ..
-rw-------    1 1001     1001           44 May 01  2020 .bash_history
-rw-r--r--    1 1001     1001          220 May 01  2020 .bash_logout
-rw-r--r--    1 1001     1001         3515 May 01  2020 .bashrc
-rw-r--r--    1 0        0            2483 May 01  2020 .other_user
-rw-r--r--    1 1001     1001          675 May 01  2020 .profile
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
226 Directory send OK.
```

Самое интересное, скорее всего в истории баш команд, но сейчас всё посмотрим внимательно.
Также я попробовал поменять директорию и понял, что всё работает.

![](../images/Pasted%20image%2020260512133753.png)

Таким образом мы нашли всех живых пользователей этого сервера.

```
└─$ cat .bash_history
Sorry I couldn't Help Other user Might help
```

Видим небольшую подсказку, что нам нужен другой пользователь, на данный момент, из других мы знаем только slade.
Но тут есть файл .other_user

```
└─$ cat .other_user 
Slade Wilson was 16 years old when he enlisted in the United States Army, having lied about his age. After serving a stint in Korea, he was later assigned to Camp Washington where he had been promoted to the rank of major. In the early 1960s, he met Captain Adeline Kane, who was tasked with training young soldiers in new fighting techniques in anticipation of brewing troubles taking place in Vietnam. Kane was amazed at how skilled Slade was and how quickly he adapted to modern conventions of warfare. She immediately fell in love with him and realized that he was without a doubt the most able-bodied combatant that she had ever encountered. She offered to privately train Slade in guerrilla warfare. In less than a year, Slade mastered every fighting form presented to him and was soon promoted to the rank of lieutenant colonel. Six months later, Adeline and he were married and she became pregnant with their first child. The war in Vietnam began to escalate and Slade was shipped overseas. In the war, his unit massacred a village, an event which sickened him. He was also rescued by SAS member Wintergreen, to whom he would later return the favor.

Chosen for a secret experiment, the Army imbued him with enhanced physical powers in an attempt to create metahuman super-soldiers for the U.S. military. Deathstroke became a mercenary soon after the experiment when he defied orders and rescued his friend Wintergreen, who had been sent on a suicide mission by a commanding officer with a grudge.[7] However, Slade kept this career secret from his family, even though his wife was an expert military combat instructor.

A criminal named the Jackal took his younger son Joseph Wilson hostage to force Slade to divulge the name of a client who had hired him as an assassin. Slade refused, claiming it was against his personal honor code. He attacked and killed the kidnappers at the rendezvous. Unfortunately, Joseph's throat was slashed by one of the criminals before Slade could prevent it, destroying Joseph's vocal cords and rendering him mute.

After taking Joseph to the hospital, Adeline was enraged at his endangerment of her son and tried to kill Slade by shooting him, but only managed to destroy his right eye. Afterwards, his confidence in his physical abilities was such that he made no secret of his impaired vision, marked by his mask which has a black, featureless half covering his lost right eye. Without his mask, Slade wears an eyepatch to cover his eye.
```

Соответственно мы точно знаем, что другой пользователь - это slade.
Далее смотрим на изображения


![](../images/Pasted%20image%2020260512134719.png)

Одно из них самое интересное, его надо восстановить.
Восстановив его видим следующее изображение

![](../images/Pasted%20image%2020260512134800.png)

Предположительно password и есть пароль, попробуем расшифровать какие-нибудь данные из jpg файла

Действительно всё сработало и мы получили архив

![](../images/Pasted%20image%2020260512135109.png)

Внутри архива два файла

```
└─$ cat passwd.txt  
This is your visa to Land on Lian_Yu # Just for Fun ***


a small Note about it


Having spent years on the island, Oliver learned how to be resourceful and 
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.

└─$ cat shado              
M3tahuman
```

Это скорее всего пароль от slade. Пробуем.

![](../images/Pasted%20image%2020260512135433.png)

Действительно, пароль подошел, читаем флаг user.txt.

Смотрим какие утилиты доступны от имени рута и видим самый простейший вектор эскалации привилегий. Через 5 секунд нам доступен рут)

![](../images/Pasted%20image%2020260512140055.png)

