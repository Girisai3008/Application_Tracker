# JobTrack — Setup & Installation Guide

This guide covers everything needed to run, deploy, or recreate this app from scratch.  
Anyone with a GitHub account and a Google account can get it fully working in about 15 minutes.

---

## Quick Start (Already Deployed)

If the app is already live at https://girisai3008.github.io/Application_Tracker/:

1. Open the URL in any browser (desktop or mobile)
2. Go to the **Sync** tab
3. Click **Sign in with Google** → sign in with `bheemisettygirisai@gmail.com`
4. Your jobs load automatically from the cloud

That's it — no installation needed.

---

## What You Need to Set Up From Scratch

If the app is gone and you need to rebuild it completely:

| Requirement | Free? | Notes |
|---|---|---|
| GitHub account | ✅ Free | For hosting the code and the live site |
| Google account | ✅ Free | For Firebase and signing into the app |
| Firebase project | ✅ Free | Realtime Database + Auth (free tier is plenty) |
| Node.js / npm | Optional | Only needed if you want a local server |

---

## Step 1 — Get the Code

### Option A: Clone from GitHub (if repo still exists)
```bash
git clone https://github.com/Girisai3008/Application_Tracker.git
cd Application_Tracker
```

### Option B: Create fresh
```bash
mkdir Application_Tracker
cd Application_Tracker
git init
# paste index.html content (copy from the live GitHub repo)
```

The entire app is **one file**: `index.html`. Everything (CSS, HTML, JavaScript) is inline.

---

## Step 2 — Firebase Setup

### 2a. Create a Firebase Project

1. Go to https://console.firebase.google.com/
2. Click **Add project**
3. Name it (e.g. `jobtrack`) → Continue → Disable Google Analytics (optional) → Create project

### 2b. Enable Google Authentication

1. In Firebase Console → left sidebar → **Build → Authentication**
2. Click **Get started**
3. Go to **Sign-in method** tab
4. Click **Google** → Enable → set support email → Save

### 2c. Add Your Domain to Authorized Domains

1. Still in Authentication → go to **Settings** tab
2. Scroll to **Authorized domains**
3. Click **Add domain**
4. Enter: `girisai3008.github.io`
5. Click Add

> If you're using a different GitHub username, enter `yourusername.github.io` instead.

### 2d. Create the Realtime Database

1. In Firebase Console → left sidebar → **Build → Realtime Database**
2. Click **Create Database**
3. Choose location: **United States (us-central1)**
4. Start in **Locked mode** → Enable

### 2e. Set Database Security Rules

1. In Realtime Database → click the **Rules** tab
2. Replace the contents with:

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

3. Click **Publish**

### 2f. Get Your Firebase Config

1. In Firebase Console → click the ⚙️ gear → **Project settings**
2. Scroll down to **Your apps** → click the **</>** (Web) icon
3. Register the app (any nickname) → skip Firebase Hosting
4. Copy the config object — it looks like:

```javascript
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "yourproject.firebaseapp.com",
  databaseURL: "https://yourproject-default-rtdb.firebaseio.com",
  projectId: "yourproject",
  appId: "1:...:web:..."
};
```

### 2g. Put the Config in index.html

Open `index.html` and find this section (around line 415):

```javascript
const FB_CONFIG={
  apiKey:'AIzaSyAXSPI6fFC6XaZSPsHe7dT0BxPn_0NnFvI',
  authDomain:'jobtrack-e09a5.firebaseapp.com',
  databaseURL:'https://jobtrack-e09a5-default-rtdb.firebaseio.com',
  projectId:'jobtrack-e09a5',
  appId:'1:911110834405:web:0674ce7be61e7c5b8dac0c'
};
```

Replace those values with your own Firebase config values.

---

## Step 3 — Deploy to GitHub Pages

### 3a. Create a GitHub Repository

1. Go to https://github.com/new
2. Name it `Application_Tracker` (or anything you like)
3. Set visibility to **Public** (required for free GitHub Pages)
4. Don't add README yet → Create repository

### 3b. Push the Code

```bash
cd Application_Tracker
git add index.html README.md SETUP.md
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOURUSERNAME/Application_Tracker.git
git push -u origin main
```

Replace `YOURUSERNAME` with your GitHub username.

### 3c. Enable GitHub Pages

1. Go to your repo on GitHub → **Settings** → **Pages** (left sidebar)
2. Under **Build and deployment → Branch**: select `main`, folder `/ (root)`
3. Click **Save**
4. Wait 1–2 minutes → your site will be live at:  
   `https://YOURUSERNAME.github.io/Application_Tracker/`

---

## Step 4 — Verify It Works

1. Open `https://YOURUSERNAME.github.io/Application_Tracker/`
2. Go to the **Sync** tab
3. Click **Sign in with Google**
4. Sign in with your Google account
5. You should see "Firebase Sync Active" and your name

---

## Making Changes and Redeploying

Every time you edit `index.html`, deploy with:

```bash
cd C:\Users\giris\Application_Tracker
git add index.html
git commit -m "Describe what you changed"
git push
```

GitHub Pages deploys automatically within ~2 minutes.

---

## Local Testing

No build tools required. Just open `index.html` directly in Chrome for most testing.

For Firebase Google Sign-In to work locally (popup auth requires HTTPS or localhost), use a simple local server:

```bash
# Option 1: Node.js (if installed)
npx serve .
# then open http://localhost:3000

# Option 2: Python
python -m http.server 8000
# then open http://localhost:8000
```

---

## Importing Existing Data

### From a JSON backup (new format):
1. Go to **Settings** tab → **Import JSON**
2. Select your `.json` backup file
3. Format must have: `{ "apps": [...] }`

### From an old-format backup:
The app also accepts the legacy format:
```json
{ "applications": [ { "dateApplied": "...", "link": "...", "resumeName": "..." } ] }
```
Fields are automatically mapped on import.

### From CSV:
1. Go to **Settings** tab → **Import CSV**
2. Columns must be in order: Company, Title, Status, Date, Location, Salary, URL, Recruiter, RecruiterEmail, Source, Notes

---

## Data Storage

| Location | What's stored |
|---|---|
| Browser `localStorage` | All jobs (`jt_apps_v2`), settings (`jt_settings_v2`) |
| Firebase Realtime Database | Same jobs data, synced under `users/{uid}/jobtrack` |

Data in localStorage is **per browser/device**. Signing in with Google syncs it across all devices.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| 404 on GitHub Pages | Check Settings → Pages → Branch is set to `main` |
| "Domain not authorized" on sign-in | Add your `username.github.io` to Firebase → Auth → Authorized domains |
| Sign-in popup blocked | Allow popups for the site in browser settings |
| Data not syncing | Check you're signed in (Sync tab) and online |
| "Cloud data format unrecognised" | Old error — fixed in current version |
| App shows old version | Hard refresh: `Ctrl+Shift+R` (Windows) or `Cmd+Shift+R` (Mac) |

---

## Current Firebase Project (Giri's setup)

> Only relevant if working on the original deployment.

| Item | Value |
|---|---|
| Firebase project | `jobtrack-e09a5` |
| Console | https://console.firebase.google.com/project/jobtrack-e09a5 |
| Database URL | `https://jobtrack-e09a5-default-rtdb.firebaseio.com` |
| Auth domain | `jobtrack-e09a5.firebaseapp.com` |
| Authorized domain | `girisai3008.github.io` |
| Sign-in account | `bheemisettygirisai@gmail.com` |
| Live URL | https://girisai3008.github.io/Application_Tracker/ |
| GitHub repo | https://github.com/Girisai3008/Application_Tracker |
| Local repo path | `C:\Users\giris\Application_Tracker\` |
