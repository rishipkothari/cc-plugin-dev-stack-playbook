# Domains and DNS

Owning a domain name and pointing it at the app (and at email providers). HTTPS certificates are almost always automatic once DNS is right.

## Decide

### Questions to ask

- Does the project need its own domain, or is the host's free subdomain fine (`*.vercel.app`)?
- Apex (`example.com`), `www`, or a subdomain of a domain you already own (`app.example.com`)?
- Will you send email from it? (Then DNS also carries SPF/DKIM/DMARC.)
- Where should DNS be managed: at the registrar, at the host, or at Cloudflare?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Cloudflare Registrar | ★ default | At-cost renewals, excellent DNS | Requires Cloudflare to run the domain's DNS |
| Porkbun | ◆ alternative | Cheap, simple registrar with free WHOIS privacy | Separate DNS setup |
| Namecheap | ◆ alternative | Popular registrar | Renewal prices higher than first-year price |
| Buy through the host (Vercel Domains) | ◆ alternative | One-click, DNS auto-configured | Domain tied to the host account |
| Cloudflare as DNS proxy (orange cloud) | ✦ distinctive | CDN, WAF, caching in front of any host | Can conflict with hosts that manage TLS themselves |
| Subdomain of an existing domain | ◆ alternative | Many projects under one domain | Shared reputation for email |

### Option notes

**Cloudflare Registrar** ★
- Pros: sells domains at wholesale cost; fast DNS; free email routing.
- Cons: you must use Cloudflare nameservers; some TLDs unsupported.
- What you'll learn: nameservers, DNS records, TTLs.

**Porkbun / Namecheap** ◆
- Pros: wide TLD selection; easy UI.
- What you'll learn: registrar vs. DNS host separation.

**Buy through the host** ◆
- Pros: zero DNS work.
- Cons: moving later means transferring.

**Cloudflare proxy** ✦
- Pros: caching, DDoS protection, firewall rules, analytics.
- Cons: in front of Vercel/Netlify it can interfere with their TLS and caching; use DNS-only (grey cloud) unless you need the proxy.
- What you'll learn: reverse proxies, CDN caching, TLS modes.

**Subdomains** ◆
- Pros: one domain for many projects (`nanny.example.com`, `blog.example.com`).
- What you'll learn: CNAME delegation.

### How to choose

1. One domain for many personal projects → buy once at Cloudflare or Porkbun, use subdomains.
2. Zero fuss for one app → buy through the host.
3. Learning networking → Cloudflare DNS, try the proxy on a non-Vercel host.

## Configure

### Common to every option

- [ ] Turn on auto-renew and registrar lock; keep the registrar account on 2FA.
- [ ] Add the domain in the host dashboard first; it tells you which records to create.
- [ ] Typical records: apex → `A` record to the host's IP (or ALIAS/flattened CNAME); `www` → `CNAME` to the host's target; subdomain → `CNAME`.
- [ ] Choose a canonical host (apex or `www`) and redirect the other.
- [ ] Keep email records (MX, SPF, DKIM, DMARC) intact when changing DNS.
- [ ] Wait for DNS to propagate (minutes, sometimes longer); HTTPS issues automatically after.

### Cloudflare Registrar

**Setup**
- [ ] Add a site to Cloudflare (or register the domain there directly).
- [ ] Records in **DNS → Records**; set proxy status per record (grey cloud for Vercel/Netlify).
- [ ] Transfer existing domains in: unlock at the old registrar, get the auth code.

**Verify**
- [ ] `dig example.com` / `dig www.example.com CNAME` return the expected values; the site loads over HTTPS.

### Porkbun / Namecheap

**Setup**
- [ ] Register; either use their DNS (add the host's records) or point nameservers at the host or Cloudflare.

**Verify**
- [ ] Same `dig` checks; HTTPS works.

### Buy through the host (Vercel)

**Setup**
- [ ] **Project → Settings → Domains → Add** → buy; records are configured automatically.

**Verify**
- [ ] Domain shows "Valid Configuration".

### Cloudflare as DNS proxy

**Setup**
- [ ] Orange-cloud the record; set SSL/TLS mode to **Full (strict)**.
- [ ] Add caching and security rules as needed.

**Verify**
- [ ] Response headers include `cf-ray`; no redirect loops (a loop usually means TLS mode is "Flexible").

### Subdomain of an existing domain

**Setup**
- [ ] Add `app.example.com` in the host; create the `CNAME` it shows at your DNS provider.

**Verify**
- [ ] Subdomain loads over HTTPS.

## Related

- [`hosting.md`](hosting.md), [`email.md`](email.md), [`auth.md`](auth.md) (OAuth redirect URLs and passkeys depend on the domain).
