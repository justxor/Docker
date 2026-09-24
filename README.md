# 🐳 Docker Cheat Sheet — шпаргалка и курс по Docker и контейнеризации

**Самый полный курс-шпаргалка по Docker на русском**: теория простым языком (что такое контейнер, чем он отличается от виртуальной машины, как устроены образы, слои, namespaces и cgroups), все важные команды, Dockerfile и BuildKit, тома, сети, Docker Compose, безопасность, продакшен, CI/CD, Kubernetes и Podman — с примерами. Плюс диагностика «что делать, если…», вопросы с собеседований и практикум с задачами и решениями.

⭐ **Поставь звезду, чтобы шпаргалка всегда была под рукой!**

**Для кого:** разработчики, DevOps / SRE-инженеры, системные администраторы, студенты, QA, все, кто готовится к собеседованию или хочет наконец разобраться, «что там внутри у контейнеров». Работает на Linux, macOS (Docker Desktop / OrbStack / Colima) и Windows (Docker Desktop + WSL 2).

> Нужна база по Linux? Смотри соседний репозиторий — [Linux шпаргалка](https://github.com/justxor/Linux-).

## Легенда

| Значок | Значение |
|---|---|
| 🟢 | База — нужно знать каждому |
| 🟡 | Продвинутый уровень |
| 🔴 | Опасно: может удалить данные или сломать окружение |
| 💡 | Совет / лайфхак |
| ⚠️ | Частая ошибка |

## 🗺 Как проходить курс

```
 Неделя 1: теория → установка → контейнеры → образы            (модули 1–4)
 Неделя 2: Dockerfile → тома → сети → Compose                  (модули 5–8)
 Неделя 3: безопасность → продакшен → CI/CD                    (модули 9–11)
 Неделя 4: оркестрация → диагностика → собеседование → практика (модули 12–15)
```

Каждый модуль — самостоятельная шпаргалка: можно читать подряд как курс, а можно открывать нужный раздел как справочник.

## 📚 Содержание

| № | Модуль | Что внутри |
|---|---|---|
| 1 | [🧠 Теория: как устроены контейнеры](course/01-theory.md) | VM vs контейнер, namespaces, cgroups, OverlayFS, образ/слой/контейнер, OCI, dockerd → containerd → runc |
| 2 | [🟢 Установка и настройка](course/02-install.md) | Linux, macOS, Windows/WSL 2, группа docker, rootless, daemon.json, зеркала, контексты |
| 3 | [🟢 Контейнеры](course/03-containers.md) | docker run и все важные флаги, жизненный цикл, exec, logs, inspect, stats, cp, лимиты |
| 4 | [🟢 Образы и реестры](course/04-images.md) | pull/push, теги и digest, Docker Hub, GHCR, свой registry, save/load, очистка |
| 5 | [🟢 Dockerfile и сборка](course/05-dockerfile.md) | все инструкции, кэш слоёв, multi-stage, BuildKit, buildx, multi-arch, шаблоны для Python/Node/Go/Java |
| 6 | [🟢 Данные: тома и монтирование](course/06-volumes.md) | volumes, bind mounts, tmpfs, права, бэкап и восстановление, драйверы |
| 7 | [🟡 Сети](course/07-networking.md) | bridge, host, none, overlay, macvlan, DNS внутри Docker, порты, iptables, диагностика |
| 8 | [🟢 Docker Compose](course/08-compose.md) | compose.yaml от А до Я, команды, healthcheck, depends_on, профили, override, watch, полный стек |
| 9 | [🟡 Безопасность](course/09-security.md) | non-root, capabilities, seccomp, read-only, секреты, сканирование (Trivy, Scout), подпись образов |
| 10 | [🟡 Продакшен](course/10-production.md) | лимиты, restart, логи, мониторинг, обновления без простоя, reverse proxy, бэкапы |
| 11 | [🟡 CI/CD](course/11-ci-cd.md) | GitHub Actions, GitLab CI, кэш сборки, теги по версиям, сканирование в пайплайне |
| 12 | [🟡 Оркестрация: Swarm, Kubernetes, Podman](course/12-orchestration.md) | Swarm за 10 минут, основы Kubernetes и kubectl, Podman, containerd/nerdctl |
| 13 | [🔴 Диагностика: что делать, если…](course/13-troubleshooting.md) | контейнер падает, нет сети, кончилось место, OOM, медленная сборка + опасные команды |
| 14 | [🎯 Вопросы с собеседований](course/14-interview.md) | 60+ вопросов с ответами: от junior до senior |
| 15 | [🎓 Практикум](course/15-practice.md) | 25 задач с решениями: от hello-world до продакшен-стека |

---

## ⚡ Шпаргалка на одну страницу

### Контейнеры

| Команда | Что делает |
|---|---|
| `docker run -d --name web -p 8080:80 nginx` | Запустить в фоне с пробросом порта |
| `docker run -it --rm ubuntu bash` | Интерактивный одноразовый контейнер |
| `docker ps` / `docker ps -a` | Работающие / все контейнеры |
| `docker logs -f --tail 100 web` | Логи в реальном времени |
| `docker exec -it web sh` | Зайти внутрь работающего контейнера |
| `docker stop web` / `docker start web` | Остановить / запустить |
| `docker restart web` | Перезапустить |
| `docker rm -f web` | 🔴 Удалить (даже работающий) |
| `docker inspect web` | Вся информация в JSON |
| `docker stats` | CPU / RAM / сеть в реальном времени |
| `docker cp web:/etc/nginx/nginx.conf .` | Скопировать файл из контейнера |
| `docker top web` | Процессы внутри контейнера |

### Образы

| Команда | Что делает |
|---|---|
| `docker build -t app:1.0 .` | Собрать образ из Dockerfile |
| `docker images` | Список образов |
| `docker pull postgres:16` | Скачать образ |
| `docker tag app:1.0 user/app:1.0` | Добавить тег |
| `docker push user/app:1.0` | Отправить в реестр |
| `docker rmi app:1.0` | Удалить образ |
| `docker history app:1.0` | Слои образа и их размер |
| `docker save app:1.0 -o app.tar` / `docker load -i app.tar` | Образ в файл и обратно |

### Тома, сети, Compose

| Команда | Что делает |
|---|---|
| `docker volume create data` | Создать том |
| `docker run -v data:/var/lib/postgresql/data postgres` | Подключить том |
| `docker run -v "$PWD":/app node` | Смонтировать текущую папку |
| `docker network create backend` | Создать сеть |
| `docker run --network backend --name db postgres` | Контейнер в сети (доступен по имени `db`) |
| `docker compose up -d` | Поднять весь стек |
| `docker compose logs -f api` | Логи сервиса |
| `docker compose down` | Остановить и удалить контейнеры и сети |
| `docker compose down -v` | 🔴 …и тома с данными |

### Уборка

| Команда | Что делает |
|---|---|
| `docker system df` | Сколько места занимает Docker |
| `docker container prune` | Удалить остановленные контейнеры |
| `docker image prune -a` | Удалить неиспользуемые образы |
| `docker builder prune` | Очистить кэш сборки |
| `docker system prune -a --volumes` | 🔴 Удалить ВСЁ неиспользуемое, включая тома |

### Минимальный Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
USER nobody
EXPOSE 8000
CMD ["python", "app.py"]
```

### Минимальный compose.yaml

```yaml
services:
  app:
    build: .
    ports: ["8000:8000"]
    environment:
      DATABASE_URL: postgres://app:secret@db:5432/app
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    volumes: [pgdata:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
volumes:
  pgdata:
```

---

## 📖 Глоссарий

| Термин | Простыми словами |
|---|---|
| **Контейнер** | Обычный процесс Linux, изолированный namespaces и ограниченный cgroups, со своей файловой системой из образа |
| **Образ (image)** | Неизменяемый шаблон: слои файловой системы + метаданные (команда запуска, переменные, порты) |
| **Слой (layer)** | Набор изменений ФС от одной инструкции Dockerfile; слои кэшируются и переиспользуются |
| **Dockerfile** | Рецепт сборки образа |
| **Реестр (registry)** | Хранилище образов: Docker Hub, GHCR, GitLab Registry, Harbor, свой `registry:2` |
| **Репозиторий** | Набор образов с одним именем и разными тегами (`nginx:1.27`, `nginx:alpine`) |
| **Тег** | Человекочитаемая, но изменяемая метка версии образа |
| **Digest** | Неизменяемый SHA256-хэш образа: `nginx@sha256:…` |
| **Том (volume)** | Хранилище данных вне жизненного цикла контейнера |
| **Bind mount** | Папка хоста, смонтированная в контейнер |
| **Compose** | Описание многоконтейнерного приложения в YAML |
| **BuildKit** | Современный движок сборки: параллельность, кэш, секреты, multi-arch |
| **OCI** | Open Container Initiative — открытые стандарты образов и рантаймов |
| **containerd** | Демон, управляющий жизненным циклом контейнеров (под Docker и Kubernetes) |
| **runc** | Низкоуровневый рантайм, который реально создаёт контейнер через системные вызовы ядра |
| **namespace** | Механизм ядра, дающий процессу «свой» вид на PID, сеть, ФС, hostname и т.д. |
| **cgroup** | Механизм ядра для лимитов и учёта ресурсов: CPU, память, I/O, число процессов |
| **Оркестратор** | Система управления контейнерами на многих серверах: Kubernetes, Swarm, Nomad |
| **Pod** | Минимальная единица Kubernetes: один или несколько контейнеров с общей сетью |
| **Rootless** | Режим, в котором демон и контейнеры работают без root на хосте |

---

## 🤝 Как помочь проекту

Нашёл ошибку или знаешь полезную команду? Открой Issue или Pull Request. Если шпаргалка помогла — поставь ⭐, это лучшая благодарность.

**Лицензия:** материалы можно свободно использовать для обучения со ссылкой на репозиторий.
