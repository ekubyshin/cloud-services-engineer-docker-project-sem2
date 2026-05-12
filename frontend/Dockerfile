FROM node:20-alpine AS builder

WORKDIR /app

ARG VUE_APP_API_URL=
ENV VUE_APP_API_URL=${VUE_APP_API_URL}

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN NODE_OPTIONS=--openssl-legacy-provider npm run build && rm -rf node_modules

# --- static assets only; entrypoint copies into volume mount /mnt (see docker-compose) ---
FROM alpine:3.22

LABEL org.opencontainers.image.title="momo-store-frontend" \
      org.opencontainers.image.description="Vue static assets for momo-store"

WORKDIR /opt/html
COPY --from=builder /app/dist/ ./
RUN chown -R 101:101 /opt/html

ENTRYPOINT ["sh", "-c", "cp -a /opt/html/. /mnt/ && exit 0"]
