# Smooth Paycheck

A 5-minute web tool that turns chaotic freelance income into a steady monthly paycheck. No spreadsheets, no signup, no accounting software.

Live at **[smoothpaycheck.com](https://smoothpaycheck.com)**.

## What's in here

```
smooth-paycheck/
├── index.html                 # The landing page (free, public)
├── app.html                   # The full tool, license-gated
├── README.md                  # This file
└── post-purchase-email.md     # Email template buyers receive after checkout
```

Two HTML files for the app, two markdown files for ops. No backend, no database, no build step.

---

## How it works

- **`index.html`** is the marketing page. Anyone can see it. The three "Get Smooth Paycheck" CTAs point to the Dodo Payments checkout URL.
- **`app.html`** is the full app. It validates license keys against Dodo's License API at `live.dodopayments.com/licenses/activate` (no backend needed — the API is CORS-enabled and called directly from the browser).
- The user's data (income, expenses, results) is saved to their browser's `localStorage`. Nothing leaves their device except the license-key check.
- The PDF download is generated on the fly using jsPDF (loaded from CDN). No server needed.

---

## Deployment — GitHub Pages + custom domain

This is what's already set up. Documenting for future reference.

### GitHub Pages
- Repo is `Public` (free GitHub Pages requires this)
- `index.html` and `app.html` live in the repo root
- **Settings → Pages → Source: Deploy from a branch → main / (root)**
- GitHub Pages rebuilds in ~60 seconds after each push

### Custom domain (`smoothpaycheck.com`)
DNS records at the registrar:

| Type    | Name | Value                    |
|---------|------|--------------------------|
| A       | @    | 185.199.108.153          |
| A       | @    | 185.199.109.153          |
| A       | @    | 185.199.110.153          |
| A       | @    | 185.199.111.153          |
| CNAME   | www  | YOUR_USERNAME.github.io  |

In **Settings → Pages → Custom domain**: `smoothpaycheck.com`, with **Enforce HTTPS** checked.

---

## Payment processing — Dodo

Set up via Dodo Payments. The product is **Smooth Paycheck**, priced at **$29** one-time, with license keys enabled.

- **Checkout URL:** `https://checkout.dodopayments.com/buy/pdt_0Ne8DXvv3Zm35mYFVBr5i`
- **Redirect URL after purchase:** `https://smoothpaycheck.com/app.html`
- **License key delivery:** Dodo sends the email automatically. Sender is `noreply@dodopayments.com`, subject `🥳 Your License Key is Ready – Activate Your Product Now!`
- **Custom post-purchase email content:** see `post-purchase-email.md`

The license-gate code in `app.html` reads the product ID and calls Dodo's `/licenses/activate` endpoint on first use, then `/licenses/validate` on returning visits. Stored in `localStorage` under `smoothpaycheck_license`.

---

## Where to edit common things

### Change the headline price
In `index.html`, search for `$29` — it appears in the nav bar, pricing card, and final CTA. Update all three. If changing price, also update it in Dodo dashboard.

### Change the colors
Both files use the same CSS variables at the top. Change once, change everywhere:
```css
--accent: #2A6F4D;   /* primary green — change to rebrand */
--hi: #F4D35E;       /* yellow highlight */
```

### Change the hero number
In `index.html`, search for `$4,280` — that's the placeholder paycheck number on the preview card. Pick something representative of the audience.

### Change the calculation logic
The math lives in `app.html`, in the `calculate()` function (search for `THE CALCULATION ENGINE`). The two key levers are:
- `p30Index` — currently uses the 30th percentile of monthly income (conservative). Bump to 40th if buyers complain the number is too low.
- `paycheck = afterTax * 0.85` — leaves 15% headroom. Adjust freely.

### Update the checkout URL
If the Dodo product is ever re-created or migrated, search `index.html` for `checkout.dodopayments.com` — appears in three places, all three need to match.

---

## Periodic verification

Things worth re-checking every few weeks or after any code change:

- [ ] Visit `smoothpaycheck.com` on mobile and desktop — confirm CTAs land on Dodo checkout, not anywhere broken
- [ ] Visit `/app.html` directly, paste a random string — confirm it shows "That key didn't work" (not unlocking the tool)
- [ ] Confirm Pinterest verification still active (`smoothpaycheck.com` should appear claimed in Pinterest Settings → Link to Pinterest)
- [ ] Hard-refresh the site in an incognito tab and confirm the favicon (green circle, "S") renders in the browser tab

---

## Help

- **GitHub Pages docs:** [docs.github.com/en/pages](https://docs.github.com/en/pages)
- **Dodo Payments docs:** [docs.dodopayments.com](https://docs.dodopayments.com)
