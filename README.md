# Momo Store Docker Project

Проект содержит Go backend и Vue frontend, подготовленные к запуску в Docker и Docker Compose. Статика собирается в образе `frontend` и копируется в общий volume; раздачу и прокси к API выполняет отдельный контейнер `nginx` (образ `nginxinc/nginx-unprivileged`).

Основной docker-стек (`backend`, `frontend`, nginx) включён в профиль **`local`**, чтобы не мешать профилю **`dev`** (hot reload в `docker-compose.dev.yml`).

## Запуск со скейлингом backend

Несколько реплик сервиса `backend` поднимаются флагом **`--scale`**. Контейнер **`nginx`** проксирует запросы к `/api` на имя сервиса `backend` в Docker-сети; встроенный DNS при нескольких репликах отдаёт несколько адресов, а в `frontend/nginx/default.conf` включены **`resolver 127.0.0.11`** и **`proxy_pass` через переменную**, чтобы nginx периодически заново резолвил имя и распределял нагрузку между инстансами.

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml --profile prod up -d --scale backend=3
```

## Ресурсы контейнеров

В `docker-compose.yml` для сервисов **`backend`** и **`nginx`** заданы `deploy.resources` (лимиты и минимальные резервации).

**Backend (лимит 0.75 CPU, 256 Mi RAM; резервация 0.05 CPU, 64 Mi RAM).** Небольшой сервис на Go типичное потребление памяти у такого процесса невелико, но лимит 256 Mi даёт запас под рост кучи при нагрузке, работу сборщика мусора и всплески одновременных запросов, не позволяя контейнеру выходить за пределы.

**Nginx (лимит 0.50 CPU, 128 Mi RAM; резервация 0.05 CPU, 32 Mi RAM).** Здесь только раздача статики и проксирование `/api`; рабочая нагрузка CPU обычно низкая, 0.50 CPU хватает на пики при одновременной отдаче файлов и буферизации ответов бэкенда. Память 128 Mi покрывает процесс nginx, кэш и служебные буферы при типичных значениях по умолчанию; tmpfs для кэша и временных файлов ограничен отдельно в compose и не должен раздувать RSS без лимита контейнера. Резервация 32 Mi отражает, что в простое прокси потребляет мало памяти.

## Переменные Окружения

Основные переменные:

- `APP_VERSION` — тег собираемых образов, по умолчанию `1.0.0`.
- `TZ` — часовой пояс контейнеров, по умолчанию `UTC`.
- `FRONTEND_PORT` — порт nginx (статика и `/api`), по умолчанию `80`.
- `VUE_APP_API_URL` — API base URL для production-сборки frontend, по умолчанию `/api`.
- `BACKEND_SECRET_FILE` — путь к локальному файлу Docker Secret.
- `DOCKER_USER` — логин Docker Hub для образов в `docker-compose.prod.yml` (формат `user/docker-project-backend`).
- `docker-compose.dev.yml` — `backend-dev` / `frontend-dev` (профиль `dev`); основной compose подключайте тем же `-f docker-compose.yml`.
- `IMAGE_TAG` — тег образов в prod-compose, по умолчанию `latest` (CI пушит `latest` на main).
- `DEV_API_URL` — API URL для dev frontend, по умолчанию `http://localhost:8081`.
- `FRONTEND_DEV_PORT` — порт dev frontend, по умолчанию `3000`.
- `BACKEND_DEV_PORT` — порт dev backend, по умолчанию `8081`.

## Trivy

Сканирование образов настроено в GitHub Actions через `aquasecurity/trivy-action`.
