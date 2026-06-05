# Docker Debug — Решения

---

## app1 — ошибка монтирования конфига

**Команда диагностики**
```bash
docker inspect app1 --format='{{.State.Error}}'
```

**Ошибка**
```
cannot create subdirectories in ".../default.conf": not a directory
→ попытка смонтировать файл как директорию
```

**Причина** — файл `app1.conf` отсутствовал, Docker создал директорию вместо файла.

**Решение**
```bash
cp /config/app1.conf.example /config/app1.conf
```

**Итог:** `Up`

---

## app2 — порт уже занят

**Команда диагностики**
```bash
docker inspect app2 --format='{{.State.Error}}'
```

**Ошибка**
```
Bind for 0.0.0.0:8080 failed: port is already allocated
```

**Причина** — порт `8080` занят контейнером app1.

**Решение**
```yaml
app2:
  image: nginx:alpine
  container_name: app2
  ports:
    - "8081:80"
```

**Итог:** `Up`

---

## app3 — запись в read-only volume

**Команда диагностики**
```bash
docker logs app3
```

**Ошибка**
```
sh: can't create /data/test.txt: Read-only file system
```

**Причина** — том смонтирован с флагом `:ro`, но команда пытается записать файл.

**Решение** — убрать `:ro`:
```yaml
app3:
  image: alpine:latest
  container_name: app3
  command: sh -c "echo 'test' > /data/test.txt && cat /data/test.txt"
  volumes:
    - ./data:/data
```

**Итог:** `Exited (0)`

---

## app4 — неверный exit code

**Команда диагностики**
```bash
docker-compose ps -a
```

**Проблема** — контейнер завершается с кодом `0` вместо ожидаемого `1`. Без `set -e` ошибка `ping` не прерывает выполнение скрипта.

**Решение** — добавить `set -e`:
```yaml
app4:
  image: alpine:latest
  container_name: app4
  command: >
    sh -c "
      set -e
      ping nonexistent-host
    "
```

**Итог:** `Exited (1)`

---

## app5 — бесконечный перезапуск

**Команда диагностики**
```bash
docker-compose ps -a
```

**Проблема** — политика `restart: always` перезапускает контейнер при каждом `exit 1`, создавая бесконечный цикл.

**Решение** — изменить политику перезапуска:
```yaml
app5:
  image: alpine:latest
  container_name: app5
  command: sh -c "exit 1"
  restart: no
```

**Итог:** `Exited (1)`

---

## app6 — не задан пароль PostgreSQL

**Команда диагностики**
```bash
docker logs app6
```

**Ошибка**
```
Database is uninitialized and superuser password is not specified.
You must specify POSTGRES_PASSWORD to a non-empty value for the superuser.
```

**Решение** — добавить переменную окружения:
```yaml
app6:
  image: postgres:15-alpine
  container_name: app6
  environment:
    POSTGRES_PASSWORD: postgres
  ports:
    - "5432:5432"
```

**Итог:** `Up`

---

## Итоговые статусы

| Контейнер | Образ               | Статус       | Порты                  |
|-----------|---------------------|--------------|------------------------|
| app1      | nginx:alpine        | Up           | 0.0.0.0:8080→80        |
| app2      | nginx:alpine        | Up           | 0.0.0.0:8081→80        |
| app3      | alpine:latest       | Exited (0)   | —                      |
| app4      | alpine:latest       | Exited (1)   | —                      |
| app5      | alpine:latest       | Exited (1)   | —                      |
| app6      | postgres:15-alpine  | Up           | 0.0.0.0:5432→5432      |
