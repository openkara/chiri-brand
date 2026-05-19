# Chiri Landing Pages — Reference & Webmaster Instructions

---

## Landing Page Reference

| Cadence | Persona | Target URL | Manus Preview URL |
|---|---|---|---|
| 1A — CEO/COO Founder Story | CEO, COO, President | `chiri.io/about` | https://chirilanding-3xipdmvm.manus.space/about |
| 1B — CHRO-to-CHRO | CHRO, CPO, VP People | `chiri.io/hr` | https://chirilanding-3xipdmvm.manus.space/hr |
| 2 — Shadow AI / IP Risk | CIO, CISO, VP IT | `chiri.io/cio` | https://chirilanding-3xipdmvm.manus.space/cio |
| 2B — Healthcare Vertical | Healthcare HR, COO, CIO | `chiri.io/healthcare` | https://chirilanding-3xipdmvm.manus.space/healthcare |
| 3 — Integration & Model Flexibility | CTO, VP Engineering | `chiri.io/cio` | https://chirilanding-3xipdmvm.manus.space/cio |
| 4 — Elastic Workforce Vision | CEO, President, COO | `chiri.io/about` | https://chirilanding-3xipdmvm.manus.space/about |

---

## Webmaster Instructions: Connecting chiri.io

This is a **separate domain** from the main Chiri website. All your webmaster needs to do is point the `chiri.io` domain at this Manus-hosted app. There is no code to deploy, no build process, and nothing to touch on the main chiri.ai website.

### Step 1: Publish the app from Manus

1. Open the Manus project for these landing pages.
2. Click the **Publish** button in the top-right of the Management UI.
3. The app goes live immediately at `chirilanding-3xipdmvm.manus.space`.

### Step 2: Connect chiri.io in Manus Settings

1. In the Manus Management UI, go to **Settings > Domains**.
2. Click **Add custom domain** and enter `chiri.io`.
3. Manus will provide you with DNS records to add — typically a **CNAME** or **A record**.

### Step 3: Your webmaster adds the DNS records

Send your webmaster the DNS records from Step 2. They need to:

1. Log into your domain registrar (wherever `chiri.io` is registered — GoDaddy, Namecheap, Cloudflare, etc.).
2. Add the DNS records Manus provides.
3. DNS propagation typically takes 15 minutes to a few hours.

That's it. Once DNS propagates, `chiri.io` will serve these landing pages and all four paths will be live:

- `chiri.io/about`
- `chiri.io/hr`
- `chiri.io/healthcare`
- `chiri.io/cio`

### Important Notes

- **No hidden menu needed.** The pages are already invisible to each other. There are no nav links between them. You send someone the specific URL and that is the only page they see.
- **All CTAs are mailto links.** Every "Talk to Our Team" button opens a pre-populated email to kara@chiri.ai. No form backend or CRM integration is required.
- **This does not affect chiri.ai.** The main Chiri website at chiri.ai is on a completely separate domain and is not touched by any of this.
