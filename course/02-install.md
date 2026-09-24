# 🟢 Модуль 2. Установка и настройка Docker

[← Теория](01-theory.md) · [На главную](../README.md) · [Контейнеры →](03-containers.md)

## 2.1. Что ставить

| Система | Вариант | Комментарий |
|---|---|---|
| Linux-сервер | **Docker Engine** (docker-ce) из репозитория Docker | Стандарт для продакшена |
| Linux-десктоп | Docker Engine или Docker Desktop | Engine легче и быстрее |
| macOS | Docker Desktop, **OrbStack**, Colima | OrbStack и Colima — лёгкие альтернативы |
| Windows | Docker Desktop + **WSL 2** | Проекты храни внутри WSL (`~/project`), а не на `C:\` — в разы быстрее |

⚠️ Пакет `docker.io` из репозиториев Ubuntu/Debian часто устаревший. Ставь из официального репозитория Docker.

💡 Docker Desktop бесплатен для личного использования, обучения и малого бизнеса; большим компаниям нужна платная подписка. Docker Engine на Linux — полностью бесплатный open source. Актуальные условия — на docker.com.

## 2.2. Ubuntu / Debian

```bash
# 1. Удалить старые/неофициальные пакеты
for p in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove -y $p; done

# 2. Добавить репозиторий Docker
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 3. Установить
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 4. Проверить
sudo docker run --rm hello-world
```

Для Debian замени в URL `ubuntu` на `debian`.

## 2.3. Fedora / RHEL / Rocky / AlmaLinux

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo   # для Fedora: .../linux/fedora/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```

В новых версиях dnf5 синтаксис другой: `sudo dnf config-manager addrepo --from-repofile=URL`.

## 2.4. Быстрый скрипт (для тестовых машин)

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
less get-docker.sh          # сначала прочитай, что запускаешь!
sudo sh get-docker.sh
```

🔴 Не делай `curl ... | sh` с неизвестных сайтов. В продакшене ставь через репозиторий или Ansible.

## 2.5. После установки

```bash
sudo systemctl enable --now docker containerd   # автозапуск
sudo usermod -aG docker $USER                   # работать без sudo
newgrp docker                                   # применить без перелогина (или перезайди)
docker run --rm hello-world
```

🔴 Группа `docker` = root-доступ к хосту. На общих серверах добавляй туда только тех, кому доверяешь как админу, или используй rootless-режим.

### Проверка установки

| Команда | Что показывает |
|---|---|
| `docker version` | Версии клиента и сервера (если сервера нет — демон не запущен или нет прав) |
| `docker info` | Драйвер хранения, cgroup, число контейнеров, зеркала, предупреждения |
| `docker compose version` | Версия Compose v2 (плагин) |
| `docker buildx version` | Версия buildx |
| `systemctl status docker` | Состояние службы |
| `journalctl -u docker -f` | Логи демона |

💡 `docker-compose` (через дефис) — старая Python-версия v1, уже не поддерживается. Используй `docker compose` (через пробел).

## 2.6. Настройка демона: /etc/docker/daemon.json

```json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" },
  "live-restore": true,
  "default-address-pools": [{ "base": "172.20.0.0/14", "size": 24 }],
  "registry-mirrors": ["https://mirror.gcr.io"],
  "features": { "buildkit": true },
  "data-root": "/var/lib/docker"
}
```

```bash
sudo systemctl restart docker   # применить
docker info | grep -iA3 "logging\|mirrors\|root dir"
```

| Параметр | Зачем |
|---|---|
| `log-opts.max-size` | Без него логи контейнеров растут бесконечно и забивают диск — **настрой всегда** |
| `live-restore` | Контейнеры не останавливаются при перезапуске/обновлении dockerd (не работает со Swarm) |
| `default-address-pools` | Избежать конфликта подсетей Docker с корпоративной сетью/VPN |
| `registry-mirrors` | Зеркало Docker Hub: быстрее и меньше упираешься в лимиты pull |
| `data-root` | Перенести данные Docker на большой диск |
| `insecure-registries` | Разрешить реестр без TLS (только для лаборатории!) |
| `userns-remap` | Сопоставить root контейнера с непривилегированным UID хоста |
| `dns` | DNS-серверы для контейнеров по умолчанию |

⚠️ Ошибка в JSON = демон не стартует. Проверяй: `sudo dockerd --validate --config-file /etc/docker/daemon.json` или `jq . /etc/docker/daemon.json`.

### Перенести /var/lib/docker на другой диск

```bash
sudo systemctl stop docker docker.socket containerd
sudo rsync -aHAX /var/lib/docker/ /data/docker/
# в daemon.json: "data-root": "/data/docker"
sudo systemctl start docker
docker info | grep "Docker Root Dir"
# убедившись, что всё работает: sudo rm -rf /var/lib/docker.old
```

## 2.7. Rootless-режим 🟡

Демон и контейнеры работают от обычного пользователя. Даже побег из контейнера не даст root на хосте.

```bash
sudo apt-get install -y uidmap dbus-user-session
dockerd-rootless-setuptool.sh install
systemctl --user enable --now docker
sudo loginctl enable-linger $USER          # работать без активной сессии
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
docker context use rootless
```

Ограничения: порты < 1024 нужно разрешать отдельно (`net.ipv4.ip_unprivileged_port_start=80`), сеть немного медленнее, нет `--net=host` в привычном смысле, некоторые лимиты cgroups требуют делегирования.

## 2.8. Контексты и удалённый Docker

CLI может управлять демоном на другой машине — удобнее всего через SSH:

```bash
docker context create prod --docker "host=ssh://deploy@203.0.113.10"
docker context ls
docker context use prod           # все команды теперь идут на prod
docker ps
docker context use default

docker --context prod ps          # разово
DOCKER_HOST=ssh://deploy@203.0.113.10 docker ps
```

🔴 Никогда не открывай API демона на `tcp://0.0.0.0:2375` без TLS — боты находят такие серверы за минуты и ставят майнеры.

## 2.9. Полезные настройки клиента

```bash
# Автодополнение (обычно ставится с пакетом)
docker completion bash | sudo tee /etc/bash_completion.d/docker > /dev/null

# Алиасы в ~/.bashrc
alias d='docker'
alias dc='docker compose'
alias dps='docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"'
alias dlog='docker logs -f --tail 100'
dsh() { docker exec -it "$1" sh -c 'command -v bash >/dev/null && exec bash || exec sh'; }
```

Файл `~/.docker/config.json` хранит токены реестров (`docker login`), формат вывода по умолчанию и настройки прокси:

```json
{
  "psFormat": "table {{.Names}}\t{{.Status}}\t{{.Ports}}",
  "credsStore": "desktop",
  "proxies": { "default": { "httpProxy": "http://proxy:3128", "noProxy": "localhost,127.0.0.1" } }
}
```

💡 Без credential helper токены в `config.json` лежат в base64 — по сути, открытым текстом. На рабочих машинах используй `docker-credential-pass` / `osxkeychain` / `wincred`.

## 2.10. Прокси для демона (корпоративная сеть)

Демону прокси нужен для `docker pull`, и он не читает переменные твоей оболочки:

```bash
sudo systemctl edit docker
# [Service]
# Environment="HTTP_PROXY=http://proxy:3128" "HTTPS_PROXY=http://proxy:3128" "NO_PROXY=localhost,127.0.0.1,.corp"
sudo systemctl daemon-reload && sudo systemctl restart docker
```

## 2.11. Удаление Docker 🔴

```bash
sudo apt-get purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm -rf /var/lib/docker /var/lib/containerd    # 🔴 удалит ВСЕ образы, контейнеры и тома
```

## ✅ Проверь себя

1. Почему после `usermod -aG docker` команда всё ещё требует sudo?
2. Какая настройка daemon.json спасёт от забитого логами диска?
3. Как выполнить `docker ps` на удалённом сервере, не заходя на него по SSH вручную?

<details><summary>Ответы</summary>

1. Группы применяются при входе в систему — нужно перезайти или выполнить `newgrp docker`.
2. `"log-opts": {"max-size": "10m", "max-file": "3"}`.
3. `docker -H ssh://user@host ps` или через `docker context`.

</details>

[← Теория](01-theory.md) · [На главную](../README.md) · [Контейнеры →](03-containers.md)
