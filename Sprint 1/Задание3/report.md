## Запуск многоконтейнерного приложения без Docker Compose

Ситуация как в 4 задании. Требуется прокся что бы приложение работало, поэтому добавил ещё один контейнер.

### Запуск контейнеров по одному
1. db
``` bash
docker run -d --name db --network myapp-network -e POSTGRES_PASSWORD=secret123 -e POSTGRES_USER=appuser -e POSTGRES_DB=myappdb -v ./init.sql:/docker-entrypoint-initdb.d/init.sql postgres:15-alpine
```

2. backend
Сборка образа и запуск контейнера
``` bash
 docker build -t backend-app .

docker run -d --name backend --network myapp-network -e DB_HOST=db -e DB_PORT=5432 backend-app:latest
```

3. frontend
Так же в app.py поправил метод fetch():
``` python
<button onclick="fetch('/increment').then(() => location.reload())">
```
Сборка образа и запуск контейнера
``` bash
docker build -t frontend-app .

docker run -d --name frontend --network myapp-network -e BACKEND_URL=http://backend:5000 frontend-app:latest
```

4. nginx
Запуск контейнера
``` bash
docker run -d --name proxy --network myapp-network -v ./default.conf:/etc/nginx/conf.d/default.conf -p 8080:80 nginx:alpine
```

default.conf:
``` nginx
server {
  listen  80;
  location / {
    proxy_pass http://frontend:8080;
  }
  location /increment {
    proxy_pass http://backend:5000/increment;
}
```

Все контейнеры:
``` bash
docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                                     NAMES
a094ca2bd5ea   nginx:alpine                    "/docker-entrypoint.…"   12 minutes ago   Up 12 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   proxy
4b76c9c980fa   frontend-app:latest             "python app.py"          14 minutes ago   Up 13 minutes   8080/tcp                                  frontend
62f4b379697b   backend-app:latest              "python app.py"          24 minutes ago   Up 24 minutes   5000/tcp                                  backend
d3f84f49bcf6   postgres:15-alpine              "docker-entrypoint.s…"   29 minutes ago   Up 28 minutes   5432/tcp                                  db
```

