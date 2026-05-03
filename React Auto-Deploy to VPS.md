# 🚀 React Auto-Deploy to VPS — Complete Guide
> **Stack:** React + Vite → GitHub Actions → Apache / Nginx VPS  
> **Trigger:** Push to `main` branch → Auto build → Auto deploy  
> **Status:** ✅ Production Ready

---

## 📋 Overview

Every time you push code to the `main` branch, GitHub Actions automatically:
1. Builds your React project (`npm run build`)
2. SSHs into your VPS securely
3. Deploys the new `dist/` files to your server's serving folder

No manual deployment ever needed again.

---

## 🔑 GitHub Secrets Setup

Go to: **GitHub Repo → Settings → Secrets and variables → Actions → New repository secret**

Add these 6 secrets:

| Secret Name | Value | How to Get |
|---|---|---|
| `VPS_SSH_KEY` | Your VPS private key content | `cat ~/.ssh/id_ed25519` on VPS |
| `VPS_KNOWN_HOST` | VPS fingerprint | `ssh-keyscan -H YOUR_VPS_IP` on VPS |
| `VPS_HOST` | `YOUR_VPS_IP` | e.g. `123.456.789.000` |
| `VPS_PORT` | `22` | Default SSH port |
| `VPS_USERNAME` | `root` | Your VPS login user |
| `VPS_TARGET_DIR` | `/var/www/your-project/dist` | Where Apache/Nginx serves files |

---

## 🛠️ VPS One-Time Setup

### 1. Get your private key (VPS_SSH_KEY)
```bash
cat ~/.ssh/id_ed25519
```
Copy everything including `-----BEGIN OPENSSH PRIVATE KEY-----` and `-----END OPENSSH PRIVATE KEY-----`

> If you don't have this file, check what keys exist first:
> ```bash
> ls ~/.ssh/
> ```
> Then `cat` whichever private key file you see (no `.pub` extension).

### 2. Allow the key to login (authorize it)
```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 3. Get the known host fingerprint (VPS_KNOWN_HOST)
```bash
ssh-keyscan -H YOUR_VPS_IP
```
Copy the full output line(s) and paste as the secret value.

---

## 📁 VPS Folder Structure (Production)

```
/var/www/
└── your-project/
    └── dist/              ← Apache/Nginx serves THIS folder
        ├── index.html
        └── assets/
```

> ⚠️ Only keep the `dist/` folder on VPS. The `src/`, `node_modules/`, `package.json` etc. are development files — they belong only in your GitHub repo, not on the server.

---

## 📄 GitHub Actions Workflow File

**File location in your repo:** `.github/workflows/deploy.yml`

```yaml
name: Deploy React Frontend to VPS

on:
  push:
    branches:
      - main
    paths:
      - 'src/**'
      - 'public/**'
      - 'package.json'
      - 'index.html'
      - 'vite.config.ts'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build
        env:
          CI: false  # Prevents treating warnings as errors

      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.VPS_SSH_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          echo "${{ secrets.VPS_KNOWN_HOST }}" >> ~/.ssh/known_hosts

      - name: Test SSH Connection
        run: |
          ssh -i ~/.ssh/deploy_key \
            -p ${{ secrets.VPS_PORT }} \
            -o StrictHostKeyChecking=yes \
            ${{ secrets.VPS_USERNAME }}@${{ secrets.VPS_HOST }} \
            "echo 'SSH connection successful to VPS'"

      - name: Deploy to VPS
        run: |
          if [ -d "dist" ]; then
            BUILD_FOLDER="dist"
          elif [ -d "build" ]; then
            BUILD_FOLDER="build"
          else
            echo "No build folder found (dist or build)"
            exit 1
          fi

          echo "Deploying from $BUILD_FOLDER folder"

          rsync -avz --delete \
            -e "ssh -i ~/.ssh/deploy_key -p ${{ secrets.VPS_PORT }} -o StrictHostKeyChecking=yes" \
            $BUILD_FOLDER/ \
            ${{ secrets.VPS_USERNAME }}@${{ secrets.VPS_HOST }}:${{ secrets.VPS_TARGET_DIR }}/

      - name: Verify Deployment
        run: |
          ssh -i ~/.ssh/deploy_key \
            -p ${{ secrets.VPS_PORT }} \
            ${{ secrets.VPS_USERNAME }}@${{ secrets.VPS_HOST }} \
            "ls -la ${{ secrets.VPS_TARGET_DIR }} | head -10"
```

---

## 🔄 How It Works (Step by Step)

```
You push code to main
        ↓
GitHub Actions triggers
        ↓
Runner installs Node.js 18
        ↓
npm ci → npm run build → creates dist/
        ↓
SSH key loaded from GitHub Secrets
        ↓
rsync copies dist/ → VPS /var/www/your-project/dist/
(--delete removes any old files automatically)
        ↓
Apache/Nginx serves updated site ✓
SSL remains active ✓
```

---

## ✅ Trigger Conditions

The workflow only runs when you push changes to these files:

```
src/**           → any file inside src folder
public/**        → any file inside public folder
package.json     → dependency changes
index.html       → root HTML changes
vite.config.ts   → build config changes
```

> Pushing changes to `README`, `.github/`, or other files will **not** trigger a deploy.

---

## 🐛 Troubleshooting

### Workflow not showing in Actions tab
The workflow file must exist at exactly: `.github/workflows/deploy.yml`

Then push a change to a trigger file:
```bash
echo "" >> src/main.tsx
git add src/main.tsx
git commit -m "test deploy trigger"
git push origin main
```

### SSH Permission denied error
```bash
# On VPS — re-authorize the key
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
Then update `VPS_SSH_KEY` secret in GitHub with:
```bash
cat ~/.ssh/id_ed25519
```

### Wrong branch name
Check your branch:
```bash
git branch
```
If it shows `master`, update the workflow:
```yaml
branches:
  - master   # change main → master
```

### Verify deployment manually
```bash
# On VPS
ls -la /var/www/your-project/dist/
curl -I https://yourdomain.com
```

---

## 🗑️ Clean Up VPS Dev Files (When Ready)

Run once on VPS to remove unnecessary development files:

```bash
cd /var/www/your-project

rm -rf node_modules src public
rm -f package.json package-lock.json vite.config.ts
rm -f tsconfig.json tsconfig.app.json tsconfig.node.json
rm -f eslint.config.js README.md

# Verify only dist/ remains
ls /var/www/your-project/
```

> These files will never come back — the workflow only deploys built `dist/` files.

---

## 📝 Quick Checklist

Before your first deploy, confirm all these are done:

- [ ] `.github/workflows/deploy.yml` exists in your repo
- [ ] `VPS_SSH_KEY` secret added in GitHub
- [ ] `VPS_KNOWN_HOST` secret added in GitHub
- [ ] `VPS_HOST` secret added in GitHub
- [ ] `VPS_PORT` secret added in GitHub
- [ ] `VPS_USERNAME` secret added in GitHub
- [ ] `VPS_TARGET_DIR` secret added in GitHub
- [ ] Public key added to VPS `authorized_keys`
- [ ] Apache/Nginx `DocumentRoot` points to your `dist/` folder

---

*Feel free to use and share this guide freely.*  
*Replace all `YOUR_VPS_IP`, `your-project`, and `yourdomain.com` placeholders with your actual values.*
