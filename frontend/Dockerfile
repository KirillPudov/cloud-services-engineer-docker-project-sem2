FROM node:18-alpine AS builder

ARG NODE_ENV

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

ENV NODE_ENV=${NODE_ENV}
ENV NODE_OPTIONS=--openssl-legacy-provider

RUN npm run build

FROM nginx:1.27-alpine

ARG NODE_ENV

ENV NODE_ENV=${NODE_ENV}
ENV VUE_APP_API_URL=/api

COPY nginx.conf /etc/nginx/conf.d/default.conf

COPY --from=builder --chown=appuser:appgroup /app/dist /usr/share/nginx/html

RUN apk update && \
    apk add --no-cache curl &&\
    addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup &&\
    chown -R appuser:appgroup /var/cache/nginx && \
    chown -R appuser:appgroup /var/log/nginx && \
    chown -R appuser:appgroup /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R appuser:appgroup /var/run/nginx.pid &&\
    chown -R appuser:appgroup /etc/nginx

USER appuser

EXPOSE 80

# healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

ENTRYPOINT ["/app/docker-entrypoint.sh"]
CMD ["forego", "start", "-r"]