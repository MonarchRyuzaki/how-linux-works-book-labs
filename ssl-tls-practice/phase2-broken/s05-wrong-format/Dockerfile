FROM alpine:3.19
RUN apk add --no-cache nginx openssl bash curl

RUN mkdir -p /etc/nginx/ssl /var/www/html /run/nginx
RUN echo "<h1>Scenario 05 - Wrong Format</h1>" > /var/www/html/index.html

# Generate CA
RUN openssl genrsa -out /etc/nginx/ssl/ca.key 2048 && \
    openssl req -new -x509 -days 3650 \
      -key /etc/nginx/ssl/ca.key \
      -out /etc/nginx/ssl/ca.crt \
      -subj "/CN=LabRootCA"

# Generate server key (PEM — correct format)
RUN openssl genrsa -out /etc/nginx/ssl/server.key 2048

# Generate PEM cert, then CONVERT to DER format
# nginx.conf points to server.crt which is DER binary, not PEM text — nginx fails
RUN openssl req -new -x509 -days 365 \
      -key /etc/nginx/ssl/server.key \
      -out /tmp/server_pem.crt \
      -subj "/CN=localhost" && \
    openssl x509 \
      -in /tmp/server_pem.crt \
      -out /etc/nginx/ssl/server.crt \
      -outform DER

COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 443
CMD ["nginx", "-g", "daemon off;"]
