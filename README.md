# Praxis Revenue Partners — Marketing Site

Static marketing site for Praxis Revenue Partners, a managed RCM service for independent and small group medical practices.

Stack: plain HTML + Tailwind (via CDN). Hosted on Firebase Hosting. CI/CD via GitHub Actions.

---

## Project structure

```
website/
├── public/                          # Files Firebase Hosting serves
│   ├── index.html                   # Landing page
│   ├── how-it-works.html            # Process / methodology
│   ├── services.html                # Services & specializations
│   └── 404.html                     # Not found page
├── .github/workflows/
│   ├── firebase-hosting-merge.yml          # Deploys main → live channel
│   └── firebase-hosting-pull-request.yml   # Deploys PRs → preview channel
├── firebase.json                    # Hosting config (clean URLs, cache headers)
├── .firebaserc                      # Firebase project mapping (edit before first deploy)
├── .gitignore
└── README.md
```

---

## One-time setup

You'll do three things, in order:

1. **Create a GitHub repo and push this code**
2. **Create a Firebase project + Hosting site**
3. **Wire GitHub Actions to deploy via a Firebase service account**

After that, every `git push` to `main` will auto-deploy to your live site.

---

### Step 1 — Push to GitHub

Open Terminal and `cd` into this folder:

```bash
cd "/Users/sampathvelaga/Downloads/stitch_practice_claim_success/website"
```

Initialize git and make your first commit:

```bash
git init -b main
git add .
git commit -m "Initial commit: Praxis Revenue Partners marketing site"
```

Create the GitHub repo. Easiest way is via the GitHub CLI (`gh`):

```bash
# install gh if you don't have it: brew install gh
gh auth login
gh repo create praxis-revenue-partners --public --source=. --remote=origin --push
```

**Or** create it manually in the browser:

1. Go to https://github.com/new
2. Repo name: `praxis-revenue-partners` (or whatever you like)
3. Public or private — your choice
4. Do **NOT** initialize with a README, .gitignore, or license (we have those locally)
5. Click "Create repository"
6. Back in Terminal, copy the commands GitHub shows you. They will look like:

```bash
git remote add origin https://github.com/<your-username>/praxis-revenue-partners.git
git push -u origin main
```

You should now see all your files on GitHub.

---

### Step 2 — Create the Firebase project

1. Go to https://console.firebase.google.com
2. Click **Add project**
3. Project name: `praxis-revenue-partners` (or whatever you like). Firebase will generate a project ID like `praxis-revenue-partners-1a2b3c`. **Copy that project ID** — you'll need it.
4. Disable Google Analytics for now (you can add it later if needed). Click **Create project**.
5. Once the project is created, in the left sidebar click **Build → Hosting** and then **Get started**. You can skip the CLI setup wizard — we already have `firebase.json` configured. Just click through to finish.

**Update `.firebaserc`** in your repo with your real project ID:

Open `.firebaserc` and replace `YOUR_FIREBASE_PROJECT_ID` with the project ID from step 3 above. For example:

```json
{
  "projects": {
    "default": "praxis-revenue-partners-1a2b3c"
  }
}
```

Commit and push:

```bash
git add .firebaserc
git commit -m "Set Firebase project ID"
git push
```

---

### Step 3 — Wire GitHub Actions to Firebase

The workflows in `.github/workflows/` need two GitHub secrets to deploy:

- `FIREBASE_PROJECT_ID` — your project ID (e.g. `praxis-revenue-partners-1a2b3c`)
- `FIREBASE_SERVICE_ACCOUNT` — a JSON service account key with permission to deploy

**Easiest path: use the Firebase CLI to auto-create everything.**

Install the Firebase CLI if you don't have it:

```bash
npm install -g firebase-tools
```

Log in:

```bash
firebase login
```

From inside the `website/` folder, run:

```bash
firebase init hosting:github
```

Answer the prompts:

- "Which project?" → select your Firebase project
- "Which GitHub repo?" → `<your-username>/praxis-revenue-partners`
- "Set up workflow file?" → **No** (we already have ours)
- "Set up automatic deployment on merge?" → **No** (we already have ours)

What this command does behind the scenes:

1. Creates a Google Cloud service account with the right Hosting permissions
2. Generates a JSON key
3. **Uploads that key as the `FIREBASE_SERVICE_ACCOUNT_<PROJECT_ID>` secret on your GitHub repo**

After it finishes, go to your GitHub repo → **Settings → Secrets and variables → Actions**.

You'll see a secret named something like `FIREBASE_SERVICE_ACCOUNT_PRAXIS_REVENUE_PARTNERS_1A2B3C`.

Two more secrets to add manually on that same page (click **New repository secret**):

| Secret name | Value |
| --- | --- |
| `FIREBASE_SERVICE_ACCOUNT` | Copy the entire JSON value from the secret Firebase created. (Or rename that secret to `FIREBASE_SERVICE_ACCOUNT` — either works.) |
| `FIREBASE_PROJECT_ID` | Your project ID, e.g. `praxis-revenue-partners-1a2b3c` |

> **Why two names?** Firebase's helper creates a secret with the project ID baked into the name. Our workflow reads `FIREBASE_SERVICE_ACCOUNT` for portability. The simplest fix is to either rename the auto-created secret or paste its JSON value into a new `FIREBASE_SERVICE_ACCOUNT` secret.

**Manual alternative** (if `firebase init hosting:github` doesn't work):

1. In Firebase Console → ⚙️ **Project settings → Service accounts → Generate new private key**. Save the JSON file somewhere safe — **do not commit it to git**.
2. In GitHub → repo → Settings → Secrets and variables → Actions → New repository secret:
   - Name: `FIREBASE_SERVICE_ACCOUNT`
   - Value: paste the entire contents of the JSON file
3. Add another secret:
   - Name: `FIREBASE_PROJECT_ID`
   - Value: your project ID

---

### Step 4 — Trigger your first deploy

Once the secrets are in place, push any change to `main`:

```bash
# trivial change to trigger deploy
echo "" >> README.md
git add README.md
git commit -m "Trigger first deploy"
git push
```

Go to your repo on GitHub → **Actions** tab. You should see the workflow running. After ~30–60 seconds it'll finish and show a green check.

Your site will be live at:

```
https://<your-project-id>.web.app
https://<your-project-id>.firebaseapp.com
```

For example: `https://praxis-revenue-partners-1a2b3c.web.app`

You can find the exact URL in **Firebase Console → Hosting**.

---

## Local development

You can preview the site locally without Firebase:

```bash
cd public
python3 -m http.server 8000
# then open http://localhost:8000
```

Or with the Firebase CLI (closer to production behavior, honors `firebase.json` rules):

```bash
firebase emulators:start --only hosting
# opens http://localhost:5000
```

---

## Editing pages

All three pages are self-contained HTML files in `public/`. Tailwind is loaded from CDN, so no build step is required — edit, save, push, and your changes deploy.

Want to add a new page? Drop `public/new-page.html` and update the nav `<header>` block on each existing page to link to it.

---

## Custom domain (optional, later)

When you're ready to point a real domain (e.g. `praxisrevenue.com`) at the site:

1. Firebase Console → Hosting → **Add custom domain**
2. Follow the DNS instructions Firebase shows you (one TXT record to verify ownership, then A/AAAA records pointing at Firebase's IPs)
3. SSL certificates are issued automatically — usually within a few hours

---

## Troubleshooting

- **Workflow fails with `Error: Process completed with exit code 1`** — almost always a missing or malformed `FIREBASE_SERVICE_ACCOUNT` secret. Re-paste the JSON, make sure no extra whitespace or quotes wrap it.
- **`Error: Project not found`** — your `.firebaserc` still has the placeholder. Update it with your real project ID and push.
- **404 on `/how-it-works`** — `cleanUrls: true` in `firebase.json` lets you link to `/how-it-works` without the `.html`. If links break locally, use the Firebase emulator (above) rather than plain `python -m http.server`.
- **Site loads but Tailwind classes don't render** — check the browser console for CSP errors. The site uses the Tailwind CDN; corporate networks sometimes block it.

---

## What's intentionally NOT in this repo

- No Firebase Auth, Firestore, or backend functions. This is marketing-only by design.
- No build step / npm dependencies. Tailwind is loaded from CDN for fastest iteration.
- No analytics. Add Google Analytics or Plausible later by injecting a `<script>` tag into each page's `<head>`.
