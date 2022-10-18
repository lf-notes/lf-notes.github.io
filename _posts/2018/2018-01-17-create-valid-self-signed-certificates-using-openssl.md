---
layout: default
title: Create valid self-signed certificates using OpenSSL
tags: ssl openssl tls
comments: true
---
# Create valid self-signed certificates using OpenSSL

I was debugging a WebSocket connection failing with error `net::ERR_INSECURE_RESPONSE`, in Chrome, when I learnt that the self-signed certificate I was using was missing [subject alternative names](https://tools.ietf.org/html/rfc5280).

This post brings together information I found in several different places, to create valid self-signed server certificates using OpenSSL that work with internet browsers such as Chrome.

![Valid Certificate on IIS](/assets/img/valid-certificate-iis.png)

OpenSSL is available in the [Git](https://git-scm.com/download/) Bash shell on Windows.

To create self-signed certificate in ASCII PEM format

```bash
openssl req -x509 -newkey rsa:4096 -nodes -subj '/CN=localhost' -keyout key.pem -out cert.pem -days 365 -config openssl.cnf -extensions req_ext
```

Additional distinguished name properties may be specified by changing the `subj` option

```conf
-subj "/C=US/ST=private/L=province/O=city/CN=hostname.example.com"
```

A minimalist `openssl.cnf` file that contains `req_ext` extension section with `subjectAltName`

```conf
[ req ]
distinguished_name = req_distinguished_name
req_extensions     = req_ext
[ req_distinguished_name ]
[ req_ext ]
subjectAltName = @alt_names
[alt_names]
DNS.1   = localhost
DNS.2   = example.com
```

Print certificate to view subject alternative names and thumbprint/fingerprint

```bash
openssl x509 -noout -text -fingerprint -in cert.pem
```

Create PFX containing private key from self-signed certificate

```bash
openssl pkcs12 -inkey key.pem -in cert.pem -export -out key.pfx
```

Create CRT file in binary DER format from self-signed certificate

```bash
openssl x509 -outform der -in cert.pem -out cert.crt
```

Create CRT file in ASCII PEM format from self-signed certificate

```bash
openssl x509 -outform pem -in cert.pem -out cert.crt
```

Add private key to the appropriate key store and reconfigure server application.

Add certificate file to trusted root authorities key store. Restart the browser. It should be happy with the certificate provided by the server.

## Using PowerShell to create self-signed certificates

PowerShell's [New-SelfSignedCertificate](https://docs.microsoft.com/en-us/powershell/module/pkiclient/new-selfsignedcertificate) command can also be used to automate self-signed certificate creation and installation

```powershell
New-SelfSignedCertificate -DnsName "localhost", "example.com" -CertStoreLocation "cert:\LocalMachine\My"
```

You can export the private key to PFX format using **Manage computer certificates** Control Panel widget, and [convert it to PEM format](_posts/2017/2017-09-18-export-private-key-in-pfx-or-p12-file-to-pem-format.md) using OpenSSL.
