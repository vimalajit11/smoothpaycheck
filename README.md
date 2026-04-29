# Smooth Paycheck

Your complete mini web-app, ready to ship. This folder contains everything you need to launch.

## What's in here

```
smooth-paycheck-app/
├── index.html                 # Landing page (free, public)
├── app.html                   # The full tool (license-gated)
├── README.md                  # This file
├── DODO-PAYMENTS-SETUP.md     # Payment setup — your primary guide
├── LEMON-SQUEEZY-SETUP.md     # Backup guide if Dodo rejects you
└── post-purchase-email.md     # Email template for buyers
```

The app is **two HTML files** plus markdown documentation. No backend, no database, no build step. The `.md` files are reference material for you — buyers never see them.

---

## How it works

- **`index.html`** is your marketing page. Anyone can see it. The three "Get Smooth Paycheck" buttons currently point to `#REPLACE_WITH_DODO_CHECKOUT_URL` placeholders — you'll replace those with your real Dodo checkout URL when your account is approved.
- **`app.html`** is the full tool. It validates license keys against Dodo Payments' public License API. By default it runs in **placeholder mode** — any non-empty key unlocks it so you can test freely. When you're ready to launch, flip one variable to `false` for real validation.
- **All user data** (income, expenses, currency choice, results) is saved to the browser's `localStorage`. Nothing leaves their device except the license-key check.
- **The PDF download** is generated client-side using jsPDF (loaded from CDN), with the brand fonts (Fraunces + Instrument Serif) embedded directly so it works offline and looks identical for every buyer.
- **The currency picker** in Step 1 lets users pick from 10 currencies (USD, EUR, GBP, INR, AUD, CAD, BRL, MXN, JPY, SGD). All formatting throughout the app and PDF respects their choice.

---

## Deployment — Step by Step

### Phase 1: Get it on GitHub Pages (15 minutes)

You don't need to know Git. This works through the GitHub web interface.

1. **Create a GitHub account** at [github.com](https://github.com) if you don't have one. Free tier works.

2. **Create a new repository.** Click `+` → New repository → name it `smooth-paycheck` (or anything) → make it **Public** (private repos can't use free GitHub Pages) → check "Add a README file" → Create.

3. **Upload your files.** On the repo page → "Add file" → "Upload files" → drag in **all 6 files** (`index.html`, `app.html`, all 4 `.md` files). Scroll down → "Commit changes."

4. **Turn on GitHub Pages.** Settings tab → Pages (left sidebar) → Source: "Deploy from a branch" → Branch: `main`, folder: `/ (root)` → Save. Wait 1–2 minutes.

5. **Test the site.** Visit `https://YOUR_USERNAME.github.io/smooth-paycheck/`. The landing page should appear. Click any CTA — it'll go to a broken `#REPLACE_WITH_DODO_CHECKOUT_URL` link, which is correct for now.

---

### Phase 2: Custom domain (10 minutes)

You've already got `smoothpaycheck.com` from Porkbun. To point it at GitHub Pages:

1. **In Porkbun**, find your domain → Details → DNS Records → Manage. Delete any default records (parking page, etc.) and add:

   | Type    | Host      | Answer                  | TTL |
   |---------|-----------|-------------------------|-----|
   | A       | (blank)   | 185.199.108.153         | 600 |
   | A       | (blank)   | 185.199.109.153         | 600 |
   | A       | (blank)   | 185.199.110.153         | 600 |
   | A       | (blank)   | 185.199.111.153         | 600 |
   | CNAME   | www       | YOUR_USERNAME.github.io | 600 |

2. **In GitHub** → Settings → Pages → Custom domain → type `smoothpaycheck.com` → Save.

3. **Wait ~10 minutes**, then visit `https://smoothpaycheck.com` in an incognito window. Your landing page should load.

4. **Once it works**, return to GitHub Pages settings and tick **Enforce HTTPS**. Your site is now live with HTTPS, free forever.

---

### Phase 3: Set up Dodo Payments (the payment processor)

You've decided on **Dodo Payments** — it's built for Indian sellers, handles global tax compliance, accepts individual sellers, lower fees than Lemon Squeezy (4% + $0.40 vs 5% + $0.50), and pays out cleanly to Indian banks with auto-generated FIRC documents.

The full setup walkthrough is in **`DODO-PAYMENTS-SETUP.md`** — a dedicated step-by-step guide that takes ~30 minutes once your account is verified.

The short version:
1. Sign up at [dodopayments.com](https://dodopayments.com) (free, KYC required)
2. Submit business verification — they review your live site (which is why Phase 2 comes first)
3. Once verified, create the Smooth Paycheck product at $29 with license keys enabled
4. Copy your checkout URL → paste into the 3 CTAs in `index.html`
5. Set `DODO_PLACEHOLDER_MODE = false` in `app.html`
6. Test with Dodo's sandbox before going live

The post-purchase email template buyers will receive is in **`post-purchase-email.md`** — paste it into Dodo's email settings before going live.

**If Dodo rejects you:** fall back to Lemon Squeezy. The full guide is in `LEMON-SQUEEZY-SETUP.md`. The trade-off is Lemon Squeezy's payouts are Stripe-mediated and Stripe is invite-only for Indian sellers since May 2024 — you may end up on PayPal-only payouts with a 4% FX hit.

---

## Where to edit common things

### Change the headline price
In `index.html`, search for `$29` — appears in the nav, pricing card, and final CTA. Update all three.

### Change the colors
Both files use the same CSS variables at the top. Change once, change everywhere:
```css
--accent: #2A6F4D;   /* primary green — change to rebrand */
--hi: #F4D35E;       /* yellow highlight */
```

### Change the hero number
In `index.html`, search for `$4,280` — that's the placeholder paycheck number on the hero preview card.

### Change the calculation logic
The math lives in `app.html`, in the `calculate()` function (search for `THE CALCULATION ENGINE`). Two key levers:
- `p30Index` — currently uses the 30th percentile of monthly income (conservative). Bump to 40th if buyers complain the number is too low.
- `paycheck = afterTax * 0.85` — leaves 15% headroom. Adjust freely.

### Add or remove a currency
In `app.html`, search for `CURRENCIES = {`. Add a new entry following the same pattern (`code: { symbol, locale }`). Then add a matching `<option>` in the picker dropdown (search for `currency-select`).

### Turn off placeholder mode for the license gate
In `app.html`, find `DODO_PLACEHOLDER_MODE = true` near the License Gate section. Change to `false`. Full setup steps are in `DODO-PAYMENTS-SETUP.md`.

---

## Pre-launch checklist

Before you announce on Instagram:

- [ ] Test the full flow on **mobile** (most Instagram traffic is mobile)
- [ ] Test the **currency picker** — switch between USD and INR mid-flow, confirm everything updates
- [ ] Test the **PDF download** — open the PDF on a phone and a desktop
- [ ] Try entering bad data (negative numbers, decimals, very large numbers)
- [ ] Confirm "Start over" actually clears localStorage
- [ ] **Make a test purchase** using Dodo's test mode — confirm the license key arrives in your email and unlocks the app
- [ ] Test the rejection cases: a fake key, an empty submit, an offline reload
- [ ] Set up a customer support email (`hello@smoothpaycheck.com`) and paste it into Dodo's product settings
- [ ] Customize the post-purchase email using `post-purchase-email.md`
- [ ] **Buy your own product once with a real card** to confirm the live flow works end-to-end. Refund yourself afterward.

---

## Post-launch: things worth adding later

In rough order of impact:

1. **Email capture on the landing page** — even just a "get notified about new tools" Beehiiv/Substack signup. Most visitors won't buy on first visit; email gets them back.
2. **Real testimonials** — once 5–10 customers have used it, ask for a one-line quote and a first name. Add them between Features and Pricing.
3. **Analytics** — [Plausible](https://plausible.io) ($9/mo) or [Umami](https://umami.is) (free, self-hosted). Skip Google Analytics; it's overkill and slows the site down.
4. **A "Made by [your name]" link in the footer** — a face on the product builds trust.
5. **Localized currency for the *checkout* (not just the app)** — once you have ~100 sales of data, look at where buyers came from. If 30% are from one country, localize the checkout price for that currency.

---

## Help

- **GitHub Pages docs**: [docs.github.com/en/pages](https://docs.github.com/en/pages)
- **Dodo Payments docs**: [docs.dodopayments.com](https://docs.dodopayments.com)
- **Stuck?** Open an issue in your repo, or come back to this conversation and ask.

Good luck with the launch.
