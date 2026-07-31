FROM alpine:3.19
RUN apk add --no-cache nginx openssl bash curl

RUN mkdir -p /etc/nginx/ssl /var/www/html /run/nginx
RUN echo "<h1>mTLS Server — Auth OK</h1>" > /var/www/html/index.html

COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 443
CMD ["nginx", "-g", "daemon off;"]
