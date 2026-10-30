# Email

Sending transactional email (sign-in links, password resets, receipts, notifications) and occasionally receiving it. Deliverability depends more on domain setup (SPF, DKIM, DMARC) than on which provider you pick.

## Decide

### Questions to ask

- Transactional only, or marketing/newsletters too? (Keep them on separate domains or streams.)
- Volume per day/month?
- Do you own a domain to send from? (You need one for good deliverability.)
- Do you need to receive and parse incoming email?
- What do you want to learn: templating with components, raw SMTP/DNS, or AWS?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Resend (+ React Email) | ★ default | Developer-friendly transactional email, templates as React components | Free tier daily/monthly limits |
| Postmark | ◆ alternative | Deliverability-focused transactional email | No meaningful free tier beyond testing |
| AWS SES | ◆ alternative | Cheapest at volume | Sandbox approval, more setup |
| Inbound email (Postmark/Resend/Cloudflare Email Routing) | ✦ distinctive | Apps that act on emails sent to them | Parsing and spam handling |
| Mailpit (local) | ✦ distinctive | Catching all email during local development | Local only |

### Option notes

**Resend** ★
- Pros: simple API and SDK; React Email for building templates as components; good docs; domain verification walkthrough.
- What you'll learn: email DNS records, component-based templates.

**Postmark** ◆
- Pros: strong deliverability reputation; separate transactional and broadcast streams; inbound parsing.
- What you'll learn: message streams, bounce handling.

**AWS SES** ◆
- Pros: very cheap per email.
- Cons: starts in a sandbox (only verified recipients) until you request production access; more AWS setup.
- What you'll learn: AWS IAM, SES identities, bounce/complaint notifications via SNS.

**Inbound email** ✦
- Pros: email-to-app features ("forward receipts to this address").
- What you'll learn: MX records, parsing MIME, webhook-driven processing.

**Mailpit** ✦
- Pros: local SMTP server with a web UI; nothing leaves your machine.
- What you'll learn: SMTP, testing email flows safely.

### How to choose

1. Most apps → Resend.
2. Deliverability-critical (sign-in links for many users) → Postmark.
3. High volume on a tight budget → SES.
4. Add Mailpit for local testing in any case if you send much email.

## Configure

### Common to every option

- [ ] Send from a subdomain you own (e.g. `mail.example.com`), not a free mailbox address.
- [ ] Add the provider's DNS records: SPF (TXT), DKIM (CNAME/TXT), and a DMARC record (`_dmarc` TXT, start with `p=none`).
- [ ] Send email from server code or a [job](jobs.md); never expose the API key.
- [ ] Include a plain-text version; keep transactional emails free of marketing content.
- [ ] Handle bounces/complaints (webhooks) if volume grows.

### Resend

**Setup**
- [ ] Create an account and API key; add and verify your domain (add the DNS records it lists).
- [ ] `npm i resend` and optionally `@react-email/components`.
- [ ] Templates in `emails/` as React components; `resend.emails.send({ from, to, subject, react })`.
- [ ] `npx react-email dev` (or `email dev`) to preview templates.

**Env vars**
- `RESEND_API_KEY`, `EMAIL_FROM`.

**Verify**
- [ ] Domain shows verified; a test email arrives in Gmail (check "Show original": SPF, DKIM and DMARC = PASS).

### Postmark

**Setup**
- [ ] Create a server; add a sender signature/domain with DKIM and Return-Path records.
- [ ] `npm i postmark`; send via the server API token.

**Env vars**
- `POSTMARK_SERVER_TOKEN`, `EMAIL_FROM`.

**Verify**
- [ ] Test email passes SPF/DKIM/DMARC.

### AWS SES

**Setup**
- [ ] Verify a domain identity (Easy DKIM records) in your region.
- [ ] Request production access (leaves the sandbox).
- [ ] IAM credentials limited to `ses:SendEmail`; `@aws-sdk/client-sesv2`.
- [ ] Configure bounce/complaint notifications.

**Env vars**
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `EMAIL_FROM`.

**Verify**
- [ ] Email to an unverified address succeeds (out of sandbox).

### Inbound email

**Setup**
- [ ] Choose a receiving address/subdomain; set MX records per the provider (Postmark inbound, Resend inbound, or Cloudflare Email Routing to a Worker).
- [ ] Webhook endpoint receives parsed JSON (from, subject, text, attachments); verify the source.

**Verify**
- [ ] An email sent to the address reaches the handler and is processed once.

### Mailpit

**Setup**
- [ ] Run via Docker (`axllent/mailpit`) or binary; SMTP on port 1025, UI on 8025.
- [ ] In development, point the mailer at it (or switch providers by env).

**Verify**
- [ ] Emails sent locally appear in the Mailpit UI.

## Related

- [`auth.md`](auth.md), [`domains.md`](domains.md), [`jobs.md`](jobs.md).
