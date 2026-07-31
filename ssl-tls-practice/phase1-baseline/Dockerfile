FROM alpine:3.19
RUN apk add --no-cache nginx openssl bash curl

RUN mkdir -p /etc/nginx/ssl /var/www/html /run/nginx

RUN echo "<h1>SSL Lab - Phase 1</h1><p>HTTPS is not configured yet. Follow task.md.</p>" \
    > /var/www/html/index.html

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
