# 🟢 Модуль 3. Контейнеры: запуск и управление

[← Установка](02-install.md) · [На главную](../README.md) · [Образы →](04-images.md)

## 3.1. Анатомия docker run

```
docker run -d --name web -p 8080:80 -e TZ=Europe/Moscow -v ./html:/usr/share/nginx/html:ro nginx:1.27 nginx -g 'daemon off;'
           ── ────────── ────────── ───────────────── ────────────────────────────────────── ────────── ───────────────────────
           │   │          │          │                 │                                      │          └─ команда (заменяет CMD)
           │   │          │          │                 │                                      └─ образ:тег
           │   │          │          │                 └─ том/монтирование (только чтение)
           │   │          │          └─ переменная окружения
           │   │          └─ порт ХОСТА:порт КОНТЕЙНЕРА
           │   └─ имя контейнера
           └─ в фоне (detached)
```

`docker run` = `docker pull` (если образа нет) + `docker create` + `docker start` (+ `attach`, если нет `-d`).

⚠️ Опции Docker пишутся **до** имени образа. Всё, что после образа, — это команда и аргументы для процесса внутри контейнера.

## 3.2. Флаги docker run, которые нужны каждый день

| Флаг | Что делает |
|---|---|
| `-d` | Запустить в фоне |
| `-it` | Интерактивно (`-i` stdin + `-t` терминал) |
| `--rm` | Удалить контейнер после завершения |
| `--name web` | Имя (иначе случайное вроде `happy_turing`) |
| `-p 8080:80` | Проброс порта хост:контейнер |
| `-p 127.0.0.1:8080:80` | Порт только на localhost (не виден из сети) |
| `-p 5353:53/udp` | UDP-порт |
| `-P` | Опубликовать все `EXPOSE`-порты на случайные порты хоста |
| `-e KEY=value` | Переменная окружения |
| `--env-file .env` | Переменные из файла |
| `-v data:/data` | Именованный том |
| `-v "$PWD":/app` | Bind mount текущей папки |
| `--mount type=bind,src=...,dst=...,readonly` | То же, явный синтаксис |
| `-w /app` | Рабочая папка |
| `-u 1000:1000` | Запустить от пользователя/группы |
| `--network backend` | Подключить к сети |
| `--restart unless-stopped` | Политика перезапуска |
| `--entrypoint sh` | Переопределить ENTRYPOINT |
| `--init` | Добавить init-процесс (сигналы, зомби) |
| `-h myhost` | Hostname |
| `--add-host db.local:10.0.0.5` | Запись в /etc/hosts |
| `--add-host host.docker.internal:host-gateway` | Доступ к хосту из контейнера на Linux |
| `--platform linux/amd64` | Запустить образ другой архитектуры (эмуляция) |
| `--pull always` | Всегда скачивать свежую версию тега |
| `-l env=prod` | Метка (label) |

### Политики перезапуска

| Политика | Поведение |
|---|---|
| `no` | Не перезапускать (по умолчанию) |
| `on-failure[:5]` | Только при ненулевом коде выхода, максимум 5 раз |
| `always` | Всегда, включая после перезагрузки хоста (даже если остановил вручную — поднимется после рестарта демона) |
| `unless-stopped` | Как always, но уважает ручную остановку — **оптимально для сервисов** |

Изменить у работающего: `docker update --restart unless-stopped web`.

### Лимиты ресурсов

```bash
docker run -d --name api \
  --memory=512m --memory-swap=512m \   # без swap
  --cpus=1.5 \                          # полтора ядра
  --cpuset-cpus=0,1 \                   # только ядра 0 и 1
  --pids-limit=256 \
  --ulimit nofile=65536:65536 \
  myapi:1.0

docker update --memory=1g --memory-swap=1g api   # поменять на лету
```

## 3.3. Жизненный цикл

| Команда | Что делает |
|---|---|
| `docker create --name x img` | Создать, но не запускать |
| `docker start x` | Запустить существующий |
| `docker start -ai x` | Запустить и подключиться |
| `docker stop x` | SIGTERM → 10 с → SIGKILL |
| `docker stop -t 30 x` | Дать 30 секунд на корректное завершение |
| `docker kill x` | Сразу SIGKILL |
| `docker kill -s HUP x` | Послать сигнал (например, перечитать конфиг) |
| `docker restart x` | stop + start |
| `docker pause x` / `unpause` | Заморозить все процессы (cgroup freezer) |
| `docker rename old new` | Переименовать |
| `docker wait x` | Дождаться завершения и вывести код выхода |
| `docker rm x` | Удалить остановленный |
| `docker rm -f x` | 🔴 Удалить даже работающий |
| `docker rm -v x` | Удалить вместе с анонимными томами |
| `docker container prune` | Удалить все остановленные |

💡 Команды принимают имя или ID, причём хватит нескольких первых символов ID: `docker stop 3f2`.

## 3.4. Список и фильтры: docker ps

```bash
docker ps                          # работающие
docker ps -a                       # все
docker ps -q                       # только ID (для подстановки в другие команды)
docker ps -s                       # + размер записываемого слоя
docker ps -n 5                     # 5 последних созданных
docker ps -f status=exited         # упавшие/остановленные
docker ps -f name=api -f label=env=prod
docker ps -f ancestor=nginx        # созданные из образа nginx
docker ps -f health=unhealthy
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
docker ps --format '{{json .}}' | jq .
```

| Поле --format | Значение |
|---|---|
| `{{.ID}}` `{{.Names}}` `{{.Image}}` | ID, имя, образ |
| `{{.Status}}` `{{.State}}` | «Up 5 minutes (healthy)» / running |
| `{{.Ports}}` `{{.Networks}}` | Порты, сети |
| `{{.Command}}` `{{.CreatedAt}}` `{{.Size}}` | Команда, дата, размер |
| `{{.Label "com.docker.compose.project"}}` | Значение метки |

## 3.5. Внутрь контейнера: exec, attach, cp

```bash
docker exec -it web bash               # оболочка (в alpine/distroless — sh или вообще нет)
docker exec web cat /etc/os-release    # одна команда
docker exec -u root -it web sh         # от root, даже если образ с USER app
docker exec -w /app -e DEBUG=1 web env # с рабочей папкой и переменной
docker exec -d web touch /tmp/flag     # в фоне

docker attach web                      # подключиться к stdin/stdout PID 1
# отключиться без остановки: Ctrl+P, Ctrl+Q  (Ctrl+C остановит контейнер!)

docker cp web:/etc/nginx/nginx.conf ./nginx.conf   # из контейнера
docker cp ./index.html web:/usr/share/nginx/html/  # в контейнер
docker cp web:/var/log - | tar -t                  # как tar-поток
```

⚠️ `exec` — для отладки. Изменения, сделанные руками внутри, пропадут при пересоздании контейнера.

### Отладка контейнера без оболочки (distroless, scratch) 🟡

```bash
# Запустить рядом контейнер с инструментами в тех же namespaces
docker run --rm -it --pid=container:app --network=container:app nicolaka/netshoot
#   ps aux, ss -tlnp, curl localhost:8080, tcpdump -i any — всё «изнутри» app
# ФС целевого контейнера видна через /proc/1/root/

docker debug app     # Docker Desktop / Docker Pro: встроенная отладочная оболочка
```

## 3.6. Логи

| Команда | Что делает |
|---|---|
| `docker logs web` | Весь лог (stdout + stderr) |
| `docker logs -f web` | Следить в реальном времени |
| `docker logs --tail 100 web` | Последние 100 строк |
| `docker logs --since 10m web` | За 10 минут (или `2026-09-24T10:00:00`) |
| `docker logs --until 1h web` | До момента час назад |
| `docker logs -t web` | С метками времени |
| `docker logs web 2>&1 \| grep -i error` | Искать по логу (stderr тоже!) |
| `docker inspect -f '{{.LogPath}}' web` | Где лежит файл лога на хосте |

💡 `docker logs` показывает только то, что процесс пишет в **stdout/stderr**. Поэтому официальный образ nginx делает `access.log -> /dev/stdout`.

## 3.7. Инспекция и мониторинг

```bash
docker inspect web                                          # всё в JSON
docker inspect -f '{{.State.Status}} {{.State.ExitCode}}' web
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web   # IP
docker inspect -f '{{.State.Health.Status}}' web             # healthy / unhealthy
docker inspect -f '{{json .Mounts}}' web | jq                # смонтированные тома
docker inspect -f '{{json .Config.Env}}' web | jq            # переменные окружения
docker inspect -f '{{.State.OOMKilled}}' web                 # убит ли из-за памяти
docker inspect -f '{{.RestartCount}}' web                    # сколько раз перезапускался

docker stats                          # live: CPU%, MEM, NET I/O, BLOCK I/O, PIDS
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker top web                        # процессы контейнера (с PID хоста)
docker port web                       # проброшенные порты
docker diff web                       # что изменено в ФС: A — добавлено, C — изменено, D — удалено
docker events --since 1h              # события: start, die, oom, health_status…
docker events -f event=die -f event=oom
```

## 3.8. Рецепты одноразовых контейнеров

Контейнер — отличный способ запустить инструмент, не устанавливая его:

```bash
docker run --rm -it python:3.12 python                         # REPL нужной версии
docker run --rm -v "$PWD":/app -w /app node:22 npm install     # npm без Node на хосте
docker run --rm -v "$PWD":/src -w /src golang:1.23 go build -o app .
docker run --rm -it --network host nicolaka/netshoot           # сетевые утилиты
docker run --rm -it -e POSTGRES_PASSWORD=pass -p 5432:5432 postgres:16   # БД для теста
docker run --rm -v "$PWD":/data -w /data mikefarah/yq '.services' compose.yaml
docker run --rm -it alpine sh -c "apk add curl && curl -I https://example.com"
docker run --rm -u "$(id -u):$(id -g)" -v "$PWD":/w -w /w alpine touch file   # файлы не от root
```

## 3.9. Массовые операции

```bash
docker stop $(docker ps -q)                         # остановить все
docker rm $(docker ps -aq -f status=exited)         # удалить все остановленные
docker ps -q -f label=env=test | xargs -r docker rm -f
docker ps -a --filter "name=^tmp-" -q | xargs -r docker rm -f
docker restart $(docker ps -q -f name=worker)
```

## 3.10. Сохранить контейнер как образ (и почему так лучше не делать)

```bash
docker commit -m "debug tools" web myweb:debug   # снимок состояния контейнера
docker export web > web.tar                       # только ФС, без слоёв и метаданных
cat web.tar | docker import - myweb:flat
```

⚠️ `docker commit` создаёт «чёрный ящик»: никто не знает, как он получен. Используй только для расследований (сохранить состояние сломанного контейнера), а образы собирай через Dockerfile.

## 3.11. Переменные окружения

```bash
docker run -e APP_ENV=prod -e DEBUG app          # DEBUG без значения — возьмётся из текущей оболочки
docker run --env-file ./prod.env app
docker exec web env                              # посмотреть внутри
```

Формат `.env`: `KEY=value` по одной на строку, без `export`; кавычки в `--env-file` воспринимаются буквально.

🔴 Переменные окружения видны в `docker inspect` всем, у кого есть доступ к демону. Для паролей лучше секреты-файлы (модуль 9).

## ✅ Проверь себя

1. Чем `docker stop` отличается от `docker kill`?
2. Как отключиться от `docker attach`, не остановив контейнер?
3. Контейнер завершился с кодом 137. Что проверить первым делом?
4. Как сделать, чтобы порт БД был доступен только с самого сервера?

<details><summary>Ответы</summary>

1. `stop` сначала шлёт SIGTERM и даёт время завершиться, `kill` сразу шлёт SIGKILL (или указанный сигнал).
2. Ctrl+P, затем Ctrl+Q.
3. `docker inspect -f '{{.State.OOMKilled}}'` и лимиты памяти; также `dmesg -T | grep -i oom` на хосте.
4. `-p 127.0.0.1:5432:5432` — или вообще не публиковать порт, а ходить в БД через общую docker-сеть.

</details>

[← Установка](02-install.md) · [На главную](../README.md) · [Образы →](04-images.md)
