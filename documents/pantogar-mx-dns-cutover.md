# pantogar.mx — DNS cutover to Vercel (CSC Global)

Status: domain is registered and sitting on CSC DNS, pointing at a CSC parking IP.
CSC (Pauline Belloir, cscglobal.com) needs us to send the zone records; she then
gets admin approval on the account and applies them.

## 1. Decide the canonical hostname

Recommendation: **www.pantogar.mx** as the primary, with the apex `pantogar.mx`
also pointed at Vercel so Vercel issues the 308 redirect itself. This matches how
the other market sites run (www.pantogar.com, www.pantogar.lt, etc.).

Do NOT use CSC "URL Forwarding" for the apex — it breaks HTTPS on the bare domain
and hides the redirect from Vercel. Point both hostnames at Vercel instead.

## 2. Get the exact records from Vercel (authoritative source)

Vercel now issues **project-specific** CNAME targets, so do not copy generic values
from a blog post. Pull them from the dashboard:

1. Vercel → the Pantogar MX project → **Settings → Domains**
2. **Add Domain** → `www.pantogar.mx` → Add
3. **Add Domain** → `pantogar.mx` → Add (choose "Redirect to www.pantogar.mx"
   if offered, or leave both as sites)
4. Vercel shows an "Invalid Configuration" panel with a table:
   *Set the following record on your DNS provider to continue.*
   Copy **Type / Name / Value** verbatim from that table.

Add the domains in Vercel **before** CSC applies the records, otherwise the TLS
certificate issuance will not start.

## 3. Records to send to CSC

Confirmed from the Vercel dashboard (project → Settings → Domains):

| Type  | Name (host) | Value                                  | TTL |
|-------|-------------|----------------------------------------|-----|
| A     | `@`         | `216.198.79.1`                         | 300 |
| CNAME | `www`       | `1349c248c0044da5.vercel-dns-017.com.` | 300 |

Notes:
- The CNAME hash is **hexadecimal** (`0-9`, `a-f`). The characters that can look
  like a capital `O` are zeros. Always copy the value with Vercel's copy button
  rather than retyping it.
- Vercel's legacy targets (`cname.vercel-dns.com`, `76.76.21.21`) still work, but
  the records above are the ones Vercel recommends for this project.
- No `_vercel` TXT verification record is required — Vercel is not asking for one,
  so the domain is not claimed on another account.
- Both hostnames are already added in Vercel: `www.pantogar.mx` is Production and
  the apex `pantogar.mx` is set to 308-redirect to it. No CSC URL Forwarding needed.

## 4. Ask CSC two extra questions

1. **CAA records** — if the zone has any CAA record, Let's Encrypt will be refused
   and the site will serve no certificate. Ask CSC to confirm there is no CAA, or
   to add: `pantogar.mx. CAA 0 issue "letsencrypt.org"`
2. **MX / mail** — confirm whether any mail (MX/SPF/DKIM) is expected on
   pantogar.mx. If Megalabs wants mail on this domain, those records must be kept
   or added; the A/CNAME change alone does not affect them, but the zone should be
   reviewed while it is open.

Also request **TTL 300** on the new records during cutover — CSC parking records
often carry a long TTL, so ask them to lower it a day ahead if possible.

## 5. Sequence

1. ~~Add both hostnames in Vercel → capture the records.~~ Done.
2. Send the table to Pauline; she requests admin approval on the CSC account.
3. CSC applies the zone change and confirms.
4. Watch Vercel → Settings → Domains until both hostnames show **Valid
   Configuration** and the certificate is issued (usually minutes).
5. Verify: `https://www.pantogar.mx` serves the site, `https://pantogar.mx`
   redirects to it, and the padlock is valid.

## 6. Open item before go-live (not DNS)

Every page in this repo still carries `.com` metadata pointing at the German/global
site — `<link rel="canonical" href="https://www.pantogar.com/...">`, `og:url`, and
`rel="shortlink"`. If that ships unchanged, Google will treat the Mexican site as a
duplicate of pantogar.com and will not index it. These need rewriting to the final
MX hostname before or at launch. Decide the hostname first (section 1), then the
rewrite is a single sitewide pass.
