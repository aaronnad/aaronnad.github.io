---
title: EC SSL Certs and Keys
date: 2023-01-03 15:00:00 -0000
categories: [Tech, Guide]
tags: [ec,ssl,certificates,guides,eliptic_curve]
---
Quite often than not i find myself needing to refresh on how to do some simple EC cert generation now that things are moving toward a more automated approach.
Below should give some insight in the general commands for generating an EC key, pem and CSR as well as converting some certs.

# Generating an EC SSL Cert using openssl on the CLI

First there's a few things to consider - what strength key do you want to employ

```bash
# the following command lists the available ciphers
openssl ecparam -list_curves
```
To generate an SSL Key using a specific key, in this instance a 256bit X9.62/SECG curve
```bash
# Use openssl to generate the key
openssl ecparam -name prime256v1 -genkey -out private-key.key
```
The following enables the generation of a CSR for signing
```bash
# Generate CSR from a .cnf and the private key file
openssl req -new -sha256 -nodes -key private.key -out sslcert.csr -config sslcertconfig.cnf
```
To generate a self signed .pem file use the following. If needed, you can also generate a .pfx file from the pem and private key
```bash
# Create a self-signed certificate for 360 days
openssl req -new -x509 -key private-key.key -out cert.pem -days 360

# Conver the newly created self signed cert from pem to pfx
openssl pkcs12 -export -inkey private-key.key -in cert.pem -out cert.pfx
```
If you want create a trust chain withthe public key from the private key, you can do that here
```bash
# generate corresponding public key
openssl ec -in private-key.key -pubout -out public-key.key
```


## Below are some examples of the config output
```bash
openssl ecparam -out private.key -name prime256v1 -genkey
```
```text
cat private.key

-----BEGIN EC PARAMETERS-----
BggqhkjOPQMBBw==
-----END EC PARAMETERS-----
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIGeOdfaNpamOcXY5YvrdsvOdERQVMe8z5gwKMdVSTTSOoAoGCCqGSM49
AwEHoUQDQgAE30hvfthqLkH57Z9rnpwj+pEqqdjUjPvjRWp3+Jjo6EFkveVHnYkx
YgvOClXF9imi2rLld22UjnPTvLHib05EVQ==
-----END EC PRIVATE KEY-----
```

```bash
openssl ecparam -noout -out private.key -name prime256v1 -genkey
```
```text
cat private.key

-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIFHn7mwM/LBid7MSKNYmPtO5SZavtY9RhyEDtjckM6W7oAoGCCqGSM49
AwEHoUQDQgAEz1Uje2mbTJR73Jl+cUjMxd9f5paBPm/ju3eFQE+0Hjv21T6dwEHg
+GBXIAAmFpTBLpB39naAnFYmqRGqc8YHuw==
-----END EC PRIVATE KEY-----
```
Notice that specifying the `-noout` parameter meant that the EC Paramters were ommitted from the .key file - these EC paramters are an OID format specifying what the key is

```bash
echo "BggqhkjOPQMBBw==" | openssl base64 -d | openssl asn1parse -inform DER
    0:d=0  hl=2 l=   8 prim: OBJECT            :prime256v1
```

## Once you have your CSR, inspect it with the following
You can use the optional tag of `-verify` in the openssl comand below to verify the config - this adds the following at the top of the output if its verified as being 'good'
`verify OK`

```bash
openssl req -noout -text -in sslcert.csr
```
```
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=GB, ST=test, L=test, O=test, OU=test, CN=test.localodmain.local/emailAddress=test@localdomain.local
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:d4:5d:be:95:91:cb:06:4c:d1:7f:15:72:a5:0a:
                    9c:38:62:db:a3:59:6d:7a:4d:b1:19:6c:43:a8:a4:
                    77:ae:d1:8f:af:1c:4c:2f:4c:dc:8b:bd:95:c3:a3:
                    da:4b:a8:ff:f0:4b:22:fa:4b:88:17:9f:c3:da:68:
                    7d:8e:6b:f4:bb
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        Attributes:
        Requested Extensions:
            X509v3 Key Usage:
                Digital Signature, Non Repudiation, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            X509v3 Subject Key Identifier:
                61:F3:F8:78:04:B2:26:8A:A8:52:CF:3D:87:4D:CD:4B:BE:C0:A7:B7
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Cert Type:
                SSL Server
            X509v3 Subject Alternative Name:
                DNS:test, DNS:test1.localdomain.local, DNS:test1, DNS:test2.localdomain.local, DNS:test2, DNS:test3.localdomain.local DNS:test3
            Netscape Comment:
                hostname
    Signature Algorithm: ecdsa-with-SHA256
         30:44:02:20:34:3a:46:e5:90:12:da:0c:5f:ad:05:f0:2b:9a:
         99:7d:72:0e:5b:d5:8e:be:9a:26:bf:11:e1:89:b9:2b:30:11:
         02:20:03:c9:f5:63:72:e6:f2:88:50:cb:99:fb:6b:42:d3:23:
         e4:1a:58:e7:6d:7a:6b:8c:be:7a:2e:c9:b0:db:7a:f1
```