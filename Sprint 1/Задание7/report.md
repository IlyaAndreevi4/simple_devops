## Модификация существующего образа

Получившийся dockerfile:
```dockerfile
FROM mostafamoradian/xk6-kafka:latest
USER 0
RUN apk add --no-cache nano
USER 12345
ENTRYPOINT ["/bin/sh"]
```
---

USER 0 необходим что бы можно было запустить apk add

В базовом образе USER 12345, его и отсавил полсе установки.

ENTRYPOINT переопределил.

---
### Итог
Переопределение ENTRYPOINT:
``` bash
docker image inspect k6-nano:latest --format '{{json .Config}}'
```
``` json
{
	"Hostname": "",
	"Domainname": "",
	"User": "12345",
	"AttachStdin": false,
	"AttachStdout": false,
	"AttachStderr": false,
	"Tty": false,
	"OpenStdin": false,
	"StdinOnce": false,
	"Env": [
		"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
	],
	"Cmd": null,
	"Image": "",
	"Volumes": null,
	"WorkingDir": "/home/k6",
	"Entrypoint": [
		"/bin/sh"
	],
	"OnBuild": null,
	"Labels": {
		"org.opencontainers.image.created": "2026-04-19T13:17:16.206Z",
		"org.opencontainers.image.description": "k6 extension to load test Apache Kafka with support for various serialization formats, SASL, TLS, compression, Schema Registry client and beyond",
		"org.opencontainers.image.licenses": "Apache-2.0",
		"org.opencontainers.image.revision": "5eb8eda7e620d8e2dfd1bf6771a3772cf0b89297",
		"org.opencontainers.image.source": "https://github.com/mostafa/xk6-kafka",
		"org.opencontainers.image.title": "xk6-kafka",
		"org.opencontainers.image.url": "https://github.com/mostafa/xk6-kafka",
		"org.opencontainers.image.version": "2.0.0"
	}
}
```



Наличие nano:
``` bash
docker run -it --name temp-container k6-nano:latest
~ $ nan
nanddump   nandwrite  nano
~ $ exit
```