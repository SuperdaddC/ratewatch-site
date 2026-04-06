# RateWatch Project Context

> **Purpose of this file:** This document exists so that a new Claude session can pick up exactly where we left off. It covers architecture, services, decisions made, and the full history of work done across all chat sessions.

---

## Project Overview

**RateWatch** is a single-page mortgage refinance savings calculator and lead-capture tool for **Michael Colyer** (NMLS #276626, DRE #01842442), Broker Associate at **Five M Realty Group, Inc.** (DRE #02253302), based in Carlsbad, CA.

- **Live URL:** https://ratewatch.thecolyerteam.com
- **Repo:** https://github.com/SuperdaddC/ratewatch-site (branch: `main`)
- **Local path:** `C:\Users\ColyerTeam\Developer\ratewatch-site`
- **Owner:** Mike Colyer (mike@thecolyerteam.com / mjcolyer@gmail.com)

---

## Architecture

### File Structure
```
ratewatch-site/
  index.html       # The entire app — HTML, CSS, and JS all inline (~850 lines)
  netlify.toml     # Build config + security headers (CSP, X-Frame-Options, etc.)
  .gitignore       # Ignores .netlify/
```

This is a **zero-build static site**. No bundler, no framework, no npm. Just one HTML file deployed directly to Netlify.

### Hosting & Deployment
- **Netlify** — auto-deploys from `main` branch on push
- Custom domain: `ratewatch.thecolyerteam.com` (DNS configured in Netlify)
- Build command: none (publish directory is `.`)

### Cloud Services

#### Supabase (apuctuqlmykeemtcasji.supabase.co)
- **Anon key** is in `index.html:455` (client-side, expected for anon role)
- **Two tables used:**
  1. `mortgage_rates_daily` — READ only. Queried on page load to get the latest 30yr fixed rate.
     - Query: `?select=product,rate&product=eq.30yr_fixed&order=rate_date.desc&limit=1`
     - This table is populated externally (not by this site) — likely by a separate rate-fetching job
  2. `rate_watch_leads` — WRITE only. The signup form POSTs lead data here.
     - Fields written: `first_name`, `last_name`, `email`, `phone`, `current_rate`, `target_rate`, `current_balance`, `occupancy_type`, `property_address`, `estimated_value`, `origination_year`, `remaining_term_months`, `original_loan_balance`, `years_to_retirement`, `calculated_savings_monthly`, `source` (always "website")
- **Storage bucket** `profile-photos` — hosts Mike's headshot used in the trust section and OG image
- **RLS should be enabled** on both tables (verify in Supabase dashboard — we flagged this but didn't confirm)

#### Plausible Analytics (added 2026-04-05)
- Script tag is in the `<head>`: `<script defer data-domain="ratewatch.thecolyerteam.com" src="https://plausible.io/js/script.js"></script>`
- **NOT YET ACTIVATED** — Mike needs to create a Plausible account (plausible.io, $9/mo) and add the domain. Until then, the script is harmless.
- Alternatives discussed: Google Analytics (free), Cloudflare Web Analytics (free, cookie-less)

#### Other External Links
- **Schedule a Call:** https://outlook.office365.com/owa/calendar/MeetingWithMike@NETORG19778106.onmicrosoft.com/bookings/
- **Apply Online:** https://michaelcolyer.zipforhome.com
- **NMLS Consumer Access:** https://www.nmlsconsumeraccess.org/
- **Contact email shown on site:** judy@vip.thecolyerteam.com (this is the Judy AI assistant email, used across Mike's projects)

---

## How the App Works

### Calculator Flow
1. Page loads with **pre-filled default values**: rate 6.875%, year 2020, balance $500k, value $650k
2. `rate30yr` defaults to 6.5% so results render immediately
3. `loadRates()` fires async — fetches live 30yr fixed rate from Supabase `mortgage_rates_daily`, updates the slider and recalculates
4. User adjusts any input or the rate slider -> `recalc()` fires
5. Three result cards show:
   - **Monthly Savings** (green) — old payment minus new payment
   - **Payoff Timeline** (blue) — if savings reinvested as extra principal
   - **401(k) Growth** (purple) — if savings invested at 7% annual return
6. All cards also show "Balance at retirement" based on the retirement years input

### Loan Term Support
- Loan term selector (15/20/30yr) was added 2026-04-05. The JS uses `calcTerm` value instead of hardcoded 30 for:
  - Remaining months calculation
  - Balance amortization
  - Term display text

### Signup / Lead Capture Flow
1. User fills out name, email, optional phone, property address, target rate (pre-filled from slider)
2. **Honeypot field** (`#website`) is hidden off-screen — if filled, the form fakes success to fool bots
3. Validation: email format, phone 10-11 digits if provided, address 8+ chars if provided, names 2+ chars
4. On submit: spinner shows, data POSTs to Supabase `rate_watch_leads`
5. Success: form hides, confetti animation, success message with email and target rate
6. Failure: inline red error banner (not an alert()) with fallback phone number

---

## Compliance (California Lending)

The site has full CA lending compliance built in across multiple commits:
- **NMLS #276626** and **DRE #01842442** displayed in trust section and footer
- **Five M Realty Group, Inc. DRE #02253302** in footer
- **Equal Housing Lender** badge in footer
- **TILA/Reg Z APR disclosure** directly below rate slider
- **Privacy Policy modal** — CCPA/CPRA rights, data collection disclosure
- **Do Not Sell or Share modal** — CCPA opt-out instructions
- **Terms of Service modal** — "informational purposes only" disclaimer
- **Accessibility Statement modal** — WCAG 2.1 Level AA commitment
- **NMLS Consumer Access** link in footer
- Rate disclaimer: "market averages for informational purposes only, do not constitute a commitment to lend"

---

## Security Headers (netlify.toml)

```
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' https://plausible.io; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; img-src 'self' https://apuctuqlmykeemtcasji.supabase.co data:; connect-src 'self' https://apuctuqlmykeemtcasji.supabase.co https://plausible.io
```

---

## Git History (oldest to newest)

| Commit | Description |
|--------|-------------|
| `4ca1f49` | Initial Rate Watch landing page |
| `1b4daeb` | Redesign: savings calculator with slider, 3 result boxes, remove rates section |
| `5be1111` | Redesign: savings calculator with slider, smart-fill inputs, 3 result cards |
| `362dcad` | Add balance at retirement to all 3 cards, retirement years input |
| `addf96f` | Dramatically compact hero + tighten calculator to fit above the fold on 1080p |
| `63e4eec` | Add property address + estimated value fields, capture all data to Supabase |
| `2afcbbb` | Move Property Address from calculator to signup form |
| `a330560` | Add bot protection: honeypot field, input validation, update URLs |
| `835ae8f` | Update address to 2214 Faraday Ave, Carlsbad, CA 92008 |
| `02db30d` | Update Apply Online URL to apply.thecolyerteam.com |
| `3756a5e` | Update apply link to michaelcolyer.zipforhome.com |
| `411126e` | Add compliance footer: privacy policy, contact info, NMLS, ADA, APR disclosure |
| `01f7056` | Add TILA/Reg Z APR disclosure directly below rate slider |
| `cade8df` | Remove incorrect DRE #01526140 from footer |
| `4042e9f` | Full CA lending compliance: responsible broker, EHL, NMLS, privacy, TOS, accessibility |
| `7a19838` | Improve UX, SEO, security, and trust signals (latest, 2026-04-05) |

---

## Work Done in Chat Session (2026-04-04 to 2026-04-06)

### Session Goals
Mike asked for a site review with improvement suggestions, then said "get'er done" to implement all of them.

### Changes Implemented (commit `7a19838`)
1. **Pre-filled calculator inputs** — rate 6.875%, year 2020, balance $500k, value $650k. Set `rate30yr = 6.5` as default so results render before Supabase responds. Added `updateSliderDisplay(); recalc();` before `loadRates()`.
2. **Loan term selector** — `<select id="calcTerm">` with 15/20/30yr options. Updated `getLoanState()`, `updateTermCalc()`, and all amortization math to use `loanTerm` instead of hardcoded 30. Added to input listener array.
3. **SVG favicon** — inline data URI, blue "R" on rounded rect, no external file needed.
4. **Twitter card meta** — `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`.
5. **Robots meta** — `<meta name="robots" content="index, follow">`.
6. **Plausible analytics** — `<script defer>` tag (needs account activation).
7. **Rate loading indicator** — slider label shows pulsing "Loading today's rate..." during fetch, restores on success/failure.
8. **Form submission spinner** — CSS `.spinner` class, `btn.innerHTML` swap during submit.
9. **Inline error banner** — red `#formError` div inserted after button on failure (replaces `alert()`), includes fallback phone number.
10. **CSP header** — added to `netlify.toml`, locked to self + Supabase + Plausible + Google Fonts.
11. **Mobile toggle styling** — smaller font/padding on `.toggle-tabs button` at `max-width:600px`.
12. **Trust value props** — three items below Mike's name: Fast Closings (21-day avg), Low-Cost Refis, 5-Star Reviews.

### Review Findings NOT Yet Implemented
- **Supabase RLS audit** — verify Row Level Security is enabled on `mortgage_rates_daily` and `rate_watch_leads`. The anon key is exposed client-side (expected) but RLS must be tight.
- **sitemap.xml / robots.txt** — not created yet.
- **Google Fonts `font-display: swap`** — not yet optimized (using Google's CDN default).
- **Testimonials / review count** — trust section has generic "5-Star Reviews" but no linked proof.
- **"The Colyer Team" text** in trust section is not linked to anything.

---

## Related Mike Colyer Projects (for context)

Mike is also building:
- **ComplyWithJudy** (`C:\Users\ColyerTeam\Developer\ca-compliance-platform`) — CA real estate/lending compliance scanner SaaS. FastAPI + Playwright on Mac Mini, Supabase backend, Netlify frontend, Stripe billing. Same Supabase project (`apuctuqlmykeemtcasji`).
- **OneMLS** (`SuperdaddC/oneMLS`) — National independent MLS platform with Larry, anti-NAR free market approach, launching CA/CO/FL.

These share the same Supabase instance. The `mortgage_rates_daily` table is likely populated by a job in the compliance platform or a standalone script.

---

## Tools & Services Summary

| Tool/Service | Purpose | Access |
|---|---|---|
| GitHub | Repo hosting | github.com/SuperdaddC/ratewatch-site |
| Netlify | Static hosting, auto-deploy from main | Custom domain configured |
| Supabase | Database (rates + leads), image storage | apuctuqlmykeemtcasji.supabase.co |
| Google Fonts | Inter typeface | CDN link in `<head>` |
| Plausible | Analytics (not yet activated) | plausible.io (needs account) |
| Outlook Bookings | Schedule a Call link | outlook.office365.com |
| ZipForHome | Apply Online link | michaelcolyer.zipforhome.com |

---

## How to Pick Up

1. `cd C:\Users\ColyerTeam\Developer\ratewatch-site`
2. `git pull origin main` to get latest
3. Edit `index.html` — it's the entire app
4. `git push origin main` to deploy (Netlify auto-builds)
5. Check https://ratewatch.thecolyerteam.com after ~30 seconds

No build step. No dependencies. No env files. Just HTML.
