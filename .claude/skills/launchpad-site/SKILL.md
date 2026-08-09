---
name: launchpad-site
description: >
  Build a 3-page Conversion Launchpad preview site (Landing, About, Client Success) for a prospect and
  deploy it to sites.optimally.ltd/<slug>. Use when the user wants a Launchpad site, a
  3-page site, a "real website" preview, or "build the launchpad for <company>". Takes EITHER a company
  website URL (scraped for brand and content) OR a written business-context block (prospects with no
  website). This is the LOW-TICKET product preview: everything is visually finished but nothing is
  wired, so there is no working form, no live CTAs and no custom domain until they buy.
  Optimally-internal.
---

# Launchpad Site (3-page preview)

Turns one company URL **or** one business-context block into a deployed 3-page site that looks exactly
like the finished £97/month Conversion Launchpad product, and posts the link to Slack.

## What this is (and is NOT)

This is the **preview of a paid product**, not a funnel demo. Read this before writing a line:

| | Launchpad site (this skill) | VSL demo (`vsl-funnel-demo`) |
|---|---|---|
| Pages | 3: Landing, About, Client Success | 1 long funnel page |
| Whose CTAs | **The client's own** (their phone, their enquiry form) | Optimally's booking link |
| VSL / video | **Never.** No video, no play button, no placeholder | Hero VSL is the centrepiece |
| Forms & links | **Inert.** Rendered fully, wired to nothing | CTAs live, tracked |
| Audience | Low-ticket buyers, sent the link in the DM | Qualified leads, revealed on a call |
| Purpose | Show the finished product so buying is an easy yes | Sell a call |

**Everything visual must look finished. Nothing functional may work.** That contrast is the offer: they
see their real site, and £97/month turns it on (domain, hosting, working lead form, edits).

## Inputs

Exactly one of:
- **`url`** - the company website. Scrape it for brand, offer, services, proof, contact details.
- **`context`** - a written block for prospects with no site, shaped:
  `BUSINESS: ... / OFFER / RESULT: ... / IDEAL CLIENT: ... / EXTRA: ... / BRAND LOOK: <label> (#hex, #hex)`

Optionally: `business_name`, `lead_email`, `lead_name` (the CRM contact this build belongs to).

Never abort because there is no URL. A context block is a complete, valid brief.

## Fixed configuration

| Thing | Value |
|---|---|
| Repo | `ReubenShears/launchpad-sites` (local checkout `D:/Claude Cowork/launchpad-sites`) |
| Live root | `https://sites.optimally.ltd` |
| Client site | `<repo>/<slug>/` → `/<slug>`, `/<slug>/about`, `/<slug>/success` |
| Reference build | `example-halstead/` - copy its structure and CSS system |
| Baserow lead row | `Lead Data` (1000548), field **`Launchpad URL`** |
| Slack | `#5-asset-generation` (`C08UWMXTNGH`) via the Optimally OS bot |

## Steps

### 1. Derive the slug
- **URL mode:** first domain label, lowercased, no TLD (`www.halstead.co.uk` → `halstead`).
- **Context mode:** slugify the business name, dropping `ltd`/`limited`/`the` (`Halstead Digital Ltd` →
  `halstead-digital`).
- **Check `ls` in the repo for a collision first** and append `-2` if taken. Business names collide far
  more often than domains do.

### 2. Gather the brief
**URL mode:** scrape with Firecrawl (`firecrawl_scrape`, formats `["markdown","html"]`). Pull the real
business name, what they sell, who for, services, any proof/testimonials/stats, phone, email, location,
hours, and the brand colours from their CSS/logo. Map the site (`firecrawl_map`) and scrape an About or
Services page too if one exists - the About page needs real material.

**Context mode:** the block IS the brief. Take the palette from the `BRAND LOOK` hex codes. If none are
given, choose a tasteful pair yourself (a deep neutral plus one confident accent) that suits the trade.

Either way you will have gaps. **Fill them with plausible, on-brand copy rather than leaving sections
out** - the whole point is that it looks finished. Never write lorem, never write `[PLACEHOLDER]`, never
invent a specific false claim like a named award or a precise revenue figure for a business you know
nothing about. Generic-but-true framing ("we work across Yorkshire and the North East") beats a
fabricated specific.

### 3. Build the three pages
Copy `example-halstead/` as the structural reference. Each client folder is **self-contained**:

```
<slug>/styles.css          <- brand tokens at the top
<slug>/index.html          <- Landing
<slug>/about/index.html    <- About
<slug>/success/index.html  <- Client Success
```

**Use RELATIVE paths for every internal link and the stylesheet. Never absolute ones, never the slug.**

```html
Root page:      <link rel="stylesheet" href="styles.css">   <a href="about/">About</a>
Sub-pages:      <link rel="stylesheet" href="../styles.css"> <a href="../">Home</a> <a href="../success/">Client Success</a>
```

Two rules make this safe and portable:
1. **Always write internal links WITH a trailing slash** (`about/`, `../success/`), matching the repo's
   `vercel.json` `trailingSlash: true`. Without that config, slashless URLs make browsers resolve
   `../styles.css` to the domain root and every sub-page renders as an unstyled white page - never
   remove that vercel.json.
2. **Never hardcode the slug or the domain in a path.** The folder must be fully portable: when a
   client BUYS, their folder is lifted unchanged into its own Vercel project (so their custom domain
   can be attached), and any `/<slug>/...` path would break at that moment.

**Re-skin by editing the three tokens at the top of `styles.css`** (`--brand`, `--accent`,
`--accent-ink`). `--accent-ink` must be legible ON the accent - dark ink on a light/warm accent, white
on a deep one. Check it; a pale accent with white text is unreadable and is the single most common way
these builds look cheap.

**Landing** - header (wordmark, nav, phone, CTA), hero (headline, sub, two CTAs, assurances, proof
panel), trust strip, problem (3), services (3), process (3 steps), featured testimonial, stat band, FAQ
(5), contact section with full form, footer.

**About** - hero, story (2-3 paragraphs of real narrative), founder card with a quote, three "how we
work" principles, stat band, coverage area, CTA band, footer.

**Client Success** - hero, stat band, three case studies (situation → what was done → three outcome
bullets), four-quote testimonial wall, CTA band, footer.

**Social share (OG) tags - every page's `<head>`, right after the description meta.** These make the
link unfurl properly when forwarded (WhatsApp, iMessage, Facebook). The image is the shared branded
card at the repo root; it is the ONE permitted absolute URL because OG images must be absolute:

```html
<meta property="og:title" content="{Business Name} | Website Built For {Lead First Name}">
<meta property="og:description" content="Your new website is ready to view. Take a look.">
<meta property="og:type" content="website">
<meta property="og:image" content="https://sites.optimally.ltd/og.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta name="twitter:card" content="summary_large_image">
```

`{Lead First Name}` is the first word of the lead name you were given, tidied to Title Case (e.g. lead
name "jay smith" gives `Unorthodox Digital | Website Built For Jay`). If no lead name was supplied (a
manual one-off build), use `{Business Name} | Website Built For You`.

**Hard rules for every page:**
- **No video, no VSL, no play button, anywhere.** Especially not in the hero.
- **All CTAs inert.** Buttons stay `<a href="#contact">` or in-site links; the form is
  `<form onsubmit="return false;">`; phone and email are plain text, never `tel:` or `mailto:`.
- **No external images or fonts.** Every visual is CSS (gradient panels, initial avatars, glyph icons).
  A client's scraped image can vanish and break the page; a gradient cannot. System font stack only.
- Footer carries `Site made by <a href="https://optimally.ltd">Optimally</a>` on all three pages.
- **No em dashes anywhere**, including in the CSS comments. Use commas, colons or full stops.
- British or American spelling: match the client's own.

### 4. Deploy (git push)
```bash
git -C "D:/Claude Cowork/launchpad-sites" add <slug>
git -C "D:/Claude Cowork/launchpad-sites" -c user.email="132842611+ReubenShears@users.noreply.github.com" -c user.name="ReubenShears" commit -m "launchpad: <slug>"
git -C "D:/Claude Cowork/launchpad-sites" push origin main
```
Remote/headless: if already in the checkout, just add/commit/push. Prefer `GITHUB_TOKEN` if provided:
`git remote set-url origin "https://x-access-token:${GITHUB_TOKEN}@github.com/ReubenShears/launchpad-sites.git"`.
Commit author email MUST be `132842611+ReubenShears@users.noreply.github.com` or the build is blocked.
Push straight to `main`. Then poll all three pages until each returns 200 (allow a couple of minutes):
```bash
curl -sS -o /dev/null -w "%{http_code}" https://sites.optimally.ltd/<slug>
curl -sS -o /dev/null -w "%{http_code}" https://sites.optimally.ltd/<slug>/about
curl -sS -o /dev/null -w "%{http_code}" https://sites.optimally.ltd/<slug>/success
```
The old `launchpad-sites-one.vercel.app` URL still serves as a fallback alias, but never hand it out: Slack, Baserow, GHL and build-complete all get the `sites.optimally.ltd` form.
If a push lands but the alias still 404s after ~3 minutes, the production alias has not rolled over.
Do not rebuild: report it in Slack and stop.

### 5. Write the URL back to the lead (when a lead email was supplied)
Funnel-triggered runs always supply one, and this write is **mandatory** - the link is the deliverable.
- **Baserow:** `Lead Data` (1000548), find the lead by that email (`list_table_rows` `search=<email>`,
  most recent match) and `update_rows` setting **`Launchpad URL`** to the live URL.
- **GHL:** find the contact by that email (`contacts_get-contacts` `query=<email>`) and write the same
  URL into the **Demo Landing Page URL** custom field, **field ID `6dtdKnKMkB659ZVlsRof`** (the field
  *key* silently fails to persist). Never create a contact.
- If either write fails, say so loudly in the Slack post with the lead email and the live URL so it can
  be attached by hand. Never fail silently.

Do **not** call the ManyChat API directly from this skill. Instead, **once the three pages are live and
returning 200, call the build-complete webhook** - this is what actually gets the site sent to them:

```bash
curl -sS -G "https://optimally.app.n8n.cloud/webhook/build-complete" \
  --data-urlencode "lead_email=<the lead email you were given>" \
  --data-urlencode "url=https://sites.optimally.ltd/<slug>" \
  --data-urlencode "kind=launchpad"
```

n8n then saves the URL to the lead row, writes it into the ManyChat **Demo Website URL** field and
applies the **Landing Page Ready** tag, which is what triggers the DM. Skipping this call means the
site is built but nobody ever receives it. Only skip it when no lead email was supplied (a manual
one-off build). If it returns anything other than 200, say so in the Slack post.

### 6. Post to Slack
Channel `C08UWMXTNGH`, Optimally OS bot, mrkdwn (`*bold*`, `<url|label>`, `>` quote groups, no em dashes):
```
:rocket:  *Launchpad site ready:  {{Company}}*

>  :globe_with_meridians:  <https://sites.optimally.ltd/{{slug}}|View the site>
>  :page_facing_up:  Landing  ·  About  ·  Client Success
>  :art:  Brand: {{brandHex}} / {{accentHex}}
>  :bust_in_silhouette:  Lead: {{leadName}} ({{leadEmail}})
>  :white_check_mark:  Saved to Baserow and CRM

---
```
If step 5 could not attach the URL, replace that last line with:
```
>  :rotating_light:  *NOT ATTACHED* - save `{{liveUrl}}` to `{{leadEmail}}` by hand
```

## Failure handling
- **No url and no context:** only then report a missing input. Check for a context block before failing.
- **Scrape thin or blocked:** fall back to the homepage scrape, then to writing from whatever you have.
  Still produce all three complete pages.
- **Slug collision:** append `-2`, never overwrite another client's folder.
- **Baserow `update_rows` rejects the table id as a string:** reload that tool with a KEYWORD ToolSearch
  (not `select:`), then retry.

## Notes on scope / side effects
Pushes a real commit and deploys three real pages on every run. One brief in, one client site out.

**This site is sent to the prospect** (unlike the VSL demo, which is revealed on a call), so it must be
genuinely presentable. But it is a preview: no domain, no working form, nothing that could take a real
enquiry and drop it. Buying is what turns those on.
