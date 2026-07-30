# Scenario 04 — Certificate / Private Key Mismatch

## Situation

After an emergency cert rotation, nginx refuses to start.
The on-call engineer rotated the cert but something went wrong.
The previous engineer is offline. You need to fix it.

## Setup

```bash
docker compose up -d --build
```

## Symptom

Check container logs immediately:
```bash
docker compose logs server
```

You will see nginx failing to start with something like:
```
nginx: [emerg] SSL_CTX_use_PrivateKey_file("/etc/nginx/ssl/server.key") failed
(SSL: ... key values mismatch)
```

The service is completely down — nginx never started.

## Your Mission

1. **Confirm** the mismatch. Prove that `server.crt` and `server.key` are not a pair.
2. **Find** the correct key that was originally used to sign `server.crt`.
3. **Fix it**: Replace the mismatched key with the correct one.
4. **Start** nginx and verify HTTPS works.

## Hints

<details>
<summary>Hint 1 — How to check if a cert and key match</summary>

Compare the public key fingerprints — they must be identical:
```bash
docker compose exec server sh -c \
  'openssl x509 -noout -modulus -in /etc/nginx/ssl/server.crt | openssl md5'

docker compose exec server sh -c \
  'openssl rsa -noout -modulus -in /etc/nginx/ssl/server.key | openssl md5'

# If the MD5 hashes differ, the cert and key are NOT a pair.
```
</details>

<details>
<summary>Hint 2 — Find the correct key</summary>

```bash
# List all keys in the ssl directory
docker compose exec server ls -la /etc/nginx/ssl/

# There's another key file — check its modulus against the cert
docker compose exec server sh -c \
  'openssl rsa -noout -modulus -in /etc/nginx/ssl/real.key | openssl md5'
```
</details>

<details>
<summary>Hint 3 — Fix it</summary>

```bash
docker compose exec server sh -c \
  'cp /etc/nginx/ssl/real.key /etc/nginx/ssl/server.key'

docker compose exec server nginx -t
docker compose exec server nginx  # start nginx (it wasn't running)
```
</details>

## Verification

```bash
openssl s_client -connect localhost:9004 \
  -CAfile <(docker compose exec server cat /etc/nginx/ssl/ca.crt)
# Verify return code: 0 (ok)
```

## Key Lesson

> A cert and its private key are mathematically linked (same public key).
> Mismatches happen during cert rotations when new certs are deployed without their matching key.
>
> **Quick verification trick (memorize this):**
> ```bash
> diff \
>   <(openssl x509 -noout -modulus -in server.crt | openssl md5) \
>   <(openssl rsa -noout -modulus -in server.key | openssl md5)
> # No output = they match. Any output = mismatch.
> ```
