# Dodo Payments Setup — Step by Step

This guide gets your $29 product live on Dodo Payments and wired into the app. Total time: ~30 minutes once your Dodo account is verified.

You'll do this in three parts: **(1) get your Dodo account verified**, **(2) create the product and copy the keys**, **(3) wire your code and test end-to-end**.

---

## Why Dodo (and not Lemon Squeezy) for Indian sellers

A quick sanity check before you start, since this changed mid-build:

- **Lemon Squeezy** is excellent globally but routes Indian payouts through Stripe, which is invite-only for India since May 2024. You'd risk being stuck with PayPal-only payouts and a 4% FX hit.
- **Paddle** works for Indian sellers but has a 2–4 day manual review process and weaker community sentiment for indie sellers.
- **Dodo Payments** is built specifically for Indian sellers shipping to a global audience. Onboarding is hours not days. FEMA-compliant by design. Auto-generates the FIRC document you'll need for tax filing. Lower fees (4% + $0.40 vs 5% + $0.50). Built-in license keys with a public API that drops cleanly into the gate code already in `app.html`.

If Dodo rejects you for any reason, fall back to Paddle (then Lemon Squeezy as last resort). The instructions below assume Dodo.

---

## Part 1 — Get your Dodo account verified

### 1. Sign up

Go to [dodopayments.com](https://dodopayments.com) → click **Get Started**. Use your real email and Indian phone number.

You'll need to provide:
- Personal name (matches your PAN — important for payouts)
- Indian phone number for OTP verification
- Email address
- Business name (use **Smooth Paycheck**)
- Country (India)

You do **not** need a registered company. Dodo accepts individual sellers. See the founder testimonials in their docs — multiple solo devs explicitly chose Dodo because it accepts individuals where others don't.

### 2. Submit business verification

Inside the dashboard, you'll be prompted to verify your business. For a sole proprietor / individual, you'll typically need:

- **PAN card** (yours, as the proprietor)
- **Aadhaar** for KYC
- **Bank account details** — account number, IFSC, account holder name (must match PAN)
- A **brief description of what you sell** — write something like: *"Smooth Paycheck is a one-page web tool that helps freelancers calculate a sustainable monthly paycheck from their income history. Single $29 purchase, license-key activated, used in browser. Target audience is independent creators and side-hustlers globally."*
- **Website URL** — they need to see the landing page. This means **deploy `index.html` to GitHub Pages first** (using the README guide), even if Dodo isn't ready yet. They'll review the site to confirm it's a real product.

### 3. Wait for verification

Per Dodo's docs and customer testimonials, this usually takes a few hours, sometimes up to a day. While you wait, do everything else you can — domain setup, push the site live, draft your Instagram launch post.

If you hit a snag during verification, Dodo's support team is famously responsive. Reach out to them directly via the dashboard chat or [hello@dodopayments.com](mailto:hello@dodopayments.com).

---

## Part 2 — Create the product

Once verified, log into the dashboard.

### 1. Create the product

- **Products** → **New product** (or "Create product")
- **Name:** `Smooth Paycheck`
- **Description:** Paste this:
  > Turn your chaotic freelance income into a steady monthly paycheck — in under 5 minutes. Calculates your sustainable paycheck, allocation system, buffer target, and stability score. One-time payment. Use forever.
- **Pricing:** Single payment (not subscription)
- **Price:** `$29.00 USD`
- Optional: upload a thumbnail image (square, simple, on-brand)

### 2. Enable license keys

In the same product setup, find the **License keys** section.

- Toggle **Enable license keys** to **ON**
- **Activation limit:** **5** (lets the buyer use it on up to 5 devices)
- **Expires after:** Leave blank / "Never" (perpetual license — they paid once, they keep access)
- Save

Each buyer will now automatically receive a unique license key, both in their purchase email and on the success page.

### 3. Set the post-purchase return URL

Buyers should land directly on the app after paying, with the license key already filled in.

In the product settings, find **Return URL** (sometimes called **Redirect URL** or **Success URL**).

Set it to:
```
https://smoothpaycheck.com/app.html
```

Use whatever your real domain is. If you haven't bought a domain yet, use your GitHub Pages URL like `https://yourusername.github.io/smooth-paycheck/app.html`.

**Bonus:** Dodo automatically appends the license key as a query parameter to your return URL (`?license_key=XXX`), so you can pre-fill the gate input. We'll wire that up in Part 3.

### 4. Copy the checkout URL

- On the product page, click **Share** or **Copy checkout link**
- The URL will look like:
  ```
  https://checkout.dodopayments.com/buy/prod_xxxxxxxxxxxx
  ```

Save this — you'll paste it into `index.html` in the next step.

---

## Part 3 — Wire your code

Three small edits.

### A. Paste the checkout URL into `index.html`

Open `index.html` in any text editor.

**Find and replace** all instances of:
```
#REPLACE_WITH_DODO_CHECKOUT_URL
```

with your real Dodo checkout URL from step 4. There are **three** of these — hero CTA, pricing card, final CTA.

In VS Code: `Cmd/Ctrl + Shift + H` opens find-and-replace across the file. Click "Replace All."

### B. Turn off placeholder mode in `app.html`

Open `app.html`. Search for `DODO_PLACEHOLDER_MODE`. You'll see:

```javascript
const DODO_PLACEHOLDER_MODE = true;
const DODO_API = 'https://live.dodopayments.com';
```

Change `DODO_PLACEHOLDER_MODE` to `false`. Leave `DODO_API` as `live.dodopayments.com` for production — only change to `test.dodopayments.com` if you're using Dodo's sandbox.

```javascript
const DODO_PLACEHOLDER_MODE = false;
const DODO_API = 'https://live.dodopayments.com';
```

That's the only code change needed for real validation. The gate now calls Dodo's License API on every unlock and on every page load (silently, to catch refunds).

### C. (Optional but smart) Auto-fill the license key from the URL

When buyers complete checkout, Dodo redirects them to `app.html?license_key=XXX&payment_id=YYY&...`. You can pre-fill the gate input so they don't have to copy-paste.

Add this near the top of `app.html`'s main script block, just after the `DODO_API` line:

```javascript
// Auto-fill the gate input if Dodo redirected here with a key in the URL
(function autoFillLicenseFromURL() {
  const params = new URLSearchParams(window.location.search);
  const keyFromURL = params.get('license_key');
  if (keyFromURL) {
    // Wait for the gate input to exist, then fill it
    document.addEventListener('DOMContentLoaded', () => {
      const input = document.getElementById('license-input');
      if (input) input.value = keyFromURL;
    });
  }
})();
```

That's pure friction-killer. Buyer pays → land on the app → key already in the box → click Unlock → they're in. No copy-paste.

### D. Push to GitHub

Drag the updated `index.html` and `app.html` into your GitHub repo. Commit. Within ~60 seconds your live site uses real Dodo validation.

---

## Part 4 — Test end-to-end before announcing

The single most-skipped step in indie launches. Do not skip it.

### Test mode setup

Dodo offers a test/sandbox environment at `test.dodopayments.com`. To use it:

1. In your Dodo dashboard, look for **Test mode** or **Sandbox** toggle (usually top-right or in settings)
2. Switch it on. Your dashboard will visually indicate you're in test mode (often with a banner or tinted UI)
3. In `app.html`, temporarily change:
   ```javascript
   const DODO_API = 'https://test.dodopayments.com';
   ```
4. Use Dodo's test card for fake purchases — check the docs for the current test card number. Most platforms use `4242 4242 4242 4242` with any future expiry and any CVV
5. Complete a "purchase" in test mode

### What you're verifying

**The happy path:**
- After paying in test mode, you land on `app.html` with the license key in the URL
- The gate input pre-fills with the key (if you added the optional auto-fill)
- Clicking Unlock validates against Dodo's test API and unlocks the app
- You can walk through all 3 input steps and see the results
- The PDF downloads correctly

**The rejection paths:**
- Type a random string like `not-a-key` → gate shows "We couldn't find that license key."
- Click Unlock with the field empty → gate shows "Please paste your license key."
- Open a private/incognito window, try a key that's already been used 5 times → gate shows the activation-limit message

**The offline / refund paths:**
- Disconnect from the internet, reload `app.html` → it should let existing buyers back in (offline grace mode)
- Refund yourself in the Dodo dashboard, then reload `app.html` → the validation should fail and kick you back to the gate

### Going live

When test passes:

1. **Switch Dodo back to live mode** (toggle off Test mode in dashboard)
2. **Change `DODO_API` back** to `https://live.dodopayments.com` in `app.html`
3. **Confirm `DODO_PLACEHOLDER_MODE = false`**
4. Push the final commit to GitHub
5. **Buy your own product once with your real card** — confirm the live flow works end-to-end
6. **Refund yourself** from the dashboard (you'll be out the ~$1.56 in fees, cheapest insurance you'll ever buy)

---

## Recommended product settings

While you're in the dashboard, also set:

- **Refund policy:** 30 days, no questions (matches what your landing page promises)
- **Customer support email:** A real address you check daily. Buyers will email this when keys don't arrive
- **Email branding:** Replace Dodo's default order email with the template in `post-purchase-email.md`. You'll find this under **Settings → Emails** or similar

---

## Tax & compliance — the Indian-seller angle

Dodo handles global tax compliance for you (VAT, GST in other countries, US sales tax, etc.) — that's the whole point of using a Merchant of Record.

For **your Indian taxes**, here's the practical version:

- Dodo treats your payouts as **export of services revenue** to your Indian bank account. This is FEMA-compliant by design.
- You'll receive an **FIRC** (Foreign Inward Remittance Certificate) for each payout, auto-generated in the Dodo dashboard. Save these — your CA will need them for filing.
- File your earnings under **personal income tax** (presumptive taxation under Section 44ADA may apply if you qualify — talk to a CA).
- **GST registration** is only required if you cross ₹20 lakh annual turnover (or ₹10 lakh in certain states), or if a client asks for GST invoices. For digital exports specifically, GST is generally not charged on the export side, but you may still need to register and file zero-rated returns once you cross the threshold.
- **Udyam (MSME) registration** is free and takes 5 minutes online. Worth doing.

I'm not a CA. The above is my best understanding from research; your actual tax obligations depend on your situation. Talk to an accountant once you've made your first $1,000 — it's worth the ₹2,000 fee.

---

## When things go wrong

**A buyer says their key doesn't work.**
99% of the time, they're typing it instead of pasting. Have them copy directly from the email, or paste directly from the URL after checkout. If still broken, look up their key in **Customers → find them → see their orders + license keys** in your Dodo dashboard.

**A buyer hit the activation limit.**
You set it to 5, so this is rare. If it happens, find the key in the dashboard and either click **Reset activations** or increase the limit. Tell them you've cleared it and they can try again.

**A buyer wants a refund.**
Go to the order → click **Refund**. Dodo can be configured to automatically disable license keys when a refund is processed. If not, manually disable the key from the dashboard. The next time they open the app, the validation will fail and they'll be locked out.

**The app shows "Couldn't reach the license server."**
Almost always a buyer's network. The app is designed to fall back to "trust saved key" mode when offline, so existing buyers won't be locked out. If it persists, check if Dodo is having an outage (their status page or X account).

**You stopped getting payouts.**
Check your payout settings haven't been flagged for re-verification. Dodo sometimes asks for additional KYC if there's unusual activity. Respond promptly via dashboard chat.

---

## Cheat sheet

| Thing | Where | Value |
|---|---|---|
| Sign-up | dodopayments.com | Free, requires KYC |
| Fee per $29 sale | — | $1.56 (you keep $27.44) |
| Payout currency | INR (settled to your Indian bank) | — |
| Payout cadence | Per Dodo's schedule (typically 1st & 15th, varies) | — |
| Test API | `https://test.dodopayments.com` | Use during testing |
| Live API | `https://live.dodopayments.com` | Production default |
| Test card | `4242 4242 4242 4242` | Any future expiry + CVV |
| Activate endpoint | `POST /licenses/activate` | Public, no API key needed |
| Validate endpoint | `POST /licenses/validate` | Public, no API key needed |

That's everything. Once Dodo verifies your account, the rest is a 30-minute job.
