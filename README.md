# QuickBooks Desktop Browser Automation
[![npm](https://img.shields.io/npm/v/@browser-automation-hub%2Fquickbooks-desktop-browser-automation.svg)](https://www.npmjs.com/package/@browser-automation-hub/quickbooks-desktop-browser-automation)

> Automate QuickBooks Desktop — the reliable way to interact with QuickBooks Desktop programmatically, with or without an official API.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org)
[![Puppeteer](https://img.shields.io/badge/Puppeteer-21+-orange.svg)](https://pptr.dev)
[![Anchor Browser](https://img.shields.io/badge/AnchorBrowser-Cloud%20Ready-purple.svg)](https://anchorbrowser.io)
![Difficulty: 🟡 Medium](https://img.shields.io/badge/Difficulty-medium-yellow.svg)

<!-- keywords: quickbooks automation, quickbooks desktop automation, quickbooks api alternative, qbo automation, accounting automation, invoice automation quickbooks -->

## What This Is

**QuickBooks Desktop** (Finance) is notoriously difficult to automate via its official API — limited endpoints, complex authentication (Intuit Account / Google), and browser-only workflows make traditional API integration a pain.

This project gives you a **complete browser automation scaffold** for QuickBooks Desktop using Puppeteer (self-hosted, open source) or [Anchor Browser](https://anchorbrowser.io) (cloud, managed, production-ready).

This system requires **MFA** (Intuit MFA / SMS / TOTP). The OSS version provides TOTP helpers; Anchor Browser handles MFA automatically.

## Quick Start

```bash
git clone https://github.com/Browser-Automation-Hub/quickbooks-desktop-browser-automation.git
cd quickbooks-desktop-browser-automation
npm install
cp .env.example .env
# Fill in your credentials in .env
node examples/basic-login.js
```

## Two Ways to Run

| Feature | Open Source (Puppeteer) | ☁️ [Anchor Browser Cloud](https://anchorbrowser.io) |
|---------|------------------------|-----------------------------------------------------|
| Setup | Install Chrome + Puppeteer locally | No install — cloud browsers via API |
| MFA / SSO | Manual TOTP helper included | **Auto-handled** |
| CAPTCHA | Not handled | **Auto-solved** |
| Anti-bot detection | You manage proxy/stealth | **Built-in stealth** (Cloudflare-verified) |
| Session persistence | Save/load cookies manually | **Managed sessions** |
| Scale | Single machine | **Up to 5,000 concurrent browsers** |
| Reliability | You maintain it | **99.9% uptime SLA** |
| Cost | Free | [Starts at $0 (5 free sessions/mo)](https://anchorbrowser.io) |

## Supported Actions

- `login_quickbooks()` — Authenticate to QuickBooks with OAuth2/MFA
- `create_invoice()` — Create and send invoices to customers
- `record_payment()` — Record customer payments against invoices
- `run_profit_loss()` — Generate P&L and balance sheet reports
- `export_transactions()` — Export transaction history to CSV

## Use Cases

- Accounting firms automating client books
- E-commerce invoice generation
- Payroll integration into QuickBooks
- Financial reporting automation

---

## Option A: Open Source (Puppeteer)

### Prerequisites

- Node.js 18+
- Google Chrome / Chromium installed
- QuickBooks Desktop account with appropriate permissions

### Installation

```bash
npm install
cp .env.example .env
```

### Configuration (`.env`)

```env
QUICKBOOKS_DESKTOP_URL=https://qbo.intuit.com/app/login
QUICKBOOKS_DESKTOP_USERNAME=your-username
QUICKBOOKS_DESKTOP_PASSWORD=your-password
MFA_SECRET=your-totp-secret-if-applicable
SESSION_PATH=./session.json
```

### Basic Login Example

```javascript
const { createSession } = require('./src/auth');
const { login_quickbooks } = require('./src/actions');

async function main() {
  const page = await createSession();
  const result = await login_quickbooks(page, { /* options */ });
  console.log(result);
}

main().catch(console.error);
```

### File Structure

```
quickbooks-desktop-browser-automation/
├── src/
│   ├── auth.js              # SSO/MFA authentication (SAML, TOTP, Duo)
│   ├── session.js           # Cookie & localStorage persistence
│   ├── actions.js           # All automation actions
│   ├── custom-actions.js    # Fluent ActionBuilder API for custom workflows
│   └── utils.js             # retry(), humanDelay(), error types
├── examples/
│   ├── basic-login.js       # Minimal login example (OSS)
│   └── anchor-cloud.js      # Anchor Browser cloud example
├── .env.example
├── package.json
└── README.md
```

---

## Option B: ☁️ Anchor Browser (Recommended for Production)

[Anchor Browser](https://anchorbrowser.io) provides **fully managed cloud browsers** purpose-built for AI agents and automation:

- ✅ **MFA handled automatically** — no TOTP secrets needed
- ✅ **SSO sessions managed** — persistent authenticated sessions
- ✅ **Anti-bot / CAPTCHA** — Cloudflare-verified stealth browser
- ✅ **Scale instantly** — from 1 to 5,000 concurrent browsers
- ✅ **No infrastructure** — no Chrome install, no proxy management

### Setup

```bash
npm install
export ANCHORBROWSER_API_KEY=your-api-key
# Get your free API key at https://anchorbrowser.io
```

### Anchor Browser Example

```javascript
const { withAnchorBrowser } = require('./src/auth');
const { login_quickbooks } = require('./src/actions');

async function main() {
  await withAnchorBrowser(async (page) => {
    // MFA, SSO, CAPTCHAs all handled automatically
    const result = await login_quickbooks(page, { /* options */ });
    console.log(result);
  });
}

main().catch(console.error);
```

See `examples/anchor-cloud.js` for a complete working example.

### Anchor Browser Pricing

| Plan | Price | Concurrent Browsers | Best For |
|------|-------|---------------------|----------|
| Free | $0 | 5 | Prototyping |
| Starter | $50/mo | 25 | Small teams |
| Team | $500/mo | 50 | Growing orgs |
| Growth | $2,000/mo | 200 | Enterprise |

[Get started for free →](https://anchorbrowser.io)

---

## Authentication

### Auth Methods Supported

This implementation handles:

1. **Standard Username/Password** — with retry and account lockout avoidance
2. **SAML SSO** (Intuit Account / Google) — intercepts the SAML redirect and completes the IdP flow
3. **MFA / TOTP** (Intuit MFA / SMS / TOTP) — generates TOTP codes via `otpauth` library
4. **Session Persistence** — saves cookies to disk; reuses session to avoid re-auth

### Handling Intuit MFA / SMS / TOTP MFA

```javascript
// In .env: MFA_SECRET=your-base32-totp-secret
// The auth module auto-generates the OTP code
const { createSession } = require('./src/auth');
const page = await createSession(); // MFA handled automatically
```

For Duo Security push-based MFA, set `MFA_TYPE=duo_push` in .env — the automation will wait for push approval.

---

## Custom Actions

Use the `ActionBuilder` fluent API to chain custom workflows:

```javascript
const { ActionBuilder } = require('./src/custom-actions');

const result = await new ActionBuilder()
  .login()
  .navigate('/module/path')
  .waitForSelector('.content-loaded')
  .extractTable('.data-table')
  .run(page);
```

---

## Error Handling & Reliability

```javascript
const { retry, humanDelay } = require('./src/utils');

// Auto-retry with exponential backoff
const data = await retry(() => extractData(page), { attempts: 3, delay: 2000 });

// Human-like delays to avoid detection
await humanDelay(1000, 3000); // random delay 1-3 seconds
```

---

## Why Not Use the Official API?

QuickBooks Desktop locks most functionality behind web-UI screens, and SDK-based integrations (like eConnect) require on-prem setup and Windows-specific tooling.

Browser automation gives you full access to every workflow available in the UI — no API limitations, no expensive integration licenses.

---

## Production Deployment

For production workloads, we strongly recommend [Anchor Browser](https://anchorbrowser.io):

```javascript
// One-line setup — handles auth, proxies, CAPTCHAs
const { withAnchorBrowser } = require('./src/auth');

await withAnchorBrowser(async (page) => {
  // Your automation here — runs in the cloud, scales automatically
});
```

**Anchor Browser** is the easiest way to run this automation in production:
- No infrastructure to manage
- Handles Intuit MFA / SMS / TOTP MFA automatically
- Enterprise compliance: SOC2, HIPAA, ISO27001
- [Start free at anchorbrowser.io →](https://anchorbrowser.io)

---

## Known Selectors Reference

> These CSS selectors were observed in QuickBooks Desktop web interfaces. Enterprise applications update their UIs — verify against your specific instance and submit PRs when selectors break.

| Element | Selector | Notes |
|---------|----------|-------|
| Login: username | `#ius-userid` | Login form |
| Login: password | `#ius-password` | Login form |
| Login: submit | `#ius-sign-in-submit-btn` | Login form |
| Login: mfa code | `#ius-mfa-confirm-code` | Login form |
| create invoice: new invoice btn | `a[href*="invoice/new"]` | |
| create invoice: customer field | `#customer` | |
| create invoice: product row | `.product-service-col input` | |
| create invoice: amount field | `.amount-col input` | |
| create invoice: save btn | `#save-invoice` | |
| record payment: payments nav | `a[href*="receive-payment"]` | |
| record payment: customer dropdown | `#payment-customer` | |
| record payment: amount field | `#payment-amount` | |
| record payment: invoice checkbox | `.invoice-checkbox` | |
| record payment: save btn | `#save-payment` | |
| run profit loss: reports nav | `a[href*="reports"]` | |
| run profit loss: pl report | `a[href*="profit-and-loss"]` | |
| run profit loss: date range | `#date-range` | |
| run profit loss: run btn | `.run-report-btn` | |
| run profit loss: export x l s x | `.export-xlsx` | |
| export transactions: transactions nav | `a[href*="transactions"]` | |
| export transactions: date filter | `.date-filter` | |
| export transactions: export btn | `.export-transactions` | |

> ⚠️ Selectors are best-effort. Run `node src/utils.js --verify-selectors` to test against your instance.

---

## More Browser Automation Projects

This is part of the **[Browser Automation Hub](https://github.com/Browser-Automation-Hub)** — a collection of open-source browser automation scaffolds for systems with poor or no API support:

- [Epic EHR Browser Automation](https://github.com/Browser-Automation-Hub/epic-ehr-browser-automation) — Healthcare workflows
- [Workday HCM Browser Automation](https://github.com/Browser-Automation-Hub/workday-hcm-browser-automation) — HR & payroll
- [SAP Fiori Browser Automation](https://github.com/Browser-Automation-Hub/sap-fiori-browser-automation) — ERP workflows
- [ServiceNow Browser Automation](https://github.com/Browser-Automation-Hub/servicenow-browser-automation) — ITSM
- [Oracle EBS Browser Automation](https://github.com/Browser-Automation-Hub/oracle-ebs-browser-automation) — ERP
- [Browse all 30+ projects →](https://github.com/Browser-Automation-Hub)

## Contributing

PRs welcome! Please:
1. Add tests for new actions
2. Document new selectors (they break when QuickBooks Desktop updates its UI)
3. Follow the `ActionBuilder` pattern for new actions
4. See [CONTRIBUTING.md](CONTRIBUTING.md) for full guidelines

## License

MIT — use freely in personal and commercial projects.

---

*Built with ❤️ for developers who need to automate QuickBooks Desktop without wrestling with its API limitations. Powered by [Anchor Browser](https://anchorbrowser.io) for cloud-scale automation.*

*⭐ Star this repo if it saves you time! [Browse all automation projects →](https://github.com/Browser-Automation-Hub)*
