# Scenario 03 — Missing Certificate Chain (Incomplete Trust Chain)

## Situation

A new microservice was deployed with a cert signed by an Internal Intermediate CA.
It works in staging but clients in production are getting verification errors.
The cert hasn't expired. The hostname is correct.

## Setup

```bash
docker compose up -d --build
```

## Symptom

```bash
openssl s_client -connect localhost:9003 \
  -CAfile <(docker compose exec server cat /etc/nginx/ssl/root-ca.crt)
```

You will see:
```
verify error:num=20:unable to get local issuer certificate
...
Verify return code: 20 (unable to get local issuer certificate)
```

## Your Mission

1. **Understand** the 3-tier chain in this container: Root CA → Intermediate CA → Server cert.
   - What files exist in `/etc/nginx/ssl/`?
   - Who signed `server.crt`? Who signed `inter-ca.crt`?

=>  server, root-ca, inter-ca crt, key, csr and srl.
=>  server.crt is signed by inter-ca , inter-ca.crt is signed by root-ca.

2. **Explain** why the client can't verify. (Hint: what is nginx actually sending?)
=> nginx is only sending server.crt not inter-ca.crt

3. **Fix it**: Build a `fullchain.crt` that bundles the server cert + intermediate cert,
   and update `nginx.conf` to use it.
=> Did it 

4. **Verify** the full chain is now sent by the server.
=> Did it
## Hints

<details>
<summary>Hint 1 — Inspect who signed server.crt</summary>

```bash
docker compose exec server openssl x509 -in /etc/nginx/ssl/server.crt -noout -text \
  | grep -E "Issuer:|Subject:"
# Issuer should be "LabIntermediateCA", not "LabRootCA"
```
</details>

<details>
<summary>Hint 2 — See what the server is currently sending</summary>

```bash
openssl s_client -connect localhost:9003 -CAfile \
  <(docker compose exec server cat /etc/nginx/ssl/root-ca.crt) 2>&1 \
  | grep -E "Certificate chain|s:|i:"
# You'll only see 1 cert in the chain — the leaf. The intermediate is missing.
```
</details>

<details>
<summary>Hint 3 — Build a fullchain and fix nginx</summary>

Inside the container:
```bash
# Order matters: leaf cert FIRST, then intermediate
cat /etc/nginx/ssl/server.crt /etc/nginx/ssl/inter-ca.crt > /etc/nginx/ssl/fullchain.crt

# Update nginx.conf to use fullchain.crt
sed -i 's|server.crt|fullchain.crt|' /etc/nginx/nginx.conf

nginx -t && nginx -s reload
```
</details>

## Verification

```bash
openssl s_client -connect localhost:9003 \
  -CAfile <(docker compose exec server cat /etc/nginx/ssl/root-ca.crt) 2>&1 \
  | grep -E "Certificate chain|depth|Verify return"
# Should show depth=2 (root), depth=1 (intermediate), depth=0 (server)
# Verify return code: 0 (ok)
```

## Key Lesson

> This is one of the most common production cert issues.
> nginx's `ssl_certificate` must contain the **full chain**: leaf cert + all intermediate CAs.
> The Root CA is NOT included in the bundle (clients have that in their trust store).
>
> Format: `cat server.crt intermediate.crt > fullchain.crt`
> NOT: `cat intermediate.crt server.crt` (wrong order!)
