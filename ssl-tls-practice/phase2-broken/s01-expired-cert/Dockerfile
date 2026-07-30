FROM alpine:3.19
RUN apk add --no-cache nginx openssl bash curl

RUN mkdir -p /etc/nginx/ssl /var/www/html /run/nginx
RUN echo "<h1>Scenario 01 - Expired Cert</h1>" > /var/www/html/index.html

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
      -subj "/CN=localhost"

# Sign with CA but with deliberately expired dates
# Valid: expired yesterday (-days -1)
RUN openssl x509 -req \
      -in /etc/nginx/ssl/server.csr \
      -CA /etc/nginx/ssl/ca.crt \
      -CAkey /etc/nginx/ssl/ca.key \
      -CAcreateserial \
      -out /etc/nginx/ssl/server.crt \
      -days -1

COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 443
CMD ["nginx", "-g", "daemon off;"]
