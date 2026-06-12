nmap показал два открытых порта, всё обычное
```
Host is up (0.0094s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up)
```

Исследуем сайт

![](../images/Pasted%20image%2020260612131449.png)

Фаззинг показал, что существует какие то директории img и r, перейдем туда и посмотрим что там

![](../images/Pasted%20image%2020260612131627.png)

В директории img лежат 3 файла, возможно эта стеганография, потом проверим

![](../images/Pasted%20image%2020260612131612.png)

Директория r не дает особого смысла, поэтому фаззим дальшь

![](../images/Pasted%20image%2020260612131652.png)

Кроме первых двух директорий нашлась poem, смотрим что там

![](../images/Pasted%20image%2020260612132512.png)

В poem просто текст поэмы, связанный с Алисой, ничего для решения он мне не дал

![](../images/Pasted%20image%2020260612132445.png)

Далее в директории r нашлась директория a, я понял, что это отсылка к слову rabbit и начал друг за другом спускаться на всё более низкие уровни, пока не дошел до пути /r/a/b/b/i/t/.
Вот весь текст, который был на этих страницах:

- "Would you tell me, please, which way I ought to go from here?"
- "That depends a good deal on where you want to get to," said the Cat.
- "I don’t much care where—" said Alice.
- "Then it doesn’t matter which way you go," said the Cat.
- "—so long as I get somewhere,"" Alice added as an explanation.
- "Oh, you’re sure to do that," said the Cat, "if you only walk long enough." Alice felt that this could not be denied, so she tried another question. "What sort of people live about here?" "In that direction,"" the Cat said, waving its right paw round, "lives a Hatter: and in that direction," waving the other paw, "lives a March Hare. Visit either you like: they’re both mad."

На последней странице я нашел скрытый текст

![](../images/Pasted%20image%2020260612133103.png)

Это оказались креды для входа по ssh
```
alice:HowDothTheLittleCrocodileImproveHisShiningTail
```

Заходим в систему и смотрим какие есть файлы. В домашнем каталоге Алисы есть флаг рута, но мы его не можем открыть из-за прав. Также есть скрипт пайтона, который показывает рандомные строчки из поэмы.

![](../images/Pasted%20image%2020260612133229.png)

Далее я проверил на самую быструю уязвимость эскалации и понял, что система подходит для этого.

![](../images/Pasted%20image%2020260612135456.png)

Я перекинул эксплоит через scp и запустил его.
В итоге получил рут пользователя.

![](../images/Pasted%20image%2020260612135540.png)

Далее поиском я нашел файл user.txt, он оказался в папке рута и прочитал его.

![](../images/Pasted%20image%2020260612135841.png)

