# Portfolio Tracker — Cloud Sync Setup & Usage Guide

---

## What This Does

Your portfolio data now syncs automatically to the cloud via **JSONBin.io** (free).  
Open the app on any PC, any browser — the same data loads every time.

- localStorage remains the **primary store** — the app works fully offline at all times.
- The cloud is a **mirror** that updates itself quietly in the background.
- No account is needed on *your* end to use the app. You only need a JSONBin account to *set up* the sync target.

---

## One-Time Setup (5 minutes)

### Step 1 — Create a free JSONBin account

Go to **https://jsonbin.io** and sign up using Google or GitHub. No credit card needed.

### Step 2 — Create a Bin

1. After signing in, click **"Create a Bin"** in the dashboard.
2. In the editor that appears, paste exactly this and hit Save:
```json
{}
```
3. You now have a bin. The page URL will look like:  
   `https://jsonbin.io/b/65a3f...`  
   The **alphanumeric part after `/b/`** is your **Bin ID**. Copy it somewhere.

### Step 3 — Get your Master Key

1. In the top-right corner, click your profile icon → **Account**.
2. Under **API Keys**, you will see a **Master Key** string.  
   Copy it.

> ⚠️ This key gives full access to all your bins. Don't share it. It's fine for personal use like this.

### Step 4 — Paste into the app

Open `portfolio-tracker.html` in any text editor (Notepad, VS Code, anything).  
Near the very top of the `<script>` section you will see these two lines:

```javascript
const SYNC_BIN_ID     = '';   // e.g. '65a3f...'
const SYNC_MASTER_KEY = '';   // from jsonbin.io → Account → API Keys
```

Replace the empty strings with your values:

```javascript
const SYNC_BIN_ID     = '65a3f...';          // ← your Bin ID here
const SYNC_MASTER_KEY = 'v2_abc123...';      // ← your Master Key here
```

Save the file. That's it — sync is now active.

---

## How It Works Day-to-Day

### When you open the app

1. It loads whatever is in localStorage instantly (so the page is never blank).
2. It then quietly fetches the cloud copy.
3. If the cloud copy is **newer** than what's on this machine, it swaps it in and re-renders.
4. If the local copy is newer (or they're the same), nothing changes.

You never see a loading screen or a delay. The page is usable in under a second.

### While you're editing

- Every cell edit auto-saves to localStorage every 5 seconds (same as before).
- After you stop editing, a **60-second timer** starts counting down.
- When the timer fires, the app pushes your full data to the cloud in a single PUT request.
- If you keep editing, the timer resets — so it only pushes after a minute of quiet.

### When you close the tab

- The app fires one final push to the cloud immediately on close, so nothing is lost even if the 60-second timer hadn't fired yet.

### On your second PC

- Open the same `portfolio-tracker.html` file (with the same Bin ID and Master Key).
- On load it pulls the cloud copy — which has everything the first PC pushed.
- Everything is there.

---

## The Sync Badge

A small badge sits next to the title in the header. It tells you exactly what's happening:

| Badge | Meaning |
|---|---|
| **● Synced** (green) | Cloud is up to date with your local data |
| **● Pending** (yellow) | You've made edits; waiting for the 60-second timer to push them |
| **↻ Syncing…** (cyan, pulsing) | A push or pull is in progress right now |
| **⚠ Offline** (red) | The network request failed (no internet, or API error). Your data is safe in localStorage. It will retry next time you make an edit or reopen the app. |
| **● No Sync** (grey) | The Bin ID or Master Key fields are empty — sync is disabled |

---

## What Happens If Something Goes Wrong

| Scenario | What happens |
|---|---|
| **No internet** | App works exactly as before — local only. Badge shows Offline. Next successful connection will sync. |
| **API request fails** | Automatic retry: 3 attempts with increasing wait (5 s → 15 s → 45 s). If all fail, badge turns red. Next edit or page open triggers a fresh attempt. |
| **Two PCs editing at the same time** | Last one to close its tab wins — its push overwrites the cloud. This is intentional and fine for single-user use. |
| **JSONBin is down** | Same as no internet. Your data never disappears — it's always in localStorage. |
| **You exhaust the free 10,000 requests** | The push/pull calls will start returning 403 errors. Badge stays red. All your data is still safe locally. At that point you can either upgrade on JSONBin (cheap) or switch the two config lines to a different service — the rest of the code doesn't change. |

---

## Request Usage Estimate

| Action | Requests/day |
|---|---|
| Pull on page open | 1–3 |
| Debounced push (60 s cooldown) | ~10–20 |
| Push on tab close | 1 |
| **Realistic daily total** | **~15–25** |
| **Free budget (10,000 lifetime)** | **~400+ days** |

---

## Quick Reference: The Two Lines You Control

```javascript
// ─── These are the ONLY two things you ever need to change ───
const SYNC_BIN_ID     = 'YOUR_BIN_ID_HERE';
const SYNC_MASTER_KEY = 'YOUR_MASTER_KEY_HERE';
```

- **To disable sync:** clear either field back to `''`.
- **To switch to a different bin:** just change the Bin ID.
- **To move to a different service in the future:** change the URL and headers inside `pullFromCloud()` and `pushToCloud()`. The badge, debounce, retry, and everything else stays the same.
