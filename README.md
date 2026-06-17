# JobTrack — Job Application Tracker

A single-file PWA (Progressive Web App) for tracking job applications, with Google sign-in and cross-device cloud sync via Firebase.

**Live URL:** https://girisai3008.github.io/Application_Tracker/  
**GitHub Repo:** https://github.com/Girisai3008/Application_Tracker (Private)  
**Owner:** Giri — bheemisettygirisai@gmail.com

---

## What It Does

- Track job applications with company, title, status, date, URL, recruiter, notes, and more
- 6 statuses: Applied → Phone Screen → Interview → Offer → Rejected → Ghosted
- Filter, search, and sort your jobs list
- Daily view (jobs applied on a specific date)
- Analytics tab with charts (status breakdown, daily activity)
- Dark/light mode
- Export to JSON or CSV, import from either format
- **Works offline** — all data stored in browser `localStorage`
- **Cloud sync** — sign in with Google to sync across devices via Firebase

---

## Tech Stack

| Piece | Detail |
|---|---|
| App | Single HTML file (`index.html`) — all CSS, HTML, JS inline |
| Hosting | GitHub Pages (deploys automatically on push to `main`) |
| Charts | Chart.js 4.4.1 from cdnjs |
| Cloud sync | Firebase Realtime Database |
| Auth | Firebase Auth — Google Sign-In (popup) |
| Firebase SDK | Compat v10.12.0 from gstatic CDN |
| Offline | Service Worker registered via Blob URL (inline SW) |
| Storage | `localStorage` keys: `jt_apps_v2` (jobs), `jt_settings_v2` (settings) |

---

## Firebase Project Details

> These are hardcoded in `index.html`. Safe to store here — Firebase security is enforced via Realtime Database Rules, not by hiding the config.

```javascript
const FB_CONFIG = {
  apiKey:      'AIzaSyAXSPI6fFC6XaZSPsHe7dT0BxPn_0NnFvI',
  authDomain:  'jobtrack-e09a5.firebaseapp.com',
  databaseURL: 'https://jobtrack-e09a5-default-rtdb.firebaseio.com',
  projectId:   'jobtrack-e09a5',
  appId:       '1:911110834405:web:0674ce7be61e7c5b8dac0c'
};
```

**Firebase Console:** https://console.firebase.google.com/project/jobtrack-e09a5  
**Database URL:** https://jobtrack-e09a5-default-rtdb.firebaseio.com  
**Auth Provider:** Google (Email/Password disabled — Google-only)  
**Authorized domain:** `girisai3008.github.io` (added in Firebase Console → Auth → Settings → Authorized domains)

### Firebase Realtime Database Rules

```json
{
  "rules": {
    "users": {
      "$uid": {
        ".read": "$uid === auth.uid",
        ".write": "$uid === auth.uid"
      }
    }
  }
}
```

### Data Structure in Firebase

```
users/
  {uid}/
    jobtrack/
      apps: [ { id, company, title, status, date, url, recruiter, notes, ... } ]
      lastModified: 1718655600000  (Unix timestamp)
```

---

## How Sync Works

1. User clicks **"Sign in with Google"** on the Sync tab
2. Google popup opens → user authenticates
3. On success, `pullFromFirebase()` runs:
   - If cloud has data → loads it into localStorage and renders
   - If cloud is empty → pushes local data up to Firebase
4. After every save (add/edit/delete), `debouncedSync()` waits 2 seconds then calls `pushToFirebase()`
5. The sync badge in the header shows: ✅ synced / ⟳ syncing / ⚠ error

### Key Firebase Functions

| Function | What it does |
|---|---|
| `initFirebaseFromSettings()` | Initializes Firebase app, sets up `onAuthStateChanged` listener |
| `signInWithGoogle()` | Opens Google popup, calls `pullFromFirebase()` on success |
| `pushToFirebase()` | Writes `{apps, lastModified}` to `users/{uid}/jobtrack` |
| `pullFromFirebase()` | Reads from Firebase; if empty, pushes local data instead |
| `debouncedSync()` | Debounced 2s wrapper — called after every `save()` |
| `updateFirebaseUI()` | Renders sign-in button or signed-in user card on Sync tab |

---

## App Data Fields (per job)

| Field | Key | Notes |
|---|---|---|
| ID | `id` | Auto-generated `jt_{timestamp}_{random}` |
| Company | `company` | Required |
| Job Title | `title` | Required |
| Status | `status` | One of 6 values (see above) |
| Date Applied | `date` | YYYY-MM-DD |
| Job URL | `url` | |
| Location | `location` | |
| Salary | `salary` | |
| Recruiter Name | `recruiter` | Also used for resume name during import |
| Recruiter Email | `recruiterEmail` | |
| Source | `source` | LinkedIn / Indeed / etc. |
| Notes | `notes` | Free text |
| Last Updated | `updated` | Unix timestamp |

---

## Import / Export

### Export
- **JSON** — full data, use for backup and re-import
- **CSV** — spreadsheet-compatible

### Import
The app handles **two JSON formats**:

**New format** (`apps` array — current):
```json
{ "apps": [ { "id": "...", "company": "...", ... } ] }
```

**Old format** (`applications` array — legacy backup):
```json
{
  "applications": [
    {
      "id": "...",
      "company": "...",
      "dateApplied": "2026-06-17",
      "link": "https://...",
      "resumeName": "...",
      ...
    }
  ]
}
```

Old format field mappings applied during import:
- `dateApplied` → `date`
- `link` → `url`
- `resumeName` → `recruiter`
- `updatedAt` (ISO string) → `updated` (Unix timestamp)

---

## GitHub Pages Deployment

- GitHub Pages is enabled on the `main` branch, root folder
- Every `git push` to `main` triggers a new deploy (~1–2 minutes)
- Works with **private repos** on GitHub free plan
- No build step — single static HTML file

### To deploy a change:
```bash
cd C:\Users\giris\Application_Tracker
git add index.html
git commit -m "Your message"
git push
```

---

## Local Development

No build tools needed. Just open `C:\Users\giris\Application_Tracker\index.html` in Chrome.

For Firebase auth to work (Google popup), the page must be on HTTPS or localhost — `file://` blocks the popup. Use a local server if testing auth:

```bash
cd C:\Users\giris\Application_Tracker
npx serve .
# open http://localhost:3000
```

Or just use the live GitHub Pages URL.

---

## File Structure

```
C:\Users\giris\Application_Tracker\
├── index.html     ← Entire app (CSS + HTML + JS, ~942 lines)
└── README.md      ← This file
```

---

## History of Changes (Claude Cowork Session — June 17, 2026)

### Before
- Single-file HTML app tracking "Applications"
- Had GitHub Gist sync (broken — required manual token, gist ID, username)

### Changes Made

1. **Replaced GitHub Gist sync with Firebase Realtime Database**
   - Created Firebase project `jobtrack-e09a5`
   - Enabled Realtime Database (us-central1 region)
   - Enabled Google Sign-In in Firebase Auth
   - Added `girisai3008.github.io` as authorized domain
   - Set per-user DB rules
   - Hardcoded `FB_CONFIG` in `index.html`
   - Replaced sync tab UI and all sync JS with Firebase functions

2. **Simplified sync UX**
   - Old: config form requiring GitHub token + gist ID + username
   - New: single "Sign in with Google" button — zero config for user

3. **Fixed `pullFromFirebase()` error handling**
   - Old: showed "Cloud data format unrecognised" when DB was empty
   - New: empty/null cloud → automatically pushes local data up

4. **Fixed JSON import for old backup format**
   - Added detection of `applications[]` array (old format)
   - Maps old field names to new schema on import

5. **Renamed "Applications" → "Jobs" throughout UI**
   - Page title, tab label, buttons, empty states, footer count, modals, dialogs

6. **Made GitHub repo private**
   - No longer using GitHub Gist, no need to be public
   - GitHub Pages still works on free plan with private repos

---

## Recovery Instructions

If you ever lose context or need to resume work on this app:

1. **App file:** `C:\Users\giris\Application_Tracker\index.html`
2. **Firebase Console:** https://console.firebase.google.com/project/jobtrack-e09a5
3. **Firebase config:** hardcoded in `index.html` — search for `FB_CONFIG`
4. **Live app:** https://girisai3008.github.io/Application_Tracker/
5. **Push changes:** run `git add index.html && git commit -m "..." && git push` from `C:\Users\giris\Application_Tracker\`

Sign in with `bheemisettygirisai@gmail.com` Google account to access your synced job data.
