FROM golang:1.25-alpine AS builder

WORKDIR /src

ENV CGO_ENABLED=0

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN go build \
    -trimpath \
    -ldflags="-s -w" \
    -o /out/momo-store-api \
    ./cmd/api

# --- runtime: small image, non-root user ---
FROM alpine:3.22

LABEL org.opencontainers.image.title="momo-store-backend" \
      org.opencontainers.image.description="Go API backend for momo-store"

RUN addgroup -S app && adduser -S -G app -H -h /nonexistent app \
    && apk add --no-cache ca-certificates

WORKDIR /app

COPY --from=builder --chown=app:app /out/momo-store-api ./momo-store-api

USER app

EXPOSE 8081

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -q -O /dev/null http://127.0.0.1:8081/health || exit 1

ENTRYPOINT ["./momo-store-api"]
