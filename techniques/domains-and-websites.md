# Domains and websites

Techniques for learning who runs a domain, what it runs on, and what it used to look like.

- [Email provider (MX records)](#email-provider-mx-records)
- [Registration date](#registration-date)
- [Past owners](#past-owners)
- [Old versions of a site (Wayback)](#old-versions-of-a-site-wayback)
- [Finding an organization's vendors](#finding-an-organizations-vendors)

---

## Email provider (MX records)

MX records name the servers that receive a domain's mail, so they reveal the email provider.

```bash
dig MX example.com +short
```

No terminal? Use `https://dns.google/resolve?name=example.com&type=MX` in a browser.

| MX hostname contains | Provider |
| --- | --- |
| `protonmail.ch` | Proton Mail |
| `google.com` / `googlemail.com` | Google Workspace |
| `outlook.com` / `protection.outlook.com` | Microsoft 365 |
| `zoho` | Zoho Mail |

The lower the priority number, the more preferred the server.

---

## Registration date

```bash
whois example.com | grep -i creat
```

Or RDAP, the modern replacement for whois, which returns JSON:

```bash
curl -s https://rdap.verisign.com/com/v1/domain/example.com | jq '.events'
```

The creation date is public even when owner details are redacted. A creation date newer than you'd expect can mean the domain was dropped and re-registered — so there was an earlier owner.

---

## Past owners

Regular `whois` only shows the current record. Historical whois services keep old snapshots.

| Source | What it gives |
| --- | --- |
| [whoxy.com](https://www.whoxy.com) | Free whois history |
| [ViewDNS](https://viewdns.info) | No whois history any more, but **IP History** (hosting changes) and **Reverse Whois** (other domains owned by the same name or email) |
| [SecurityTrails](https://securitytrails.com) | Historical DNS; some whois history with a free account |
| [Wayback Machine](https://web.archive.org) | Old copies of the site, often with names on About or contact pages |

Since GDPR (2018), most registrars redact owner details, so real names usually appear only in older records.

---

## Old versions of a site (Wayback)

When a question asks what a site *used to* show — or the live site has been replaced — go to the Wayback Machine.

1. Open `web.archive.org/web/*/example.com` to see the calendar of snapshots.
2. Pick a snapshot from the right period and view source (`Ctrl+U`).
3. Search the source for what you need (`version`, `footer`, `ver`, a name).
4. Add **`id_`** after the timestamp to get the original page without Wayback's injected code:
   ```
   https://web.archive.org/web/20260909152343id_/https://example.com/
   ```

**Watch for red herrings:** third-party scripts carry their own version numbers. For example, `"version":"2024.11.0"` inside `data-cf-beacon` belongs to Cloudflare's analytics script, which appears on every Cloudflare-hosted site.

---

## Finding an organization's vendors

Organizations reveal the software they use (MFA, SSO, email, HR systems) through help pages, DNS records, subdomains and job postings.

1. **Dorks on their own site**
   ```
   site:example.gov MFA
   site:example.gov "multi-factor authentication"
   site:example.gov Duo OR Okta OR "Microsoft Authenticator" OR RSA
   ```
   Look for employee setup guides and PDFs.
2. **DNS TXT records** — vendors ask customers to add verification records:
   ```bash
   dig TXT example.gov +short
   ```
   Look for `duo_sso_verification`, `okta-verification`, `MS=`, `atlassian-domain-verification` and so on.
3. **Subdomains from certificate logs** — search `%.example.gov` on [crt.sh](https://crt.sh) for names like `okta.`, `duo.`, `sso.` or `vpn.`.
4. **Job postings** — IT listings often name the products staff must know.

Two independent sources that agree give a confident answer.
