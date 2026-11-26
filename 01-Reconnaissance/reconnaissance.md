# Phase 1: Initial Reconnaissance

## 1. Objective

The objective of this phase was to gather publicly available technical information about the target domain and establish an initial understanding of its web infrastructure.

## 2. Environment

- Operating System: Kali Linux
- Assessment Type: Black-box
- Target: Mediroza General Hospital
- Domain: medirozahospital.com

## 3. DNS Enumeration

### A Record

Command:

```bash
dig medirozahospital.com A
```

Result:

- IPv4 Address: 199.188.201.16
- DNS Status: NOERROR

### AAAA Record

Command:

```bash
dig medirozahospital.com AAAA
```

Result:

- No IPv6 address returned.

### Nameserver Enumeration

Command:

```bash
dig medirozahospital.com NS
```

Identified nameservers:

- dns1.namecheaphosting.com
- dns2.namecheaphosting.com

### MX Record Enumeration

Command:

```bash
dig medirozahospital.com MX
```

Three mail exchange records were identified.

## 4. Web Technology Fingerprinting

Tool: WhatWeb

Command:

```bash
whatweb https://medirozahospital.com
```

Observations:

- Web server: LiteSpeed
- Identified IP: 199.188.201.16
- Initial response: HTTP 403 Forbidden

## 5. HTTP Assessment

Tool: cURL

Commands:

```bash
curl --http1.1 -sS -D - -o /dev/null https://medirozahospital.com
```

```bash
curl --http2 -sS -D - -o /dev/null https://medirozahospital.com
```

Results:

- HTTP/1.1: 200 OK
- HTTP/2: 200 OK
- Server: LiteSpeed
- Content-Length: 5445 bytes

## 6. TLS Certificate Inspection

Tool: OpenSSL

The certificate inspection identified:

- Subject: medirozahospital.com
- Issuer: SSL.com
- Valid From: 14 August 2026
- Expiry: 28 February 2027

The certificate was within its validity period at the time of testing.

## 7. Preliminary Observations

1. The domain resolves to an IPv4 address.
2. Two nameservers were identified.
3. Three MX records were discovered.
4. LiteSpeed was identified as the web server.
5. HTTP/1.1 and HTTP/2 both returned successful responses.
6. WhatWeb returned a 403 response while cURL returned 200 in subsequent tests.

The inconsistent responses require further investigation.

## 8. Conclusion

Initial reconnaissance successfully identified the domain's DNS configuration, web server technology and TLS certificate information.

No confirmed security vulnerabilities have been established during this phase.

Further assessment is required.

---
**Project:** Networkwalks Week 4  
**Batch:** B083  
**Phase:** Initial Reconnaissance
