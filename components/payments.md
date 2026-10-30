# Payments

Taking money: one-off purchases, subscriptions, or payouts to others. The big fork is whether you are the seller (you handle sales tax/VAT) or a merchant of record sells on your behalf.

## Decide

### Questions to ask

- One-time payments, subscriptions, usage-based billing, or a marketplace paying out to others?
- Physical or digital goods? Selling internationally?
- Do you want to handle sales tax/VAT yourself, or have a merchant of record (MoR) take it on?
- Do you need your own checkout UI, or is a hosted checkout page fine?
- What do you want to learn: the full Stripe model (webhooks, customers, subscriptions), or the fastest way to sell?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Stripe Checkout | ★ default | One-off and simple subscription payments with a hosted page | You are the seller; you handle tax (Stripe Tax helps) |
| Stripe Billing (subscriptions + Customer Portal) | ◆ alternative | SaaS subscriptions, trials, plan changes | More webhook states to handle |
| Lemon Squeezy | ◆ alternative | Digital products and SaaS with tax handled (MoR) | Higher fees; less flexible |
| Paddle | ◆ alternative | SaaS as MoR, strong for B2B and global tax | Approval process; higher fees |
| Polar | ✦ distinctive | Developer-focused MoR for digital products, subscriptions, usage billing | Young platform |
| Stripe Connect | ✦ distinctive | Marketplaces that pay out to sellers | Onboarding, compliance, complex flows |
| Payment links / buttons | ◆ alternative | No-code: a link from the Stripe dashboard | No integration with your app's data |

### Option notes

**Stripe Checkout** ★
- Pros: hosted, PCI-compliant payment page; cards, wallets, many local methods; test mode with test cards; the most documented payments API.
- Cons: you are the seller of record; tax registration and remittance are your job (Stripe Tax calculates and reports but you still register).
- Pick when: most projects, and the best way to learn how payments work.
- What you'll learn: Checkout Sessions, webhooks, idempotency, test vs. live mode.

**Stripe Billing** ◆
- Pros: products/prices, trials, proration, invoices; the Customer Portal handles plan changes and cancellations for you.
- Cons: subscription lifecycle has many events to handle correctly.
- Pick when: recurring revenue.
- What you'll learn: subscription states, entitlement syncing from webhooks.

**Lemon Squeezy** ◆ (owned by Stripe)
- Pros: merchant of record: handles global sales tax/VAT; storefront, license keys, affiliates.
- Cons: higher fees than plain Stripe; less control.
- Pick when: selling digital products or SaaS without dealing with tax.
- What you'll learn: the MoR model, license key delivery.

**Paddle** ◆
- Pros: established MoR for SaaS; handles tax, invoicing, B2B billing.
- Cons: account approval; higher fees.
- Pick when: SaaS selling globally, especially to businesses.

**Polar** ✦
- Pros: MoR built for developers (open source); subscriptions, one-off, usage-based billing, benefits like GitHub repo access or license keys; framework adapters.
- Cons: younger than the others.
- Pick when: selling to developers or trying the newest MoR model.
- What you'll learn: usage-based billing, MoR integration.

**Stripe Connect** ✦
- Pros: take payments on behalf of others, route funds, take a platform fee.
- Cons: connected-account onboarding, compliance, refunds and disputes across parties.
- Pick when: a marketplace or platform.
- What you'll learn: platform payments, KYC onboarding flows.

**Payment links** ◆
- Pros: no code at all.
- Cons: nothing in your app knows about the payment unless you add webhooks.
- Pick when: a quick tip jar or a one-off sale.

### How to choose

1. Learning payments properly → Stripe Checkout, then Billing.
2. Digital product/SaaS and you don't want to handle tax → Lemon Squeezy, Paddle or Polar.
3. Marketplace → Stripe Connect.
4. Just need to get paid once → payment link.

## Configure

### Common to every option

- [ ] Build and test entirely in test/sandbox mode first.
- [ ] Treat the webhook, not the redirect back to your site, as the source of truth for "paid".
- [ ] Verify webhook signatures using the raw request body.
- [ ] Make webhook handlers idempotent (store processed event IDs; providers retry).
- [ ] Store the provider's customer/subscription IDs in your database against your user.
- [ ] Never trust prices from the client; look them up server-side by price/product ID.
- [ ] Before going live: switch to live keys, re-create the webhook endpoint for live mode, run one real small payment and refund it.

### Stripe Checkout

**Setup**
- [ ] Create a Stripe account; stay in test mode.
- [ ] Create products and prices in the dashboard (or via API).
- [ ] `npm i stripe`.
- [ ] Server route creates a Checkout Session (`mode: "payment"` or `"subscription"`, `line_items` with the price ID, `success_url`, `cancel_url`, `client_reference_id` or `metadata` with your user ID) and redirects to `session.url`.
- [ ] Webhook route handles `checkout.session.completed` (and failures); verify with `stripe.webhooks.constructEvent(rawBody, sig, secret)`.
- [ ] Local webhook testing: `stripe listen --forward-to localhost:3000/api/stripe/webhook` (Stripe CLI), which prints a local signing secret.
- [ ] Optional: Stripe Tax, receipts, coupons.

**Env vars**
- `STRIPE_SECRET_KEY` (server), `STRIPE_WEBHOOK_SECRET` (per endpoint), `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` (only if using client-side Stripe.js/Elements).

**Verify**
- [ ] Pay with test card `4242 4242 4242 4242`; the webhook marks the order paid in your database.
- [ ] `stripe trigger checkout.session.completed` hits the handler.
- [ ] Replaying the same event doesn't double-apply.

**Gotchas**
- Next.js route handler: read the body with `await req.text()` for signature verification, not `req.json()`.
- Test and live mode have different keys and different webhook secrets.

### Stripe Billing

**Setup**
- [ ] Recurring prices; Checkout in `subscription` mode.
- [ ] Handle `customer.subscription.created/updated/deleted` and `invoice.paid` / `invoice.payment_failed`; store status and current period end.
- [ ] Enable the Customer Portal in the dashboard; create portal sessions from a server route.
- [ ] Gate features on stored subscription status, not on each request to Stripe.

**Verify**
- [ ] Use test clocks to simulate renewals and failed payments.

### Lemon Squeezy

**Setup**
- [ ] Create a store and products; use test mode.
- [ ] Checkout via hosted links or the API (`@lemonsqueezy/lemonsqueezy.js`), passing custom data (user ID).
- [ ] Webhook endpoint with signing secret; handle order and subscription events.

**Env vars**
- `LEMONSQUEEZY_API_KEY`, `LEMONSQUEEZY_STORE_ID`, `LEMONSQUEEZY_WEBHOOK_SECRET`.

**Verify**
- [ ] Test purchase triggers the webhook and updates your database.

### Paddle

**Setup**
- [ ] Sandbox account; products and prices.
- [ ] Paddle.js overlay or inline checkout with a client-side token.
- [ ] Webhooks (notification destinations) with signature verification.

**Env vars**
- `PADDLE_API_KEY`, `PADDLE_WEBHOOK_SECRET`, `NEXT_PUBLIC_PADDLE_CLIENT_TOKEN`.

**Verify**
- [ ] Sandbox checkout completes and the webhook is received.

### Polar

**Setup**
- [ ] Create an organization in sandbox; create products.
- [ ] Use the SDK or framework adapter (e.g. `@polar-sh/nextjs`) for checkout and webhooks.
- [ ] Configure benefits (license keys, file downloads, repo access) if relevant.

**Env vars**
- `POLAR_ACCESS_TOKEN`, `POLAR_WEBHOOK_SECRET`.

**Verify**
- [ ] Sandbox purchase triggers the webhook.

### Stripe Connect

**Setup**
- [ ] Enable Connect; choose account type (Express is the usual start).
- [ ] Onboard sellers with Account Links; store connected account IDs.
- [ ] Create payments with `transfer_data.destination` and `application_fee_amount` (destination charges).
- [ ] Handle `account.updated` to know when sellers can receive payouts.

**Verify**
- [ ] Test-mode seller onboarding completes; a test payment shows the fee split.

### Payment links

**Setup**
- [ ] Create a link in the Stripe dashboard; put it behind a button.
- [ ] Optional: webhook on `checkout.session.completed` to record it.

## Related

- [`apis.md`](apis.md) (webhooks), [`auth.md`](auth.md), [`environments.md`](environments.md).
