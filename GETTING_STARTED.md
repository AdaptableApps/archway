![Archway](https://adaptableapps.net/images/Archway_Logo_v1_Banner_White_On_Black.svg)

# Getting started

Welcome to Archway. This guide takes you from nothing to a working storefront that sells **your** product
subscriptions to **your** customers, with the payments going straight into your own Stripe account.

> ### 👉 [https://adaptableapps.live.us.app.archwayportal.com](https://adaptableapps.live.us.app.archwayportal.com)
>
> Start here. This is where you create your account and subscribe to Archway.

**The steps at a glance:**

1. [Create your account](#step-1---create-your-account)
2. [Subscribe to Archway](#step-2---subscribe-to-archway)
3. [Create your tenancy](#step-3---create-your-tenancy) - your company's own space, with its own address
4. [Connect your Stripe account](#step-4---connect-your-stripe-account)
5. [Publish your products and prices](#step-5---publish-your-products-and-prices)
6. [Try your store as a customer, then go live](#step-6---try-your-store-as-a-customer-then-go-live)
7. [Optional extras](#step-7---optional-extras) - sign-in options, a second administrator, connecting your own
   software

Allow about **30-45 minutes** for steps 1-6. You can stop after any step and pick up where you left off.

---

## Before you start

**You will need:**

- A modern browser - Chrome, Edge, Safari or Firefox, kept up to date.
- An email address you can receive mail at.
- A card to pay for your Archway subscription.
- A **[Stripe](https://stripe.com) account** for your company, **activated for live payments** - that is, with
  your business details and the bank account your customers' payments should be paid out to. Stripe walks you
  through this when you activate the account. You can set Archway up with Stripe's **test** keys first and
  switch to live ones when you are ready (step 6).

---

## Step 1 - Create your account

1. Open the Archway address above. The first thing you are asked for is your **region** - choose the one closest
   to you.

   <!-- SCREENSHOT NEEDED: images/getting-started/01-region-select.png - the region selection page, before any account exists. -->

   > **About the list.** The regions on offer are the ones where our payment provider supports business bank
   > accounts, because that is what decides where a SaaS company can actually be *paid*. It governs where
   > **your company** can be based - not where your customers can be. They can be anywhere.
   >
   > If your country is not listed, please [get in touch](#getting-help) - more regions are being added.

2. Choose **Sign Up** and fill in your name, email address, password, time zone and language.

   <!-- SCREENSHOT NEEDED: images/getting-started/02-sign-up.png - the completed sign-up form with example details. -->

3. Sign in with the email address and password you just used.

---

## Step 2 - Subscribe to Archway

Your Archway subscription is what lets you open a storefront of your own.

1. Open the **Store**. You will see the products on offer with their plans and prices.

   <!-- SCREENSHOT NEEDED: images/getting-started/03-store.png - the Store page showing a product with its Plans & Pricing. -->

2. Choose an **Archway** plan, add it to your **Cart** and check out. You are handed to Stripe's secure payment
   page to pay.

   <!-- SCREENSHOT NEEDED: images/getting-started/04-checkout.png - the cart immediately before checkout. -->

3. After paying you are returned to Archway. Open **My Subscriptions** and check your new subscription is listed
   and active. Keep this page open - the next step starts from it.

   <!-- SCREENSHOT NEEDED: images/getting-started/05-my-subscriptions.png - My Subscriptions listing one active subscription. -->

---

## Step 3 - Create your tenancy

A **tenancy** is your company inside Archway. It owns your workspaces, your products and your customers - and it
gets its own address, which is where your customers will shop.

1. On your **Archway subscription**, choose **Create Tenancy**.

   The button appears on the **Archway** subscription only - if it seems to be missing, check which subscription
   you are looking at.

   <!-- SCREENSHOT NEEDED: images/getting-started/06-create-tenancy-button.png - the Archway subscription and its Create Tenancy button. -->

2. Fill in the details it asks for. One of them is your **subdomain**, and it decides your address:

   > a subdomain of `yourcompany` gives you
   > **`https://yourcompany.live.us.app.archwayportal.com`**

   Choose it carefully - short, lowercase and recognisably yours. It is the address you will give your
   customers, and the name your own software uses to talk to Archway.

   <!-- SCREENSHOT NEEDED: images/getting-started/07-create-tenancy-dialog.png - the input dialog, filled in, with the subdomain field visible. -->

3. Archway creates the tenancy, refreshes the subscription, and gives you a **button to open your new tenancy**.
   Click it - that takes you to your own address, and everything from here happens there.

   <!-- SCREENSHOT NEEDED: images/getting-started/08-open-tenancy.png - the refreshed subscription showing the button that opens the tenancy. -->

4. Your new address asks you to **choose a region** first. That is expected - the region is recorded per tenancy,
   because your storefront serves your customers from it.

   > **If you briefly see a "maintenance" message while your tenancy is being created**, that is expected too.
   > Archway is preparing your tenancy's own database. It clears itself within moments - there is nothing to do
   > but wait.

5. **Sign in again** at your new address, with the same email address and password. You are the tenancy's
   **owner**. Your sign-in does not follow you from one address to another - each tenancy is its own storefront,
   which is exactly how your customers will experience yours.

---

## Step 4 - Connect your Stripe account

Archway takes payments through **your** Stripe account, so your customers' money goes straight to you.

Everything you set up as an owner lives in **Tenant Center**, reachable from the menu once you are signed in at
your tenancy's address: your payment provider, your products and your prices.

1. In **Tenant Center**, open your tenancy and add a **Payment Provider**.

2. Choose the **region** it serves. You add the provider **once per region** - if you sell into two regions,
   that is two entries, each with its own keys. Adding the same provider twice for the same region is refused on
   purpose.

3. Enter your Stripe keys. There are two sets - **test** keys for trying things out and **live** keys for real
   payments - and each set has two values:

   | Field in Archway | What to enter | Where to find it in Stripe |
   |---|---|---|
   | **Sandbox Payment Provider Account Key** | Your Stripe **account ID** - starts with `acct_` | Dashboard → **Settings** → **Business** → *Account details* |
   | **Sandbox Payment Provider Private Api Key** | Your **test** secret key - starts with `sk_test_` | Dashboard → **Developers** → **API keys**, with test mode on |
   | **Live Payment Provider Account Key** | Your Stripe **account ID** - starts with `acct_` | as above |
   | **Live Payment Provider Private Api Key** | Your **live** secret key - starts with `sk_live_` | Dashboard → **Developers** → **API keys**, with test mode off |

   The keys are hidden on screen once entered, to keep them safe.

4. Leave **Live Mode** switched **off** for now. While it is off, Archway uses your test keys and no real money
   moves; step 6 is where you switch it on.

   <!-- SCREENSHOT NEEDED: images/getting-started/09-payment-provider.png - the Tenant Payment Provider screen, with the key values blurred. -->

> **This step is what opens your store.** Until a payment provider is linked and active for a region, your
> storefront will not open for visitors in it - there would be no way to take their money. If your store will
> not open for a signed-out visitor later on, come back here first.

**Treat the secret keys like passwords.** Never send them by email or paste them anywhere other than these fields.
If you think one has been exposed, roll it in your Stripe dashboard and enter the new one here.

---

## Step 5 - Publish your products and prices

Products and prices live in **Tenant Center** too.

1. Create a **product** - name, description and image.

   <!-- SCREENSHOT NEEDED: images/getting-started/10-create-product.png - the Product form filled in with an example product. -->

2. Add a **price** to it: the amount, the currency and how often it recurs.

   <!-- SCREENSHOT NEEDED: images/getting-started/11-create-price.png - the Product Price form filled in. -->

3. **Approve** the price. A price is not offered for sale until it has been approved - so a half-finished price
   is never visible to a customer.

   <!-- SCREENSHOT NEEDED: images/getting-started/12-approve-price.png - the approval step, or the price showing as approved. -->

4. Open the **Store** at your tenancy's address and check your product and price appear exactly as you intended -
   especially the amount, the currency and the decimal point.

---

## Step 6 - Try your store as a customer, then go live

Before you send customers to your store, buy from it yourself.

1. Sign out.

2. Go to **your own address** - `https://yourcompany.live.us.app.archwayportal.com` - and sign up there with a
   **different email address**, as a plain customer.

3. Buy one of your products. With **Live Mode** still off, use one of Stripe's test cards - no real money moves:

   | Purpose | Card number | Expiry | CVC |
   |---|---|---|---|
   | Payment succeeds | `4242 4242 4242 4242` | any future date | any 3 digits |
   | Payment is declined | `4000 0000 0000 0002` | any future date | any 3 digits |

4. Check the subscription appears under that customer's **My Subscriptions**, and the payment appears in your
   Stripe dashboard's **test** data.

5. **Go live.** Sign back in as the owner, open your Payment Provider in **Tenant Center**, make sure the **live**
   keys are entered, and switch **Live Mode** on. From now on your store takes real payments, paid out to your
   Stripe account.

> **Tip:** after switching Live Mode on, a single small real purchase - refunded afterwards from your Stripe
> dashboard - is a good final check that everything is connected.

Your store is open. Share your address - `https://yourcompany.live.us.app.archwayportal.com` - with your customers.

---

## Step 7 - Optional extras

### Let customers sign in with Google, GitHub and others

On your tenancy in **Tenant Center**, the **Authentication Providers** section lists the ways people can sign in
to your store. Add one to let people sign in with that service instead of a password.

### Add a Tenant Admin

The tenancy **owner** - you - can hand day-to-day administration to someone else. On your tenancy in **Tenant
Center**, choose **Change Admin** and enter the email address the person uses for their Archway account - they
need to have signed up already. They get administrator access to the tenancy; ownership stays with you.

### Connect your own software

If your product is software, it can check for itself whether a customer's subscription is active - when they sign
in, when it starts, or a few times a day.

Every **customer account** and every **subscription** in Archway has a **secret key**. Your customer can copy them
with **Copy Key** in the menu at the top right of their customer account and of the subscription, and enter them
into your software. See the **[API integration guide](API_INTEGRATION.md)** for the details, with examples in
many programming languages and for no-code tools such as Zapier.

---

## Good to know

- **Finding things in long lists.** Sections that can hold many rows - subscriptions, products, customers - have a
  **search button beside the expander arrow**. Type a few letters and press **Enter**; clearing the box brings the
  full list back. A section shows the **100 most recent** entries, newest first, so on a long list search is the
  way to reach older ones.
- **Cancelling takes up to an hour to take effect.** A subscription shows as cancelled straight away, but what it
  paid for can keep working for up to an hour.
- **Error messages are deliberately brief.** If something goes wrong you will usually see *"Something went wrong.
  Please try again."* rather than technical detail - that belongs in our logs, not on your screen. If it keeps
  happening, [get in touch](#getting-help) and tell us the time you saw it.
- **On a phone or tablet** - Archway works in any modern mobile browser too.

---

## Getting help

Email **[support@adaptableapps.net](mailto:support@adaptableapps.net)**. A couple of sentences is fine.

**To help us help you quickly, include:**

1. **What you did** - the steps, in order.
2. **What happened**, and **what you expected** instead.
3. **When** it happened, with your time zone - "about 14:35, SAST" is enough. This is the most important line,
   because the time is how we find the details in our logs.
4. **A screenshot**, if the problem is something you can see.
5. **Your browser and device** - "Chrome on Windows", "Safari on iPhone".

**Never include your password or your Stripe secret keys.** We will never need either of them to look into
something.
