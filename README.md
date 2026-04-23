# 🛡️ ExpiryGuard v4.0.5

**Product Expiry Tracking SaaS — Full Stack**
Stack: Node.js/Express (Railway) · Supabase (PostgreSQL + Auth) · Twilio (SMS) · Resend (Email) · Claude Vision (OCR) · Paystack (Payments) · Netlify (Frontend)

---

## All Issues Fixed in v4.0.5

| Issue | Root Cause | Fix Applied |
|---|---|---|
| **Profile overlay read-only** | Profile fields (name, username, timezone) were removed from settings but no editable form was added to the overlay | Overlay now has View and Edit modes. Tap "Edit Profile" to open inline form with name, username, timezone editing and username availability check |
| **Cron alerts stopped firing** | `.eq("profiles.is_blocked", false)` on a Supabase join does not work in Supabase JS v2 — silently returns zero rows, so no products were fetched at all | Fixed: fetch blocked user IDs in a separate query, then filter in JavaScript before sending alerts |
| **Icons showing as boxes in Kodular** | Some Android WebView versions don't support all emoji code points | No code change needed — the HTML is correctly UTF-8 encoded. If boxes appear on-device, enable emoji fonts in Kodular or use text labels |
| **CSV download fails in Kodular WebView** | `URL.createObjectURL()` + anchor click doesn't trigger file download in WebView | Falls back to `data:text/csv` URI opened via `window.open()`. User taps ⋮ in their browser to save the file |
| **Blocked users still received cron alerts** | Same bad join query above — all users' alerts were being skipped, not just blocked ones | Fixed with the two-step query approach |
| **Login returns to settings page** | `localStorage` was persisting last navigation state across sessions | `last_dpage` and `last_mpage` cleared on logout. App always opens at Dashboard on fresh login |
| **Forgot password broken** | `FRONTEND_URL` env var incorrect, and Resend free tier restricts sending | Fixed email template with plain text link fallback. Ensure `FRONTEND_URL` matches your Netlify URL exactly |
| **Change password missing** | Feature not implemented | New overlay modal in settings: enter current password + new password (dual entry) → verified server-side |
| **Railway URL visible to all** | API settings section was shown to everyone | Hidden entirely for non-admin users. Only users with `plan: "admin"` see the backend URL section |
| **Test-mode products persist** | No tracking of which products were added during test mode | `added_in_test_mode` column on products table. Disabling test mode (from admin) auto-deletes these products |
| **Plan badge missing** | Not implemented | Plan chip (Free / Paid / Admin / Test) shown in sidebar footer (desktop) and available in profile overlay |
| **OCR gated correctly** | OCR was available to all users | Now only visible to Paid / Admin / Test mode users, or users with ≥ `sms_min_credits` credits |
| **Logo not inline** | Font-size too large, flexbox not applied correctly | Reduced to 14px (mobile) / 13px (desktop), flex row with `align-items:center` |
| **Logo click shows profile** | Not implemented | Clicking 🛡️ ExpiryGuard on both mobile and desktop opens the profile overlay |
| **Logo wiggle animation** | Not implemented | `logoWiggle` CSS animation plays twice on load to hint users to click the logo |
| **Username change limit** | Not enforced | `username_changes` counter in profiles, `max_username_changes` admin setting. Profile edit shows remaining changes |
| **FAB hidden on settings** | FAB appeared on all pages | Hidden (opacity 0, pointer-events none) when on settings or notifications page |
| **Dark mode flash on toggle** | `body { transition: background .25s }` fired during theme switch | Fix: `body.style.transition = 'none'` before switching, restored via `requestAnimationFrame` after paint |
| **Notifications page** | Not implemented | Full notifications page on both desktop (sidebar nav) and mobile (bottom nav). Credit additions and system events create notifications. Unread dot on bell icon |
| **Bank transfer fallback** | Not implemented | Configured in Admin → Settings → Bank Account Fallback. Shown in credits modal when Paystack fails |
| **Stop registrations toggle** | Not implemented | `registration_open` admin setting. Toggle in Admin → Settings. Signup page shows "Registration closed" and button is disabled |
| **Auto logout not enforced** | Frontend read from localStorage, not server | Frontend fetches `/api/config` on boot — gets `auto_logout_minutes` from DB. Admin changes take effect on next user boot |
| **Settings (terms URL) not enforced** | Same as above | Terms URL, SMS threshold, paystack key all fetched from `/api/config` on every boot |

---

## Files in This Version

```
expiryguard_v405/
├── backend/
│   ├── server.js              — Complete backend v4.0.5
│   ├── supabase_schema.sql    — Run in Supabase SQL Editor
│   ├── package.json           — Dependencies
│   ├── Procfile               — Railway startup
│   └── .env.example           — All environment variables
├── frontend/
│   ├── index.html             — Main user platform
│   └── admin.html             — Admin management panel
├── BANK_ACCOUNTS_TEMPLATE.txt — Guide for setting up bank transfer
├── TERMS_TEMPLATE.txt         — Terms & Conditions template
└── README.md                  — This file
```

---

## What to Update on GitHub

Replace these files from v4.0.4:
- `backend/server.js` — cron fix, profile route, notifications, new admin routes
- `backend/supabase_schema.sql` — new columns and tables
- `frontend/index.html` — all user-facing fixes
- `frontend/admin.html` — registration toggle, bank accounts field, SMS threshold

Add new files:
- `BANK_ACCOUNTS_TEMPLATE.txt`
- `TERMS_TEMPLATE.txt`

---

## Supabase — Run Schema Again

Run `supabase_schema.sql` in Supabase SQL Editor. It safely adds:
- `username_changes` column to profiles
- `added_in_test_mode` column to products
- `notifications` table (new)
- New admin_settings keys: `registration_open`, `max_username_changes`, `bank_accounts`

All statements use `IF NOT EXISTS` or `ON CONFLICT DO NOTHING` — safe to re-run.

---

## New Environment Variables

No new Railway environment variables are needed for v4.0.5. All new settings are managed from the Admin Panel → Settings.

---

## Setting Up Terms & Conditions

1. Open `TERMS_TEMPLATE.txt` from this package
2. Fill in your name, app name, email, and jurisdiction
3. Upload the file to your GitHub repository as `TERMS.md`
4. Copy the raw GitHub URL of your file, e.g.:
   `https://raw.githubusercontent.com/yourusername/yourrepo/main/TERMS.md`
   Or use the rendered version:
   `https://github.com/yourusername/yourrepo/blob/main/TERMS.md`
5. In the Admin Panel → Settings, paste this URL into **Terms & Conditions URL**
6. Save settings — the signup page will immediately show the link

---

## Setting Up Bank Transfer Fallback

1. In the Admin Panel → Settings → Bank Account Fallback, enter your accounts
2. One account per line, format: `AccountNumber|BankName|AccountName|ContactEmail`
3. Example:
   ```
   0123456789|First Bank Nigeria|John Doe|payments@yourdomain.com
   9876543210|GTBank|John Doe|payments@yourdomain.com
   ```
4. The bank transfer section appears in the credits modal when:
   - Paystack is not configured, OR
   - Paystack payment initialization fails

When a user pays via bank transfer:
1. They send the screenshot to your contact email
2. You verify in your banking app
3. In Admin → Users → Manage, tap "Add Credits" with the appropriate amount

---

## Kodular WebView — Complete Setup Guide

### Why the Logo Didn't Load on Initialization

Kodular's WebView has a timing bug: `goto_url` in `Screen.Initialize` fires before the WebView component is fully mounted. The page gets a "connection refused" error.

**Fix:** Use a `Clock` component:
1. Drag a `Clock` component onto your screen
2. Set `Enabled = False` initially
3. Set `TimerInterval = 600` (600ms)
4. In `Screen.Initialize`: set `Clock1.Enabled = True`
5. In `Clock1.Timer`: 
   - Set `Clock1.Enabled = False` (fire only once)
   - Call `WebViewer1.GoToUrl(your_netlify_url)`

This gives the WebView time to mount before navigation.

### Emoji Showing as Boxes

If emoji characters appear as ☐ or ? on the device:
- This is an Android system font issue, not a code issue
- The HTML is correctly UTF-8 encoded
- Solution: In Kodular, set `WebViewer.UserAgent` to a modern Chrome UA string, or update the device's Android System Webview from Play Store

### CSV Download

CSV downloads via the ⬇ CSV button:
- In **Chrome/Firefox**: downloads normally as a file
- In **Kodular WebView**: opens as a `data:text/csv` URI. The user must tap the browser menu (⋮) and select "Download" or "Save"
- The app shows a toast: "CSV opened — tap ⋮ to save"

### Restricting to Play Store Only

True Play Store enforcement requires Google's Play Integrity API (complex server-side integration). Practical alternative:
1. Add a secret header in Kodular: `WebViewer.RequestHeaders = "x-app-source: kodular"`
2. Create a backend check route that validates this header
3. Show a "Please install from Play Store" message if header is missing
4. This stops casual browser users but not determined hackers — sufficient for most cases

---

## Dynamic Railway URL (Why Not to Do It)

**The request:** Store the Railway URL in a Railway environment variable so future URL changes don't require a frontend update.

**Why this is a bad idea:**
- The frontend (HTML) runs in the user's browser, not on Railway. It cannot read Railway env vars.
- Serving the URL via an API endpoint is circular: to fetch the URL, the frontend needs to already know the URL.
- The only practical solutions are: (a) hardcode the URL in the frontend (what we do — one file change when it changes), or (b) use a custom domain pointed at Railway so the URL never changes.

**Recommended solution — Custom Domain:**
1. Register a domain (e.g. `api.expiryguard.com.ng`)
2. In Railway → Settings → Networking → Custom Domain, add your domain
3. Add a CNAME record in your DNS: `api` → `your-railway-url.up.railway.app`
4. Update `API_BASE` in `index.html` to `https://api.expiryguard.com.ng`
5. Future Railway URL changes: just update the CNAME record, no code change needed

---

## Admin Panel — Feature Reference

### Users Table
Columns: Name, Username, Email, Plan, Credits (live), Products (live), Joined, Status (Active / Test Mode / Blocked)

### User Management Panel (click "Manage")
- **Credits** — Add positive numbers to add, negative to remove
- **Plan** — Change between Free / Paid / Admin
- **Test Mode** — Enable for N days (1–90). Disable immediately. Disabling auto-deletes all products the user added during test mode
- **Block/Unblock** — Blocking immediately invalidates their session; next API request returns HTTP 403 and the frontend forces logout
- **Clear User Data** — Deletes all their products, contacts, alert logs. Credits reset to free default. Account remains intact
- **Delete Account** — Permanently removes user and all data (irreversible)

### Settings Reference

| Setting | Default | Effect |
|---|---|---|
| `cron_expired_time` | `07:00` | UTC time for expired product alerts (daily) |
| `cron_warning_time` | `19:00` | UTC time for warning/critical alerts (daily) |
| `cron_enabled` | `true` | Toggle all scheduled alerts on/off |
| `free_credits` | `10` | Credits given to new users on signup |
| `credit_price_20` | `150000` | Price for 20 credits in kobo (₦1,500) |
| `credit_price_50` | `300000` | Price for 50 credits in kobo (₦3,000) |
| `credit_price_100` | `500000` | Price for 100 credits in kobo (₦5,000) |
| `sms_min_credits` | `50` | Minimum credits to unlock SMS alerts |
| `paystack_public_key` | — | Sent to frontend for payment initialisation |
| `bank_accounts` | — | Pipe-delimited bank account fallback |
| `terms_url` | — | URL for Terms & Conditions on signup page |
| `auto_logout_minutes` | `60` | Idle session timeout (0 = disabled) |
| `max_username_changes` | `3` | How many times a user can change their username (0 = never) |
| `registration_open` | `true` | Toggle user registration on/off |
| `maintenance_mode` | `false` | Only admins can log in |
| `app_version` | `4.0.5` | Kodular app checks this for update prompts |
| `update_message` | — | Message shown in Kodular when version mismatch |

---

## SMS / OCR Gate Logic

A user can use SMS alerts and OCR label scan if **any** of the following are true:
1. Their `plan` is `paid` or `admin`
2. Their `test_mode` is active
3. Their `credits` count is ≥ `sms_min_credits` (configurable, default 50)

This logic is checked:
- In the frontend when rendering the contact "Notify Via" options
- In the `addContact()` function before showing SMS radio button
- In the backend's product creation route (SMS preference respected regardless, but the frontend enforces the gate)

---

## Notifications System

Notifications are created automatically when:
- A new account is created (welcome message)
- Credits are added via Paystack payment
- Credits are added manually by admin
- Test mode is granted by admin
- Test mode is disabled by admin (with product cleanup notice)
- Test mode expires automatically

Users see notifications via:
- Bell icon (🔔) in topbar with red dot when unread
- Notifications page (desktop sidebar + mobile bottom nav)
- Notifications modal (bell icon click)

---

## Deployment Checklist

**Supabase:**
- [ ] Run `supabase_schema.sql` in SQL Editor

**GitHub:**
- [ ] Replace `backend/server.js`
- [ ] Replace `backend/supabase_schema.sql`
- [ ] Replace `frontend/index.html`
- [ ] Replace `frontend/admin.html`
- [ ] Add `TERMS_TEMPLATE.txt` and `BANK_ACCOUNTS_TEMPLATE.txt`

**Railway:**
- [ ] Redeploys automatically on GitHub push
- [ ] No new env vars needed for v4.0.5

**Netlify:**
- [ ] Upload new `index.html`
- [ ] Upload `admin.html` (accessible at `/admin.html`)

**Admin Panel (first-time setup after deploy):**
- [ ] Set Terms URL
- [ ] Set Bank Account Fallback entries
- [ ] Set Paystack Public Key
- [ ] Configure credit prices if different from defaults
- [ ] Verify cron times are correct for your timezone

---

## Revenue Projections

| Monthly Active Users | Revenue (if 20% convert at ₦3,000 avg) | Infrastructure |
|---|---|---|
| 100 | ₦60,000 | Railway Hobby $5/mo |
| 500 | ₦300,000 | Railway Hobby $5/mo |
| 2,000 | ₦1,200,000 | Railway Pro $20/mo + Supabase Pro $25/mo |
| 10,000 | ₦6,000,000 | Railway Pro + Supabase Pro + Twilio volume discount |

---

## Support Contact

For users who pay via bank transfer, direct them to your contact email set in the Bank Account Fallback field. Credits are added manually from Admin → Users → Manage within 3–8 hours.
