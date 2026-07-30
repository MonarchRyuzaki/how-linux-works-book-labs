FROM alpine:3.19
RUN apk add --no-cache nginx openssl bash curl

RUN mkdir -p /etc/nginx/ssl /var/www/html /run/nginx
RUN echo "<h1>Scenario 03 - Missing Chain</h1>" > /var/www/html/index.html

# Root CA
RUN openssl genrsa -out /etc/nginx/ssl/root-ca.key 2048 && \
    openssl req -new -x509 -days 3650 \
      -key /etc/nginx/ssl/root-ca.key \
      -out /etc/nginx/ssl/root-ca.crt \
      -subj "/CN=LabRootCA"

# Intermediate CA key + CSR
RUN openssl genrsa -out /etc/nginx/ssl/inter-ca.key 2048 && \
    openssl req -new \
      -key /etc/nginx/ssl/inter-ca.key \
      -out /etc/nginx/ssl/inter-ca.csr \
      -subj "/CN=LabIntermediateCA"

# Sign intermediate with root (mark it as a CA)
RUN printf 'basicConstraints=CA:TRUE\nkeyUsage=keyCertSign,cRLSign\n' > /tmp/inter.ext && \
    openssl x509 -req \
      -in /etc/nginx/ssl/inter-ca.csr \
      -CA /etc/nginx/ssl/root-ca.crt \
      -CAkey /etc/nginx/ssl/root-ca.key \
      -CAcreateserial \
      -out /etc/nginx/ssl/inter-ca.crt \
      -days 3650 \
      -extfile /tmp/inter.ext

# Server cert — signed by INTERMEDIATE (not root)
RUN openssl genrsa -out /etc/nginx/ssl/server.key 2048 && \
    openssl req -new \
      -key /etc/nginx/ssl/server.key \
      -out /etc/nginx/ssl/server.csr \
      -subj "/CN=localhost" && \
    openssl x509 -req \
      -in /etc/nginx/ssl/server.csr \
      -CA /etc/nginx/ssl/inter-ca.crt \
      -CAkey /etc/nginx/ssl/inter-ca.key \
      -CAcreateserial \
      -out /etc/nginx/ssl/server.crt \
      -days 365

# THE BUG: nginx.conf points to server.crt ONLY (no intermediate bundled)
# client only gets the leaf cert and cannot build the chain to root-ca

COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 443
CMD ["nginx", "-g", "daemon off;"]
