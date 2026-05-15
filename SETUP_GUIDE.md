# 🤖 AI Outreach Automation — Complete Setup Guide
### n8n + Apollo.io + Claude AI + Gmail + Google Sheets

---

## 🗺️ System Overview

```
Apollo.io API ──► n8n Workflow 1 ──► Google Sheets (Leads DB)
                                          │
                                          ▼
                              n8n Workflow 2 (daily 10AM)
                                          │
                                          ▼
                              Claude AI (personalize each email)
                                          │
                                          ▼
                              Gmail (send email)
                                          │
                                          ▼
                              Google Sheets (mark "Sent")
```

---

## STEP 1 — Install & Launch n8n

### Option A: Cloud (Easiest — Recommended for Beginners)
1. Go to https://n8n.io and click **"Start for free"**
2. Create an account
3. You get a hosted n8n instance instantly — no setup needed

### Option B: Self-hosted (Free, advanced)
```bash
# Requires Node.js 18+ installed
npx n8n
# Then open http://localhost:5678
```

---

## STEP 2 — Create Your Google Sheet (Lead Database)

1. Go to https://sheets.google.com and create a **new blank spreadsheet**
2. Name it: `AI Outreach — Leads Database`
3. In the **first sheet tab**, rename it to: `Leads`
4. Create these exact column headers in **Row 1** (copy-paste this exactly):

```
First Name | Last Name | Full Name | Email | Title | Company | Industry | Website | LinkedIn | City | Country | Employees | Apollo ID | Scraped At | Status | Email Sent At | Email Subject | Replied | Notes
```

5. **Copy your Google Sheet ID** from the URL:
   - URL looks like: `https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms/edit`
   - The Sheet ID is: `1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms`
   - Save this — you'll need it in both workflows

---

## STEP 3 — Get Your API Keys

### 3A. Apollo.io API Key
1. Sign up at https://app.apollo.io (free tier gives 50 leads/month; paid from $49/mo)
2. Go to **Settings → Integrations → API**
3. Click **"Create new API key"**
4. Copy and save your key: `apollo_XXXXXXXXXXXX`

### 3B. Anthropic (Claude) API Key
1. Go to https://console.anthropic.com
2. Sign in (or create account)
3. Click **"API Keys"** → **"Create Key"**
4. Copy and save: `sk-ant-XXXXXXXXXXXXXXXX`
5. Add credits: minimum $5 recommended (each email personalization costs ~$0.002)

### 3C. Gmail OAuth Setup
1. Go to https://console.cloud.google.com
2. Create a new project (name it "n8n Outreach")
3. Enable **Gmail API** and **Google Sheets API**
4. Go to **Credentials → Create Credentials → OAuth 2.0 Client ID**
5. Download the credentials JSON
6. You'll connect this inside n8n (see Step 4)

---

## STEP 4 — Import Workflows Into n8n

### Import Workflow 1 (Lead Scraper):
1. Open your n8n dashboard
2. Click **"+ New workflow"** (top right)
3. Click the **three dots menu (⋮)** → **"Import from file"**
4. Upload `workflow_1_lead_scraper.json`
5. The workflow loads with all nodes connected

### Import Workflow 2 (Email Outreach):
1. Repeat the same steps
2. Upload `workflow_2_email_outreach.json`

---

## STEP 5 — Configure Credentials in n8n

Go to **Settings → Credentials** in n8n and create each of these:

### 5A. Google Sheets Credential
1. Click **"Add Credential"** → Search "Google Sheets OAuth2"
2. Click **"Connect with Google"**
3. Sign in with your Gmail account
4. Grant the permissions requested
5. Name it exactly: `Google Sheets OAuth2`

### 5B. Gmail Credential
1. Click **"Add Credential"** → Search "Gmail OAuth2"
2. Click **"Connect with Google"**
3. Use the same Gmail account you want to send FROM
4. Name it exactly: `Gmail OAuth2`

### 5C. Apollo.io API Key
1. Click **"Add Credential"** → Search "HTTP Header Auth"
2. Name: `Apollo API Key`
3. Header name: `x-api-key`
4. Header value: paste your Apollo key

---

## STEP 6 — Update the Workflow Placeholders

Open **each workflow** and replace these values:

### In BOTH workflows:
| Placeholder | Replace With |
|-------------|-------------|
| `YOUR_GOOGLE_SHEET_ID_HERE` | Your actual Sheet ID from Step 2 |

### In Workflow 2 only:
| Placeholder | Replace With |
|-------------|-------------|
| `YOUR_ANTHROPIC_API_KEY_HERE` | Your Claude API key from Step 3B |
| `YOUR_REPLY_TO_EMAIL@yourdomain.com` | The email where you want replies sent |

**How to edit a node:**
1. Click on the node in the canvas
2. A panel opens on the right
3. Edit the fields directly

---

## STEP 7 — Test Before Going Live

### Test Workflow 1 (Lead Scraper):
1. Open Workflow 1 in n8n
2. Click **"Execute Workflow"** (the play button)
3. Watch nodes turn green as they execute
4. Check your Google Sheet — leads should appear!
5. If any node turns red, click it to see the error

### Test Workflow 2 (Email Outreach):
1. **IMPORTANT**: First add one test row to your Google Sheet manually:
   - Fill in a real email you own (to receive the test)
   - Set Status = `Pending`
2. Open Workflow 2 → Click **"Execute Workflow"**
3. Check your Gmail Sent folder — you should see a personalized email
4. Check your Google Sheet — Status should change to `Sent`

---

## STEP 8 — Activate the Workflows (Go Live!)

1. Open Workflow 1 → Toggle **"Active"** switch (top right) to ON
2. Open Workflow 2 → Toggle **"Active"** switch to ON

Now they run automatically:
- **Workflow 1** runs at 9AM Mon-Fri → Scrapes new leads into your sheet
- **Workflow 2** runs at 10AM Mon-Fri → Sends personalized emails to all Pending leads

---

## 🔧 Customization Options

### Change Target Industries (Workflow 1 — Apollo node)
Add industry filters to the Apollo HTTP Request body:
```json
"q_organization_industry_tag_ids": [5567, 5568, 5569]
```
Find industry IDs at: https://docs.apollo.io/reference/people-search

### Change Sending Volume
In Workflow 2, edit the **"Split Into Batches"** node:
- `batchSize: 10` = sends 10 emails per run
- Increase to 50 for higher volume (watch Gmail limits: max 500/day on free, 2000/day on Workspace)

### Change Send Schedule
Edit the **Schedule Trigger** node in either workflow:
- `0 9 * * 1-5` = 9AM Monday to Friday
- `0 9 * * *` = 9AM every day including weekends
- `0 9,14 * * 1-5` = 9AM AND 2PM weekdays

### Customize the Email Prompt
In Workflow 2, open the **"Claude AI — Personalize Email"** node.
Edit the `content` field inside the messages body to change the tone, CTA, or products pitched.

---

## 📊 Tracking Replies (Manual Step)

When someone replies positively:
1. Open your Google Sheet
2. Find their row
3. Change `Replied` column to `Yes`
4. Add notes in the `Notes` column
5. You handle the negotiation personally from here

**Tip**: Set up a Gmail filter to label incoming emails that are replies to your outreach campaign, so they're easy to spot.

---

## ⚠️ Important Limits & Best Practices

| Platform | Limit | Recommendation |
|----------|-------|----------------|
| Gmail (free) | 500 emails/day | Stay under 400 to be safe |
| Gmail (Workspace) | 2,000 emails/day | Ideal for high volume |
| Apollo.io (free) | 50 exports/month | Upgrade for 100k contacts |
| Claude API | Pay-per-use | ~$0.002/email, budget $10/mo |

### Avoid Spam Filters:
- Warm up your email domain for 2 weeks before launching (use tools like Instantly or Mailreach)
- Send max 50-100 emails/day for first 2 weeks
- Always include an unsubscribe line in your emails (add it to the Claude prompt)
- Use Google Workspace email, not personal Gmail, for sending

---

## 🚀 Scaling to 100,000 Contacts

For massive scale, upgrade your stack:
1. **Apollo.io Professional** ($79/mo) — Bulk exports, no limit on contact views
2. **Google Workspace** ($6/user/mo) — 2,000 emails/day sending limit
3. **Dedicated SMTP** (e.g., SendGrid, Mailgun) — Replace Gmail node with SMTP node for unlimited sending
4. **Multiple sending accounts** — Use 3-5 Gmail accounts, rotate with n8n

To add SendGrid instead of Gmail:
- In n8n, replace the Gmail node with **"Send Email"** node
- Use SMTP credentials from SendGrid
- Get 100 free emails/day, or paid plans for unlimited

---

## 🆘 Troubleshooting

| Problem | Solution |
|---------|----------|
| Apollo node returns empty | Check your API key; verify your search filters aren't too restrictive |
| Google Sheets node fails | Re-authenticate your Google credentials in n8n Settings |
| Claude AI returns error | Check your Anthropic API key and ensure you have credits |
| Gmail fails to send | Re-authenticate Gmail; check you haven't hit daily limits |
| Workflow not auto-running | Make sure the "Active" toggle is ON |

---

## 📁 Files in This Package

| File | Purpose |
|------|---------|
| `workflow_1_lead_scraper.json` | Import into n8n — scrapes leads from Apollo to Google Sheets |
| `workflow_2_email_outreach.json` | Import into n8n — sends AI-personalized emails daily |
| `SETUP_GUIDE.md` | This document |

---

*Built for: AI Solutions Cold Outreach System*
*Stack: n8n + Apollo.io + Claude AI (Anthropic) + Gmail + Google Sheets*
