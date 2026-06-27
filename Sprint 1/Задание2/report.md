# Базовый образ из Yandex Registry

---

## Dockerfile

```dockerfile
FROM cr.yandex/mirror/python:3.11-slim
WORKDIR /app
ENV REGISTRY="Yandex Mirror"
COPY app.py .
USER 1001
EXPOSE 8080
ENTRYPOINT ["python", "app.py"]
```

---

## Проверка работы

**HTTP-запрос**
```bash
curl -v http://localhost:8080
```

**Ответ сервера**
```
* Host localhost:8080 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:8080...
* Connected to localhost (::1) port 8080
* using HTTP/1.x
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/8.12.1
> Accept: */*
>
* Request completely sent off
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Server: BaseHTTP/0.6 Python/3.11.4
< Date: Fri, 05 Jun 2026 15:19:28 GMT
< Content-type: text/html
<

        <html>
        <head><title>Task 2</title></head>
        <body style="font-family: Arial; text-align: center; padding: 50px;">
            <h1>Задание 2 выполнено!</h1>
            <h2>Yandex Mirror Registry</h2>
            <p>Registry: <strong>Yandex Mirror</strong></p>
            <p>Базовый образ: <code>cr.yandex/mirror/python:3.11-slim</code></p>
        </body>
        </html>
* shutting down connection #0

```

Страница возвращает HTML с подтверждением использования `cr.yandex/mirror/python:3.11-slim` и значением переменной `REGISTRY`.

---

**Переменная окружения**
```bash
docker exec yandex-app env | grep REGISTRY
```
```
REGISTRY=Yandex Mirror
```

---

## Итог

| Параметр       | Значение                          |
|----------------|-----------------------------------|
| Базовый образ  | `cr.yandex/mirror/python:3.11-slim` |
| `REGISTRY`     | `Yandex Mirror`                   |
| Порт           | `8080`                            |
| Пользователь   | `1001` (non-root)                 |
| Статус         | `200 OK`                          |
