# Simple TLS Example with Caddy

This project aims to provide the simplest starter-set to watch the TLS behavior for biginners.

## Requirements

- [Caddy](https://caddyserver.com)
- [OpenSSH](https://www.openssl.org/source/)
	- macOS: `brew install openssl`

## Setup

### Generate a Slef-signed Certificate

You must generate your self-signed certificate like below before start the server.
It generates a signed public key `cert.pem` and its paired private key `key.pem` under the `certs/` directory.

```sh
$ openssl req -x509 -newkey rsa:4096 -keyout certs/key.pem -out certs/cert.pem -days 365 -nodes
Generating a 4096 bit RSA private key
.......................................................................................................................++
..............................................................................++
writing new private key to 'key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:.
State or Province Name (full name) []:.
Locality Name (eg, city) []:.
Organization Name (eg, company) []:.
Organizational Unit Name (eg, section) []:.
Common Name (eg, fully qualified host name) []:localhost
Email Address []:.
```

then you will complete following ingredients.

```
$ tree
.
├── Caddyfile.http
├── Caddyfile.https
├── README.md
├── certs
│   ├── cert.pem
│   └── key.pem
└── public
    └── index.html
```

### Serve and Send Requests

Start the caddy server by

```
$ caddy -conf Caddyfile.https
```

then, in another terminal, you can send a HTTPS request by `curl`, simply, with `--insecure` option(it make the client skips verifying the certificate).

```
$ curl --insecure https://localhost:8443
A Simple TLS Example with Caddy works!
```

or you also be able to set `certs/cert.pem` as a trusted certificate.

```
$ curl --cacert certs/cert.pem https://localhost:8443
A Simple TLS Example with Caddy works!
```

## Extra Lessons

### Use curl verbose option

Using `-v` (`--verbose`) is a good way to watch the TLS handshake protocol's behaviors. 

```
$ curl -v --cacert certs/cert.pem https://localhost:8443
* Rebuilt URL to: https://localhost:8443/
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: certs/cert.pem
  CApath: none
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
:
```

and compare with patterns by `--insecure` option.

### Wireshark

[Wireshark](https://www.wireshark.org/#download) is a powerful tool to
analyze and display the IP-level packat traffic with the semantics of the application protocols, containing TLS byte value to semantics mapping, on each packets.

This repository contains HTTP(not HTTPS) Caddy config [Caddyfile.http](./Caddyfile.http) to help you understand and become aware the basic TCP traffic patterns(e.g. a TCP session gains a lot of ACK packets, which also appear a lot in TLS).
