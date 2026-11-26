# Reconnaissance and Information Gathering

## 1. Project Information

| Field | Details |
|---|---|
| Project | Networkwalks Week 4 — Penetration Testing |
| Batch | B083 |
| Target | `https://medirozahospital.com` |
| Assessment Type | Black-Box Security Assessment |
| Phase | Initial Reconnaissance |
| Tester | Oluwaferanmi Idowu |
| Environment | Kali Linux |

## 2. Objective

The objective of this phase was to gather publicly accessible information about the target domain and establish an initial understanding of its external attack surface.

The reconnaissance activities focused on:

- DNS enumeration.
- Domain and IP address identification.
- Web server fingerprinting.
- HTTP response and header analysis.
- HTTP protocol verification.
- TLS certificate inspection.
- Homepage structure analysis.
- Identification of publicly referenced website paths and resources.

All activities were conducted within the stated assessment scope.

## 3. Tools and Technologies

| Tool | Purpose |
|---|---|
| Kali Linux | Penetration testing environment |
| dig | DNS record enumeration |
| WhatWeb | Web technology fingerprinting |
| curl | HTTP response and header inspection |
| OpenSSL | TLS certificate inspection |
| grep | HTML content and resource analysis |
| file | Identification of downloaded file types |

## 4. DNS Enumeration

DNS enumeration was performed using `dig` to identify publicly available DNS records associated with the target domain.

### 4.1 A Record Enumeration

**Command:**

```bash
dig medirozahospital.com A
```

**Results:**

- Status: `NOERROR`
- IPv4 Address: `199.188.201.16`

The A record resolved the target domain to the identified IPv4 address.

**Evidence:** `screenshots/02-dns-a-record.png`

### 4.2 Name Server Enumeration

**Command:**

```bash
dig medirozahospital.com NS
```

**Identified Name Servers:**

- `dns1.namecheaphosting.com`
- `dns2.namecheaphosting.com`

These records identify the name servers responsible for the domain's DNS service.

### 4.3 Mail Exchange Enumeration

**Command:**

```bash
dig medirozahospital.com MX
```

**Identified Mail Servers:**

| Priority | Mail Server |
|---:|---|
| 5 | `mx1-hosting.jellyfish.systems` |
| 10 | `mx2-hosting.jellyfish.systems` |
| 20 | `mx3-hosting.jellyfish.systems` |

The MX records reveal the mail servers configured to receive email for the domain.

**Evidence:** `screenshots/03-dns-ns-mx-records.png`

### 4.4 IPv6 Record Enumeration

**Command:**

```bash
dig medirozahospital.com AAAA
```

The query returned `NOERROR` but did not return any AAAA records.

This indicates that no IPv6 address was returned for the queried domain during this test.

## 5. Web Server Fingerprinting

Web technology identification was performed using WhatWeb.

**Command:**

```bash
whatweb https://medirozahospital.com
```

### Observations

| Attribute | Result |
|---|---|
| Target | `medirozahospital.com` |
| IP Address | `199.188.201.16` |
| Web Server | LiteSpeed |
| Identified Technology | HTML5 |
| Initial WhatWeb Response | 403 Forbidden |
| Additional Header | `x-turbo-charged-by` |

WhatWeb identified LiteSpeed as the web server technology.

The initial request returned a `403 Forbidden` response. However, subsequent requests using curl returned successful responses.

This inconsistency was recorded for further investigation rather than classified as a confirmed vulnerability.

**Evidence:** `screenshots/04-whatweb-fingerprinting.png`

## 6. HTTP Response and Protocol Analysis

HTTP response headers were inspected using curl.

**Command:**

```bash
curl -I https://medirozahospital.com
```

### Observations

The response included:

- HTTP status: `200 OK`
- Content type: `text/html`
- Web server: LiteSpeed
- Content length: `5445` bytes
- Additional header: `x-turbo-charged-by`

### 6.1 HTTP/1.1 Verification

**Command:**

```bash
curl --http1.1 -sS -D - -o /dev/null https://medirozahospital.com
```

**Result:** `HTTP/1.1 200 OK`

### 6.2 HTTP/2 Verification

**Command:**

```bash
curl --http2 -sS -D - -o /dev/null https://medirozahospital.com
```

**Result:** `HTTP/2 200`

Both protocol tests returned successful responses.

These results confirm that the server responded over HTTP/1.1 and HTTP/2 during the assessment.

**Evidence:**

- `screenshots/05-http-headers.png`
- `screenshots/06-http-protocol-testing.png`

## 7. TLS Certificate Inspection

The target's TLS certificate was inspected using OpenSSL.

**Command:**

```bash
echo | openssl s_client -connect medirozahospital.com:443 -servername medirozahospital.com 2>/dev/null | openssl x509 -noout -subject -issuer -dates
```

### Certificate Information

| Attribute | Result |
|---|---|
| Subject | `CN=medirozahospital.com` |
| Issuer | SSL.com TLS Issuing RSA CA R1 |
| Valid From | 14 August 2026 |
| Expiry Date | 28 February 2027 |

The certificate's validity dates were checked and found to include the assessment date.

This inspection confirms the certificate's subject, issuer, and validity period. It does not constitute a complete assessment of the server's TLS configuration.

**Evidence:** `screenshots/07-tls-certificate.png`

## 8. Homepage Structure Analysis

The homepage was downloaded locally for basic HTML inspection.

**Command:**

```bash
curl -sS https://medirozahospital.com -o evidence/homepage.html
```

The downloaded file was approximately 5,445 bytes.

### 8.1 Page Identification

The extracted HTML title was:

`Mediroza General Hospital | Compassionate Care, Advanced Medicine`

### 8.2 HTML Form Inspection

The following command was used to identify HTML form elements:

```bash
grep -oi '<form[^>]*>' evidence/homepage.html
```

No form opening tags were returned.

This observation applies only to the downloaded homepage. It does not establish that forms are absent from other pages or dynamically loaded content.

### 8.3 Identified Website Paths

The following paths were identified in the homepage's HTML links:

| Path | Observation |
|---|---|
| `/styles.css` | Stylesheet resource |
| `/staff/login.php` | Staff login entry point |
| `/patient/login.php` | Patient login entry point |
| `/contact.html` | Contact page |
| `/index.html` | Homepage |
| `/about.html` | About page |
| `/doctors.html` | Doctors page |

The staff and patient login paths were recorded as publicly referenced entry points for further authorised assessment.

Their presence alone does not establish a security vulnerability.

### 8.4 JavaScript Reference Inspection

The homepage was inspected for external JavaScript references using:

```bash
grep -oEi 'src="[^"]+\.js[^"]*"'
```

No matching external JavaScript source references were returned.

This result is limited to the specific HTML pattern searched and does not rule out inline JavaScript or dynamically loaded scripts.

**Evidence:** `screenshots/08-homepage-inspection.png`

## 9. Summary of Reconnaissance Findings

| No. | Finding | Observation |
|---:|---|---|
| 1 | Domain resolution | Resolved to `199.188.201.16` |
| 2 | Name servers | Two name servers identified |
| 3 | Mail infrastructure | Three MX records identified |
| 4 | Web server | LiteSpeed identified |
| 5 | HTTP protocols | HTTP/1.1 and HTTP/2 returned successful responses |
| 6 | TLS certificate | Validity period includes assessment date |
| 7 | Authentication paths | Staff and patient login paths identified |
| 8 | Homepage inspection | No HTML form opening tags found in downloaded homepage |
| 9 | Response inconsistency | WhatWeb returned 403 while subsequent curl tests returned 200 |

## 10. Preliminary Security Observations

The reconnaissance phase identified publicly observable infrastructure information and website entry points that may be relevant to subsequent security assessment.

The following areas were noted for further authorised review:

- Authentication entry points for staff and patients.
- HTTP response and security-header behaviour.
- Server configuration and information exposure.
- Access-control behaviour of publicly referenced resources.

No confirmed security vulnerability was established solely from the reconnaissance activities documented here.

Any further testing involving authentication, access control, or sensitive information must remain within the written authorisation and use approved test accounts and synthetic records.

## 11. Evidence and Supporting Screenshots

The following screenshots provide visual evidence of the reconnaissance activities performed during this assessment.

| Evidence ID | Screenshot | Description |
|---|---|---|
| E01 | `01-kali-environment.png` | Kali Linux testing environment |
| E02 | `02-dns-a-record.png` | DNS A record resolution |
| E03 | `03-dns-ns-mx-records.png` | Name server and mail exchange records |
| E04 | `04-whatweb-fingerprinting.png` | Web server fingerprinting results |
| E05 | `05-http-headers.png` | HTTP response headers |
| E06 | `06-http-protocol-testing.png` | HTTP/1.1 and HTTP/2 response tests |
| E07 | `07-tls-certificate.png` | TLS certificate inspection |
| E08 | `08-homepage-inspection.png` | Homepage HTML analysis |

All screenshots should be stored in the accompanying `screenshots/` directory.

Evidence must be reviewed before publication to ensure that credentials, patient information, employee information, and other confidential data are not disclosed.

## 12. Conclusion

The initial reconnaissance phase established a baseline understanding of the target's publicly observable infrastructure, DNS configuration, web technologies, TLS certificate information, HTTP behaviour, and homepage structure.

The assessment identified the target's IPv4 address, name servers, mail servers, LiteSpeed web server, supported HTTP protocols, and publicly referenced staff and patient login paths.

These observations provide a foundation for the subsequent vulnerability assessment phase.

No real patient records, employee information, credentials, or other confidential information were retrieved or included in this documentation.

---

**Assessment Status:** Initial Reconnaissance Completed  
**Next Phase:** Vulnerability Assessment  
**Document Owner:** Oluwaferanmi Idowu  
**Programme:** Networkwalks Cybersecurity Internship  
**Batch:** B083
