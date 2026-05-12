# Momo Store Docker Project

Проект содержит Go backend и Vue frontend, подготовленные к запуску в Docker и Docker Compose.

#  Если не будет хватать до отлично, пожалуйста, верните на доработку!

## Быстрый Запуск

Подготовьте локальный файл секрета:

```bash
mkdir -p secrets
printf 'change-me-local-secret\n' > secrets/backend_api_token
```

```bash
BACKEND_SECRET_FILE=./secrets/backend_api_token docker compose up -d --build
```

Backend напрямую наружу не публикуется. Frontend проксирует API-запросы через nginx:

```text
http://localhost:8080/api/health
```

## Development Profile

Dev-сервисы запускаются отдельным профилем:

```bash
docker compose --profile dev up -d --build
```

## Переменные Окружения

Основные переменные:

- `APP_VERSION` — тег собираемых образов, по умолчанию `1.0.0`.
- `FRONTEND_PORT` — порт frontend в production, по умолчанию `8080`.
- `VUE_APP_API_URL` — API base URL для production-сборки frontend, по умолчанию `/api`.
- `BACKEND_SECRET_FILE` — путь к локальному файлу Docker Secret.
- `DEV_API_URL` — API URL для dev frontend, по умолчанию `http://localhost:8081`.
- `FRONTEND_DEV_PORT` — порт dev frontend, по умолчанию `3000`.
- `BACKEND_DEV_PORT` — порт dev backend, по умолчанию `8081`.

## Trivy

Сканирование образов настроено в GitHub Actions через `aquasecurity/trivy-action`.
