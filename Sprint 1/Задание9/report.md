## Docker Networks

### Проблемы
---
``` bash
docker exec worker ping -c 2 redis
ping: bad address 'redis'

docker exec frontend ping -c 2 api
ping: bad address 'api'

docker exec api ping -c 2 db
PING db (172.21.0.3): 56 data bytes
64 bytes from 172.21.0.3: seq=0 ttl=64 time=0.061 ms
64 bytes from 172.21.0.3: seq=1 ttl=64 time=0.136 ms

--- db ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.061/0.098/0.136 ms
```


- Redis изолирован - никто не может подключиться
- Frontend не видит API
- Worker использует хардкод IP 192.168.1.100 (несуществующий)
- backend_net помечена `internal: true` - контейнеры не могут наружу

### Решение
---
1. Redis изолирован

Добавить redis в backend_net.

``` yaml
  redis:
    image: redis:alpine
    container_name: redis
    networks:
      - redis_net
      - backend_net
```

2. Frontend не видит API

Добавить frontend в backend_net.

``` yaml
  frontend:
    image: nginx:alpine
    container_name: frontend
    ports:
      - "8080:80"
    networks:
      - frontend_net
      - backend_net
```

3. Хардкод IP worker

Изменить IP на DNS-имя сервиса.

``` yaml
  worker:
    image: alpine:latest
    container_name: worker
    command: sh -c "while true; do echo 'Trying to connect to redis:6379'; sleep 5; done"
    networks:
      - backend_net
```

4. backend_net internal

Убрать `internal: true`

``` yaml
networks:

  backend_net:
    driver: bridge
```


### Проверка после исправления

``` bash
docker exec worker ping -c 2 redis
PING redis (172.20.0.6): 56 data bytes
64 bytes from 172.20.0.6: seq=0 ttl=64 time=0.125 ms
64 bytes from 172.20.0.6: seq=1 ttl=64 time=0.288 ms

--- redis ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.125/0.206/0.288 ms


docker exec frontend ping -c 2 api
PING api (172.20.0.3): 56 data bytes
64 bytes from 172.20.0.3: seq=0 ttl=64 time=0.047 ms
64 bytes from 172.20.0.3: seq=1 ttl=64 time=0.153 ms

--- api ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.047/0.100/0.153 ms


docker-compose logs worker
worker  | Trying to connect to redis:6379
worker  | Trying to connect to redis:6379
worker  | Trying to connect to redis:6379
worker  | Trying to connect to redis:6379
worker  | Trying to connect to redis:6379
worker  | Trying to connect to redis:6379
worker  | Trying to connect to redis:6379
```