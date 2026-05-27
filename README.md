# 🛡️ ShieldOps — Multi-Tenant Website Security Monitoring Platform

> A full-stack SaaS platform that provides 24/7 website security monitoring, AI-powered threat triage, and compliance reporting for small and medium businesses.

---

## 🚀 Live Demo

| Portal | URL | Credentials |
|--------|-----|-------------|
| 👑 Admin Console | _your-admin.netlify.app_ | admin@shieldops.io |
| 🔍 SOC Analyst Console | _your-soc.netlify.app_ | analyst@shieldops.io |
| 🌐 Customer Portal | _your-customer.netlify.app_ | customer@shieldops.io |

---

## 📸 What It Does

ShieldOps is a **3-portal security SaaS** built for the Irish/EU SMB market. It monitors websites 24/7, detects threats in real time, and gives non-technical business owners plain-English explanations of what happened — and what to do.

### Customer Portal
- Real-time security dashboard with uptime, SSL, and threat status
- Plain-English AI-generated alert summaries (powered by Google Gemini)
- One-click approval/rejection for analyst response actions
- GDPR breach notification, ISO 27001 evidence packs, and insurance claim reports
- AI chat assistant: ask "What happened to my site last night?"

### SOC Analyst Console
- Multi-tenant alert queue with SLA countdown timers (1h / 4h / 24h)
- Full case investigation workspace: IOC enrichment, MITRE ATT&CK mapping, findings, commands, tools
- One-click response actions: block IP via Cloudflare, rate-limit endpoints, geo-block countries, force WordPress password resets
- Customer approval workflow with live Firestore listeners
- Shared threat intelligence feed across all tenants
- AI-assisted triage with case context

### Admin Console
- MRR / ARR revenue dashboard
- Full tenant CRUD with plan and analyst assignment
- Create analyst accounts directly from the UI
- Global detection rule configuration (brute force thresholds, SSL expiry warnings, page change sensitivity)
- Billing breakdown per tenant and plan

---

## 🏗️ Architecture

```
Browser → shieldops-customer (Netlify)
            ↓ Firebase Auth (Email/Password)
            ↓ Firestore (tenant-scoped reads)
            ↓ Google Gemini API (AI chat + summaries)

Browser → shieldops-soc (Netlify)
            ↓ Firebase Auth (analyst/admin roles)
            ↓ Firestore (cross-tenant reads)
            ↓ Cloudflare API (response actions)
            ↓ Google Gemini API (AI triage)

Browser → shieldops-admin (Netlify)
            ↓ Firebase Auth (admin role only)
            ↓ Firestore (full read/write)

Shared Backend: Firebase (soc-dashboard-1c74c)
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + Vite |
| Routing | React Router v6 (HashRouter) |
| Backend | Firebase 10 (Auth + Firestore) |
| AI | Google Gemini 2.0 Flash (free tier) |
| Charts | Recharts |
| PDF Generation | jsPDF + jspdf-autotable |
| Hosting | Netlify (free tier) |
| Design | Custom dark theme CSS (Inter + JetBrains Mono) |

---

## 🗃️ Firestore Data Model

| Collection | Purpose |
|-----------|---------|
| `users` | Auth user profiles + roles (admin / analyst / owner) |
| `tenants` | Customer site configurations |
| `alerts` | Security alerts with AI summaries and evidence |
| `iocs` | Indicators of Compromise with enrichment data |
| `threat_intel` | Shared cross-tenant IOC feed |
| `action_logs` | Response action audit trail |
| `approval_requests` | Customer approval workflow |
| `cases` | Investigation case records |
| `findings` | Case findings with MITRE mapping |
| `compliance_docs` | Generated GDPR / ISO 27001 / insurance documents |
| `config` | Global detection rule configuration |

---

## 💳 Plan Tiers

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

## ⚡ Quick Setup

### 1. Firebase Setup
1. Create a Firebase project at [console.firebase.google.com](https://console.firebase.google.com)
2. Enable **Email/Password** authentication
3. Create a **Firestore database**
4. Set Firestore rules to allow authenticated reads/writes
5. Open `firebase-setup.html` in a browser to auto-create all accounts and seed data

### 2. AI Setup
1. Get a free API key at [aistudio.google.com](https://aistudio.google.com)
2. Update `GEMINI_KEY` in `src/ai.js` in each portal

### 3. Deploy
```bash
# Build each portal
cd shieldops-admin && npm install && npm run build
cd shieldops-soc && npm install && npm run build
cd shieldops-customer && npm install && npm run build
```
Drag each `dist/` folder to [netlify.com](https://netlify.com) — get 3 live URLs instantly.

---

## 🔐 Security Notes

- Firebase API config is client-side (protected by Firestore security rules and Auth)
- For production: move AI API key calls to a Firebase Function backend
- Restrict Firebase API key to your Netlify domains in Google Cloud Console
- Tighten Firestore rules to role-based access before going live with real customers

---

## 👤 Author

Built by **Prasanna Kumar** — [LinkedIn](https://linkedin.com/in/YOUR_PROFILE)

---

## 📄 Licence

MIT — free to use, modify, and deploy.
