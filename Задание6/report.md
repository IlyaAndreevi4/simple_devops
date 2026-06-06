## Оптимизация Dockerfile

dockerfile:
``` dockerfile
FROM python:3.11-slim
WORKDIR /app
LABEL maintainer="Linus Torvalds" description="This is description" version="1.0.0"
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        wget \
        git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080')" || exit 1
RUN groupadd -g 1001 usergroup && \
    useradd -u 1001 -g usergroup user && \
    chown -R 1001:1001 /app
USER 1001
EXPOSE 8080
CMD ["python", "app.py"]
```

|Чек-лист:                          |Описание                         |
|-----------------------------------|---------------------------------|
|✅ Конкретный тег (не latest)		|FROM python:3.11-slim
|✅ Slim/Alpine образ				|FROM python:3.11-slim
|✅ Минимум слоев (объединенные RUN)|Объединение команд на этапе установки пакетов (apt-get)
|✅ Правильный порядок COPY			|Сначала копирую requirements, а после установки pip install копирую код
|✅ Очистка кэшей (apt, pip)		|apt-get clean && rm -rf /var/lib/apt/lists/* и  --no-cache-dir для pip
|✅ USER (не root)					|После установки пакетов и зависимостей создаю непривилигерованного пользователя и запускаю из-под него USER 1001
|✅ Exec форма CMD					|CMD ["python", "app.py"]
|✅ LABEL метаданные				|LABEL maintainer="Linus Torvalds" description="This is description" version="1.0.0"
|✅ EXPOSE порт						|EXPOSE 8080
|✅ WORKDIR							|WORKDIR /app
|✅ Нет секретов в ENV				|Передаю при запуске контейнера.  docker run -d -p 8080:8080 -e DATABASE_PASSWORD=secret123 -e API_KEY=sk-abc123 --name test-container test-app
|✅ .dockerignore файл				|Прописал в dockerignore *.bad *.md dockerfile .dockerignore