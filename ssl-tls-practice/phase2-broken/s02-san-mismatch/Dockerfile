FROM alpine:3.19
RUN apk add --no-cache nginx openssl bash curl

RUN mkdir -p /etc/nginx/ssl /var/www/html /run/nginx
RUN echo "<h1>Scenario 02 - SAN Mismatch</h1>" > /var/www/html/index.html

# Generate CA
RUN openssl genrsa -out /etc/nginx/ssl/ca.key 2048 && \
    openssl req -new -x509 -days 3650 \
      -key /etc/nginx/ssl/ca.key \
      -out /etc/nginx/ssl/ca.crt \
      -subj "/CN=LabRootCA"

# Generate server key + CSR
RUN openssl genrsa -out /etc/nginx/ssl/server.key 2048 && \
    openssl req -new \
      -key /etc/nginx/ssl/server.key \
      -out /etc/nginx/ssl/server.csr \
      -subj "/CN=internal-api"

# SAN extension file: cert only valid for "internal-api", NOT "localhost"
RUN printf '[v3_req]\nsubjectAltName=DNS:internal-api\n' > /tmp/san.ext

# Sign with CA + the restricting SAN extension
RUN openssl x509 -req \
      -in /etc/nginx/ssl/server.csr \
      -CA /etc/nginx/ssl/ca.crt \
      -CAkey /etc/nginx/ssl/ca.key \
      -CAcreateserial \
      -out /etc/nginx/ssl/server.crt \
      -days 365 \
      -extfile /tmp/san.ext \
      -extensions v3_req

COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 443
CMD ["nginx", "-g", "daemon off;"]
