# DNS + SSL Configuration Resolution (DonWeb, Cloudflare & PXsol)

This repository documents the complete process of **diagnosing, resolving, and stabilizing** a production website affected by SSL certificate errors, email delivery issues, redirection problems, and conflicting configurations across multiple providers.

The infrastructure involved three different services:

- **DonWeb (Ferozo)** – Hosting provider and original DNS server  
- **Cloudflare** – CDN, SSL termination and proxy  
- **PXsol** – External SaaS platform (domain target: `secure.pxsol.com`)  

---

## Initial Problem

The domain `abedulmardelaspampas.com.ar` was experiencing the following issues:

- Emails bouncing when sent to Gmail accounts
- Browser warning: `NET::ERR_CERT_COMMON_NAME_INVALID`
- Invalid certificate (`*.ferozo.com`) when accessing the site without `www`
- Inconsistent HTTP / HTTPS redirections
- Duplicate and conflicting DNS records
- SSL certificate valid only for the `www` subdomain, not for the root domain

---

## Diagnosis

1. SPF, DKIM and DMARC records were incorrectly configured in Ferozo.
2. Cloudflare was partially enabled and only issuing certificates for `www`.
3. The root domain (`@`) was pointing directly to the Ferozo server without proxying (DNS Only).
4. PXsol SaaS required **only the `www` subdomain** to be proxied (CNAME → `secure.pxsol.com`).
5. The root domain could not obtain a valid SSL certificate from the remote server.
6. A **301 redirect** from the root domain to `www` was required.

---

## Implemented Solution

### Ferozo Configuration
- Created SPF record
- Created DKIM record
- Created DMARC record

### Cloudflare Configuration
- Configured SPF, DKIM and DMARC records
- **A record (@)** set to **Proxied**
- **CNAME record (www)** set to **Proxied**, pointing to `secure.pxsol.com`
- SSL mode set to **Full**
- Redirect rules enabled:

**Rule 1 — HTTPS → WWW**  
`https://abedulmardelaspampas.com.ar/* → https://www.abedulmardelaspampas.com.ar/$1` (301)

**Rule 2 — HTTP → HTTPS + WWW**  
`http://abedulmardelaspampas.com.ar/* → https://www.abedulmardelaspampas.com.ar/$1` (301)

---

## Validation

- Correct access with and without `www`
- Automatic redirection to the secure version
- Website served correctly from PXsol behind Cloudflare proxy
- Email delivery fully functional (SPF, DKIM, DMARC verified)
- DNS propagation completed successfully

---

## Final Result

The website is now fully functional across all scenarios:

- Incoming and outgoing emails working correctly
- HTTP traffic redirected to HTTPS
- Root domain redirected to `www`
- Valid HTTPS certificate provided by Cloudflare
- No browser security warnings
- PXsol hotel management platform operating correctly behind the proxy

---

## Skills Applied

- Ferozo configuration (SPF / DKIM / DMARC)
- Advanced DNS administration
- Cloudflare (SSL, Proxy, Rules, Redirects)
- SSL troubleshooting and resolution
- Multi-provider infrastructure analysis
- Technical coordination with hosting support
- Technical documentation and client communication

---

## Diagrams

This repository includes flow and architecture diagrams illustrating the **before and after**
states of the DNS and SSL configuration.

![DNS Flow](img/flujo-de-dns.png)  
*Figure 1 — DNS flow*

![Before vs After](img/antes-vs-despues-dns-y-ssl-diagrama-de-arquitectura.png)  
*Figure 2 — Before vs after DNS & SSL architecture*

![Problem → Solution Flow](img/diagrama-de-flujo-problema-diagnostico-solucion.png)  
*Figure 3 — Problem, diagnosis and solution process*

![301 Redirect Rules](img/diagrama-de-reglas-de-redireccion-301.png)  
*Figure 4 — 301 redirects configured in Cloudflare*

