![Archway](https://adaptableapps.net/images/Archway_Logo_v1_Banner_White_On_Black.svg)

# Beta testing guide

Thank you for helping us test Archway. This guide walks you through the whole product the way a real SaaS
company would use it: create an account, open a storefront, connect Stripe, publish products, and then buy
from your own store as a customer.

It should take about **30-45 minutes** to work through everything. You are welcome to stop after any part -
each one is useful on its own.

> ### 👉 [https://adaptableapps.demo.us.app.archwayportal.com](https://adaptableapps.demo.us.app.archwayportal.com)
>
> This is the beta environment. It is separate from anything live, and its data may be reset while we work.

---

## Before you start

**You will need:**

- A modern browser - Chrome, Edge, Safari or Firefox, kept up to date.
- An email address you can receive mail at.
- A **Stripe test-mode account** if you want to try the storefront parts (parts 3-6). It is free to create
  at [stripe.com](https://stripe.com) and you will need your **test** publishable and secret keys.

**Nothing here involves real money.** Archway is running against Stripe in **test mode**, so use Stripe's
test card numbers and never a real card:

| Purpose | Card number | Expiry | CVC |
|---|---|---|---|
| Payment succeeds | `4242 4242 4242 4242` | any future date | any 3 digits |
| Payment is declined | `4000 0000 0000 0002` | any future date | any 3 digits |

**Please do not put real customer data into the beta.** Use made-up company names, products and prices. Beta
data may be wiped without notice while we work.

---

## Known issues - please don't report these

We already know about these. Everything *else* is worth telling us about.

- **The menu shows pages you cannot open.** Depending on your permissions you may see entries such as
  *Tenant Center* or *Security Center*. Tapping one gives a message saying you do not have permission. That
  message is correct - the menu simply should not have offered it.
- **Dark mode only.** Light mode exists but is not finished, so the beta runs in dark. If you switch to
  light you will find styling that has not been done yet - that is known, and not worth reporting.
- **Free-trial length is not displayed.** If a price includes a free trial, the Store and the subscription
  pages do not say so yet.
- **Checkout is slow when the cart is large.** Every item has to be confirmed with Stripe one at a time, so a
  cart with a dozen or more subscriptions can sit on the checkout page for a minute or more, and may look as
  though it has failed when it has not. Give it time before retrying - and if you do want to test the slow
  case deliberately, that is genuinely useful to us. A cart of two or three items is quick.
- **Error messages are deliberately vague.** If something goes wrong you will usually see *"Something went
  wrong. Please try again."* rather than a technical description. That is on purpose - the details belong in
  our logs, not on your screen - but it means **the time you saw it is the most useful thing you can tell
  us**. See *What to report* at the end.
- **Cancelling takes up to an hour to bite.** The subscription itself shows as cancelled straight away, but
  the tenancy it paid for can keep working for up to **an hour** before it starts showing the "Not Active"
  page. That is a caching window, not a mistake - so if you cancel something and can still get in, that is
  expected for now. Worth telling us if it is still letting you in the next day.
- **Older tenancies have several workspaces all called "Public".** Fixed for tenancies created from 25
  August - their workspaces are named after their region. If yours were created before that, the
  Restrict-to-Workspace picker will offer several identically named entries and there is no way to tell them
  apart. Nothing is wrong with the data, only the names.
- **Safari: the very bottom of a long page can be hard to reach** - the last 30-40 pixels. Other browsers are
  fine.

---

## Part 1 - Create your account

1. Open the beta URL. The first thing you are asked for is your **region** - choose the one closest to you.

   ![Screenshot: region selection](images/beta/01-region-select.png)
   <!-- SCREENSHOT NEEDED: the region selection page, before any account exists. -->

   > **About the list.** The regions on offer are the ones where our payment provider supports business bank
   > accounts, because that is what decides where a SaaS company can actually be *paid*. Between them they
   > cover most of the world's SaaS businesses. It governs where **your company** can be based - not where
   > your customers can be, and they can be anywhere.
   >
   > If your country is not listed you can still test everything a customer does (Parts 1, 2 and 7), but you
   > will not be able to open a storefront of your own. We are adding a second payment provider after launch
   > to bring more of Africa in, and other regions will follow the same way.

2. Choose **Sign Up** and fill in your name, email address, password, time zone and language.

   ![Screenshot: sign-up form](images/beta/02-sign-up.png)
   <!-- SCREENSHOT NEEDED: the completed sign-up form with example (fake) details. -->

3. Sign in with the email address and password you just used.

**What we're testing:** that sign-up is clear, that nothing is asked for twice, and that you end up signed in
without having to guess what to do next.

---

## Part 2 - Subscribe to Archway

This part does two jobs: you see the store from your customer's side, and you subscribe to **Archway**
itself - which is what lets you open a storefront of your own in Part 3.

1. Open the **Store**. You will see the products on offer with their plans and prices.

   ![Screenshot: the Store](images/beta/03-store.png)
   <!-- SCREENSHOT NEEDED: the Store page showing a product with its Plans & Pricing. -->

2. Choose an **Archway** plan and add it to your **Cart**, then check out. You will be handed to Stripe's
   payment page - use the test card above.

   ![Screenshot: checkout](images/beta/04-checkout.png)
   <!-- SCREENSHOT NEEDED: the cart immediately before checkout (not the Stripe page itself). -->

3. After paying you are returned to Archway. Open **My Subscriptions** and check your new subscription is
   listed and active. Keep this page open - Part 3 starts from it.

   ![Screenshot: my subscriptions](images/beta/05-my-subscriptions.png)
   <!-- SCREENSHOT NEEDED: My Subscriptions listing one active subscription. -->

**What we're testing:** that the price you saw is the price you paid, that returning from Stripe works
cleanly, and that the subscription appears without you having to refresh or hunt for it.

---

## Part 3 - Create your tenancy

A **tenancy** is your company inside Archway. It owns your workspaces, your products and your customers -
and it gets its own address.

1. On your **Archway subscription**, choose **Create Tenancy**.

   The button appears on the **Archway** subscription only. Other products have nothing to provision, so you
   will not find it on them - if it seems to be missing, check which subscription you are looking at.

   ![Screenshot: the Archway subscription with the Create Tenancy button](images/beta/06-create-tenancy-button.png)
   <!-- SCREENSHOT NEEDED: My Subscriptions showing the Archway subscription and its Create Tenancy button. -->

2. Fill in the details it asks for. One of them is your **subdomain**, and it decides your address:

   > a subdomain of `yourcompany` gives you
   > **`https://yourcompany.demo.us.app.archwayportal.com`**

   Choose it as you would a real one - short, lowercase, and recognisably yours.

   ![Screenshot: the create-tenancy dialog](images/beta/07-create-tenancy-dialog.png)
   <!-- SCREENSHOT NEEDED: the input dialog, filled in, with the subdomain field visible. -->

3. Archway creates the tenancy, refreshes the subscription, and gives you a **button to open your new
   tenancy's page**. Click it - that takes you to your own address, and everything from here happens there.

   ![Screenshot: the open-tenancy button](images/beta/08-open-tenancy.png)
   <!-- SCREENSHOT NEEDED: the refreshed subscription showing the button that opens the tenancy. -->

4. Your new address asks you to **choose a region** first. That is expected - the region is recorded per
   tenancy, not per person, because your storefront serves your customers from it.

   > **If you briefly see a "maintenance" message while your tenancy is being created**, that is expected
   > too. Archway is preparing your tenancy's own database, and it holds only *your* tenancy still for those
   > few seconds - other people using the beta at the same time are not affected. It clears itself; there is
   > nothing to do but wait.

5. **Sign in again** at your new address, with the same email address and password you used to create the
   tenancy. You are its owner. (Your sign-in does not follow you across addresses - each one is its own
   storefront, which is exactly how your customers will experience yours.)

**What we're testing:** whether it is clear what a tenancy is and what the subdomain will be used for before
you commit to one, and whether the hand-off to your new address is smooth.

> **Note:** the **Owner User** and **Admin User** fields on your tenancy are deliberately read-only.
> Changing who administers a tenancy will be done through a dedicated action later in the beta.

---

## Part 4 - Set up your store: connect Stripe

Archway takes payments through **your** Stripe account, so your customers' money goes straight to you.

Parts 4 and 5 together are how you set up your store. Everything an owner sets up lives in **Tenant
Center**, reachable from the menu once you are signed in to your own tenancy: your payment provider, your
products and your prices.

1. In **Tenant Center**, link a **payment provider** and enter your Stripe **test-mode** keys.

   ![Screenshot: payment provider](images/beta/09-payment-provider.png)
   <!-- SCREENSHOT NEEDED: the Tenant Payment Provider screen. Blur or fake the key values. -->

2. Choose the region or regions you want that provider to serve. You link the provider **once per region**
   - so if you sell into two regions, that is two entries, each with its own keys. Trying to add the same
   provider twice for the same region is refused on purpose, and should tell you so plainly.

**Please use test keys only.** Never paste a live secret key into the beta.

> **This step is what opens your store.** Until a payment provider is linked and active for a region, your
> storefront will not open for visitors in it - there would be no way to take their money. If you get to
> Part 6 and your own store will not open for a signed-out visitor, come back here first.

**What we're testing:** whether it is clear which keys are needed and where to find them in Stripe, and
whether anything about handing over your keys feels uncomfortable. We want to hear that.

---

## Part 5 - Set up your store: publish a product

Products and prices live in **Tenant Center** too.

1. Create a **product** - name, description and image.

   ![Screenshot: create product](images/beta/10-create-product.png)
   <!-- SCREENSHOT NEEDED: the Product form filled in with an example product. -->

2. Add a **price** to it: the amount, the currency and how often it recurs.

   ![Screenshot: create price](images/beta/11-create-price.png)
   <!-- SCREENSHOT NEEDED: the Product Price form filled in. -->

3. **Approve** the price. A price is not offered for sale until it has been approved - this is deliberate, so
   a half-finished price is never visible to a customer.

   ![Screenshot: approving a price](images/beta/12-approve-price.png)
   <!-- SCREENSHOT NEEDED: the approval step, or the price showing as approved. -->

4. Open the **Store** and confirm your product and its price now appear.

**What we're testing:** whether the approval step makes sense to you, and whether anything you entered comes
back looking different from how you typed it - especially prices, currencies and decimal points.

---

## Part 6 - Buy from your own store

The most valuable test in this guide: be your own customer.

1. Sign out.
2. Go to **your own address** - `https://yourcompany.demo.us.app.archwayportal.com` - and sign up there with
   a **different email address**, as a plain customer with no special rights.
3. Buy one of the products you published, using the test card.
4. Check that the subscription appears under **My Subscriptions**.

**What we're testing:** that an ordinary customer sees exactly what they should - your products, your prices,
their own subscriptions - and nothing that belongs to you or to anyone else. **If you ever see data that
isn't yours, please tell us immediately.**

---

## Part 7 - Everyday things

Worth a few minutes each, because they are what people do most:

- Sign out and back in again.
- Edit your own user details and your account.
- Switch between tenancies or workspaces if you have more than one.
- **Try searching a long list.** Sections that can hold many rows - subscriptions, products, tenants - have a
  **search button beside the expander arrow**. Click it to open the box, type a few letters, then either
  press **Enter** or click the **search icon** in the box; clearing the box brings the full list back.
  Worth knowing: a section shows the **100 most recent** entries, newest first, so on a very long list the
  search box is the way to reach the older ones.
- Resize the browser window, and try it on a phone or tablet if you have one to hand.

---

## What to report, and how

Whichever is easier for you:

- **Email** [support@adaptableapps.net](mailto:support@adaptableapps.net), or
- **Post it in the beta testers WhatsApp group.**

Both reach us. There is nothing to sign up for and no form to fill in - a couple of sentences is fine.

**A good report includes:**

1. **What you did** - the steps, in order.
2. **What happened**, and **what you expected** instead.
3. **When** it happened, with your time zone - roughly is fine, to the nearest minute is better. **This is
   the most important line in your report.** Because error messages on screen are deliberately vague (see
   *Known issues*), the time is usually how we find what actually went wrong in our logs. "About 14:35, SAST"
   is enough.
4. **Which part of this guide** you were on, if you were following it.
5. **A screenshot**, if the problem is something you can see.
6. **Your browser and device** - "Chrome on Windows", "Safari on iPhone".

**Please don't include your password or your Stripe secret key** - not in an email, and especially not in
the group, where everyone else can see it. We will never need either of them to look into something. If you
think a key has been exposed, roll it in your Stripe dashboard; in test mode nothing is at risk, but it is a
good habit.

Small things are worth reporting too. Wording that confused you, a button that was not where you expected,
a page that felt slow - that feedback is as useful to us as an error message.

Thank you for your time. It genuinely makes the product better.
