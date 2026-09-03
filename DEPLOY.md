# Naik Labs — Project Deployment Playbook

Reusable instruction set for taking any Naik Labs project from localhost to a live subdomain on `naiklabs.dev`. Built on [jhammant/ship-what-you-built](https://github.com/jhammant/ship-what-you-built).

---

## Prerequisites (one-time setup)

### 1. Install the ship-what-you-built skill

```bash
git clone https://github.com/jhammant/ship-what-you-built.git /tmp/ship-what-you-built
mkdir -p ~/.claude/skills/first-site
cp -r /tmp/ship-what-you-built/skill/scripts ~/.claude/skills/first-site/
cp /tmp/ship-what-you-built/skill/SKILL.md ~/.claude/skills/first-site/
rm -rf /tmp/ship-what-you-built
```

This gives you five scripts (all take `--help`, all safe to run twice):

| Script | What it does |
|---|---|
| `detect.sh` | Identifies project type, framework, build command, output directory |
| `preflight.sh` | Checks for secrets, credentials, large files before publishing |
| `deploy.sh` | Deploys to Cloudflare Pages (or AWS). Remembers config after first run |
| `status.sh` | Walks DNS → server → certificate → OG image for a live domain |
| `og-image.sh` | Generates a 1200x630 OG preview image via headless Chrome |

### 2. Install Wrangler (Cloudflare CLI)

```bash
npm install -g wrangler
wrangler login
```

### 3. Cloudflare account

- Sign up at [dash.cloudflare.com](https://dash.cloudflare.com/sign-up) (free, no card)
- Enable 2FA (My Profile → Authentication)
- Add `naiklabs.dev` domain (if not already added)
- Add per-subdomain DNS records as each project goes live

### 4. Resend account (if any project sends email)

- Sign up at [resend.com](https://resend.com) — free tier: 100 emails/day, 3,000/month
- Add domain: Resend dashboard → Domains → `naiklabs.dev`
- Add the 3 DNS records Resend gives you (SPF, DKIM, DMARC) in Cloudflare DNS
- Verify the domain in Resend
- Create an API key scoped to `naiklabs.dev` sending

Once verified, any `<project>@naiklabs.dev` address works as a sender.

---

## Per-project deployment

Run these steps for each project. Replace the placeholders:

| Placeholder | Example |
|---|---|
| `PROJECT_NAME` | Inaugural Parkrun Scanner |
| `PROJECT_SLUG` | inaugural-parkrun |
| `PROJECT_SUBDOMAIN` | inauguralparkrun |
| `PROJECT_TAGLINE` | Find parkruns running their first-ever event near you |
| `PROJECT_ACCENT` | `#b5432a` (Naik Labs red) or project-specific colour |

---

### Step 1: Create the project directory

```bash
mkdir -p ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG
cd ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG
```

Each project is a standalone static site: one `index.html`, one OG image, deployed as its own Cloudflare Pages project.

---

### Step 2: Build the project page

Create `index.html` in the project directory. Every project page follows this structure:

```
┌─────────────────────────────────┐
│  Title                          │
│  Tagline                        │
│─────────────────────────────────│
│  What is this?                  │
│  [1-2 paragraph explanation]    │
│─────────────────────────────────│
│  How it works                   │
│  [Numbered steps / pipeline]    │
│─────────────────────────────────│
│  [Screenshots / demo if any]    │
│─────────────────────────────────│
│  Status / tech tags / links     │
│  Footer: naiklabs.dev           │
└─────────────────────────────────┘
```

#### HTML template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>PROJECT_NAME — by NaikLabs</title>

  <!-- OG tags — update after generating preview.png -->
  <meta property="og:title"       content="PROJECT_NAME">
  <meta property="og:description" content="PROJECT_TAGLINE">
  <meta property="og:image"       content="https://PROJECT_SUBDOMAIN.naiklabs.dev/preview.png">
  <meta property="og:url"         content="https://PROJECT_SUBDOMAIN.naiklabs.dev">
  <meta property="og:type"        content="website">
  <meta name="twitter:card"       content="summary_large_image">

  <style>
    *, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

    :root {
      --bg: #fafafa;
      --text: #1a1a1a;
      --text-muted: #6b7280;
      --accent: PROJECT_ACCENT;
      --accent-light: PROJECT_ACCENT_LIGHT;
      --border: #e5e7eb;
      --max-w: 720px;
      --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", Inter, Helvetica, Arial, sans-serif;
      --font-mono: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
    }

    body { background: var(--bg); color: var(--text); font-family: var(--font-sans); font-size: 16px; line-height: 1.6; }
    .container { max-width: var(--max-w); margin: 0 auto; padding: 48px 24px 64px; }
    .header { text-align: center; padding-bottom: 32px; border-bottom: 2px solid var(--accent); }
    .header h1 { font-size: clamp(28px, 4vw, 40px); font-weight: 700; color: var(--accent); line-height: 1.2; }
    .header p { margin-top: 12px; color: var(--text-muted); }
    section { margin-top: 40px; }
    section h2 { font-size: 20px; font-weight: 700; margin-bottom: 12px; }
    section p { line-height: 1.7; }
    .steps { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 16px; margin-top: 16px; }
    .step { background: var(--accent-light); border-radius: 12px; padding: 20px; text-align: center; }
    .step-num { display: inline-flex; align-items: center; justify-content: center; width: 32px; height: 32px; border-radius: 50%; background: var(--accent); color: white; font-weight: 700; font-size: 14px; margin-bottom: 8px; }
    .step h3 { font-size: 15px; font-weight: 600; margin-bottom: 4px; }
    .step p { font-size: 13px; color: var(--text-muted); }
    .tags { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 16px; }
    .tag { font-family: var(--font-mono); font-size: 12px; padding: 4px 12px; border: 1px solid var(--border); border-radius: 4px; color: var(--text-muted); }
    footer { margin-top: 64px; padding-top: 20px; border-top: 1px solid var(--border); text-align: center; font-size: 13px; color: var(--text-muted); }
    footer a { color: var(--text-muted); text-decoration: underline; text-underline-offset: 3px; }
    .powered { font-size: 10px; font-weight: 400; color: var(--text-muted); margin-left: 6px; }
    @media (max-width: 480px) { .steps { grid-template-columns: 1fr; } .container { padding: 32px 16px 48px; } }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>PROJECT_NAME<span class="powered">by NaikLabs</span></h1>
      <p>PROJECT_TAGLINE</p>
    </div>
    <section>
      <h2>What is this?</h2>
      <p>WHAT_IT_DOES — 1-2 paragraphs explaining the project.</p>
    </section>
    <section>
      <h2>How it works</h2>
      <div class="steps">
        <div class="step"><div class="step-num">1</div><h3>Step name</h3><p>Step description</p></div>
        <div class="step"><div class="step-num">2</div><h3>Step name</h3><p>Step description</p></div>
        <div class="step"><div class="step-num">3</div><h3>Step name</h3><p>Step description</p></div>
      </div>
    </section>
    <!-- Optional: screenshots, subscribe form, status info -->
    <div class="tags">
      <span class="tag">TECH_1</span>
      <span class="tag">TECH_2</span>
    </div>
    <footer><p>Powered by <a href="https://naiklabs.dev">NaikLabs</a></p></footer>
  </div>
</body>
</html>
```

**Design rules:** system font stacks only, no webfonts, no external CSS, self-contained single file. Each project can have its own accent colour. The parkrun page is the reference.

---

### Step 3: Preview locally

```bash
cd ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG
python3 -m http.server 8765
# Open http://localhost:8765
```

Check: renders correctly, responsive at 375px, no external resource loads.

---

### Step 4: Generate OG image

Option A — ship-what-you-built:

```bash
"$HOME/.claude/skills/first-site/scripts/og-image.sh" \
  --title "PROJECT_NAME" \
  --subtitle "PROJECT_TAGLINE" \
  --domain PROJECT_SUBDOMAIN.naiklabs.dev \
  --accent "PROJECT_ACCENT" \
  --out ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG/preview.png
```

Option B — Remotion still (if the launch-videos project is set up):

```bash
cd ~/Documents/GitHub/naiklabs-launch-videos
npx remotion still PROJECT_NAMECard out/PROJECT_SLUG-preview.png
cp out/PROJECT_SLUG-preview.png ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG/preview.png
```

**OG image rules:** 1200x630px exactly, PNG or JPG, under 5 MB, publicly reachable absolute URL. `og:image` must point to `https://PROJECT_SUBDOMAIN.naiklabs.dev/preview.png`.

---

### Step 5: Run preflight checks

```bash
cd ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG
"$HOME/.claude/skills/first-site/scripts/preflight.sh"
```

Checks for: tracked secrets (`.env`/`.pem`/`.key`), credential-shaped strings, `.env` in git history, missing `.gitignore`, large files, email exposure.

**Do not deploy if preflight fails.**

---

### Step 6: Deploy to Cloudflare Pages

First deploy:

```bash
cd ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG

npx --yes wrangler@latest pages project create PROJECT_SUBDOMAIN-naiklabs \
  --production-branch main

npx --yes wrangler@latest pages deploy . \
  --project-name PROJECT_SUBDOMAIN-naiklabs \
  --commit-dirty=true
```

Or use ship-what-you-built (remembers config after first run):

```bash
"$HOME/.claude/skills/first-site/scripts/deploy.sh" \
  --host cloudflare --project PROJECT_SUBDOMAIN-naiklabs \
  --dir . --domain PROJECT_SUBDOMAIN.naiklabs.dev

# Subsequent deploys:
"$HOME/.claude/skills/first-site/scripts/deploy.sh"
```

---

### Step 7: Set up custom subdomain

Cloudflare dashboard → Workers & Pages → `PROJECT_SUBDOMAIN-naiklabs` → Custom domains → Add `PROJECT_SUBDOMAIN.naiklabs.dev`. Auto-creates DNS and SSL certificate. Wait 1-2 minutes.

---

### Step 8: Validate the live site

```bash
"$HOME/.claude/skills/first-site/scripts/status.sh" PROJECT_SUBDOMAIN.naiklabs.dev
```

Walks: DNS → server → certificate → OG image.

Then validate OG previews before sharing:

| Platform | Validator |
|---|---|
| LinkedIn | [linkedin.com/post-inspector](https://www.linkedin.com/post-inspector/) |
| Facebook | [developers.facebook.com/tools/debug](https://developers.facebook.com/tools/debug/) |
| X | Post in a draft and check |

Platforms cache the first preview they see — always validate first.

---

### Step 9: Verify NaikLabs branding

Every page deployed under `naiklabs.dev` must carry **NaikLabs** branding in three places:

| Location | What to check | Example |
|---|---|---|
| **Page title** | `<title>` ends with `— by NaikLabs` | `<title>Meal Planner — by NaikLabs</title>` |
| **Header / logo** | Small "by NaikLabs" label next to the project name | `<span class="powered">by NaikLabs</span>` |
| **Footer** | "Powered by NaikLabs" with a link to naiklabs.dev | `Powered by <a href="https://naiklabs.dev">NaikLabs</a>` |

```bash
PAGE=$(curl -s https://PROJECT_SUBDOMAIN.naiklabs.dev)

echo "$PAGE" | grep -qi "<title>.*by NaikLabs" \
  && echo "✓ Title branding" \
  || echo "✗ MISSING — <title> must end with '— by NaikLabs'"

echo "$PAGE" | grep -qi 'class="powered".*by NaikLabs' \
  && echo "✓ Header branding" \
  || echo "✗ MISSING — header needs <span class=\"powered\">by NaikLabs</span>"

echo "$PAGE" | grep -qi "powered by.*naiklabs" \
  && echo "✓ Footer branding" \
  || echo "✗ MISSING — footer needs 'Powered by NaikLabs'"
```

**Do not deploy if any branding check fails.**

---

### Step 10: Update the portfolio

The holding page at `~/Documents/GitHub/naiklabs/index.html` needs two additions: a **card** in the projects grid and a **detail page** section. Then redeploy.

#### 10a. Add the project card

Find the `<div class="projects-grid">` block. Add a new card at the end (before the closing `</div>`). Update the card number (`06`, `07`, etc.), the `onclick` target, hero style, status, and tags:

```html
<div class="project-card" onclick="showProject('PROJECT_SLUG')">
  <div class="project-card-hero muted">
    <!-- Hero styles: "dark" (black), "accent" (red), "muted" (grey) -->
    <span class="card-num">06</span>
    <span class="card-title">PROJECT_NAME</span>
  </div>
  <div class="project-meta">
    <span class="project-name">PROJECT_NAME</span>
    <span class="status">
      <!-- Status dots: "shipped", "in-progress", "archived" -->
      <span class="status-dot shipped"></span>Shipped
    </span>
  </div>
  <p class="project-desc">PROJECT_TAGLINE</p>
  <div class="tags">
    <span class="tag">TECH_1</span>
    <span class="tag">TECH_2</span>
  </div>
</div>
```

Also update the `<span class="section-count">` text (e.g. "5 entries" → "6 entries").

#### 10b. Add the detail page

Add a new detail section **after** the last `<!-- PROJECT DETAIL -->` block and **before** the `<!-- RESUME PAGE -->` comment:

```html
<!-- PROJECT DETAIL: PROJECT_NAME -->
<div id="page-PROJECT_SLUG" class="page">
  <a class="back-link" onclick="showPage('projects')">&larr;&nbsp; All Projects</a>
  <div class="detail-hero muted">
    <!-- Match the hero style from the card above -->
    <span class="card-num">06</span>
    <span class="card-title">PROJECT_NAME</span>
  </div>
  <div class="detail-status">
    <span class="status"><span class="status-dot shipped"></span>Shipped</span>
  </div>
  <h2 class="detail-title">PROJECT_NAME</h2>
  <p class="detail-subtitle">PROJECT_TAGLINE</p>
  <div class="detail-section-label">What it does</div>
  <p class="detail-body">WHAT_IT_DOES — 1-2 paragraphs.</p>
  <div class="detail-section-label">Why I built it</div>
  <p class="detail-body">WHY_I_BUILT_IT — 1-2 paragraphs.</p>
  <div class="detail-section-label screens-header">
    <span>Screens</span><span class="section-count">N views</span>
  </div>
  <div class="screens-grid">
    <div><div class="screen-slot">Image Slot</div><p class="screen-label">View 1</p></div>
    <div><div class="screen-slot">Image Slot</div><p class="screen-label">View 2</p></div>
  </div>
  <div class="info-table">
    <div class="info-row"><span class="info-label">Status</span><span class="info-value">Shipped</span></div>
    <div class="info-row"><span class="info-label">Year</span><span class="info-value">2026</span></div>
    <div class="info-row"><span class="info-label">Visibility</span>
      <span class="info-value">Public</span>
      <!-- Use "Private" if no public site -->
    </div>
    <div class="info-row"><span class="info-label">Source</span>
      <span class="info-value">
        <!-- Public repo: -->
        <a href="https://github.com/ADAS-Ash/PROJECT_SLUG" target="_blank" rel="noopener">github.com/ADAS-Ash/PROJECT_SLUG</a>
        <!-- Private repo: just text "Private repository" -->
      </span>
    </div>
  </div>
  <div class="tags" style="margin-top:20px">
    <span class="tag">TECH_1</span>
    <span class="tag">TECH_2</span>
  </div>
  <!-- Public project with a live site: -->
  <a class="cta-btn" href="https://PROJECT_SUBDOMAIN.naiklabs.dev" target="_blank" rel="noopener">Visit PROJECT_NAME</a>
  <!-- Private project: -->
  <!-- <div class="cta-btn private">Private Repository</div> -->
</div>
```

#### 10c. Redeploy the main site

```bash
cd ~/Documents/GitHub/naiklabs
npx --yes wrangler@latest pages deploy . --project-name naiklabs --commit-dirty=true
```

#### Checklist

- [ ] Card number is sequential and unique
- [ ] `onclick` id matches the detail section's `id="page-..."` (both use `PROJECT_SLUG`)
- [ ] Section count updated ("N entries")
- [ ] Detail section placed before `<!-- RESUME PAGE -->`
- [ ] CTA button points to the live subdomain (or shows "Private Repository")
- [ ] Main site redeployed

---

### Step 11: Subsequent deploys

```bash
cd ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG
"$HOME/.claude/skills/first-site/scripts/preflight.sh"
npx --yes wrangler@latest pages deploy . --project-name PROJECT_SUBDOMAIN-naiklabs --commit-dirty=true
"$HOME/.claude/skills/first-site/scripts/status.sh" PROJECT_SUBDOMAIN.naiklabs.dev
```

---

## Project registry

| Project | Subdomain | CF Project Name | Status |
|---|---|---|---|
| Inaugural Parkrun | `inauguralparkrun.naiklabs.dev` | `inauguralparkrun-naiklabs` | Live |
| Mindle | `mindle.naiklabs.dev` | `mindle-naiklabs` | Not started |
| Puffer Properties | `pufferproperties.co.uk` | *(separate)* | Live |
| Meal Planner | `mealplanner.naiklabs.dev` | `mealplanner-naiklabs` | Not started |
| GPX Exporter | `gpxexporter.naiklabs.dev` | `gpxexporter-naiklabs` | Live |
| Crossword Generator | — | Docker self-hosted | Live (Docker) |
| HA House Bible | — | — | Private, no public page |

---

## Cloudflare Pages Functions (backends)

For projects that need server-side logic (API proxies, form handlers, notifications), add a `functions/` directory at the **project root**. File paths become API routes:

```
functions/api/hello.js   →   https://PROJECT_SUBDOMAIN.naiklabs.dev/api/hello
functions/api/save.js    →   https://PROJECT_SUBDOMAIN.naiklabs.dev/api/save
```

`onRequestPost` = POST, `onRequestGet` = GET, `onRequest` = all methods.

### Environment variables / secrets

Pages project → Settings → Variables and Secrets → Add → Production → Encrypt. Encrypted values cannot be read back. **Redeploy after adding** — existing deployments don't pick them up.

### Data storage

| Need | Service | Free tier |
|---|---|---|
| Key-value | Workers KV | Generous |
| SQL database | D1 (SQLite) | Generous |
| File/image uploads | R2 (S3-compatible, no egress) | 10 GB |
| Auth, Postgres, realtime | Supabase (external) | Generous |

Add under Settings → Bindings → Add. Available on `context.env`.

### Common Cloudflare issues

| Symptom | Fix |
|---|---|
| Build succeeds, site blank | Wrong output directory |
| Build fails, works locally | Set `NODE_VERSION` env var to match your `node -v` |
| Custom domain stuck pending | Domain not in this Cloudflare account, or nameservers not switched — `dig NS yourdomain.com` |
| Function returns 500 | Missing environment variable — check Functions real-time logs |
| `/api/...` returns HTML | `functions/` must be at repo root, not inside `src/` |
| Env var change no effect | Redeploy — variables apply at build time |
| Old version showing | Browser cache — Cmd+Shift+R |

---

## Optional: Email notifications (Resend)

For projects that email users (weekly digests, subscribe confirmations, alerts). Each project sends from its own address: `<project>@naiklabs.dev`.

### Per-project setup

Add the Resend API key as a Wrangler secret:

```bash
cd your-project
wrangler secret put RESEND_API_KEY
```

### Email module (copy per project, change FROM)

```typescript
// src/email.ts
const FROM = "Project Name <projectname@naiklabs.dev>";

export async function sendEmail(
  apiKey: string, to: string, subject: string, html: string
): Promise<void> {
  const res = await fetch("https://api.resend.com/emails", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${apiKey}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ from: FROM, to: [to], subject, html }),
  });
  if (!res.ok) {
    const body = await res.text();
    throw new Error(`Resend API error ${res.status}: ${body}`);
  }
}
```

### Calling from your Worker

```typescript
import { sendEmail } from "./email";

if (env.RESEND_API_KEY) {
  await sendEmail(env.RESEND_API_KEY, "user@example.com", "Subject", "<p>Body</p>");
}
```

Guard with `if (env.RESEND_API_KEY)` so the worker doesn't crash in local dev.

### Sending on a schedule (cron)

In `wrangler.toml`:

```toml
[triggers]
crons = ["0 18 * * 5"]  # every Friday at 6pm UTC
```

### Email template pattern

Keep emails simple — inline styles only, max-width 600px. Always include an unsubscribe link (legally required under CAN-SPAM/GDPR).

```typescript
export function notificationEmailHtml(content: string, unsubscribeUrl: string): string {
  return `<!DOCTYPE html>
<html><head><meta charset="UTF-8"></head>
<body style="font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,sans-serif;max-width:600px;margin:0 auto;padding:20px;color:#212529;">
  ${content}
  <hr style="border:none;border-top:1px solid #dee2e6;margin:2rem 0;">
  <p style="font-size:0.85em;color:#6c757d;">
    <a href="${unsubscribeUrl}" style="color:#6c757d;">Unsubscribe</a>
  </p>
</body></html>`;
}
```

### Subscriber management (Workers KV)

```typescript
const KV_PREFIX = "sub:";

export async function subscribe(kv: KVNamespace, subscriber: Subscriber): Promise<"subscribed" | "already_subscribed"> {
  const key = `${KV_PREFIX}${subscriber.email}`;
  if (await kv.get(key)) return "already_subscribed";
  await kv.put(key, JSON.stringify(subscriber));
  return "subscribed";
}

export async function listSubscribers(kv: KVNamespace): Promise<Subscriber[]> {
  const list = await kv.list({ prefix: KV_PREFIX });
  const subs: Subscriber[] = [];
  for (const key of list.keys) {
    const val = await kv.get(key.name);
    if (val) subs.push(JSON.parse(val));
  }
  return subs;
}
```

### Project address convention

| Project | From address |
|---|---|
| Inaugural Parkrun | `inauguralparkrun@naiklabs.dev` |
| Meal Planner | `mealplanner@naiklabs.dev` |
| *(new project)* | `<projectslug>@naiklabs.dev` |

### Costs

| Tier | Emails/day | Emails/month | Price |
|---|---|---|---|
| Free | 100 | 3,000 | $0 |
| Pro | Unlimited | 50,000 | $20/mo |

Free tier is enough for personal projects. Resend stops sending at the cap — no surprise bills.

---

## Optional: SMS / text messages

For time-sensitive alerts (not newsletters — use email for those). SMS is expensive relative to email; use it for genuinely urgent, short alerts.

### When to use which channel

| Use case | Channel |
|---|---|
| Weekly digest / report | Email |
| Time-sensitive alert (event today, something broke) | SMS |
| Welcome / onboarding | Email |
| One-off notification the user must see now | SMS |

### Provider comparison

| Provider | Outbound SMS (UK) | Phone number | Best for |
|---|---|---|---|
| **Twilio** | ~$0.04/msg | $1.15/mo | Widest docs, most examples |
| **Telnyx** | ~$0.03/msg | $0.50/mo | Cheapest per-message |
| **AWS SNS** | ~$0.04/msg | No number needed (one-way) | One-way alerts, already on AWS |

### Setup (Twilio)

1. Sign up at [twilio.com](https://www.twilio.com) — trial account includes credit
2. Get a phone number (Messaging → Phone Numbers → Buy a number)
3. Add secrets to your Worker:

```bash
wrangler secret put TWILIO_ACCOUNT_SID
wrangler secret put TWILIO_AUTH_TOKEN
wrangler secret put TWILIO_FROM_NUMBER   # e.g. +441234567890
```

### SMS module (Twilio)

```typescript
// src/sms.ts
export async function sendSms(
  accountSid: string, authToken: string, from: string, to: string, body: string
): Promise<void> {
  const res = await fetch(
    `https://api.twilio.com/2010-04-01/Accounts/${accountSid}/Messages.json`,
    {
      method: "POST",
      body: new URLSearchParams({ To: to, From: from, Body: body }),
      headers: {
        Authorization: `Basic ${btoa(`${accountSid}:${authToken}`)}`,
        "Content-Type": "application/x-www-form-urlencoded",
      },
    }
  );
  if (!res.ok) {
    const error = await res.text();
    throw new Error(`Twilio API error ${res.status}: ${error}`);
  }
}
```

### SMS module (Telnyx — cheaper alternative)

```typescript
// src/sms.ts
export async function sendSms(
  apiKey: string, from: string, to: string, text: string
): Promise<void> {
  const res = await fetch("https://api.telnyx.com/v2/messages", {
    method: "POST",
    headers: { Authorization: `Bearer ${apiKey}`, "Content-Type": "application/json" },
    body: JSON.stringify({ from, to, text }),
  });
  if (!res.ok) {
    const error = await res.text();
    throw new Error(`Telnyx API error ${res.status}: ${error}`);
  }
}
```

### UK-specific notes

- UK mobile numbers: `+447...` format (E.164)
- Register a Twilio Alphanumeric Sender ID (free) to show `NaikLabs` instead of a random number
- UK regulations require opt-in consent before sending marketing SMS

---

## Optional: Web push notifications

Browser push notifications — alerts that appear even when your site isn't open. Free, no third-party service.

### How it works

1. User clicks "Enable notifications" on your site
2. Browser provides a push subscription (endpoint URL + keys)
3. Store the subscription in Workers KV
4. Your Worker sends encrypted payloads to the browser's push service

### Libraries for Cloudflare Workers

- **[PushForge](https://github.com/draphy/pushforge)** — zero dependencies, TypeScript, built for edge runtimes
- **[webcrypto-web-push](https://github.com/block65/webcrypto-web-push)** — uses Web Crypto APIs

Standard `web-push` npm package uses Node.js crypto and won't work on Workers.

### Setup

1. Generate VAPID keys: `npx web-push generate-vapid-keys`
2. Store private key as a Worker secret, public key in your frontend
3. Frontend: register service worker, subscribe to push, POST subscription to your Worker
4. Worker: store in KV, send push messages when events happen

### When to use push vs SMS vs email

| Channel | Cost | Reach | Urgency | Opt-in friction |
|---|---|---|---|---|
| Email | ~free | Universal | Low-medium | Email address |
| Web push | Free | Only opted-in browsers | Medium-high | One click |
| SMS | $0.03-0.04/msg | Universal, highest open rate | Highest | Phone number |

---

## Optional: Google Ads (driving traffic)

For projects with a public audience — drive search traffic to the project page.

### Setup

1. Create account at [ads.google.com](https://ads.google.com)
2. Note your **Conversion ID** (`AW-XXXXXXXXX`)
3. Add to your project's `<head>`:

```html
<script async src="https://www.googletagmanager.com/gtag/js?id=AW-XXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'AW-XXXXXXXXX');
</script>
```

### Track conversions

```javascript
document.getElementById('subscribe-btn').addEventListener('click', function() {
  gtag('event', 'conversion', {
    send_to: 'AW-XXXXXXXXX/CONVERSION_LABEL',
    value: 1.0, currency: 'GBP',
  });
});
```

Get `CONVERSION_LABEL` from Google Ads → Goals → Conversions → Create → Website → Manual setup.

### Campaign setup

For Naik Labs projects, use **Search campaigns**:

1. New campaign → Leads or Website traffic → Search
2. Daily budget: $2-5/day to test
3. Keywords per project:

| Project | Keywords |
|---|---|
| Inaugural Parkrun | `new parkrun events`, `inaugural parkrun`, `parkrun near me` |
| Puffer Properties | `property investment UK`, `buy to let analysis` |
| Mindle | `markdown reader mac`, `markdown viewer macos` |
| Meal Planner | `meal planning app`, `weekly meal planner`, `grocery list app` |

### Add Google Analytics (optional, recommended)

Add GA4 alongside the Ads tag — one gtag snippet, two `config` lines:

```html
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
  gtag('config', 'AW-XXXXXXXXX');
</script>
```

---

## Optional: Monetisation (earning from traffic)

### Google AdSense

For content-heavy pages with traffic. Add to `<head>`:

```html
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-XXXXXXXXXXXXXXXX"
  crossorigin="anonymous"></script>
```

**Requirements for approval:** substantial original content, privacy policy page, `ads.txt` at domain root, no copyrighted content. Can take days to weeks.

**Performance impact:** AdSense significantly slows page load. Consider only for content-heavy pages, not landing pages.

### Alternatives

| Service | Min traffic | Payout threshold | Notes |
|---|---|---|---|
| **Carbon Ads** | Developer audience | Via network | Clean single-ad format, good for dev tools |
| **Media.net** | None | $100 | Yahoo/Bing contextual ads |
| **Buy Me a Coffee / Ko-fi** | None | None | Donation link, zero effort |
| **GitHub Sponsors** | None | None | For open source projects |
| **Stripe** | None | Configurable | One-off or subscription via Cloudflare Functions |

For early-stage projects, a donation link is more appropriate than ads on a page with 50 visitors.

---

## Optional: Launch videos (Remotion)

### Setup

```bash
cd ~/Documents/GitHub
npx create-video@latest    # pick "blank" template
cd naiklabs-launch-videos
npm install
npx remotion skills add    # gives Claude the official Remotion instructions
npm run dev
```

One Remotion project holds all launch videos as separate compositions.

### Key concepts

A video is a pure function of the frame number. Use 30 fps (1 second = 30 frames).

- `useCurrentFrame()` — current frame number
- `<Sequence from={N} durationInFrames={M}>` — shows children during that window; **resets frame counter to 0 inside**
- `interpolate(frame, [fromFrame, toFrame], [fromValue, toValue], {extrapolateRight: 'clamp'})` — maps frames to CSS values
- `spring({frame, fps, config: {damping: 200}})` — physics-based 0→1 (damping 200 = no wobble)

### The Rise helper

```tsx
const Rise: React.FC<{ start: number; children: React.ReactNode; distance?: number }> = ({
  start, children, distance = 26,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const s = spring({ frame: frame - start, fps, config: { damping: 200 } });
  return (
    <div style={{ opacity: s, transform: `translateY(${interpolate(s, [0, 1], [distance, 0])}px)` }}>
      {children}
    </div>
  );
};
```

### Video sizing

| Size | Ratio | Use for |
|---|---|---|
| **1080x1350** | 4:5 | LinkedIn/Instagram feed (default) |
| **1080x1080** | 1:1 | Safe everywhere |
| **1920x1080** | 16:9 | YouTube, README embeds |
| **1200x630** | 1.91:1 | OG preview image (`remotion still`) |

### Rendering

```bash
npx remotion render PROJECT_NAME out/PROJECT_SLUG-launch.mp4
npx remotion still PROJECT_NAMECard out/PROJECT_SLUG-preview.png

cp out/PROJECT_SLUG-launch.mp4 ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG/
cp out/PROJECT_SLUG-preview.png ~/Documents/GitHub/naiklabs/projects/PROJECT_SLUG/preview.png
```

### Prompt template for Claude

```
Build me a launch video for **[PROJECT NAME]** in this Remotion project.

**Composition:** id `[Name]`, **1080x1350**, **30 fps**, **1080 frames** (36 seconds).

**Palette:** background `#0F1311`, panel `#161B19`, border `#1E2630`,
text `#ECF1ED`, dim text `#8A968F`, accent `#b5432a`.

**Fonts:** system stacks only — no webfonts, no `@import`.

**No audio.** It must read completely with the sound off.

**The beats, in order — one `<Sequence>` each:**

1. **0–105** (3.5s) — The problem: *"[one-line problem statement]"*
2. **105–330** (7.5s) — The answer: *"[Project Name]"*, tagline
3. **330–600** (9s) — Hero beat: [describe the key visual]
4. **600–810** (7s) — How it works: three stacked rows, staggered in
5. **810–960** (5s) — [Additional info or open source note]
6. **960–1080** (4s) — Hold on the URL, large and still

**Structure:** one component per beat, shared `Shell` + `Rise` helper.
Stagger list items by ~11 frames. Nothing under 24px at this resolution.
```

### What makes a launch video good

- Open on the **problem**, not the product name
- One idea per beat
- Text large enough for a phone (nothing under 24px at 1080 width)
- No audio dependency — everything on screen as text
- Under 40 seconds (30 is better)
- End on the URL, hold it for 4 seconds

### Common Remotion issues

| Symptom | Fix |
|---|---|
| Chrome not found on render | `npx remotion browser ensure` |
| Fonts wrong in render vs Studio | Never use `@import`; use system stacks or `@remotion/google-fonts` |
| Module version mismatch | `npx remotion versions`; install with `npx remotion add <package>` |
| Last beat cut off | `from + durationInFrames` of final Sequence must equal Composition's `durationInFrames` |
| A beat is blank | Frame counting resets to 0 inside `<Sequence>` — `start` values are relative |

---

## Per-project checklist

### Required

- [ ] Build project page (`index.html`)
- [ ] Generate OG image (1200x630 PNG)
- [ ] Run `preflight.sh`
- [ ] Deploy to Cloudflare Pages
- [ ] Set up custom subdomain
- [ ] Run `status.sh`
- [ ] Validate OG on LinkedIn Post Inspector
- [ ] Verify NaikLabs branding (title, header logo, footer)
- [ ] Update portfolio card on naiklabs.dev

### If the project sends email

- [ ] Add `src/email.ts` with `<project>@naiklabs.dev` as sender
- [ ] Add `RESEND_API_KEY` as Wrangler secret
- [ ] Include unsubscribe link in every email

### If the project sends SMS

- [ ] Choose provider (Twilio for docs, Telnyx for cost)
- [ ] Add `src/sms.ts`
- [ ] Add API keys as Wrangler secrets
- [ ] Ensure opt-in consent before sending

### If the project needs ads or analytics

- [ ] Add gtag snippet to `<head>`
- [ ] Set up conversion tracking
- [ ] Start with $2-5/day test budget

### If creating a launch video

- [ ] Write Remotion composition (1080x1350, 30fps, ~36s)
- [ ] Render video and OG still
- [ ] Copy outputs to project directory
- [ ] Validate before posting on social

---

## Costs summary

| Item | Cost | When |
|---|---|---|
| `naiklabs.dev` domain | ~$10-12/yr | Always |
| Cloudflare (DNS, Pages, SSL, bandwidth, D1, KV) | Free | Always |
| Resend email (100/day) | Free | If sending email |
| Twilio SMS (UK) | ~$1.15/mo + $0.04/msg | If sending SMS |
| Web push notifications | Free | If using push |
| Google Ads | $2-5/day | If driving traffic |
| Google Analytics / AdSense tag | Free | If tracking visitors |

Base cost for a project with no ads or SMS: **$0/month** beyond the shared domain. Free tier failure mode is 429 (rate limited), not a bill.

---

## Quick reference

```bash
# Detect project type
"$HOME/.claude/skills/first-site/scripts/detect.sh"

# Preflight check
"$HOME/.claude/skills/first-site/scripts/preflight.sh"

# Deploy
"$HOME/.claude/skills/first-site/scripts/deploy.sh"

# Check live site
"$HOME/.claude/skills/first-site/scripts/status.sh" PROJECT_SUBDOMAIN.naiklabs.dev

# Generate OG image
"$HOME/.claude/skills/first-site/scripts/og-image.sh" \
  --title "PROJECT_NAME" --subtitle "PROJECT_TAGLINE" \
  --domain PROJECT_SUBDOMAIN.naiklabs.dev --accent "PROJECT_ACCENT" \
  --out projects/PROJECT_SLUG/preview.png
```

---

*Based on [ship-what-you-built](https://github.com/jhammant/ship-what-you-built) by Jon Hammant. Adapted for Naik Labs, August 2026.*
