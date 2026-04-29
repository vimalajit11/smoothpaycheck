# Lemon Squeezy Setup — Step by Step

This guide gets your $29 product live on Lemon Squeezy and wired into the app. Total time: ~30 minutes.

You'll do this in three parts: **(1) create the Lemon Squeezy product**, **(2) wire your code to it**, **(3) test end-to-end before going live**.

---

## Part 1 — Set up your Lemon Squeezy store and product

### 1. Sign up

Go to [lemonsqueezy.com](https://www.lemonsqueezy.com) → click **Get Started**. It's free; no card required.

You'll be asked to:
- Set up a store name (use **Smooth Paycheck**)
- Pick a store URL slug (e.g., `smooth-paycheck` — this becomes part of your checkout URL)
- Verify your email

**Important:** Lemon Squeezy will eventually ask for tax/business details before you can receive payouts. You can fill these in now or later, but you can't get *paid* until they're complete. Plan for this — it can take a day or two for verification.

### 2. Create the product

In the Lemon Squeezy dashboard:

- Click **Products** → **New product**
- **Name:** `Smooth Paycheck`
- **Description:** Paste this:
  > Turn your chaotic freelance income into a steady monthly paycheck — in under 5 minutes. Calculates your sustainable paycheck, allocation system, buffer target, and stability score. One-time payment. Use forever.
- **Category:** Software / Tool (pick whichever fits best)
- **Pricing model:** Single payment (not subscription)
- **Price:** `$29.00 USD`
- Upload a thumbnail image — anything clean and on-brand. A simple logo on a beige background works.

### 3. Turn on license keys

This is the critical step that ties everything together.

In your product setup:

- Scroll to the **License keys** section
- Toggle **Generate license keys** to **ON**
- **License length:** **Perpetual** (or "No expiry")
- **Activation limit:** **5** (lets the buyer use it on up to 5 devices — phone, laptop, tablet, etc. — without needing to deactivate)
- Save

Each buyer will now automatically receive a unique license key in their purchase email.

### 4. Set up the post-purchase redirect

You want buyers to land directly on the app after paying.

- In the product settings, find **Thank-you page** or **Redirect URL**
- Set it to: `https://smoothpaycheck.com/app.html`
- (Use whatever your real domain is — if you haven't bought one yet, you can use the GitHub Pages URL like `https://yourusername.github.io/smooth-paycheck/app.html` for now)

The buyer will receive their license key via email **and** see it on the thank-you page, so they can copy it directly into the app.

### 5. Find your Product ID

This is the number you'll paste into your code so the app can verify keys belong to *your* product, not someone else's.

- Go to **Products** in the dashboard
- Click on your **Smooth Paycheck** product
- Look at the URL in your browser. It'll look like:
  `https://app.lemonsqueezy.com/products/12345`
- Copy that number (`12345` in this example) — that's your product ID.

### 6. Get your checkout URL

- On your product page, click **Share** or **Buy this product**
- Copy the checkout URL. It'll look like:
  `https://your-store.lemonsqueezy.com/buy/aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee`

Save both numbers somewhere — you need them in Part 2.

---

## Part 2 — Wire your code to Lemon Squeezy

Two files to edit. Both changes are tiny.

### A. Paste your checkout URL into `index.html`

Open `index.html` in any text editor (VS Code, Sublime, Notepad++, even GitHub's web editor).

**Find and replace** every instance of:

```
#REPLACE_WITH_LEMONSQUEEZY_CHECKOUT_URL
```

with your actual checkout URL from step 6 above. There are **three** of these — hero CTA, pricing card, final CTA.

In VS Code: `Cmd/Ctrl + Shift + H` opens find-and-replace across the file. Click "Replace All."

### B. Paste your product ID + turn off placeholder mode in `app.html`

Open `app.html`. Search for `LS_PRODUCT_ID` (around line 1320). You'll see:

```javascript
const LS_PRODUCT_ID = 0;  // e.g. 123456
const LS_PLACEHOLDER_MODE = true;
```

Change to:

```javascript
const LS_PRODUCT_ID = 12345;  // your actual product ID from step 5
const LS_PLACEHOLDER_MODE = false;
```

Save the file. **That's all the code changes needed.** The app will now validate every license key against Lemon Squeezy's License API, check the key belongs to your product, and let through only real buyers.

### C. Push to GitHub

Drag the updated `index.html` and `app.html` into your GitHub repo (web interface works fine, no Git knowledge required), commit the changes, and within ~60 seconds your live site will be using real validation.

---

## Part 3 — Test end-to-end before announcing

This is the part most indie sellers skip and regret. **Don't skip this.**

### Test mode: a free purchase

Lemon Squeezy has a built-in test mode that lets you "buy" your own product without paying anything.

1. In the Lemon Squeezy dashboard, find the **Test mode** toggle in your settings (often top-right of the dashboard)
2. Turn it **ON**
3. Go to your checkout URL in a private/incognito browser window
4. Use Lemon Squeezy's test card: `4242 4242 4242 4242` with any future expiry and any CVV
5. Complete the "purchase"

You should now:
- See the thank-you page with your license key
- Receive a test email with the same key
- Get redirected to `app.html`

### Test the gate

1. Open `app.html` in a fresh private browser window
2. Paste the test license key
3. Click Unlock

You should see the app unlock. Walk through all 3 input steps, click "Show my paycheck," confirm the results screen renders, hit "Download PDF" and verify the PDF opens.

### Test the rejection cases

These are the tests most people skip:

- Type a random string like `not-a-real-key` → app should show "We couldn't find that license key."
- Type a real key from a different Lemon Squeezy product (if you have one) → app should show "This key is for a different product."
- Click Unlock with the field empty → app should show "Please paste your license key."
- Disconnect from the internet, reload the app → it should let you back in (offline grace mode for existing buyers).

### Go live

When everything works:

1. Turn **Test mode OFF** in Lemon Squeezy
2. Make sure `LS_PLACEHOLDER_MODE = false` in `app.html` and your product ID is set
3. Push the final commit to GitHub
4. **Buy it once with your own real card** to confirm the live flow works
5. Refund yourself from the Lemon Squeezy dashboard once you've confirmed it works (you'll be out the ~$2 in fees, which is the cheapest insurance you'll ever buy)

---

## Recommended product settings (worth doing now)

Inside your product → Settings, also configure:

- **Refund policy:** Toggle on, set to **30 days, no questions** (matches what your landing page promises)
- **Customer support email:** Use a real address you check daily. Buyers will email this when keys don't arrive
- **Receipt customization:** Add a brief note like *"Welcome in! Your license key is below — paste it into the app at smoothpaycheck.com/app.html. If anything breaks, just reply to this email."*

---

## The post-purchase email (use the template)

Lemon Squeezy will send buyers a default receipt automatically. It's fine, but you should override it with something more personal. See `post-purchase-email.md` in this folder for the recommended template.

To set it up: **Store settings → Emails → Order confirmation** in the Lemon Squeezy dashboard.

---

## When things go wrong

**A buyer says their key doesn't work.**
99% of the time, they're typing it instead of pasting (license keys have hyphens that are easy to miss). Have them copy directly from the email. If still broken, you can manually look up their key in **Customers** → find them → see their orders + keys.

**A buyer hit the activation limit.**
You set it to 5, so this is rare, but if it happens, go to the license key in the dashboard and click **Reset activations**. Tell them you've cleared it and they can try again.

**A buyer wants a refund.**
Go to the order → click **Refund**. Lemon Squeezy automatically disables their license key when you refund, so the next time they open the app it will lock back to the gate. You don't need to do anything else.

**The app shows "Couldn't reach the license server."**
This is almost always a buyer's flaky network. If it persists, check Lemon Squeezy's status page (status.lemonsqueezy.com). Your app is built to fall back to "trust saved key" mode when offline, so existing buyers won't be locked out.

---

That's everything. Once Part 3 is done you're literally ready to announce on Instagram.
