Подключаемся к ip через порт, указанный в задании, нам выдается строка. Она каждый раз разная, нам надо её расшифровать.

![](../images/Pasted%20image%2020260502104931.png)

Также нам дан исходник системы

![](../images/Pasted%20image%2020260502105040.png)

Из этого куска системы видно, что берется ключ из 5 случайных цифр и букв разного регистра, выполняется XOR и всё это переводится в hex, соответственно, чтобы обратно  сделать, надо перевести hex в ASCII и XOR является симметричным, поэтому, зная ключ мы можем получить исходный флаг. Перебор всех комбинаций огромный, но мы можем упростить зная, что флаг начинается с THM{ и последний символ остается неизвестным. Делаем скрипт на питоне, который перебирает последний символ.

```
import binascii
import string

hex_data = "0c7f1738426956362d461d4f2e02462c033928511959287053347b232b672a432373472a4f15314f"

cipher = binascii.unhexlify(hex_data)
alphabet = string.ascii_letters + string.digits
key_len = 5
known_prefix = b"THM{"
key_first4 = bytes(cipher[i] ^ known_prefix[i] for i in range(4))
found = False

for key5_char in alphabet:
    key5 = ord(key5_char)
    full_key = key_first4 + bytes([key5])
    repeated_key = (full_key * (len(cipher) // key_len + 1))[:len(cipher)]
    plain_bytes = bytes(a ^ b for a, b in zip(cipher, repeated_key))
    try:
        plain_text = plain_bytes.decode('ascii')
    except UnicodeDecodeError:
        continue
    if not plain_text.startswith("THM{"):
        continue
    if not plain_text.endswith("}"):
        continue
    if all(32 <= ord(c) <= 126 for c in plain_text):
        print(f"Ключ: {full_key.decode('ascii')}")
        print("Флаг:", plain_text)
        found = True
        break
if not found:
    print("Не удалось найти ключ. Возможно, начало флага не 'THM{'")
```

После выполнения скрипта ключ и флаг найдены

![](../images/Pasted%20image%2020260502112523.png)

Отправляем данные

![](../images/Pasted%20image%2020260502112628.png)

Но ответ не приходит..
Порт, по которому выполняется подключение - 1337, что соответствует известной аббревиатуре leet, возможно ключ для получения второго флага - это написанный по нормальному первый флаг?
Проверим это:

А)) Нас просили ответить код расшифрования в ответ скинуть, ещё раз всё делаем и получаем второй флаг

![](../images/Pasted%20image%2020260502113347.png)
