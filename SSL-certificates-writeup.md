# How to create a valid self signed SSL Certificate?

I’ll give you **exact files and OpenSSL commands**, explain every option, show the exact **SAN** config, and finish with how to *trust* the root CA on Ubuntu and how to run/test a local HTTPS server.

> Work in one folder (example: `~/certs`). Run the commands there.

## 0 — Prep

```bash
mkdir -p ~/certs
cd ~/certs
```

## 1 — Create a Root CA (private key + self-signed root certificate)

**Files we’ll create**: `rootCA.key` (private key) and `rootCA.crt` (self-signed root CA cert).
**Config file**: `rootCA.cnf` — used to set CA extensions (`basicConstraints = CA:TRUE`, etc.).

Create `rootCA.cnf` (exact content — copy/paste to file):

```ini
[req]
default_bits       = 4096
prompt             = no
default_md         = sha256
distinguished_name = dn
x509_extensions    = v3_ca

[dn]
C  = MA
ST = Casablanca
L  = Casablanca
O  = MyLocalCA
CN = MyLocalRootCA

[v3_ca]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = critical, CA:TRUE
keyUsage               = critical, cRLSign, keyCertSign
```

Commands:

```bash
# 1) generate root private key (4096 bits)
openssl genrsa -out rootCA.key 4096

# 2) create a self-signed root certificate (valid 10 years = 3650 days)
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 \
  -out rootCA.crt -config rootCA.cnf
```

**Why & notes**:

* `-x509` => produce a self-signed certificate (root CA).
* `x509_extensions = v3_ca` tells OpenSSL to include CA extensions from `[v3_ca]`.
* Protect `rootCA.key` — it’s your master key. `chmod 600 rootCA.key` and store it safely.

## 2 — Create server key + CSR (with SAN config bundled)

We want a server cert that covers:

* `app.localhost`
* `go.localhost`
* `db.localhost`
* `localhost`
  and also the loopback IP `127.0.0.1` (helpful).

Create `server.cnf` (used both for CSR generation and later for signing):

```ini
[req]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
req_extensions     = req_ext

[dn]
C  = MA
ST = Casablanca
L  = Casablanca
O  = MyCompany
CN = app.localhost

[req_ext]
# this is the exact SAN configuration used when generating CSR or passing to -extfile
subjectAltName = @alt_names
subjectKeyIdentifier = hash

[alt_names]
DNS.1 = app.localhost
DNS.2 = go.localhost
DNS.3 = db.localhost
DNS.4 = localhost
IP.1  = 127.0.0.1
```

Commands:

```bash
# 1) generate server private key
openssl genrsa -out server.key 2048

# 2) generate CSR using server.cnf (the CSR will include req_extensions if requested)
openssl req -new -key server.key -out server.csr -config server.cnf
```

**Notes**:

* `req_extensions = req_ext` tells OpenSSL to look at `[req_ext]` when creating the CSR (so the CSR contains the SAN info).
* The `[alt_names]` block is the **exact SAN list** — these are the names that will be valid on the certificate.

## 3 — Sign the CSR with your Root CA (produce server certificate with SAN)

When signing the CSR with `openssl x509 -req`, OpenSSL will **not** automatically copy CSR SANs into the final cert unless you explicitly supply extensions at signing time. So we will use `-extfile server.cnf -extensions req_ext` to include the SAN and other extensions.

Command:

```bash
openssl x509 -req -in server.csr \
  -CA rootCA.crt -CAkey rootCA.key -CAcreateserial \
  -out server.crt -days 825 -sha256 \
  -extfile server.cnf -extensions req_ext
```

**What this does**:

* `-CAcreateserial` creates `rootCA.srl` (serial file).
* `-extfile server.cnf -extensions req_ext` tells OpenSSL “read the `[req_ext]` section in `server.cnf` and apply those extensions (including subjectAltName) to the issued certificate”.
* `-days 825` — recommended maximum for leaf cert lifetime for browser compatibility; since this is a local CA you could choose longer, but 825 is safe for modern browsers.

## 4 — Verify the certificate contains the SANs and is signed by your CA

Inspect certificate details:

```bash
openssl x509 -in server.crt -noout -text
```

Look for the section `X509v3 Subject Alternative Name` — it should list:

```
DNS:app.localhost, DNS:go.localhost, DNS:db.localhost, DNS:localhost, IP Address:127.0.0.1
```

Verify the chain (checks signature):

```bash
openssl verify -CAfile rootCA.crt server.crt
# should respond: server.crt: OK
```

Test an SSL handshake (assuming a server is listening on 127.0.0.1:8443):

```bash
openssl s_client -connect 127.0.0.1:8443 -servername app.localhost -CAfile rootCA.crt
# look for "Verify return code: 0 (ok)" at the end
```

## 5 — Install / trust the Root CA on Ubuntu system

Place the root CA certificate where Ubuntu expects local CA certs and run update:

```bash
sudo cp rootCA.crt /usr/local/share/ca-certificates/my-local-root-ca.crt
sudo update-ca-certificates
```

`update-ca-certificates` will:

* copy the cert into `/etc/ssl/certs/`
* add it to the system trust store so system programs (curl, apt, Chrome on Linux, etc.) will trust certificates signed by your root.

**Firefox**: Firefox uses its own certificate store. Either:

* Import `rootCA.crt` manually via *Preferences → Privacy & Security → View Certificates → Authorities → Import* (recommended for convenience), **or**
* Use `certutil` (CLI) for automated import:

```bash
# install helper if you want to use certutil
sudo apt install libnss3-tools

# find your Firefox profile folder (one example):
ls ~/.mozilla/firefox/*.default-release

# import (replace PROFILE with the exact profile directory)
certutil -d sql:$HOME/.mozilla/firefox/PROFILE -A -t "C,," -n "My Local Root CA" -i rootCA.crt
```

Finally, restart any browsers so they pick up the new trust store.

## 6 — Make sure your hostnames resolve to your local IP

Add lines to `/etc/hosts` (if needed; `.localhost` often resolves already but add to be explicit):

```bash
sudo nano /etc/hosts
# add this line:
127.0.0.1 app.localhost go.localhost db.localhost localhost
```

## 7 — Use the certificate with a local server (examples)

**Simple quick test with OpenSSL s\_server** (no extra software):

```bash
# run a basic HTTPS server on port 8443
openssl s_server -accept 8443 -cert server.crt -key server.key -www
```

Then in another terminal:

```bash
curl -v https://app.localhost:8443 --resolve app.localhost:8443:127.0.0.1
# or, to use the system trust (after installing rootCA), simply:
curl -v https://app.localhost:8443 --resolve app.localhost:8443:127.0.0.1
```

**Python http.server (quick dev)**:

```bash
# python 3.11+ supports --ssl-certfile and --ssl-keyfile
python3 -m http.server 8443 --bind 127.0.0.1 --directory . \
  --ssl-certfile server.crt --ssl-keyfile server.key
```

**Nginx example snippet**:

```nginx
server {
    listen 443 ssl;
    server_name app.localhost go.localhost db.localhost localhost;

    ssl_certificate     /home/you/certs/server.crt;
    ssl_certificate_key /home/you/certs/server.key;

    root /var/www/html;
    location / { try_files $uri $uri/ =404; }
}
```

After placing cert/key on the machine and configuring nginx, `sudo systemctl reload nginx`.

**Docker / Traefik**: mount `server.crt` & `server.key` into your container and point your TLS config to them (Traefik has specific label/config options).

## 8 — Test from browser / curl

* Browser: go to `https://app.localhost:8443` (or to `https://app.localhost` if you used port 443). You should **not** see certificate warnings after trusting root CA.
* `curl` (if you didn’t install root CA):
  `curl --cacert rootCA.crt https://app.localhost:8443 --resolve app.localhost:8443:127.0.0.1`
* `openssl s_client` test (use `-servername` to set SNI):

```bash
openssl s_client -connect 127.0.0.1:8443 -servername app.localhost -CAfile rootCA.crt
```

Check `X509v3 Subject Alternative Name` and `Verify return code: 0 (ok)`.

## 9 — Quick troubleshooting

* Browser still warns? Restart browser, ensure you installed root CA in the right place (system store vs Firefox store), and that the certificate actually includes the hostname in SAN (verify with `openssl x509 -in server.crt -noout -text`).
* `openssl verify` fails? Make sure you passed `-CAfile rootCA.crt` to the verify command.
* Wrong name in SAN? Recreate CSR/cert — SANs can only be set at signing time (or included in CSR and copied at signing if signer is instructed).
* Protect root key: `chmod 600 rootCA.key` and keep it offline if possible.
