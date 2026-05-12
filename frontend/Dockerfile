FROM node:20-alpine AS builder

WORKDIR /app

ARG VUE_APP_API_URL=
ENV VUE_APP_API_URL=${VUE_APP_API_URL}

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN NODE_OPTIONS=--openssl-legacy-provider npm run build && rm -rf node_modules

# --- runtime: minimal static server, non-root ---
FROM nginxinc/nginx-unprivileged:stable-alpine

LABEL org.opencontainers.image.title="momo-store-frontend" \
      org.opencontainers.image.description="Vue static frontend for momo-store"

WORKDIR /usr/share/nginx/html

COPY --chown=nginx:nginx nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder --chown=nginx:nginx /app/dist/ ./

USER nginx

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -q -O /dev/null http://127.0.0.1:8080/momo-store/ || exit 1
