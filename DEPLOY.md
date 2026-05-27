# ShieldOps Deployment Guide
## Three portals, one Firebase project, Netlify hosting

---

## STEP 1 — Firebase Setup

### 1.1 Firestore Security Rules
In Firebase Console → Firestore → Rules, paste:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

### 1.2 Enable Authentication
Firebase Console → Authentication → Sign-in method → Enable **Email/Password**

---

## STEP 2 — Create User Accounts in Firebase

### 2.1 Create Firebase Auth users
In Firebase Console → Authentication → Users, create:
- `admin@shieldops.io` — your admin password
- `analyst@shieldops.io` — your analyst password  
- `customer@yourcompany.com` — test customer password

### 2.2 Create Firestore user documents
In Firestore → users collection, create one doc per user:

**Admin user** (doc ID = Firebase Auth UID):
```json
{
  "uid": "<firebase-auth-uid-of-admin>",
  "email": "admin@shieldops.io",
  "name": "ShieldOps Admin",
  "role": "admin",
  "tenant_id": null,
  "created_at": "<Timestamp>"
}
```

**Analyst user**:
```json
{
  "uid": "<firebase-auth-uid-of-analyst>",
  "email": "analyst@shieldops.io",
  "name": "SOC Analyst",
  "role": "analyst",
  "tenant_id": null,
  "created_at": "<Timestamp>"
}
```

**Customer user**:
```json
{
  "uid": "<firebase-auth-uid-of-customer>",
  "email": "customer@yourcompany.com",
  "name": "Test Customer",
  "role": "owner",
  "tenant_id": "<tenant-doc-id>",
  "created_at": "<Timestamp>"
}
```

---

## STEP 3 — Create Test Tenant

In Firestore → tenants collection, create a document:
```json
{
  "company_name": "Acme Web Solutions",
  "domain": "acme.ie",
  "plan": "pro",
  "status": "active",
  "hosting_provider": "SiteGround",
  "cms_platform": "wordpress",
  "critical_urls": ["/wp-admin", "/checkout", "/login"],
  "alert_contacts": ["owner@acme.ie"],
  "cloudflare_connected": true,
  "cloudflare_zone_id": "YOUR_CF_ZONE_ID",
  "plugin_installed": true,
  "analyst_assigned": "<analyst-uid>",
  "notes": "Test tenant",
  "created_at": "<Timestamp>"
}
```

Copy the document ID — you'll need it for the customer user's `tenant_id`.

---

## STEP 4 — Create Sample Alert (Optional)

In Firestore → alerts collection:
```json
{
  "tenant_id": "<tenant-doc-id>",
  "case_id": "CASE-001",
  "title": "Brute Force Attack Detected",
  "type": "brute_force",
  "severity": "high",
  "status": "open",
  "detected_at": "<Timestamp>",
  "sla_deadline": "<Timestamp 24h from now>",
  "raw_evidence": {
    "ip": "185.234.219.47",
    "url": "/wp-admin/wp-login.php",
    "user_agent": "Mozilla/5.0 (compatible; SemrushBot)",
    "attempt_count": 847,
    "response_code": 401,
    "page_diff": ""
  },
  "ai_summary": "Your website's login page received 847 failed login attempts from a single IP address in Romania over the past 2 hours. This is a brute force attack trying to guess your admin password. Your site is currently blocking these attempts, but we recommend enabling two-factor authentication.",
  "customer_visible_summary": "Someone tried to break into your website admin area 847 times. We detected and logged this attack. Your site remains secure, but we recommend changing your admin password as a precaution.",
  "ai_recommended_actions": [
    "Block the attacking IP address at your firewall",
    "Enable two-factor authentication on WordPress",
    "Limit login attempts to 5 per 10 minutes",
    "Review all admin user accounts for any unauthorized access"
  ]
}
```

---

## STEP 5 — Configure Claude API Key

In each portal folder, create a `.env` file:
```
VITE_CLAUDE_API_KEY=sk-ant-api03-YOUR_KEY_HERE
```

**Important**: The AI chat and document generation features require this key.
Get your API key at: https://console.anthropic.com

> For production: Route API calls through your own backend (Node.js/Firebase Function) to protect the API key from browser exposure.

---

## STEP 6 — Install & Build

```bash
# Install Node.js 18+ if not installed
# https://nodejs.org

# Build Admin Console first
cd shieldops-admin
npm install
npm run build
# dist/ folder created

# Build SOC Console
cd ../shieldops-soc
npm install
npm run build

# Build Customer Portal
cd ../shieldops-customer
npm install
npm run build
```

### Local development testing:
```bash
# In each folder:
npm run dev
# Customer: http://localhost:5173
# SOC: http://localhost:5174  (run with: vite --port 5174)
# Admin: http://localhost:5175 (run with: vite --port 5175)
```

---

## STEP 7 — Deploy to Netlify

### Manual drag-and-drop (simplest):
1. Go to **https://netlify.com** → Log in
2. Click **"Add new site"** → **"Deploy manually"**
3. Drag the `shieldops-admin/dist/` folder → note the URL (e.g., `random-name.netlify.app`)
4. Repeat for `shieldops-soc/dist/` → note URL
5. Repeat for `shieldops-customer/dist/` → note URL

### Set custom domains (optional):
- In Netlify site settings → Domain management → Add custom domain
- Customer: `app.shieldops.io`
- SOC: `soc.shieldops.io`
- Admin: `admin.shieldops.io`

### HashRouter note:
All three portals use HashRouter, so no `_redirects` file is needed. Netlify will serve the single `index.html` correctly.

---

## STEP 8 — Test Everything

### Admin Console:
1. Open admin URL → login as `admin@shieldops.io`
2. Go to Tenants → verify test tenant is visible
3. Go to Analysts → create an analyst account
4. Go to Rules → adjust detection thresholds
5. Go to Billing → verify MRR calculation

### SOC Console:
1. Open SOC URL → login as `analyst@shieldops.io`
2. Dashboard → should show metrics
3. Alert Queue → click on test alert → CaseDetail opens
4. IOCs tab → add a test IP → click Enrich
5. Response Actions tab → try "Mark False Positive"
6. AI tab → ask "What MITRE techniques match?"
7. Threat Intel → add IOC to shared feed
8. Timeline → select case

### Customer Portal:
1. Open customer URL → login as `customer@yourcompany.com`
2. Dashboard → should show company name, alerts
3. Alerts → click on alert → AlertDetail
4. AI Chat → ask "What happened to my site?"
5. Compliance → try generating GDPR document (needs Managed plan)
6. Settings → update company info

---

## FIRESTORE COLLECTIONS REFERENCE

| Collection | Purpose |
|-----------|---------|
| `users` | Auth user profiles + roles |
| `tenants` | Customer site configurations |
| `alerts` | Security alerts/incidents |
| `iocs` | Indicators of Compromise |
| `threat_intel` | Shared cross-tenant IOC feed |
| `action_logs` | Response action audit trail |
| `approval_requests` | Customer approval workflow |
| `cases` | Investigation case records |
| `findings` | Case investigation findings |
| `commands` | Commands run during investigation |
| `tools` | Tools used during investigation |
| `chat_messages` | AI conversation history |
| `compliance_docs` | Generated compliance documents |
| `config` | Detection rule configuration |

---

## PLAN FEATURE MATRIX

| Feature | Starter | Pro | Managed | Agency |
|---------|---------|-----|---------|--------|
| 24/7 Monitoring | ✓ | ✓ | ✓ | ✓ |
| Email Alerts | ✓ | ✓ | ✓ | ✓ |
| AI Chat | ✗ | ✓ | ✓ | ✓ |
| IOC Enrichment | ✗ | ✓ | ✓ | ✓ |
| Response Actions | ✗ | ✓ | ✓ | ✓ |
| Compliance Docs | ✗ | ✗ | ✓ | ✓ |
| Dedicated Analyst | ✗ | ✗ | ✓ | ✓ |
| SLA | None | 24h | 4h | 1h |
| White Label | ✗ | ✗ | ✗ | ✓ |
| Price | €29/mo | €99/mo | €299/mo | €799/mo |

---

## ARCHITECTURE DIAGRAM

```
Browser → shieldops-customer (Netlify)
            ↓ Firebase Auth
            ↓ Firestore reads (tenant scoped)
            ↓ Claude API (AI chat)

Browser → shieldops-soc (Netlify)
            ↓ Firebase Auth (analyst/admin only)
            ↓ Firestore reads (all tenants)
            ↓ Cloudflare API (response actions)
            ↓ Claude API (AI triage)

Browser → shieldops-admin (Netlify)
            ↓ Firebase Auth (admin only)
            ↓ Firestore reads/writes (full access)

Shared: Firebase project soc-dashboard-1c74c
```

---

## PRODUCTION CHECKLIST

- [ ] Firebase API key restricted to your Netlify domains in Google Cloud Console
- [ ] Claude API key moved to backend (Firebase Functions) — never expose in browser
- [ ] Firestore rules tightened for production (role-based access)
- [ ] Custom domains configured in Netlify
- [ ] Firebase Auth authorized domains updated (add your Netlify domains)
- [ ] Admin and SOC URLs kept private (not linked publicly)
- [ ] Backup Firestore data regularly

---

## SUPPORT

Built with:
- React 18 + Vite
- Firebase 10 (Auth + Firestore)
- Recharts (charts)
- jsPDF + autotable (PDF reports)
- Claude Sonnet (AI features)
- Netlify (hosting)
