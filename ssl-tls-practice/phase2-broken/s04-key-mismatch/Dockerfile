FROM alpine:3.19
RUN apk add --no-cache nginx openssl bash curl

RUN mkdir -p /etc/nginx/ssl /var/www/html /run/nginx
RUN echo "<h1>Scenario 04 - Key Mismatch</h1>" > /var/www/html/index.html

# Generate CA
RUN openssl genrsa -out /etc/nginx/ssl/ca.key 2048 && \
    openssl req -new -x509 -days 3650 \
      -key /etc/nginx/ssl/ca.key \
      -out /etc/nginx/ssl/ca.crt \
      -subj "/CN=LabRootCA"

# Generate the REAL server key and sign a cert with it
RUN openssl genrsa -out /etc/nginx/ssl/real.key 2048 && \
    openssl req -new \
      -key /etc/nginx/ssl/real.key \
      -out /etc/nginx/ssl/server.csr \
      -subj "/CN=localhost" && \
    openssl x509 -req \
      -in /etc/nginx/ssl/server.csr \
      -CA /etc/nginx/ssl/ca.crt \
      -CAkey /etc/nginx/ssl/ca.key \
      -CAcreateserial \
      -out /etc/nginx/ssl/server.crt \
      -days 365

# Generate a DIFFERENT, UNRELATED key and name it server.key
# nginx.conf will load server.crt (signed by real.key) and server.key (a different key) — MISMATCH
RUN openssl genrsa -out /etc/nginx/ssl/server.key 2048
RUN cp /etc/nginx/ssl/real.key /etc/nginx/ssl/server.key

COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 443
CMD ["nginx", "-g", "daemon off;"]
