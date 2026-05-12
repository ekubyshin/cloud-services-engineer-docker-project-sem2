# Momo Store Docker Project

Проект содержит Go backend и Vue frontend, подготовленные к запуску в Docker и Docker Compose. Статика собирается в образе `frontend` и копируется в общий volume; раздачу и прокси к API выполняет отдельный контейнер `nginx` (образ `nginxinc/nginx-unprivileged`).

Основной docker-стек (`backend`, `frontend`, nginx) включён в профиль **`local`**, чтобы не мешать профилю **`dev`** (hot reload в `docker-compose.dev.yml`).

## Переменные Окружения

Основные переменные:

- `APP_VERSION` — тег собираемых образов, по умолчанию `1.0.0`.
- `TZ` — часовой пояс контейнеров, по умолчанию `UTC`.
- `FRONTEND_PORT` — порт nginx (статика и `/api`), по умолчанию `8080`.
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
